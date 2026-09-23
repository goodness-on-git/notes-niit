# Demonstration: JUnit + Mockito + Spring Context Together

This is a worked example to demonstrate combining what students already know separately:

- **JUnit + Mockito alone** — `@ExtendWith(MockitoExtension.class)`, `@Mock`, `@InjectMocks`. No Spring. Objects are built and wired by hand.
- **JUnit + Spring Context alone** — `@SpringBootTest`, real beans, real wiring, no mocks.

Neither of those covers a very common real case: a Spring-wired object graph where **one specific collaborator must not actually run** in a test (an external API, a payment gateway, an email sender). That's what this example demonstrates.

---

## The Domain

```java
// PaymentGateway.java — an interface, because the real implementation talks to a network
public interface PaymentGateway {
    String charge(String customerId, int amountInCents);
}
```

```java
// StripePaymentGateway.java — the real implementation. NEVER run this in a test.
@Component
public class StripePaymentGateway implements PaymentGateway {

    @Override
    public String charge(String customerId, int amountInCents) {
        // Real network call to a payment provider would go here.
        throw new UnsupportedOperationException("Wire up the real Stripe client here");
    }
}
```

```java
// Order.java — a plain JPA entity
@Entity
public class Order {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String customerId;
    private int amountInCents;
    private String status;

    // constructors, getters, setters
}
```

```java
// OrderRepository.java — real Spring Data JPA, same as students already know
public interface OrderRepository extends JpaRepository<Order, Long> {
}
```

```java
// OrderService.java — the class this whole example is about
@Service
public class OrderService {

    private final OrderRepository orderRepository;
    private final PaymentGateway paymentGateway;

    public OrderService(OrderRepository orderRepository, PaymentGateway paymentGateway) {
        this.orderRepository = orderRepository;
        this.paymentGateway = paymentGateway;
    }

    public Order placeOrder(String customerId, int amountInCents) {
        String confirmationId = paymentGateway.charge(customerId, amountInCents);

        Order order = new Order();
        order.setCustomerId(customerId);
        order.setAmountInCents(amountInCents);
        order.setStatus("PAID:" + confirmationId);

        return orderRepository.save(order);
    }
}
```

```java
// OrderController.java
@RestController
@RequestMapping("/orders")
public class OrderController {

    private final OrderService orderService;

    public OrderController(OrderService orderService) {
        this.orderService = orderService;
    }

    @PostMapping
    public ResponseEntity<Order> placeOrder(@RequestBody OrderRequest request) {
        Order order = orderService.placeOrder(request.customerId(), request.amountInCents());
        return ResponseEntity.ok(order);
    }
}

record OrderRequest(String customerId, int amountInCents) {}
```

**Ask:** "`OrderService` needs a real `OrderRepository` (Spring-managed, talks to a database) and a `PaymentGateway`. If we write a pure Mockito test — no Spring at all — what do we lose? If we write a plain `@SpringBootTest` — real context, no mocks — what breaks?"

**Expected:**
- Pure Mockito: fast, but you're hand-wiring everything yourself, and you're not proving the *real* Spring wiring (bean names, JPA mapping, configuration) actually works.
- Plain `@SpringBootTest`: proves real wiring, but `placeOrder` would call the real `StripePaymentGateway`, which throws (or, in production, actually charges someone).

**Say:** We want both at once: a real Spring context, with one bean swapped for a Mockito mock. That's what `@MockitoBean` does.

> **Note on the annotation name:** Since Spring Boot 3.4, the old `@MockBean` (`org.springframework.boot.test.mock.mockito.MockBean`) is deprecated in favor of `@MockitoBean` (`org.springframework.test.context.bean.override.mockito.MockitoBean`), which now lives in Spring Framework itself rather than Spring Boot. The behavior is the same for straightforward cases like this one. If your students are on an older Spring Boot version, swap the import and annotation name — everything else below is unchanged.

---

## Demo 1: Service-Level Test — Real Context, One Mocked Bean

```java
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.bean.override.mockito.MockitoBean;

import static org.mockito.Mockito.when;
import static org.junit.jupiter.api.Assertions.*;

@SpringBootTest
class OrderServiceTest {

    @Autowired
    private OrderService orderService;          // real bean, really wired by Spring

    @Autowired
    private OrderRepository orderRepository;    // real bean, really talks to the (test) database

    @MockitoBean
    private PaymentGateway paymentGateway;       // Spring replaces the real StripePaymentGateway bean with this mock

    @Test
    void placeOrderSavesAPaidOrder() {
        when(paymentGateway.charge("cust-1", 5000)).thenReturn("conf-123");

        Order saved = orderService.placeOrder("cust-1", 5000);

        assertNotNull(saved.getId());                       // proves the real repository really saved it
        assertEquals("PAID:conf-123", saved.getStatus());   // proves the real service logic ran
    }
}
```

**Ask:** "Point at each annotation. Which bean is real? Which is fake? How does Spring know to swap it?"

**Expected:** `orderService` and `orderRepository` are the real beans from the real application context — same as any `@SpringBootTest` they've written before. `paymentGateway` is a Mockito mock, and `@MockitoBean` tells Spring: "when anything in this context asks for a `PaymentGateway`, hand it this mock instead of the real `StripePaymentGateway`." Everything downstream of `OrderService` — repository, entity, database — is completely real.

