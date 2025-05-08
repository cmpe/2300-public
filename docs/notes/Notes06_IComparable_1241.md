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
  // Optional specialized version, saves a cast/conversion
  public bool Equals(CBox cbox)
  {
    // null ? return false
    if (cbox is null) return false; 

    // or just Width
    return (Width == cbox.Width && Height == cbox.Height);
  }
  public override int GetHashCode()
  {
    return 1; // ok, not used in a Hashtable or related context
  }
  // IComparible demands that you supply : int CompareTo( object ) method,
  //   - no assumptions as to argument type, must be "safe" but you are expected
  //     to throw an exception for bad types
  public int CompareTo(object arg)
  {
    if (!(arg is CBox argBox ))
      throw new ArgumentException($"CBox:CompareTo:{nameof(arg)} - Not a valid CBox");
    //return (Width * Height) - (tmp.Width * tmp.Height);
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
{ // CBox - Comparible<CBox> supporting interface example
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