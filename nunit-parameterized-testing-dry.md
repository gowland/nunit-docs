# DRY Unit Tests with NUnit Parameterized Testing

Unit testing is like paying for fire insurance. You’d be foolish not to have insurance, but you don’t have to like paying for it. So, you shop around to get a good deal because you don’t want to spend more than you have to.

Unit tests often cost more than they should. The most common culprit is duplicate code. Many unit testing fixtures contain methods which are remarkably similar to each other. This violates the DRY principle, but without the right tools, there’s not much that can be done. We’re left to suffer through the maintainability and readability nightmare that is duplicate code.

So, what can we do to dry out our tests?

## DRY Unit Tests

Fortunately, many unit testing frameworks do provide the tools we need to DRY up our unit tests. NUnit is no exception. In this post we’re going to apply a series of refactorings to a test fixture with four nearly identical test methods.

(NOTE: The following examples are in NUnit 3. As of writing, few 3rd party test runners play nice with NUnit 3, so our refactored unit tests will fail. NUnit 2.x supports all the functionality used in these examples, but uses a slightly different syntax as indicated where necessary below.)

## Wet Test Code

Let’s take a look at the unit tests for a fictitious `OrderValidator` class. The `OrderValidator` looks at the contents of a shopping cart and evaluates their validity. For the purposes of this demo, we’re going to focus on one property in particular, `HasOverlappingDiscounts`.

`HasOverlappingDiscounts` should be true in the case where both of these two conditions are true: the order has sale items in the cart and there is a coupon applied. It should be false in all other cases. It’s just simple AND logic, and so we have the following four unit tests which cover all possible inputs.

```csharp
[Test]
public void HasOverlappingDiscountsIsFalseIfNoCouponsAndNoSaleItemsTest()
{
  var factory = new OrderValidatorFactory
  {
    CouponProvider = GetCouponProviderStub(hasAppliedCoupons: false),
    SaleItemsProvider = GetSaleItemsProviderStub(hasSaleItems: false)
  };

  var sut = factory.GetOrderValidator();
  Assert.IsFalse(sut.HasOverlappingDiscounts);
}

[Test]
public void HasOverlappingDiscountsIsFalseIfHasCouponsAndNoSaleItemsTest()
{
  var factory = new OrderValidatorFactory
  {
    CouponProvider = GetCouponProviderStub(hasAppliedCoupons: true),
    SaleItemsProvider = GetSaleItemsProviderStub(hasSaleItems: false)
  };

  var sut = factory.GetOrderValidator();

  Assert.IsFalse(sut.HasOverlappingDiscounts);
}

[Test]
public void HasOverlappingDiscountsIsFalseIfNoCouponsAndHasSaleItemsTest()
{
  var factory = new OrderValidatorFactory
  {
    CouponProvider = GetCouponProviderStub(hasAppliedCoupons: false),
    SaleItemsProvider = GetSaleItemsProviderStub(hasSaleItems: true)
  };

  var sut = factory.GetOrderValidator();

  Assert.IsFalse(sut.HasOverlappingDiscounts);
}

[Test]
public void HasOverlappingDiscountsIsTrueIfHasCouponsAndHasSaleItemsTest()
{
  var factory = new OrderValidatorFactory
  {
    CouponProvider = GetCouponProviderStub(hasAppliedCoupons: true),
    SaleItemsProvider = GetSaleItemsProviderStub(hasSaleItems: true)
  };

  var sut = factory.GetOrderValidator();

  Assert.IsTrue(sut.HasOverlappingDiscounts);
}
```

As you can see, there’s a lot of repetition. The setup for each test is remarkably similar. Further, the the assert is identical in the first three methods. We’ve already introduced helper methods to create our stubs, but the tests are still very similar. There must be something more we can do to reduce the amount of duplicated code. Fortunately, NUnit has the concept of parametertized testing.

## First Step to Parameterized Testing

Parameterized testing is simply passing values into a test method through method parameters rather than hard coding values within the method itself.

Let’s refactor our first test to take advantage of parametertized testing. We can apply the `TestCase` attribute with two pieces of data.

