# Good and Bad Tests

## Good Tests

**Integration-style**: Test through real interfaces, not mocks of internal parts.

```csharp
// GOOD: Tests observable behavior
[Fact]
public async Task Checkout_WithValidCart_ReturnsConfirmed()
{
    var cart = CartFactory.Create();
    cart.Add(product);

    var result = await checkoutService.CheckoutAsync(cart, paymentMethod);

    Assert.Equal(OrderStatus.Confirmed, result.Status);
}
```

Characteristics:

- Tests behavior users/callers care about
- Uses public API only
- Survives internal refactors
- Describes WHAT, not HOW
- One logical assertion per test

## Bad Tests

**Implementation-detail tests**: Coupled to internal structure.

```csharp
// BAD: Tests implementation details
[Fact]
public async Task Checkout_CallsPaymentServiceProcess()
{
    var mockPayment = new Mock<IPaymentService>();
    var sut = new CheckoutService(mockPayment.Object);

    await sut.CheckoutAsync(cart, payment);

    mockPayment.Verify(p => p.Process(cart.Total), Times.Once);
}
```

Red flags:

- Mocking internal collaborators
- Testing private methods (via reflection or `InternalsVisibleTo`)
- Asserting on call counts/order (`mock.Verify(..., Times.Once)`) as the primary assertion
- Test breaks when refactoring without behavior change
- Test name describes HOW not WHAT
- Verifying through external means instead of interface

```csharp
// BAD: Bypasses interface to verify
[Fact]
public async Task CreateUser_SavesToDatabase()
{
    await userService.CreateUserAsync(new User { Name = "Alice" });

    var row = await dbConnection.QuerySingleAsync<UserRow>(
        "SELECT * FROM Users WHERE Name = @Name", new { Name = "Alice" });

    Assert.NotNull(row);
}

// GOOD: Verifies through interface
[Fact]
public async Task CreateUser_MakesUserRetrievable()
{
    var user = await userService.CreateUserAsync(new User { Name = "Alice" });

    var retrieved = await userService.GetUserAsync(user.Id);

    Assert.Equal("Alice", retrieved.Name);
}
```

**Tautological tests**: Expected value restates the implementation, so the test passes by construction.

```csharp
// BAD: Expected value is recomputed the way the code computes it
[Fact]
public void CalculateTotal_SumsLineItems()
{
    var items = new[] { new LineItem(10m), new LineItem(5m) };
    var expected = items.Sum(i => i.Price);

    Assert.Equal(expected, calculator.CalculateTotal(items));
}

// GOOD: Expected value is an independent, known literal
[Fact]
public void CalculateTotal_SumsLineItems()
{
    var items = new[] { new LineItem(10m), new LineItem(5m) };

    Assert.Equal(15m, calculator.CalculateTotal(items));
}
```

## XUnit conventions

- Prefer `[Fact]` for a single case, `[Theory]` + `[InlineData]` (or `[MemberData]` for complex inputs) for the same behavior across multiple inputs — don't hand-write near-duplicate `[Fact]`s that only differ by input value.
- Name tests `MethodOrBehavior_Condition_ExpectedResult` (e.g. `Checkout_WithExpiredCard_ThrowsPaymentDeclinedException`) so a failing test name alone tells you what broke.
- Use `Assert.Throws<TException>(...)` / `await Assert.ThrowsAsync<TException>(...)` for expected failures — don't wrap in try/catch and assert in the catch block.
- Constructor + `IDisposable.Dispose()` handle per-test setup/teardown; use `IClassFixture<T>` / `ICollectionFixture<T>` for expensive shared context (e.g. a test database), not static state.
