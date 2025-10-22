# Interfaces
At this point you have some familiarity with interfaces, specifically `IComparable`. When your class chose to implement the `IComparable` interface, you were required to complete the necessary `CompareTo()` method. While inherently (pun intended ) different than inheritance, there are many similarities.

An interface, like classes may be either public or internal, Like classes, interfaces default to internal. Methods in an interface definition are implicitly public and abstract. Being abstract, methods in the interface may not contain definitions. By convention interfaces are prefixed with ‘I’.

Ultimately an interface is just a named collection of abstract methods that outline a specific behavior. Classes may elect to model this behavior by implementing that interface. Consider the following interface definition:

```csharp
// make an interface for things that "have volume", internal by default
interface IVolume
{
  double GetVolume(); // implicitly public, abstract
}
// Class to illustrate use :
internal class CCube : IVolume
{
    public double GetVolume() // sample of method IVolume support
    {
        return ( volume_calculation_from_members );
    }
}
```

Interfaces don’t allow you to define field data, but they do allow property definitions, since properties are merely automated get/set methods

Interface properties, like methods, are fully and implicitly public, therefore, it is illegal to apply any accessibility modifiers on the get/set – if it is supported – it is public.

```csharp
interface IVolume
{
  double Volume { get; } // property, it is really just a method
  // this “looks” like an automatic property, but just specifies that
  // the implementing class will supply a public manual Volume with a get{}
}

internal class CCube : IVolume
{
  // sample of property interface support, here you actually complete the property
  public double Volume { get { return (volume_calc); } }
}
```

This interface describes a behavior of “having volume”, and any class that elects to implement the interface must provide an identically signatured method and calculate its volume, returning it as a double in its own implementation.

You might wonder what benefit this provides, since any class could just make a method called “GetVolume” and do the same thing. The interface provides a common template for any class that models this behavior, and forces any participating class to use the same form. This does not buy you much for interfaces you are building, but for well known interfaces the ability to provide consistent implementations for behaviors is critical. This is easily illustrated by our implementation of IComparable and its use in explicit sorting and inclusion into any sorted collections.

The syntax for using an interface in a class definition is the same as it is for inheritance:

```csharp
// make an abstract base class for abstract shapes that have volume
internal abstract class C3DShape : IVolume
{
    public abstract double GetVolume();

    // other common 3DShape members here...
}
```

This class will serve as an abstract placeholder for 3D shapes that have volume. An implementation of a derived, non-abstract class would look like this:

```csharp
// make a concrete class that has shape and volume
internal class CCube : C3DShape
{
    // the member that describes the type-specific dimensions
    private double _dLength = 0;

    // ctor for inits
    public CCube(double dInitLength)
    {
        _dLength = dInitLength;
    }

    // class-specific implementation of interface
    public override double GetVolume()
    {
        return Math.Pow(_dLength, 3);
    }
}
```

Note: the class hierarchy above could be written to include the interface inclusion at the derived shape level (it’s just arguably not good practice), so for completeness:

```csharp
// make an interface for things that "have volume"
internal interface IVolume
{
    double GetVolume();
}
    
// make an abstract base class for abstract shapes that have volume
internal abstract class C3DShape
{
    // other common 3DShape members here...
}

// make a concrete class that has shape and volume
internal class CCube : C3DShape, IVolume
{
    // the member that describes the type-specific dimensions
    private double _dLength = 0;

    // ctor for inits
    public CCube(double dInitLength)
    {
        _dLength = dInitLength;
    }

    // class-specific implementation of interface
    public double GetVolume()
    {
        return Math.Pow(_dLength, 3);
    }
}
```

Note that providing an override for an interface method does not use the override keyword!

C# only allows single inheritance with classes, but does allow multiple inheritance with interfaces. The interesting tricks interfaces allow with polymorphism could be a course in itself.

```csharp
public interface IRenderable
{
    void Render(CDrawer3D target);

    int pAuto { get; set; }
}
```

Like regular methods, properties that exist in interfaces must not include access modifiers. This translates to being a public member, so it will not be of much use to us.

