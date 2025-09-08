1.  # Inheritance {#inheritance-1}

After Encapsulation, Inheritance is the 2^nd^ "Pillar" of OOPs.
Inheritance allows construction of a "family" of classes where the
functionality of the parent (original) class can be leveraged and
modified by the child (derived) classes.

You have already ( inadvertently ) derived classes -- in any windows
forms application, the form class of the application is NOT an actual
form class, but a new class derived from a base/parent windows form
class, upon which you have added additional functionality :
```c#
namespace ice10_sep09
{
  public partial class Form1 : Form
  {
    private CDrawer canvas = new CDrawer();
    public Form1()
    {
      InitializeComponent();
      Width = 400; // Where do InitializeComponent and Width come from ? Your base class !!
    }
    private void Form1_Click(object sender, EventArgs e)
    {
      canvas.AddEllipse(...);
    }
  }
}
```
Inheritance creates an "**is-a**" relationship rather than a "**has-a**"
relationship. ie. Your new application form "is-a" child form, but the
new fields you add to the form create a "has-a" relationship with the
form. The derived class is sometimes called a subclass and the base
class referred to as the parent or superclass.

# Basic Derivation

In basic derivation the colon operator is used to define an inheritance
condition

`class myDerived : myBase {}`

Upon deriving a new class, that class "inherits" **all** fields/methods
of the base class. Careful reading of the Microsoft documentation says
the derived class inherits all non-private fields/methods. While in one
sense this is true, some clarification is required. Recall the
protection specification definitions, whereby private data was only
accessible within the class itself, while protected was like private
except for that derived classes could access the field/method. So to
clarify, **private** data can **not** be accessed from the derived
class, but it **does** still exist as a non-accessible member
field/method of your derived class. It **is** there, you just can not
directly access it. ( Perhaps other protected/public fields or methods
of the base class utilize them : like an Area property )

Protection modifiers are carried into the new derived class whereby :

- `public` -- type or member - access is not restricted
- `protected` -- type or member -access is limited to same class or a
  class derived from that class
- `private` -- type or member - exist, but are NOT (!) accessible to the
  derived class ( why, how can it work ? )
- `internal` -- type or member - Access is limited to current
  assembly
- `protected internal` -- type or member -- Access is limited to current
  assembly or types derived from that class in any assembly
- `private protected` - a new addition -- type or member - access is
  limited to the same class or class derived from that class only within
  the current assembly

Not all modifiers are valid on both types ( classes ) and members of
classes in all contexts. While members typically use :
public/protected/private. But classes often use :
public/internal/protected internal/private protected to restrict type
accessibility rather than member accessibility.

Simplest Example, extend the functionality of a previously defined class
```c#
[internal/public] class MyRandom : Random // default is internal
{
  // Along with all fields and methods, we can now add :
  public int NextOneToTen()
  {
    return Next(1, 11);
  }
}

{
  MyRandom myR = new MyRandom();
  int i = myR.Next(2); // Still supports base class public interface
  int j = myR.NextOneToTen(); // Now invoke new functionality
}
```
The example illustrates the derivation of a new class with the sole
purpose of adding some desired functionality that makes the derived
class more desirable to use than the base class. A library of such
derived classes could be useful for many repetitive tasks.

# Inheritance with default constructors

When a new class is derived, you still get the default free default
constructor for your derived class. C# guarantees that upon creation of
a derived class, the base class constructor is invoked automagically
prior to the execution of the derived class default constructor ( even
if it does nothing ). This is important, but hard to see why when the
examination only spans a parent/child class hierarchy.

This ensures that if you choose to create a constructor for the derived
class, the body of the CTOR will not be executed until the base class
has been initialized.

The previous example of MyRandom, demonstrated this since the MyRandom
class did **not** define any constructors. This meant that the system
supplied a default constructor. Consider that the Random() base class
actually did ( invisibly to us ) have its default constructor invoked
since it was properly initialized with the current time.

In a less contrived analogy, if you had a "logging" base class, where a
derived class would have a default CTOR which expects the base class
functionality of opening and preparing the log file to be complete. It
is reasonable for the derived class to assume that the base class is
"ready-to-go" once the derived constructor body is executed.

If the base class is itself a derived class, then initialization via the
constructors occurs from most base to most derived ( turtles all the way
up )

Following our basic tenet, data is private, and the class constructor is
responsible for its own data initialization. Then it follows that the
Derived class is responsible for its own data initialization and not for
the individual initialization of it's parent class ( base class ).
Likely, the base class data is private and not accessible anyway.

