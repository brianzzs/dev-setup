# When to Mock

Mock at **system boundaries** only:

- External APIs (payment, email, etc.)
- Databases (sometimes - prefer a real/test database, e.g. via `IDapperHelper` against a test instance)
- Time/randomness (wrap `DateTime.UtcNow` / `Guid.NewGuid()` behind an injected abstraction such as `ITimeProvider`)
- File system (sometimes)

Don't mock:

- Your own classes/modules
- Internal collaborators
- Anything you control

Use **Moq** as the mocking library. It is already referenced across this codebase's test projects — don't introduce NSubstitute, FakeItEasy, or another library alongside it.

## Designing for Mockability

At system boundaries, design interfaces that are easy to mock:

**1. Use dependency injection**

Pass external dependencies in rather than creating them internally:

```csharp
// Easy to mock
public class PaymentProcessor
{
    private readonly IPaymentClient paymentClient;

    public PaymentProcessor(IPaymentClient paymentClient)
    {
        this.paymentClient = paymentClient;
    }

    public Task<PaymentResult> ProcessAsync(Order order) =>
        paymentClient.ChargeAsync(order.Total);
}

// Hard to mock
public class PaymentProcessor
{
    public Task<PaymentResult> ProcessAsync(Order order)
    {
        var client = new StripeClient(ConfigurationManager.AppSettings["StripeKey"]);
        return client.ChargeAsync(order.Total);
    }
}
```

**2. Prefer SDK-style interfaces over generic fetchers**

Create specific methods for each external operation instead of one generic method with conditional logic:

```csharp
// GOOD: Each method is independently mockable
public interface IUserApiClient
{
    Task<User> GetUserAsync(int id);
    Task<IReadOnlyList<Order>> GetOrdersAsync(int userId);
    Task<Order> CreateOrderAsync(CreateOrderRequest request);
}

// BAD: Mocking requires conditional logic inside the mock setup
public interface IApiClient
{
    Task<TResponse> FetchAsync<TResponse>(string endpoint, HttpMethod method, object body = null);
}
```

The SDK approach means:
- Each `Mock<T>.Setup(...)` targets one specific method
- No conditional branching in test setup
- Easier to see which operations a test exercises
- Compile-time type safety per operation

## Moq usage

```csharp
var paymentClient = new Mock<IPaymentClient>();
paymentClient
    .Setup(c => c.ChargeAsync(It.IsAny<decimal>()))
    .ReturnsAsync(new PaymentResult(PaymentStatus.Succeeded));

var sut = new PaymentProcessor(paymentClient.Object);

var result = await sut.ProcessAsync(order);

Assert.Equal(PaymentStatus.Succeeded, result.Status);
```

- Assert on the **result**, not the mock, whenever behavior is observable through the return value. Reach for `mock.Verify(...)` only when the side effect itself (a call happening, with what arguments) is the behavior under test — e.g. confirming an email was sent — not as a substitute for asserting on output.
- Use `It.IsAny<T>()` sparingly; prefer exact expected arguments (`It.Is<T>(x => ...)` or a literal) so the test still fails if the wrong data is passed.
- `Autofac.Extras.Moq`'s `AutoMock` is already available in this codebase's `WIT.AreaServices.Tests` project for auto-mocking a class's full dependency graph — use it there rather than hand-constructing every `Mock<T>` when a class has many collaborators.
