- [Equals – When Are Apples Not Apples](#equals--when-are-apples-not-apples)
  - [Equals() - Class Instance Equivalence Comparisons](#equals---class-instance-equivalence-comparisons)
    - [When used with value types (ints, floats, structs, or other value types), *value semantics* are used.](#when-used-with-value-types-ints-floats-structs-or-other-value-types-value-semantics-are-used)
    - [When used with reference types (classes), *identity semantics* are used.](#when-used-with-reference-types-classes-identity-semantics-are-used)
  - [Overloading operator==, operator!=](#overloading-operator-operator)
# Equals – When Are Apples Not Apples

## Equals() - Class Instance Equivalence Comparisons

`Equals()` exists for all structs and classes (all objects be they value or reference types).

```csharp
if (thingOne.Equals(thingTwo))
    WriteLine($"They are equal: {thingOne} and {thingTwo} – imagine that");
```

### When used with value types (ints, floats, structs, or other value types), *value semantics* are used.

With value semantics, `Equals()` will return true if the member fields of both objects are the same value. The implication is that two distinct objects with identical member field values will be *true*. This makes sense since stack-based (value-types) are distinct and could not possibly be the same object.

### When used with reference types (classes), *identity semantics* are used.

With *identity semantics*, `Equals()` will return true if the two object references both refer to the same actual heap-based object (i.e., are they actually the same object). The implication is that two references to distinctly different objects (i.e., made with different `new()` calls) will ALWAYS return false, even if the values of the fields within the objects are identical.

<Memory Diagram Here>

This means that `Equals()` is useless to determine if you have a new reference object and you wish to determine if it holds identical field data compared to another (or container of) objects. This makes sense in this case, since references are really just “strings” attached to actual heap-based objects, so equality tests whether the “strings” are attached to the same object.

This default functionality is fine, provided you understand it and use it appropriately.

Often you want to be able to take a reference object and test it with *value semantics*. Where equality would be understood to compare all the fields, or perhaps just some of the fields (i.e., same x, y but size varies would indicate equality). This can be accomplished by overriding the `Equals()` method for a class.

Because `Equals()` already exists, we must override it, not attempt to replace it (compiler will alert you). The first `Equals()` you are overriding accepts an object NOT a class-type argument, it will be up to you to determine if the object is the type you want (`is/as` operator). You must ensure that `Equals()` does not throw exceptions. 

Steps you should perform (in order according to MS guidelines):
- If the object parameter is null, return false – right?
- Use either `is` or `as` to determine if the object is the correct type (your type) and is not null (it's a reference), if not the correct type return false.
    - Using pattern matching, the above two steps can be coded as one step (see example).
- Finally compare the fields/state conditions of your choice and return true/false based on your interpretation of equality.
    - When you do this the wizard helps a bunch but ALSO invokes the `Equals()` of the “base” class, this is yours to remove and replace with your value semantics.

When you do replace `Equals()` with your own value semantic version, there still exists a static `object.ReferenceEquals(obj1, obj2)` version that can be used for identity semantics.

```csharp
// Refresher, x is type == true if x is not null and will cast to type without exception
// y as type – returns y cast to type if y is not null and cast w/o exception
class Box
{
  private int Width;
  private int Height;
  public Box(int Width, int Height)
  {
    this.Width = Width;
    this.Height = Height;
  }
  public override bool Equals(object obj)
  { 
    // using is tests for both null and correct object
    
    // null? or not a Box? return false
    if ( obj is not Box arg) return false; 
    
    // Determine Value Eq
    return (Width == arg.Width && Height == arg.Height); 
  }
  // Optional - overload Equals() for Box type
  //  skip type test and conversion
  public bool Equals(Box box)
  {
    if (box is null) return false; // null? return false
    return (Width == box.Width && Height == box.Height); // or just Width
  }
  public override int GetHashCode()
  {
    return 1; // ok, not used in a Hashtable or related context
  }
  public override string ToString()
  {
    return $"({Width},{Height})";
  }
}
```

Then:

```csharp
// 2 different objects, same field values
Box a = new Box(1, 2);
Box b = new Box(1, 2);

// Compare our Value based Equals() override
WriteLine($"a.Equals(b) returned: {a.Equals(b)}");
// Compare to the Identity based ==/!= ( ReferenceEquals())
WriteLine($"a==b returned: {a == b} ??? What?");
```

`Equals()` - expected, we overloaded the `Equals()` to allow value semantics and selective (user-defined) `Equals` comparisons.

NOTICE: `==` did not return the same as `Equals()`, different mechanism where the default functionality is to use *identity semantics* ( ie. `ReferenceEquals()`). We can change this but we won't worry about that now.

More reasonable example:

```csharp
Box[] arr = { new Box(0, 0), new Box(1, 1), new Box(2, 2) };
foreach (Box test in arr)
    WriteLine($"testing new Box(1,1) == ({test}) returned {new Box(1, 1) == test}");
```
The c# collection and extension methods leverage the use of `Equals()` to allow custom overrides to alter the default operations.

```csharp
List<Box> list = new List<Box>();
list.Add(new Box(1, 1));
list.Add(new Box(2, 2));
int iFound = list.IndexOf(new Box(2, 2)); // Uses Box Equals()
WriteLine($"Found via IndexOf at {iFound}"); // will be -1 if Equals() not overridden
```

You will get a compiler warning about overriding `Equals()` without overriding `GetHashCode()`. Basically, overriding `Equals()` is changing the way you determine equality, then some algorithms/containers (like Hashtable) which utilize both to determine object existence will potentially get confused. The default hash code is the address in memory of the object (thus guaranteed to be unique), generally it is recommended that you generate a value that would also be unique for the object. 

Any operations that use `GetHashCode()` and find a collision ( a duplicate hashcode ) will then resort to using `Equals()` to determine equality.

For our purposes, we will return a constant - 1, for the hashcode. For operations that care, this will mean they will use `Equals()` instead.

How would you write an appropriate `GetHashCode()` for the `Box` class above, given the member data and what `Equals()` considers equality ?

## Overloading operator==, operator!=

Support exists for you to override the actual ==/!= operators. This is generally NOT recommended, but is shown as an operator override example. Some libraries do use this functionality.

For reference types, operator `==` and `!=` by default do NOT invoke `Equals()`, rather they will always perform identity equality (`ReferenceEquals(objA, objB)` - which is not overridable). The .NET library only overloads the operator `==/!=` to provide value semantics on classes that, after construction, can NOT change. These classes are known as immutable and the string class is an example. It is desirable to provide value semantics on immutable classes since by definition they will never be equal.

While supported by the framework, overloading operators is frowned upon, and should only be done as a well-documented and rationalized design decision.

You can overload operator `==/!=` for your classes to add value semantics as well – but just because you can doesn't mean you should. Users understand that `==/!=` use identity semantics and expect it. If you want operator `==` and `!=` to perform value semantics, then you must overload the operators manually. Unless you are overriding `Equals()`, do not even consider overloading operator `==/!=`. Follow these basic rules for operator `==`:
- Return true if object references are the same.
- Return false if either (but not both) object references are null.
- Provide optional value semantics, via leveraging `Equals()` - do NOT rewrite the equivalence check!
- Then overload operator `!=` to leverage the operator `==` you just added (again, more leveraging).

The syntax is simple, but important to note that the method is static (no `this`).

```csharp
// Assuming this operation:
Box A = new Box(1, 2);
Box B = new Box(1, 2);
if (A == B)  // this translates to: if (operator==(A, B)), 
// where A is the left-hand argument, B is right-hand argument
    Trace.WriteLine("Same");

class Box // overload ==, != operators
{
  ...
  // Overload == and !=, public, static, compare both objects and determine equality
  public static bool operator ==(Box argLeft, Box argRight)
  {
    // same object or both null, that's equal
    if (ReferenceEquals(argLeft, argRight))
        return true; 

    // either but not both =/is null, that's not equal
    if (argLeft == null ^ argRight == null)
        return false; 

    // leverage Equals() here instead of rewriting
    return argLeft.Equals(argRight);
  }

  public static bool operator !=(Box argLeft, Box argRight)
  {
    // leverage operator==() here instead of rewriting
    return !(argLeft == argRight); 
  }
}
```

Operator ordering is the same for all overloadable binary operators: `+`, `-`, `*`, `/`, `%`, `&`, `|`, `^`, `<<`, `>>`, `<`, `<=`, `>`, `>=`.
```