Default constructor example :
```c#
class BaseDCTOR
{
  private int _iNum = 1;
  public int Num { get { return _iNum; } }
  public BaseDCTOR()
  {
    Console.WriteLine("Base Default CTOR Invoked");
  }
}

class DerivedDCTOR : BaseDCTOR
{
  public DerivedDCTOR()
  {
    Console.WriteLine("Derived Default CTOR Invoked");
  }
}

// Then :
{
  DerivedDCTOR derived = new DerivedDCTOR();
}
```
`Base Default CTOR Invoked`

`Derived Default CTOR Invoked`

In this example, both base and derived classes have explicitly defined
default constructors, and as the final output lines indicate, the Base
CTOR was invoked **prior** to the Derived CTOR.

To see how the system accomplishes this, we need the hidden
initialization that is being done. Recall that to leverage another
overloaded CTOR within our class, we use the **: this()** syntax. For
base class initialization, the keyword **base** is used in the same form
to provide explicit base class constructor information.

```c#
class DerivedDCTOR : BaseDCTOR
{
  public DerivedDCTOR() : base() // the system puts this in for us
  {
    Console.WriteLine("Derived Default CTOR Invoked");
  }
}
```
As shown above, the system silently embeds the **:base()**
initialization into our constructor. In this case, it ensures that the
base class constructor is invoked prior to the derived.

You should adhere to the policy that each class ( base or derived )
should initialize all its "new" members, either through the CTOR or an
initializer. It is not the job of the derived class to initialize its
base class members ! It might not even be able to if the base members
are private !

It is important to realize that the system will not allow a derived
class to be constructed without base class initialization ( invoke a
base CTOR ). If your base class has supplied a custom CTOR, whereby the
default CTOR is now suppressed, your derived class **must** explicitly
initialize the base class utilizing the base keyword.

# Inheritance with custom constructors

For many reasons the base class might not support a default CTOR. A
custom CTOR would suppress the default CTOR, or perhaps the class isn't
able to initialize without supplied values. In this case the base
keyword must be used to explicitly inform the compiler which base class
CTOR will be used to instantiate the derived class instance.

Explicitly invoke the appropriate base class CTOR using the : ( colon )
operator (again, but different )
```c#
class Base
{
  private int _iNum = 1;
  public int Num { get { return _iNum; } }
  public Base()
  {
    Console.WriteLine("Base Default CTOR Invoked");
  }
  public Base( int iVal )
  {
    _iNum = iVal;
    Console.WriteLine("Base (int) CTOR Invoked");
  }
}

class Derived : Base
{
  public Derived() : base() // Utilize Base() default constructor
  {
    Console.WriteLine("Derived Default CTOR Invoked");
  }
  public Derived( int iD ) : base( iD ) // Utilize Base(int) constructor here
  {
    Console.WriteLine("Derived (int) CTOR Invoked");
  }
}

// Then :
{
  Derived da = new Derived();
  Derived db = new Derived(8);
}
```
```c#
Base Default CTOR Invoked
Derived Default CTOR Invoked
Base (int) CTOR Invoked
Derived (int) CTOR Invoked
```
It is possible to utilize whichever base CTOR is appropriate by
supplying the necessary syntax and arguments. Here we utilize the base
class custom CTOR from the derived class default CTOR.
```c#
public Derived() : base( 9 ) // Utilize Base(int) for our default constructor
{
  Console.WriteLine("Derived Default CTOR Invoked");
}
```
If you fail to provide initialization for your base class the system
**will** attempt to invoke the base class default CTOR. Be careful, as
this may not be the initialization you want. If the base class does not
support a default CTOR, the creation of a derived class instance is not
possible and you will receive the error : ClassXXX does not contain a
constructor that takes `0` arguments.