```csharp
[TestCase(false, false)]
public void HasOverlappingDiscountsIsFalseIfNoCouponsAndNoSaleItemsTest(bool hasAppliedCoupons, bool hasSaleItems)
{
  var factory = new OrderValidatorFactory
  {
    CouponProvider = GetCouponProviderStub(hasAppliedCoupons: hasAppliedCoupons),
    SaleItemsProvider = GetSaleItemsProviderStub(hasSaleItems: hasSaleItems)
  };

  var sut = factory.GetOrderValidator();

  Assert.IsFalse(sut.HasOverlappingDiscounts);
}
```

So, what’s happening here? The two parameters in `TestCase` become values passed into the test method’s parameters. If we were to run these tests, the test runner would run this test passing `false` to both `hasAppliedCoupons` and `hasSaleItems`. So, although the refactored test looks different, it is functionally equivalent to the original.

If we rerun our tests, they still all pass.

Note that the `Test` attribute is now redundant, so we’ve removed it.

So far we haven’t reduced any duplication, but we’re about to get some pay-off for our efforts.

## Multiple Test Cases Per Method

As it turns out, you can add multiple `TestCase` attributes to a single unit test. By adding two more `TestCase` attributes with the appropriate values, we can increase the amount of coverage this test provides.

```csharp
[TestCase(false, false)]
[TestCase(true, false)]
[TestCase(false, true)]
public void HasOverlappingDiscountsIsFalseIfNoCouponsAndNoSaleItemsTest(bool hasAppliedCoupons, bool hasSaleItems)
{
  var factory = new OrderValidatorFactory
  {
    CouponProvider = GetCouponProviderStub(hasAppliedCoupons: hasAppliedCoupons),
    SaleItemsProvider = GetSaleItemsProviderStub(hasSaleItems: hasSaleItems)
  };

  var sut = factory.GetOrderValidator();

  Assert.IsFalse(sut.HasOverlappingDiscounts);
}
```

Now, the test runner will execute this test method 3 times, once with both test parameters set to `false`, once with only the first parameter set to `true`, and once with only the second parameter set to `true`. In each execution it still asserts that the value of `HasOverlappingDiscounts` is `false` which is the correct behaviour.

Sure enough, all tests still pass. So far, so good.

We can now delete tests two and three since our first test now tests the same logic.

## What to Do With the Positive Test

This is nice, but it would be even nicer to eliminate the need for the fourth test. The two remaining tests still contain duplicate code. To that end, let’s start by refactoring the fourth test into the same format as the first.

```csharp
[TestCase(true, true)]
public void HasOverlappingDiscountsIsTrueIfHasCouponsAndHasSaleItemsTest(bool hasAppliedCoupons, bool hasSaleItems)
{
  var factory = new OrderValidatorFactory
  {
    CouponProvider = GetCouponProviderStub(hasAppliedCoupons: hasAppliedCoupons),
    SaleItemsProvider = GetSaleItemsProviderStub(hasSaleItems: hasSaleItems)
  };

  var sut = factory.GetOrderValidator();

  Assert.IsTrue(sut.HasOverlappingDiscounts);
}
```

In this refactoring step, we’ve removed no duplication. However, if you compare this against the first test, there is only one difference (apart from the values in the `TestCase` attribute); in this test we’re asserting the value of `HasOverlappingDiscounts` is true.

So how do we eliminate the remaining duplication?

## One Approach to Merge Tests

Given what we know so far, the obvious choice would be to add a third bool parameter to our methods called `expected`. From there, instead of having one test with `Assert.IsFalse` and one with `Assert.IsTrue`, we could have a single test with `Assert.AreEqual`.

```csharp
[TestCase(false, false, false)]
[TestCase(true, false, false)]
[TestCase(false, true, false)]
[TestCase(true, true, true)]
public void HasOverlappingDiscountsIsFalseIfNoCouponsAndNoSaleItemsTest(bool hasAppliedCoupons, bool hasSaleItems, bool expected)
{
  var factory = new OrderValidatorFactory
  {
    CouponProvider = GetCouponProviderStub(hasAppliedCoupons: hasAppliedCoupons),
    SaleItemsProvider = GetSaleItemsProviderStub(hasSaleItems: hasSaleItems)
  };

  var sut = factory.GetOrderValidator();

  Assert.AreEqual(expected, sut.HasOverlappingDiscounts);
}
```