### Predict, then run: forget the stub

**Predict:** "What happens if I delete the `when(...)` line and run the test?"

Remove it, run, then explain:

**Expected:** No exception. A Mockito mock returns `null` for any unstubbed method by default (for object return types), so `charge(...)` returns `null` and `status` becomes `"PAID:null"`. The test fails on the `assertEquals`, not with a crash — the same behavior they already know from pure Mockito tests, because it *is* the same Mockito.

### Predict, then run: use the real bean instead

**Predict:** "What happens if I remove `@MockitoBean` entirely, so the real `StripePaymentGateway` gets wired in?"

Remove it, run, then explain:

**Expected:** The test throws `UnsupportedOperationException` from inside `charge(...)` (or, against a real implementation, would attempt a real network call). This is exactly the failure `@MockitoBean` exists to prevent — nobody wants a test suite that tries to charge a real card or calls a real third-party API on every build.

---

## Demo 2: Controller-Slice Test — Mock the Layer Below

A `@SpringBootTest` loads the *entire* application context. For testing just the HTTP layer, that's more than needed and slower than it needs to be. `@WebMvcTest` loads only the web layer, and here `@MockitoBean` mocks the whole `OrderService`, not just one of its collaborators.

```java
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.test.context.bean.override.mockito.MockitoBean;
import org.springframework.test.web.servlet.MockMvc;

import static org.mockito.Mockito.when;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@WebMvcTest(OrderController.class)
class OrderControllerTest {

    @Autowired
    private MockMvc mockMvc;         // simulates HTTP requests, no real server started

    @MockitoBean
    private OrderService orderService;  // OrderService itself is mocked here — repository and gateway never load

    @Test
    void placeOrderReturnsTheCreatedOrder() throws Exception {
        Order fakeOrder = new Order();
        fakeOrder.setId(1L);
        fakeOrder.setStatus("PAID:conf-123");

        when(orderService.placeOrder("cust-1", 5000)).thenReturn(fakeOrder);

        mockMvc.perform(post("/orders")
                        .contentType("application/json")
                        .content("{\"customerId\":\"cust-1\",\"amountInCents\":5000}"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.status").value("PAID:conf-123"));
    }
}
```

**Ask:** "In Demo 1, `PaymentGateway` was mocked but `OrderService` was real. Here, what's mocked and what's real?"

**Expected:** `OrderService` itself is now the mock. Only the controller and Spring's web machinery (`@RequestMapping`, JSON conversion, `MockMvc`) are real. `OrderRepository`, `PaymentGateway`, and the database never come into play at all — `@WebMvcTest` doesn't even load those beans.

---

## The Three Kinds of Test, Side by Side

| | Pure Mockito | `@SpringBootTest` + `@MockitoBean` | `@WebMvcTest` + `@MockitoBean` |
|---|---|---|---|
| Spring context? | No | Full application context | Web layer only |
| What's real? | Only the class under test | Everything except the mocked bean(s) | Only the controller and web config |
| What's mocked? | Every collaborator, by hand (`@Mock`) | The one bean you don't want to actually run | The entire service layer below the controller |
| Speed | Fastest | Slowest (full context startup) | Fast (small context) |
| Proves | Your class's own logic, in isolation | Real wiring + real logic, minus one dangerous dependency | The controller's request/response handling only |
| Use it for | Business logic with no Spring dependencies | A service whose real wiring matters, but has one external/dangerous collaborator | Request mapping, status codes, JSON shape, validation |

**Say:** These aren't competing choices — most test suites have all three. The question to ask about any class is: "What do I actually need Spring for here, and what do I need to keep from actually running?"

---

## Practice: Add a Second Collaborator

Extend `OrderService` with an `EmailSender` that emails the customer a receipt after a successful charge.

```java
public interface EmailSender {
    void send(String to, String subject, String body);
}
```

**Tasks:**

1. Wire `EmailSender` into `OrderService` the same way `PaymentGateway` is wired.
2. Update the Demo 1 test so it still passes — predict what happens if you forget to add `@MockitoBean` for `EmailSender` before running it, then run it and check.
3. Add an assertion using `verify(emailSender).send(...)` to confirm the receipt was actually sent — this is the same `verify` they already know from pure Mockito tests.
4. Discuss: should `EmailSender` also be mocked in the `@WebMvcTest` controller test? (Expected: it doesn't need to be — `OrderService` is already fully mocked there, so nothing underneath it, including `EmailSender`, ever runs.)

---

## Teacher Notes

### What NOT to teach deeply yet

- `@DataJpaTest` and other slice annotations beyond `@WebMvcTest` (mention they exist; save for a dedicated slice-testing session if you have one)
- `MockReset` modes and mock lifecycle configuration
- Testcontainers / real external databases in tests
- Differences in behavior between `@MockBean` and `@MockitoBean` beyond the import change — only relevant if a student hits it on an older Spring Boot version

### Source

- Spring Boot 3.4 release notes and Spring Boot reference docs confirm `@MockBean`/`@SpyBean` are deprecated since 3.4.0 (for removal in 4.0.0), replaced by Spring Framework's `@MockitoBean`/`@MockitoSpyBean`.
