- [`IComparable` – the interface](#icomparable--the-interface)
- [`Comparison<>`](#comparison)
- [`IComparer<T>` - Example only](#icomparert---example-only)
# `IComparable` – the interface

Being able to compare class instances is also a common desire. This most often occurs if we want to be able to sort our data. Sorted data is a requirement for many algorithms based on starting with a sorted dataset (like BinarySearch). Unlike `Equals()` there is NO default comparison operation available as a default. In order to implement basic comparison, we must involve ourselves in implementing an interface (much more on interfaces later). Fortunately, implementing the `IComparable` interface is straight forward and provides an excellent introduction to using and implementing interfaces.

An interface looks like a class definition, but with no fields, just has method declarations only – no actual implementation. It is named interface and defines a contract whereby all classes which choose to implement the interface will be required to implement the contracted methods. In other words, by incorporating an interface into a class, the class will not be complete until all methods defined by the interface have been implemented.

Convention is that interface names start with a capital I. For comparison to be supported for our class, we implement the `IComparable` interface.

```csharp
class Box : IComparable // now with comparison via CompareTo()
```

The `IComparable` interface defines a single method, `int CompareTo(object)`, to complete our class we merely implement this method following these basic rules for CompareTo() return values :
1. if the instance object is considered “less” than the object argument, return a negative value
2. if the instance object is considered “equal” to the object argument, return 0
3. if the instance object is considered “greater” than the object argument, return a positive value

-1, 0, 1 are recommended values, and the magnitude of the values is not used ie. -88, means same as -1

If the object being compared is not of the class type, unlike `Equals()`, you should throw a new ArgumentException with an appropriate message.

Annoyingly, if you fail to support the `CompareTo()` interface, and you attempt to `Sort()`, it will compile and throw an `InvalidOperationException` at runtime indicating that: Failed to compare two elements in the array.

Upon completion, you may programmatically use `CompareTo()`, as well, all utility algorithms may now use this “contract” method to do comparisons for sorting, searching etc.

Also like `Equals()`, providing support through `IComparable` does NOT mean you can use `<`, `>`, etc. These operators do not automatically invoke your `CompareTo()`. To provide that functionality you must overload the desired operators manually. As the general convention with the c# community is to not overload operators, we will not go down this path.

In the more than you need to know right not category: Generic collections add an additional level of complexity when comparison is required, by default they implement comparison via a comparer object which has the default functionality of interrogating the underlying type to see if it implements a `IComparable`<T> interface (a generic one), if not it then looks for an implementation of `IComparable`. This fortunately works in our favor and the functionality we desire is the ultimate result. * This is also implemented for equality through IEquatable<T> which luckily defaults to our `Equals()` override.

```csharp
class CBox : `IComparable` // support comparisons for library use
{
  private int Width;
  private int Height;
  public CBox( int Width, int Height)
  {
    this.Width = Width;
    this.Height = Height;
  }
  // override, most basic Equals(object)
  public override bool Equals(object obj)
  {
    // not a CBox ? return false
    if (!(obj is CBox arg)) return false;

    // Determine Value `Equals()`
    return (Width == arg.Width && Height == arg.Height);
  }
  public override int GetHashCode()
  {
    return 1; // ok, not used in a Hashtable or related context
  }
  // IComparable demands that you supply : int CompareTo( object ) method,
  //   - no assumptions as to argument type, must be "safe" but you are expected
  //     to throw an exception for bad types
  public int CompareTo(object arg)
  {
    if (!(arg is CBox argBox ))
      throw new ArgumentException($"CBox:CompareTo:{nameof(arg)} - Not a valid CBox");
    // return (Width * Height) - (tmp.Width * tmp.Height); - valid, but not best implementation
    // Use area for its "magnitude", this will vary Or Let int.CompareTo() do the work !
    return (Width * Height).CompareTo(argBox.Width * argBox.Height); 
  }
  public override string ToString()
  {
    return $"({Width},{Height})";
  }
}
```

```csharp
{ // CBox - Comparable<CBox> supporting interface example
  // 2 different objects, same field values
  CBox a = new CBox(2, 2);
  CBox b = new CBox(1, 1);

  // Compare our Value based `Equals()` to the Identity based ==/!=
  WriteLine($"a.Equals(b) returned : {a.Equals(b)}");
  WriteLine($"a.CompareTo(b) : {a.CompareTo(b)}");
  WriteLine($"b.CompareTo(a) : {b.CompareTo(a)}");
  // CompareTo() - negative if invoking is smaller, 0 if equal and positive if larger than argument
  // Be sure that if a.CompareTo(b) returns a positive value, b.CompareTo(a) returns a negative..etc

  // Containers leverage off CompareTo(), some by default - SortedDictionary(), others via utility
  //  functions like Sort, Merge, etc
  List<CBox> list = new List<CBox>();
  list.Add(new CBox(3, 3));
  list.Add(new CBox(2, 2));
  list.Add(new CBox(1, 1));
  list.Sort(); // Needs a way to order the set, Sort will find and use your IComparable support
  WriteLine($"{list[0]} - {list[1]} - {list[2]}");
}
```
What if our requirement is for a 2-tier ordering ( or more ) ? As in SQL parlance, we want sub-sorting.

An example for our CBox example is to not sort just by area, rather order the CBoxes by Height *within* Length. Read this as Ascending Width, but if the Width is same, then Height is considered
as the 2nd tier order comparision. Algorithmically, then if Widths differ, the result is determined, BUT, if the Widths are the same, we must return based on Height.
```csharp
  public int CompareTo(object arg)
  {
    // Same sanity check required
    if (!(arg is CBox argBox ))
      throw new ArgumentException($"CBox:CompareTo:{nameof(arg)} - Not a valid CBox");

    // Check each successive tier, Width first
    int iReturn = Width.CompareTo( argBox.Width ); // Compare Widths
    if( iReturn == 0 ) // Widths are the same ! Must Check next tier, Height
      iReturn = Height.CompareTo( argBox.Height ); // Compare Height
    // iReturn should hold appropriate tier outcome
    return iReturn;
  }
```

# `Comparison<>`
To allow alternate methods of sorting there are a several choices. Examination of the `Sort()` method indicates a number of overloads. One version accepts a `Comparison<>` and another a `IComparer<T>` interface.

The simplest version is to supply a `Comparison<>` method
You generate a static method that conforms to the signature definition similar to CompareTo()

ie. `[internal/public] static int MyComparison( type arg1, type arg2 )` 

// where the type is your list type, note that since the method is static there is no `“this”`, and as such both comparison objects arrive as arguments to the method.

// `Comparison<int>` method for example, as a form member comparison
```csharp
public static int MyCompare(int arg1, int arg2) // 2 ARGS required here
{
  return -1 * arg1.CompareTo(arg2); // Leverage CompareTo, but reverse order to get descending
}

//Then :
        aList.Sort(MyCompare); // Note the MyCompare is not () invoked !, the name is the arg
```


Or a more typical use with user defined classes :
```csharp
internal class Thing : IComparable // recall internal is public WITHIN this assembly/exe/dll
{	
  public int X { get; private set; }
  public Thing( int iVal )
  {
    X = iVal;
  }
  // Other details omitted
  // Comparison<> function for use with alternate sort requirements
  // static helper for Thing based Collections to sort by
  // Note : the necessary null tests, note negation of normal singular null tests
  internal static int MyThingCompare(Thing arg1, Thing arg2) // 2 arguments here, no "this" 
  {
    if (arg1 == null && arg2 == null) return 0; // same, both null
    if (arg1 == null) return -1; // ascending, null is "less" than anything, so arg1 is LESS
    if (arg2 == null) return 1; // ascending, null is "less" than anything, so arg1 is MORE
    // Both not null here
    return arg1.X.CompareTo(arg2.X); // normal sort of int
  }
}
```
```csharp
// Now sort it, supply MyThingCompare method as comparer
// For use with : Sort(Comparison<Thing> Comparison); // signature overload
// provide EntryPoint to automatically create delegate used to Sort()

things.Sort(Thing.MyThingCompare);

WriteLine("List of Thing, sorted by MyThingCompare : ");
foreach (Thing t in things)   // Dump the list to the console
  Write("{t.X} ");
WriteLine(Environment.NewLine);
```

 # `IComparer<T>` - Example only
The second version is somewhat more complex, but will allow for greater extensibility – this is shown for example purposes only.

You generate a new class that supports the `IComparer<>` interface, which supports a Compare method with a signature similar to `CompareTo()` ie. `public int Compare( type arg1, type arg2 )` // where the type is the container type. Since the Compare is wrapped in a class instance, the `Compare()` method can be programmatically assigned to sort in multiple user defined ways.

// MyCompareClass, to support int Collections : IComparer<int>
// Using class so can implement dynamic sort selection by sort type selection member
```csharp
internal class MyCompareClass : IComparer<Thing>
{
  public enum ESortType { Ascending, Descending }; // possible sort options
  public ESortType SortType { get; set; } //auto public property - NO public field instances
  public MyCompareClass()
  {
    SortType = ESortType.Ascending; // default sort set to Ascending
  }
  // Invoked by Sort when MyCompareClass instance is passed
  public int Compare(Thing arg1, Thing arg2) 
  {
    switch (SortType)                     // Determine sort type
    {
      case ESortType.Ascending:
        return arg1.CompareTo(arg2);      // Leverage, just arrange order to suit
      case ESortType.Descending:
        return -1 * arg1.CompareTo(arg2);      // Same, Leverage, but reverse order
      default:
        break;
    }
    return 0; // unknown sort, unreachable
  }
}
```
Then to use it make an instance of your Comparer class and use it as an argument within the Sort() call, this following shows the benefit and possibilities of being able to programmatically switch sorts.
```csharp
MyCompareClass comp = new MyCompareClass();
thangs.Sort(comp); // Sort it default sort of class, Ascending
WriteLine("Union Ascending sort via MyCompareClass and IComparer<int> support");
foreach (Thing t in thangs)   // Dump the list to the console
  Write($"{t.X} ");
WriteLine(Environment.NewLine);

// Or change sort order programmatically, via SortType
comp.SortType = MyCompareClass.ESortType.Descending;
thangs.Sort(comp); // Sort it default sort of class, Ascending
WriteLine("Union Descending sort via MyCompareClass and IComparer<int> support");
foreach (Thing t in thangs)   // Dump the list to the console
  Write("{t.X} ");
WriteLine(Environment.NewLine);
```
           
