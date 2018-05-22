# NUnit 3 Assertion Constraints By Example

At some point the old style assertions like `Assert.AreEqual` will be depricated. The good news is they've been replaced with something even better: assertion constraints.

Constraints have a few of advantages:
* Intent is communicated more clearly
* In the case of failure, the test output describes the actual problem more clearly, especially for more complex cases

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

```
#### Output
```
Expected: True
But was:  False
```

### NUnit3
```csharp
var minDate = new DateTime(2018, 5, 9);
var maxDate = new DateTime(2018, 5, 10);

var listOfDates = new[]
{
    new DateTime(2018, 5, 9, 13, 59, 57),
    new DateTime(2018, 5, 19, 9, 42, 11),
    new DateTime(2018, 5, 9, 7, 1, 09),
};

Assert.That(listOfDates, Is.All.InRange(minDate, maxDate));
```
#### Output
```
Expected: all items in range (2018-05-09 12:00:00 AM,2018-05-10 12:00:00 AM)
But was:  < 2018-05-09 13:59:57, 2018-05-19 09:42:11, 2018-05-09 07:01:09, 2018-05-09 03:22:03 >
```

## Verify List is Ordered Descending
### Old
```csharp
var listOfDates = new[]
{
    new DateTime(2018, 1, 9, 13, 59, 57),
    new DateTime(2018, 12, 9, 9, 42, 11),
    new DateTime(2018, 12, 9, 7, 1, 09),
};

CollectionAssert.AreEqual(sut.ListOfDates, listOfDates.OrderByDescending(d => d));
```
#### Output
```
Expected is <System.DateTime[3]>, actual is <System.Linq.OrderedEnumerable`2[System.DateTime,System.DateTime]>
  Values differ at index [0]
  Expected: 2018-01-09 13:59:57
  But was:  2018-12-09 09:42:11
```

### NUnit3
```csharp
var listOfDates = new[]
{
    new DateTime(2018, 1, 9, 13, 59, 57),
    new DateTime(2018, 12, 9, 9, 42, 11),
    new DateTime(2018, 12, 9, 7, 1, 09),
};

Assert.That(listOfDates, Is.Ordered.Descending);
```
#### Output
```
Expected: collection ordered, descending
But was:  < 2018-01-09 13:59:57, 2018-12-09 09:42:11, 2018-12-09 07:01:09 >
```

## Verify List Order for Complex Types
### Old
```csharp
var listOfDates = new[]
{
    new DateTime(2018, 1, 9, 13, 59, 57),
    new DateTime(2018, 12, 9, 9, 42, 11),
    new DateTime(2018, 12, 9, 7, 1, 09),
};
CollectionAssert.AreEqual(listOfDates, listOfDates.OrderBy(t => t.Year).ThenByDescending(t => t.Month));
```
#### Output
```
Expected is <System.DateTime[3]>, actual is <System.Linq.OrderedEnumerable`2[System.DateTime,System.Int32]>
Values differ at index [0]
Expected: 2018-01-09 13:59:57
But was:  2018-12-09 09:42:11
```

### NUnit3
You can also use customer comparers to test order; not shown.
```csharp
var listOfDates = new[]
{
    new DateTime(2018, 1, 9, 13, 59, 57),
    new DateTime(2018, 12, 9, 9, 42, 11),
    new DateTime(2018, 12, 9, 7, 1, 09),
};
Assert.That(listOfDates, Is.Ordered.By(nameof(DateTime.Year)).Then.By(nameof(DateTime.Month)).Descending);
```
#### Output
```
Expected: collection ordered by "Year" then by "Month", descending
But was:  < 2018-01-09 13:59:57, 2018-12-09 09:42:11, 2018-12-09 07:01:09 >
```

## Verify Items in List are Distinct
### Old
```csharp
var items = new[]{3,3,4};
CollectionAssert.AreEqual(items, items.Distinct());
```
#### Output
```
Expected is <System.Int32[3]>, actual is <System.Linq.Enumerable+<DistinctIterator>d__64`1[System.Int32]>
Values differ at index [1]
Expected: 3
But was:  4
```

### NUnit3
```csharp
var items = new[]{3,3,4};
Assert.That(items, Is.Unique);
```
#### Output
```
Expected: all items unique
But was:  < 3, 3, 4 >
```
