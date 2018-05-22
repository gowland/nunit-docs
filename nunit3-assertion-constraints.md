# NUnit 3 Assertion Constraints By Example

At some point the old style assertions like `Assert.AreEqual` will be depricated. The good news is they've been replaced with something even better: assertion constraints.

Constraints have a few of advantages:
* Intent is communicated more clearly
* In the case of failure, the test output describes the actual problem more clearly

There are many more assertion constraints available than given below. Check out their documentation: https://github.com/nunit/docs/wiki/Constraints

## Verify Equality
### Old
```csharp
Assert.AreEqual(5, 4);
```
#### Output
```
Expected: 5
But was:  4
```

### NUnit3
```csharp
Assert.That(4, Is.EqualTo(5));
```
#### Output
```
Expected: 5
But was:  4
```

## Verify Exception Thrown
### Old
```csharp
Assert.Throws<ArgumentException>(() => sut.DoThingThatDoesNotThrow());
```
#### Output
```
Expected: <System.ArgumentException>
But was:  null
```

### NUnit3
```csharp
Assert.That(() => sut.DoThingThatDoesNotThrow(), Throws.ArgumentException);
```
#### Output
```
Expected: <System.ArgumentException>
But was:  no exception thrown
```

## Verify Exception Thrown with Message Containing String
### Old
```csharp
try
{
    sut.DoThingThatDoesNotThrow();
}
catch (ArgumentException e)
{
    if (e.Message.Contains("blah"))
    {
        Assert.Pass("Threw an exception containing 'blah'");
    }
}
Assert.Fail("Should have thrown an exception containing 'blah'");
```
#### Output
```
Should have thrown an exception containing 'blah'
```

### NUnit3
```csharp
Assert.That(() => sut.DoThingThatDoesNotThrow(), Throws.ArgumentException.With.Message.Contains("blah"));
```
#### Output
```
Expected: <System.ArgumentException> and property Message containing "blah"
But was:  no exception thrown
```

## Verify no Exception Thrown
### Old
```csharp
bool throws = false;
try
{
    sut.DoDangerousThing();
}
catch (Exception)
{
    throws = true;
}

Assert.IsFalse(throws);
```
#### Output
```
Expected: False
But was:  True
```

### NUnit3
```csharp
Assert.That(() => sut.DoDangerousThing(), Throws.Nothing);
```
#### Output
```
Expected: No Exception to be thrown
But was:  <System.ArgumentException: blah
```

## Verify List of Values is in Range
### Old
```csharp
var minDate = new DateTime(2018, 5, 9);
var maxDate = new DateTime(2018, 5, 10);

var listOfDates = new[]
{
    new DateTime(2018, 5, 9, 13, 59, 57),
    new DateTime(2018, 5, 19, 9, 42, 11),
    new DateTime(2018, 5, 9, 7, 1, 09),
};

Assert.IsTrue(listOfDates.All(d => d >= minDate && d < maxDate));