Creating and using interfaces in C# has some practical advantages. A class that implements an interface can be represented by that interface. Consider, for example, the following code:

```csharp
public class RedSquare : IRenderable
{
  private int _side = 0;
  public int pAuto // Must complete pAuto to complete IRenderable
  {
    get { return _side; }
    set { _side = Math.Abs(value); }
  }

  public RedSquare( int side )
  {
    _side = side;
  }

  public void Render(CDrawer target) // Must complete Render
  {
    target.AddRectangle( 0, 0, _side, _side, Color.Red);
  }
}
```

The RedSquare type or any other type that implements `IRenderable` could be referenced through that interface. This means that collections of `IRenderable` are possible:

```csharp
// container of generic renderable items allowing any object that supports the IRenderable
//  interface to be added to the collection
LinkedList<IRenderable> stufftodraw = new LinkedList<IRenderable>();

// create some items, add to linked list
RedSquare square = new RedSquare(10);
stufftodraw.AddLast(square);
BlueCircle circle = new BlueCircle(8);
stufftodraw.AddLast( circle );
while (true)
{
    dr.Clear();

    // update animation, no interface, must explicitly invoke
    square.Tick();
    circle.Tick();

    // render the shapes, what is interesting to note here is that when rend is accessed
    // in the body of the foreach(), only common elements are available – like the
    // Render() method and core( object base ) methods.
    foreach (IRenderable rend in stufftodraw)
        rend.Render(dr);   // WARNING : all stufftoddraw must be IRenderable or Exception thrown
    
    // render the image
    dr.Render();
    Thread.Sleep(20);                
}
```

To carry this example to its obvious end, another interface `ITick` would have been defined and supported by our types. This would allow the use of another `foreach( ITick`.. ) loop to move our objects rather than resorting to the individual `square.Tick()` and `circle.Tick()` calls.

# Interfaces and RTTI ( Run Time Type Information )

The syntax for interface inclusion in a class definition would lead a person to believe that the class is a derivation of the interface type. That interface contract allows the system to treat it as a type with a concrete set of behaviors that are supported. This can be proven with the use of the `is` keyword:

```csharp
object thing = new RedSquare(5);
if (thing is IRenderable rendMe) // Conditional Access : remember “is” has a “free” null check
    rendMe.Render(dr);

Or if you prefer, using old school “is” and “as”

if (thing is IRenderable)
    (thing as IRenderable).Render(dr);

Or perhaps the best version of all – Elvis ?.

(thing as IRenderable)?.Render(dr);//if ‘as’ resolves to null, Elvis skips invoking Render() !
```

Use Conditional Access with a local variable, or an object can be tested for an interface with the `is` keyword, the same way derived types can be tested through base class references. Then either a cast or `as` is applied.
This mechanism is particularly handy when a base class reference does not contain interface definitions that a derived class does:

```csharp
public interface IRenderable
{
    void Render(CDrawer target);    
}

public class CBase
{
}

public class CDerived : CBase, IRenderable
{
    public void Render(CDrawer target)
    {
    }
}
```

Test Code…
```csharp
CBase test = new CDerived();
// test will not show the render method - not part of base class
if (test is IRenderable rendMe)
    rendMe.Render(dr); // OR 

foreach (IRenderable test in ListOfThings ) // OK if all objects support IRenderable contract
    test.Render(dr);                        //  runtime error if collection element does not
                                            //  support the IRenderable interface
```
In this situation the ability to cast a type to one of its interfaces is handy, as it is only the interface, not the true derived type that is of interest here.
Be careful with foreach() iteration, unless all collection objects support the interface contract

If you are not sure, you can fallback to the failsafe :
```csharp
foreach (object test in ListOfThings ) // Not sure what is supported…
{
  if (test is IRenderable rendMe )         // Check it first, but remember rendMe persists !
    rendMe.Render(dr);  // Good to go !

  if (test is IDanceAble danceMe )         // use of rendMe identifier name is a conflict, 
    danceAble.Render(dr);

  (test as IRenderable)?.Render(); // Or the best of all - the King - Elvis ! ?.

  // continue with other interfaces
}
```