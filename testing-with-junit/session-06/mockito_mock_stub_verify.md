# Mockito — Why We Mock: The OrderService Story

## The story

Imagine you're building the checkout feature for an online store.

`OrderService` is the piece of code responsible for placing an order.
But before it can confirm an order, it needs to charge the customer —
and charging money is not `OrderService`'s job. That job belongs to
`PaymentService`, which talks to a real payment provider somewhere out
in the world (a bank, a card processor, whatever).

So `OrderService` *depends on* `PaymentService` to do its work.

Now picture testing `OrderService`. If your test uses the **real**
`PaymentService`, every single test run would try to actually contact
a payment provider. That means:

- real network calls
- real (or fake) money moving
- slow, flaky tests
- a test that fails not because your code is wrong, but because the
  payment provider's server hiccupped

That's not what you want. You want to ask a much narrower question:

> "If payment succeeds, does `OrderService` correctly complete the
> order? And if payment fails, does it correctly *not* complete it?"

To ask that question safely, you need a stand-in for `PaymentService`
— something that behaves exactly the way you tell it to, with no
network calls, no side effects, and no surprises. That stand-in is a
**mock**.

Once you have a mock, two more questions naturally follow:

1. **What should the mock do when it's called?** — that's **stubbing**.
   You're not testing `PaymentService` here, so you just tell the mock
   "when `charge(100)` is called, return `true`" and move on.
2. **Did `OrderService` actually use the dependency the way it should
   have?** — that's **verification**. Stubbing controls the mock's
   *output*; verification checks the unit's *behaviour toward* the
   mock.

The code below is exactly that story, written as a test.

---

## The dependency (what we will mock)

```java
interface PaymentService {
    boolean charge(int amount);
}
```

This is the thing `OrderService` needs but doesn't want to depend on
directly during a test — the real payment provider, standing behind an
interface.

## The unit under test (the logic we are actually testing)

```java
class OrderService {
    private final PaymentService paymentService;

    OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }

    boolean placeOrder(Order order) {
        if (paymentService.charge(order.getAmount())) {
            // save order
            return true;
        }
        return false;
    }
}
```

This is the piece of logic the test cares about. Everything else in
the test exists to put this method under controlled conditions.

## The test

```java
class OrderServiceTest {

    @Test
    void shouldPlaceOrderWhenPaymentSucceeds() {

        // MOCK: replace the real PaymentService with a controllable test double
        PaymentService paymentService = mock(PaymentService.class);

        // the unit under test, wired manually with the mock as its dependency
        OrderService orderService = new OrderService(paymentService);

        Order order = new Order(100);

        // STUB: decide what the mock returns when charge(100) is called
        when(paymentService.charge(100))
            .thenReturn(true);

        // EXECUTE: call the actual logic we are testing — OrderService.placeOrder
        boolean result = orderService.placeOrder(order);

        // ASSERT: check OrderService's returned behaviour
        assertTrue(result);

        // VERIFY: confirm OrderService actually called charge(100) on its dependency
        verify(paymentService).charge(100);
    }
}
```

---

## Tying it back to the story

| Line in the test | Question it answers |
|---|---|
| `mock(PaymentService.class)` | "Can I get a stand-in for the payment provider?" |
| `new OrderService(paymentService)` | "Is the unit under test wired to that stand-in?" |
| `when(paymentService.charge(100)).thenReturn(true)` | "What should the stand-in do when asked to charge?" |
| `orderService.placeOrder(order)` | "What actually happens when I run the real logic?" |
| `assertTrue(result)` | "Did `OrderService` behave correctly?" |
| `verify(paymentService).charge(100)` | "Did `OrderService` actually ask the dependency to charge?" |

Students should walk away able to answer, in their own words:

> **Why would a unit test replace a real dependency with a mock?**

— and be able to point to the exact line in the test that does each
job: mocking, stubbing, executing, asserting, verifying.