This would work correctly, but there is a slightly more readable way we can express the same thing.

## Expected Result

Conveniently, the `TestCase` attribute accepts a named parameter, `ExpectedResult` (or `Result` in NUnit 2.x).

This attribute tells the test runner what the result of the test method should be. Instead of explicitly asserting within the method, we simply return the value we’re testing. The test runner will do the assert for us.

Functionally, using `ExpectedResult`is equivalent to passing a third parameter to assert against. But the benefit it gives is increased readability. It clearly communicates to other developers the expected outcome of the test method.

So let’s see what that would look like and refactor both of our remaining test methods to take advantage of `ExpectedResult`.

```csharp
[TestCase(false, false, ExpectedResult = false)]
[TestCase(true, false, ExpectedResult = false)]
[TestCase(false, true, ExpectedResult = false)]
public bool HasOverlappingDiscountsIsFalseIfNoCouponsAndNoSaleItemsTest(bool hasAppliedCoupons, bool hasSaleItems)
{
  var factory = new OrderValidatorFactory
  {
    CouponProvider = GetCouponProviderStub(hasAppliedCoupons: hasAppliedCoupons),
    SaleItemsProvider = GetSaleItemsProviderStub(hasSaleItems: hasSaleItems)
  };

  var sut = factory.GetOrderValidator();

  return sut.HasOverlappingDiscounts;
}

[TestCase(true, true, ExpectedResult = true)]
public bool HasOverlappingDiscountsIsTrueIfHasCouponsAndHasSaleItemsTest(bool hasAppliedCoupons, bool hasSaleItems)
{
  var factory = new OrderValidatorFactory
  {
    CouponProvider = GetCouponProviderStub(hasAppliedCoupons: hasAppliedCoupons),
    SaleItemsProvider = GetSaleItemsProviderStub(hasSaleItems: hasSaleItems)
  };

  var sut = factory.GetOrderValidator();

  return sut.HasOverlappingDiscounts;
}
```

If we run these tests, they both pass.

Take a look at the two test methods. They are now identical, except for their respective `TestCase` attributes.

## And Then There Was One

Since we can have multiple `TestCase` attributes per test, we can move the test case from the fourth test up to the first and then delete the fourth test method.

At the same time, we’ll rename the remaining test to more accurately reflect what we’re testing.

```csharp
[TestCase(false, false, ExpectedResult = false)]
[TestCase(true, false, ExpectedResult = false)]
[TestCase(false, true, ExpectedResult = false)]
[TestCase(true, true, ExpectedResult = true)]
public bool HasOverlappingDiscountsIsTrueOnlyIfHasAppliedCouponsAndHasSaleItemsTest(bool hasAppliedCoupons, bool hasSaleItems)
{
  var factory = new OrderValidatorFactory
  {
    CouponProvider = GetCouponProviderStub(hasAppliedCoupons: hasAppliedCoupons),
    SaleItemsProvider = GetSaleItemsProviderStub(hasSaleItems: hasSaleItems)
  };

  var sut = factory.GetOrderValidator();

  return sut.HasOverlappingDiscounts;
}
```

By applying the `TestCase` attribute, we’ve been able to refactor out all the duplicated code. We’re down to a single method from four.

Compare this to the original four unit tests in terms of readability. The test name still describes the behavior of the method, but it does so in a single test name rather than across four test names. Further, all the inputs and their expected outcomes are displayed above the test. Any developer looking at this method can get an accurate picture of the method’s behavior in a single glance.

In terms of maintainability and readability, this refactoring is a clear win.

Obviously, not all unit tests are so simple, but there are similar techniques you can use to wrangle most tests into a more readable format with less duplication. If there’s interest, we can look at some of those in future posts.
