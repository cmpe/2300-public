- [Generic Extension Methods – Basic (Non-Predicate)](#generic-extension-methods--basic-non-predicate)
  - [Common Standard Query Categories and Methods](#common-standard-query-categories-and-methods)
  - [IEnumerable](#ienumerable)
    - [Common Extension Methods](#common-extension-methods)
  - [Deferred Execution](#deferred-execution)
    - [Dealing with `IEnumerable` Return Collections](#dealing-with-ienumerable-return-collections)
      - [Option 1: Use `IEnumerable` as a Collection](#option-1-use-ienumerable-as-a-collection)
      - [Option 2: Use the Collection Constructor Overloads to Make a New List](#option-2-use-the-collection-constructor-overloads-to-make-a-new-list)
      - [Option 3: Use One of the `ToXXX()` Helpers like `ToList()`](#option-3-use-one-of-the-toxxx-helpers-like-tolist)
    - [Using Various Extension Methods with Previous Options](#using-various-extension-methods-with-previous-options)
  - [Extension Methods by Design - Extras - outside cmpe2300 requirements](#extension-methods-by-design---extras---outside-cmpe2300-requirements)
# Generic Extension Methods – Basic (Non-Predicate)

Generic extension methods (methods that utilize `<>` in their declaration) that work without having to create predicates. It is important to note that all these methods do not modify the list or their contents; instead, they all return a new collection (`IEnumerable`) comprised of elements from the argument collections that fulfill the operation. These collections are not quite what they seem – more in deferred execution details to come.

The signatures of these methods often indicate that they do NOT actually return a matching type list, instead returning an `IEnumerable<T>` object.

## Common Standard Query Categories and Methods

**Set**: These operators perform set operations on the input sequence. Examples are:
 - [`Concat`](https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.concat?view=net-7.0)
 - [`Union`](https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.union?view=net-7.0)
 - [`Intersect`](https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.intersect?view=net-7.0)
 - [`Except`](https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.except?view=net-7.0)
 - [`Distinct`](https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.distinct?view=net-7.0)

**Filtering**: These operators filter the input sequence based on a predicate. Examples are:
- [`Where`](https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.where?view=net-7.0)
- [`OfType`](https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.oftype?view=net-7.0)
- [`SkipWhile`](https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.skipwhile?view=net-7.0)

**Ordering**: These operators sort the input sequence according to a specified key or comparer. Examples are:
- [`OrderBy`](https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.orderby?view=net-7.0)
- [`ThenBy`](https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.thenby?view=net-7.0)
- [`Reverse`](https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.reverse?view=net-7.0)
- [`OrderByDescending`](https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.orderbydescending?view=net-7.0)

**Conversion**: These operators convert the input sequence into a different type or format. Examples are:
- [`ToArray`](https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.toarray?view=net-7.0)
- [`ToList`](https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.tolist?view=net-7.0)
- [`ToDictionary`](https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.todictionary?view=net-7.0)
- [`AsEnumerable`](https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.asenumerable?view=net-7.0)

**Quantifier**: These operators return a boolean value that indicates whether some or all elements in the input sequence satisfy a given condition. Examples are:
- [`Any`](https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.any?view=net-7.0)
- [`All`](https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.all?view=net-7.0)
- [`Contains`](https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.contains?view=net-7.0)

**Projection**: These operators transform the input sequence into a new sequence of a different type. Examples are:
- [`Select`](https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.select?view=net-7.0)
- [`SelectMany`](https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.selectmany?view=net-7.0)
- [`Zip`](https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.zip?view=net-7.0)

**Element**: These operators return a single element from the input sequence based on a specified condition or position. Examples are:
- [`First`](https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.first?view=net-7.0)
- [`Last`](https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.last?view=net-7.0)
- [`Single`](https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.single?view=net-7.0)
- [`ElementAt`](https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.elementat?view=net-7.0)

**Generation**: These operators create a new sequence from scratch or based on an existing sequence. Examples are:
- [`Range`](https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.range?view=net-7.0)
- [`Repeat`](https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.repeat?view=net-7.0)
- [`Empty`](https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.empty?view=net-7.0)
- [`DefaultIfEmpty`](https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.defaultifempty?view=net-7.0)

**Aggregate**: These operators perform a calculation on the input sequence and return a single value as the result. Examples are:
- [`Count`](https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.count?view=net-7.0)
- [`Sum`](https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.sum?view=net-7.0)
- [`Average`](https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.average?view=net-7.0)
- [`Aggregate`](https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.aggregate?view=net-7.0)

The following fall outside the scope of our material:

**Joining**: These operators combine elements from two or more sequences based on a common key. Examples are:
- [`Join`](https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.join?view=net-7.0)
- [`GroupJoin`](https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.groupjoin?view=net-7.0)
- [`Zip`](https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.zip?view=net-7.0)

**Grouping**: These operators group the input sequence into sub-sequences based on a common key. Examples are:
- [`GroupBy`](https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.groupby?view=net-7.0)
- [`ToLookup`](https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.tolookup?view=net-7.0)


## IEnumerable

The interface describes an object (collection) that can be enumerated ( ie. `foreach()`). It also supports a set of utility methods. One of these is `ToList()` which will convert the `IEnumerable` collection into a `List` of the original type. For example:

```csharp
List<Thing> MyNewListOfBoth = OneList.Concat(TwoList).ToList(); // take the result of Concat() and invoke ToList().
```

### Common Extension Methods

- `Concat(collection)`: Generate a new collection that combines the argument collection to the invoking collection (possible duplicates – see `Union`). `A.Concat(B)` results in a collection ordered as first A elements then B elements in their original order.
- `Distinct()`: Will return a new collection with duplicates eliminated (uses `Equals()` of the collection object). `MyDistinctList = OneList.Distinct().ToList();`
- `ElementAt(index)`: Similar to subscript access `[]`, available for .NET support where languages do not have `[]`.
- `Except(collection)`: Generate a new collection based on the invoking collection, but without any elements from the argument collection (uses `Equals()`). `A.Except(B)` will return a collection with all elements from A that are NOT in collection B.
- `Intersect(collection)`: Generate a new collection that is the common elements between the collections (uses `Equals()`). `A.Intersect(B)` will return a new collection where the elements exist in both A and B collections.
- `Union(collection)`: Generate a new collection that is the combination of both collections but without duplicates (similar to `Concat()`).

*Note*: `Concat()`, `Except()`, `Intersect()`, `Distinct()`, and `Union()` do not require pre-sorting, and as such are not very efficient. To reduce the overhead in execution of the operations, these implement a paradigm known as deferred execution. In a nutshell, it means the actual operation execution time is quick (because the full operation is not performed), instead most of the processing is performed as the collection is enumerated (as values are pulled out like in a `foreach()`). Each is different, `Concat()` is the “cheapest”, and `Except()` and `Intersect()` the most expensive. For large collections, these algorithms are poor performers since the collections do not need to be sorted and that condition cannot be leveraged – Think about it, if the collections were sorted, the algorithms could be very efficient.

## Deferred Execution

Extension methods that return `IEnumerable` collections often incorporate an implementation mechanism known as deferred execution. A call to a method using deferred execution would actually not perform any meaningful work other than some initialization and returning an Enumerator (iterator - something that can be iterated – i.e., `foreach()`).

The work is done as each element is retrieved/iterated. How would this work?

```csharp
// The following example, broken down
IEnumerable<int> A = aList.Concat(bList); // Whiteboard intervention here!
```

Given a starting situation as:

```csharp
{ // Lists, Extension Methods
  List<int> aList = new List<int>();
  List<int> bList = new List<int>();
  for (int i = 0; i < 10; ++i) // Add 10 random values into each list
  {
    aList.Add(rnd.Next(10));
    bList.Add(rnd.Next(10));
  }  
  WriteLine("Both lists aList, bList"); 
  foreach (int iVal in aList)   // Dump the aList to the console
    Write($"{iVal} ");
  WriteLine();
  foreach (int iVal in bList)   // Dump the bList to the console
    Write($"{iVal} ");
  WriteLine(Environment.NewLine);
}
```

### Dealing with `IEnumerable` Return Collections

#### Option 1: Use `IEnumerable` as a Collection

```csharp
IEnumerable<int> A = aList.Concat(bList); // can't cast to List<int>, save collection
WriteLine("Concat() of lists, output via IEnumerable<>:");
foreach (int i in A) // an IEnumerable collection can be enumerated, so foreach()
  Write($"{i},");
WriteLine(Environment.NewLine);
```

#### Option 2: Use the Collection Constructor Overloads to Make a New List

- Either indirectly using the returned `IEnumerable` collection:

```csharp
List<int> newList_A = new List<int>(A); // using A from above
```

- Or directly, putting the extension operation in as the argument:

```csharp
List<int> newList = new List<int>(aList.Concat(bList));
// or by self-assignment to replace it
aList = new List<int>(aList.Concat(bList)); // concat and self-assign back with new list
```

#### Option 3: Use One of the `ToXXX()` Helpers like `ToList()`

```csharp
// can't cast to List<int>, use helper ToList() which resolves your list type
aList = aList.Concat(bList).ToList(); // note the assignment back to aList

WriteLine("Concat() of lists, output via ToList() helper:");
foreach (int i in aList)
  Write($"{i},");
WriteLine(Environment.NewLine);
```

### Using Various Extension Methods with Previous Options

```csharp
// use ToList() to allow self-assignment
bList = bList.Distinct().ToList(); 

WriteLine("Distinct only in bList");
foreach (int iVal in bList)   // Dump the list to the console
  Write($"{iVal} ");
WriteLine(Environment.NewLine);

int b = bList.ElementAt(0); // same as int b = bList[0];
int ix = bList.IndexOf(b);  // Should be index = 0, the match to int b

List<int> ExceptList; // or = null; just a reference to a List<int>, no list yet
ExceptList = aList.Except(bList).ToList(); // All except those in argument list

WriteLine("New list from aList except those in bList");
foreach (int iVal in ExceptList)   // Dump the list to the console
  Write($"{iVal} ");
WriteLine(Environment.NewLine);

// Union, like Concat, but excludes duplicates, defers execution until enumeration
List<int> UnionList = aList.Union(bList).ToList(); // declare and assign in 1 step

WriteLine("New list Union of aList and bList - invoking + arg ordering");
WriteLine("Note: Union will exclude duplicates in build of UnionList");
foreach (int iVal in UnionList)   // Dump the list to the console
  Write($"{iVal} ");
WriteLine(Environment.NewLine);

// Intersect - distinct elements common to both, defers execution until execution
// - Take Union list, intersect with b, should get b back out
List<int> IntersectList = new List<int>(UnionList.Intersect(bList)); // Use Constructor

WriteLine("New list Intersect of UnionList and bList - no presort required");
foreach (int iVal in IntersectList)   // Dump the list to the console
  Write($"{iVal} ");
WriteLine(Environment.NewLine);

// If order required, sort last, as both Union and Intersect do not maintain order
```

## Extension Methods by Design - Extras - outside cmpe2300 requirements

Extension methods are just that... methods that extend the functionality of a class.

Extension methods in the case of collections are an appropriate use of the extension construct. They are written against a managed class hierarchy and make sense as a way to leverage the design of the collection classes. Beware of writing your own extension methods if the underlying type is not your own, as it may change and thereby cause your methods to fail. That said, they work great and can be a handy tool for the toolbox.

In the simplest case, a double:

```csharp
public static class MyExtensions
{
  // Return the appropriate type, the first argument
  // is always decorated with this - and is populated with the invoking instance
  public static string ToFixed2(this double dVal) // You won't see this in intellisense
  {
    return Math.Round(dVal, 2).ToString();
  }
  // You may add more parameters, provided they come after the first "hidden" this
  public static string ToFixed(this double dVal, int iNumDigits)
  {
    return Math.Round(dVal, iNumDigits).ToString();
  }
}
```

Then:

```csharp
{// Extension Methods
  double dVal = Math.PI;
  Console.WriteLine($"dVal = {dVal.ToFixed2()}"); // Note: No arguments, this is dVal
  Console.WriteLine($"dVal = {dVal.ToFixed(6)}");
}
```

Or with a handy familiar class.... `CDrawer`:

```csharp
public static class MyExtensions
{
  public static void DrawImage(this CDrawer canvas, Image img)
  {
    if (img == null) return;
    Bitmap tmpBit = new Bitmap(img); // make temp actual bitmap
    for (int r = 0; r < canvas.ScaledHeight; r++)
      for (int c = 0; c < canvas.ScaledWidth; c++)
        canvas.SetBBPixel(c, r, tmpBit.GetPixel(c, r));
    canvas.Render();
  }
}
```

Then:

```csharp
CDrawer canvas = new CDrawer();
canvas.DrawImage(Image.FromFile("Kirk.png")); // invoke our extension method
```