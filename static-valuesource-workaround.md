# Use a Non-Static ValueSource in NUnit 3

Starting in NUnit 3, `TestCaseSource` and `ValueSource` must reference static members. This restriction really gets in the way sometimes; some methods or properties cannot be made static.

## The Work Around

When my employer switched to NUnit 3, several of the base fixtures we had created started failing. These base fixtures saved us tonnes of time writing boiler plate code. However, they relied heavily on abstract methods. As you may know, abstract methods cannot be made static. To retain our base classes, we developed the following work around taking advantage of NUnit 3’s `Assert.Multiple`.

Say you start with the following code. Our problem is that `NonStaticValueSource` cannot be made static.

```csharp
[Test]
public void MyTest([ValueSource("NonStaticValueSource")] int value)
{
  int expected = value;
  var otherClass = new OtherClass();
  otherClass.Value = value;
  var sut = new ClassUnderTest(otherClass);
  Assert.AreEqual(expected, sut.DoThing());
}
```

### Step 1, replace `ValueSource` with `foreach`

Firstly, remove the `ValueSource` attribute, and the test parameter. Next, iterate over the the result of the source method with the test method.

```csharp
[Test]
public void MyTest()
{
  var otherClass = new OtherClass();

  foreach(int value in NonStaticValueSource())
  {
    int expected = value;
    otherClass.Value = value;
    var sut = new ClassUnderTest(otherClass);
    Assert.AreEqual(expected, sut.DoThing());
  }
}
```

The test method itself is not static, so it may access `NonStaticValueSource` just fine.

The unfortunate side effect of this approach is that the test runner will stop running after the first failure. Imagine that `NonStaticValueSource` returns 1000 values. If the first value causes the assert to fail, the test runner will skip stop the method. Due to the first failure, the test runner skips the remaining 999. Fortunately, we can get around this.

### Step 2, use `Assert.Mutliple`

Next, apply `Assert.Multiple` to the problem. `Assert.Multiple` allows you to run an arbitrary code block with as many assert calls as you wish (documentation).

```csharp
[Test]
public void MyTest()
{
  var otherClass = new OtherClass();

  Assert.Multiple(() =>
  {
    foreach(int value in NonStaticValueSource())
    {
      int expected = value;
      otherClass.Value = value;
      var sut = new ClassUnderTest(otherClass);
      Assert.AreEqual(expected, sut.DoThing());
    }
  }
}
```

Now all the values of from `NonStaticValueSource` will be tested. The output from the test will show all the failures, if any.

This isn’t pretty, but it gets the job done.

## Multiple ValueSources or TestCaseSources

If you have multiple non-static `ValueSource`s, you’ll have to use nested loops.

If you have a non-static `TestCaseSourse` which passes more than one value or has specified return value, you’ll have to do more work. You can create a new private nested class which encapsulates the data, or, if there’s only two, you can use a `Tuple`.

## Warning

Note that, depending on your test runner, the test name may change. If you’re using something like Team City, that means the original test will never show as fixed because it ceases to exist. In that case, you can mark it as fixed and it will go away.