There is an additional keyword "sealed" that can be used as a class
modifier, which when applied **prevents** the class from being used as a
base class ( can't derive from this class ) - this is a design
consideration when the developer of the class does not want the class to
act as a base class -- we'll see other considerations that also need to
be examined when a class becomes polymorphic. There are many system
classes that are **sealed.** As it turns out, structures in c# are
explicitly sealed and can not be derived from.

# Method/Function Hiding

In a complex hierarchy, your derived class may attempt to use a method
name that is currently defined by your base class. The compiler will
alert you to this condition. If it is accidental, choose another name
for your derived method. If however you wish to "replace" the base
version with your "new and improved" derived version the compiler will
insist on the use of the **new** keyword preceding your derived method
signature :
```c#
class Derived : Base
{
  // Other details omitted
  new public void Show( )
  {
    Console.WriteLine("Derived Show() better than ever");
    base.Show(); // Invoke base class Show() using base keyword
  }
}
```
In this case, Show() is meant to replace the Show available in the Base
class. Now when Derived class instances invoke Show() it will be the
"new" Show(). Users can still invoke the base class Show() by performing
a cast ( an Upcast, see next ) :

```c#
{
  Derived da = new Derived();
  da.Show(); // Invoke Derived Show()
  ((Base)da).Show(); // Cast to invoke Base Show()
}
```
# Upcasting and What is it actually ?

Perhaps as a refresher, upcast refers to the binding of a derived object
to a more base reference in the same family. We saw this in override of
Equals() and our CompareTo() implementation, where the incoming argument
was of type object. The method used a cast or is/as to convert the
supposed "object" type to the class type. This was an example where any
reference type ( which is ultimately derived from "object", can bind
implicitly to an "object" argument type.

It turns out that **every** system type ( including your custom classes
) ultimately derive from "object". In the case of "naked" classes ( not
derived ), the derivation is hidden. You are reminded constantly when
your supposed empty classes magically have Equals() , ToString(),
GetHashCode() and GetType(). As such the most "upcast-ness" that can
happen is to the "object", but any valid parent class on the way up the
hierarchy is fair game.

This also occurs whenever the non-generic C# containers like ArrayList
were used. The underlying references of the ArrayList were always of
type object, thereby requiring that the element returned by the
ArrayList had to be explicitly down-cast to the real type for use.

Given :
```c#
object p = new Employee(); // p is object reference, and the new Employee() is upcast and binds to p then use of p required a cast :

Employee e = (Employee)p;
e.EmpMethod(); // cumbersome and laden with possible exceptions

//or using "as" ie.

Employee e = p as Employee;
if( e != null )
  e.EmpMethod(); // no exceptions but still cumbersome.

//Somewhat more interesting yet :

//The GetType() method returns a Type object, it has a staggering amount of info... take a peek

Text = p.GetType().Name;// Name is just one of many informational properties
```
Upcasting is core in almost all inheritance and interface based
frameworks. Recall that all the event handlers you have created
throughout windows programming utilized a parameter of type object
called sender. The control which caused the event is passed to your
handler as an object whether it be a button, timer or trackbar. Your
handler and you should know whether it can be cast and used.

# Virtual Methods and Polymorphism

Polymorphism is enabled by the inclusion of the keyword virtual on one
or more methods. The application of virtual to a method signals that the
method is a candidate for polymorphic behavior and *may* be overridden
in a derived class. An example of a virtual method is ToString() and
Equals().

Methods that are declared virtual in a base class may be overridden in a
derived class. If this occurs, an object which has been upcast to a base
type may invoke the method and rather than the base method being
invoked, the overridden derived replacement is invoked instead.

Virtual methods can not be declared private, as they cannot be invoked
-- protected or public only.

Methods may be marked virtual in a base class. Methods in derived
classes with the same signature may use the override keyword to invoke
polymorphic behavior:

```c#
class CBase
{
  virtual public void Who()
  {
    Trace.WriteLine("Who in base!");
  }
}

class CDerived : CBase
{
  public override void Who()
  {
    Trace.WriteLine("Who in derived!");
  }
}
```
If a method is marked virtual in a base class, and is overridden in a
derived class, the derived class version will be used if the method is
called through a base class object reference:
```c#
CBase baseref = new CDerived();
Trace.WriteLine(baseref.GetType().Name);

baseref.Who();
//CDerived
//Who in derived!
```
This is handy, since a generic base class object reference could point
at any class in the hierarchy, yet still invoke derived class behavior.

Consider the following class hierarchy:
```c#
class CShape
{
  public virtual void ShowArea()
  { } // empty
}

class CSquare : CShape
{
  protected int m_iLength;
  public CSquare(int iLength)
  {
    m_iLength = iLength;
  }
  public override void ShowArea()
  {
    Trace.WriteLine ("Area of this square is " +
        Math.Pow (m_iLength, 2).ToString("f2") +"units!");
  }
}

class CCircle : CShape
{
  protected double m_dRadius;
  public CCircle(double dRadius)
  {
    m_dRadius = Math.Abs(dRadius);
  }
  public override void ShowArea()
  {
    Trace.WriteLine("Area of this circle is " +
        (Math.PI * Math.Pow (m_dRadius, 2)).ToString("f2") + " units!");
  }
}
```
With calling code:

```c#
List<CShape> m_lShapes = new List<CShape>();
m_lShapes.Add(new CSquare(5));
m_lShapes.Add(new CCircle(45));

foreach (CShape shape in m_lShapes)
  shape.ShowArea();

//Area of this square is 25.00 units!
//Area of this circle is 6361.73 units!
```
The ability to maintain generic collections of base class objects will
allow you to avoid parallel arrays and the like. This is a good thing.

## Abstract Classes

Sometimes, if not often, you will find that a base class is just a
placeholder in the hierarchy. Consider the CShape base class, for
example. A shape without dimension can't actually produce an area. In
fact, the CShape class shouldn't be instantiated at all. If you run into
this situation, you may make a base class abstract. Abstract classes are
created with the sole intent of being derived from. You will never
instantiate an abstract base class; however you will use them as the
root of polymorphism.

Firstly, some background. A class is marked abstract with the abstract
keyword:
```c#
[internal/public] abstract class CShape
{

}
```
Once a class is marked abstract, it can't be instantiated. Attempting to
do so will generate an error.

Operable methods and fields may be added to an abstract base class.
Since the abstract class can't be instantiated, the implication is that
these members will be operated from derived classes:
```c#
// this class can't be instantiated directly
abstract class CBaseThing
{
  // only constructor in this can set the value
  public int piNum { get; private set; }
  public CBaseThing(int iInit)
  {
    piNum = iInit;
  }
  public virtual void ShowData()
  {
    Trace.Write (piNum.ToString());
  }
}

class CDevThingy : CBaseThing
{
  public CDevThingy(int iNum) : base (iNum)
  { }
  public override void ShowData()
  {
    base.ShowData();
    Trace.WriteLine (" is " + (piNum % 2 == 0 ? "even" : "odd"));
  }
}
```
Clearly, from looking at the sample code above, the abstract class can
be operated through the derived class; it just can't be directly
instantiated.

Methods may be marked abstract as well. An abstract method does not have
a definition and *must* be overridden in every derived class:
```c#
// this class can't be instantiated directly
abstract class CBaseThing
{
  // only constructor in this can set the value
  public int piNum { get; private set; }
  internal CBaseThing(int iInit)
  {
    piNum = iInit;
  }
  public abstract void ShowData(); // NO BODY, MUST be overridden
}

class CDevThingy : CBaseThing
{
  public CDevThingy(int iNum) : base (iNum)
  { }
  public override void ShowData()
  {
    Trace.WriteLine (piNum.ToString() + " is " + (piNum % 2 == 0 ?
"even" : "odd"));
  }
}
```
## What if the base class supports custom Equals() and the IComparable interface ?

If the base class you are deriving from supports a custom Equals(), the
default functionality is that your new derived class "inherits" the
Equals() method. When the derived class explicitly invokes Equals() or
during library algorithm methods, the base class Equals() will be used
when required.

In this case, the base class Equals() method is used and the derived
objects equality is based on their base field/method considerations. No
new derived fields/properties will be considered.

This is usually fine unless the derived class adds additional fields
that should participate in the custom Equals() check. If this is the
case you must determine what your derived class would consider equality.
Normally, this would the AND of both **base** and derived fields. Since
it is likely **base** fields are inaccessible, leverage the existing
Equals() method in combination with the new derived fields.

```c#
public override bool Equals(object obj)
{
  if (!(obj is DerivedEqCmp d))
    return false;
  // return base class equality AND _iDer the additional derived field
  return base.Equals(obj) && _iDer.Equals( d._iDer );
}
```
In this example, once the data verification is complete, the base class
Equals() is invoked utilizing the **base.** keyword pseudo-field. A
class can use the **base** keyword to explicitly invoke/access immediate
base class information. Unfortunately unlike normal fields, base may not
be chained to "drill up" the hierarchy ( ie. base.base.ToString(); //
would be invalid ). That does not mean it doesn't happen, it might, but
it would be done in the base class Equals() instead.

`IComparable`

When an interface is supported by a class, it must be implemented. Where
it is implemented is dependent on the hierarchy itself.

Consider the following class hierarchy, our first ( but not recommended
) example :
```c#
abstract class CShape : IComparable // supports IComparable, just not here...
{
  public abstract double GetArea();
  public abstract int CompareTo(object o); // a placeholder forcing implementation in derived class
}

class CSquare : CShape
{
  protected int m_iLength;
  public CSquare(int iLength)
  {
    m_iLength = iLength;
  }
  public override double GetArea()
  {
    return Math.Pow(m_iLength, 2);
  }
  public override int CompareTo(object o) // support IComparible
  {
    if (!(o is CShape))
      throw new ArgumentException("CSquare can only compare to shapes!");
    CShape arg = o as CShape; // must be cast to at least CShape, object doesn't have polymorphic GetArea()
    return this.GetArea().CompareTo(arg.GetArea());
  }
}

class CCircle : CShape
{
  protected double m_dRadius;
  public CCircle(double dRadius)
  {
    m_dRadius = Math.Abs(dRadius);
  }
  public override double GetArea()
  {
    return (Math.PI * Math.Pow(m_dRadius, 2));
  }
  public override int CompareTo(object o) // and support IComparible again here...
  {
    if (!(o is CShape))
      throw new ArgumentException("CCircle can only compare to shapes!");
    return this.GetArea().CompareTo(((CShape)o).GetArea());// alternate version using cast
  }
}
```
Once IComparable is included in the base class, the derived classes
inherit it. By making the CompareTo method abstract, the derived classes
must provide an override. The implication of making a base class include
IComparable is that all derived classes will compare on a behavior
common to all classes in the hierarchy. In the sample code above, this
includes the area calculation. This makes sense, as squares can't be
"compared" to circles, but area can be compared to area. This
implementation is a little strange, as all CompareTo methods in the
derived classes will be functionally the **same**. **Common behavior
belongs in the base class**, so CompareTo need not be polymorphic. This
could be simplified to some extent, by putting a concrete implementation
of CompareTo in the base class that uses a polymorphic GetArea

This version is more stream lined and easier to follow:
```c#
abstract class CShape : IComparable
{
  public abstract double GetArea();
  public int CompareTo(object o) // supports IComparible, utilizing polymorphic GetArea()
  {
    if (!(o is CShape))
      throw new ArgumentException("Shapes can only compare to shapes!");
    CShape arg = o as CShape; // must be cast to at least CShape, object doesn't have polymorphic GetArea()
    
    // Both GetArea() calls invoke the appropriate Derived class GetArea() implementation
    return GetArea().CompareTo(arg.GetArea());
  }
}

class CSquare : CShape
{
  protected int m_iLength;
  public CSquare(int iLength)
  {
    m_iLength = iLength;
  }
  public override double GetArea()
  {
    return Math.Pow(m_iLength, 2);
  }
}

class CCircle : CShape
{
  protected double m_dRadius;
  public CCircle(double dRadius)
  {
    m_dRadius = Math.Abs(dRadius);
  }
  public override double GetArea()
  {
    return (Math.PI * Math.Pow(m_dRadius, 2));
  }
}
```
This is a much more elegant way of implementing the concept "all shapes
can be compared by area".
```c#
List<CShape> m_lShapes = new List<CShape>();
m_lShapes.Add(new CSquare(5));
m_lShapes.Add(new CCircle(45));
m_lShapes.Add(new CCircle(1));
m_lShapes.Add(new CSquare(1));
m_lShapes.Sort();

foreach (CShape shape in m_lShapes)
  Trace.WriteLine(string.Format("{0} : {1:f2}", shape.GetType().Name, shape.GetArea());
```

## NVI - Non-virtual Interface Pattern

The NVI pattern is closely related to the Template pattern. Resulting in
a separation of interfaces into a user/client interface and a derivation
interface.

The pattern assigns all user/client interaction through a non-virtual
public interface ( properties and methods ).

The pattern assigns all polymorphic operations to function in a
hidden/protected set of properties and methods. Not all methods need be
virtual, and may support polymorphic operations.

The pattern almost universally results in the exposed user/client
interface being a set of base class non-virtual properties/methods,
which in turn invoke protected virtual methods. The derived classes
override the virtual methods specific to their intended behavior.

The downside is an extra non-virtual method is required for user/client
interface, removing the public virtual implementation.

The benefit is that if pre/post processing that is required to be
modified in the hierarchy can be managed in the non-virtual public
interface.
```c#
public partial class Form1 : Form
{
  public Form1()
  {
    InitializeComponent();
    DerLogger derLogger = new DerLogger();
    // All derived Loggers will have to modify their Log override to
    derLogger.Log("Hello"); // Will only invoke the overriden Log()
    // Output :
    // 12:48 PMDerLogger : Hello-Done
  }
}

abstract class Logger // If you have an abstract method, your class must be marked abstract
{
// Non-virtual public Log(), will invoke virtual override in derived appropriately
  public void Log(string s)
  {
    Write(DateTime.Now.ToShortTimeString());
    VLog(s); // Invoke the abstract method, but derived will be forced to   complete
    WriteLine("-Done");
  }
  protected abstract void VLog(string s); // Derived must complete log   functionality
}

class DerLogger : Logger
{
  protected override void VLog(string s)
  {
    Write($"DerLogger : {s}");
  }
}
```
Without NVI, all derived Loggers would be required to modify their
override to provide the pre/post processing of adding the TimeStamp and
Done strings.
