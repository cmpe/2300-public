- [Classes](#classes)
  - [Object Oriented Programming](#object-oriented-programming)
  - [Classes vs. Structs](#classes-vs-structs)
  - [Attributes \[optional\]](#attributes-optional)
  - [class-modifier \[optional\]](#class-modifier-optional)
  - [access-modifier](#access-modifier)
  - [Declaration of fields/data](#declaration-of-fieldsdata)
    - [other field modifiers include :](#other-field-modifiers-include-)
  - [Methods](#methods)
    - [instance methods ( non-static )](#instance-methods--non-static-)
    - [static methods - future topic](#static-methods---future-topic)
    - [Instance Constructors ( CTOR )](#instance-constructors--ctor-)
# Classes
## Object Oriented Programming

First block of foundation is encapsulation
       
Encapsulation : the act of combining object state and functionality together, in the case of programming classes this involves the inclusion of fields and their related methods into a user define object. Also included within encapsulation is  Information or Data Hiding which is the practice of restricting access of internal data and methods except through an intentional public interface. This leads to the real world analogy of objects which hold both data and functionality. This is similar to real world objects providing a natural conversion into a programming model for simulation.

## Classes vs. Structs
Both are aggregate user-defined datatypes able to provide encapsulation, but structs are stack based value types whereas classes are heap based reference types.
    
Non-static classes are reference types and must be allocated via new() operator, structs may optionally use new, but is isn't necessary.

Non-static instances reside on the GC Heap and their lifespan is managed by the system and garbage collector. A reference count is maintained of all variables referencing the object, once the count decrements to 0, the object is a candidate for garbage collection  this happens whenever, you can not destroy a reference object explicitly ( no deterministic destruction )
``` c#
[Attribute]
[class-modifier] class ClassName [ : BaseClass |+ Implemented-Interfaces ]
{
	<access-modifier> data-type member-name; // members
	<access-modifier> return-type MethodName ( [parameter-type(s)] ) // methods
}
```
## Attributes [optional]
- you have seen these before [Serializable]
- flags to the compiler to ensure necessary class conditions are in place to implement the desired attribute ( ie. if some members are NOT serializable ( handles, pointers, etc ), they must be exclude with additional attributes. )
- others, but we will not consider any user defined or custom attributes

## class-modifier [optional]
applies to accessibility of class with respect to the rest of the application, these are considered top-level ( non-nested ) types and can only have accessibility levels of :
- `internal` [ default if not specified] class is accessible within the current assembly, either the .exe or .lib file.
- `public` class is accessible from any assembly ( like CDrawer in GDIDrawer assembly )
- AND/OR one of these additional modifiers : `abstract`, `sealed` a later topic
               
## access-modifier 
apply to both data/fields and methods
- options include : 
  - `private` [default if not specified] accessibility limited to class methods only
  - `private protected` compound modifier assessibility only to derived classes in same assembly
  - `protected` accessibility only to derived classes ( later topic )
  - `public` accessibility to all

- all members/methods should have an access modifier, the default being private if omitted ( implicitly private )
  
## Declaration of fields/data
- fields when declared, may be initialized in a number of ways :
  - using initializers provide an initialization value following a = at declaration time ( ex.  `int _iX = 5;` ) - it may seem unusual since this was not allowed in structures.
  - using constructors provide an initialization value in the constructor body for the field
  - using manual method initialization ( ie. Make user call Init() method )
  - not all methods apply to all types of fields, depends on field type.. value or reference type, etc.

### other field modifiers include :
- `const`
  - non-static fields only
  - must be initialized when declared
  - literals or other const value only
  - must be known at compile time
  - can not be initialized in constructor
  - actually const fields are implicitly static/shared
    - ie. `private const int iWidth = 800;`
- `static` class, member, and method modifier, future topic
- `readonly`
  - fields only
  - may be initialized when declared or in constructor ( static / non-static )
  - once initialized, can not be altered, not even by members
  - alternative to const when value can not be determined at compile time ( ie. Random() )
- `required`
  - fields or properties
  - forced to use an object initializer* the field marked `required` must be included/initialized in the parameters.
  - this has limited use as the initializer parameters are set AFTER a default CTOR has initialized the object.
  - use of a custom CTOR as well complicates the use, and will be deferred as a proper CTOR will satisfy the equivalent of a required field/property.
- `volatile`
  - used when read/write ordering important
  - ensures reads are not cached, writes guaranteed to execute in relative order ( ** for threads )
- `new` method modifier, future topic
- `sealed` class modifier, future topic
- `abstract` class, method modifier, future topic
               
``` c#
internal class Sample
{
	private const int _iWidth = 800;	// can never change, must be initialized now
	private readonly int _iHeight = 600;	// may be modified in constructors, but this ok too
	private readonly int _iDepth;		// usually initialization is done in constructors
	private int _iSpeed = 0;		// regular instance member is most common
}
```
           
## Methods
### instance methods ( non-static )
- accessibility defined by the access modifier, same rules apply
- methods may access any static/non-static member field defined within the class, regardless of access modifier
- methods may invoke any  static/non-static method, regardless of access modifier
- invoked using the dot operator with a class instance
- must define a return type and may accept optional parameter arguments
``` c#
// Simple instance method
class Test
{
  private int _multiplier = 2;
  private int foo( int x )
  {
    return x * _multiplier; // or x * this._multipler;
  }
}
```

### static methods - future topic
-  accessibility defined by the access modifier, same rules apply
-  static methods may only access static fields ( no non-static fields )
-  static methods may invoke any static method within the class, regardless of access modifier ( no non-static )


### Instance Constructors ( CTOR )
- instance ctor is invoked when an instance of the class is created via new()
- instance ctors may be overloaded ( multiple versions differing only by their argument lists )
- a ctor that accepts no arguments is known as a default or parameterless ctor
  - if you fail to provide a default ctor, you get one free, but does nothing except initialize all members to their default values
  - if you provide a default ctor, you can initialize all members within the ctor body, even though the system ensures that all members have been initialized to their default values prior to the ctor body being executed.
- if you add ANY ctor, the compiler does not supply the default any more, and if you want your custom ctor and a default you must supply both explicitly
  
- you may invoke another overloaded ctor from within a ctor using `this` ( : `this( reqd_params )` ), this may chain yet another ctor this is handy when you want numerous ctors ranging from minimum arguments to all arguments
   -by specifying a full ( all params ctor ), then overloading addition versions which accept less arguments to which you specify a default value and invoke the master/full ctor, you can build a ctor set all the way up to a default while ensuring no redundant code is implemented
  - an alternative to this could use a full ctor, but supplementing with optional arguments allowing the same functionality, this however, will not work for all cases where special initialization is required in certain constructors.
- syntax for constructors ( ctors )
  - public ( usually, but may use any modifier, sometimes a private ctor is preferred for factories )
  - no return type - ever
  - name identical to classname  always, exactly
  - optional arguments
    
- use of `this`
  - `this` provides access to the current invoking class instance and can be used in any method, including ctors
  - handy to resolve name ambiguity : `this.X = X;` // in a ctor for example
``` c#
public class Birdie
{
  private const int _iTweets = 100; // const must use initializer
  private readonly int _iBeaks;     // only modifiable in constructor
  private int _iSeeds;
  public Birdie(int iSeeds) // custom/user defined constructor
  {
    _iBeaks = 12;     // readonly can only be changed/modified within a constructor
    _iSeeds = iSeeds;
    // or this._iSeeds = iSeeds; // not preferred unless removing ambiguity
  }
  // default constructor, leveraging previous constructor, supplying arg iSeeds = 8
  public Birdie() : this(8)
  {//this code executed after leveraged (int) constructor call, no body required, work done 
  }
  public void Chirp( ) // user defined method
  {
    for( int i = 0; i < _iSeeds; ++i )
      Console.Write("*");               // output a * for each seed
  }
}
// Now for users, accessing and using the new Birdie class
private void foo()
{
  Birdie? birde; // valid but currently a reference without an object, defaults to null
  // Since a class is a reference type, must use new to actually allocate one

  Birdie bird = new Birdie(8);  // invokes user-defined ctor to make Birdie and
                                // our local reference bird now references the new object
  Birdie birdz = new Birdie( iSeeds : 8); // this time with parameter names
                                         // our local reference bird now references the new object

  Birdie birdy = new Birdie(); // invokes default ctor, which leverages other ctor to make Birdie

  // OR using Target-typed expressions when the target type is explicitly known
  Birdie birdm = new(8); // same as Birdie(8); since bird type is explicitly known
  Birdie birdn = new( iSeeds : 8 );
  Birdie birdp = new(); // same as Birdie();


  Birdie[] ArrBirds = new Birdie[8]; // make array of Birdie references, NO actual Birdies are made,
                                  // just an array of references to them, we must populate manually
  for (int i = 0; i < ArrBirds.Length; ++i)
    ArrBirds[i] = new Birdie(i); // Allocate a new Birdie for this reference

  // Or we can use other collections
  List<Birdie> BirdList = new List<Birdie>();//List of Birdie references, again, NO Birdies are made

  Birdie tmp = new Birdie(1); // Birdie's gotta come from somewhere
  BirdList.Add(tmp); // Add dat bird

  BirdList.Add(new Birdie(9)); // Or do it all in one step, no temporary reference is required

  while (BirdList.Count < 10)
    BirdList.Add(new Birdie(BirdList.Count)); // Fillup til count=10, using a non-literal ctor value

  bird.Chirp(); // Invoke Chirp() on the original bird, in this case bird is our invoking instance

  foreach (Birdie b in ArrBirds) // References b is populated with each Birdie in array
    b.Chirp(); // Chirp that bird

  foreach (Birdie b in BirdList) // Or from List, note this is allowed b is foreach scoped reference
    b.Chirp(); // that will refer to each of our BirdList Birdie objects in turn
}
```
c#12 - `Primary Constructors`
Primary Constructor Declaration: The parameters of the primary constructor are defined directly after the class or struct name. These parameters are in scope throughout the entire body of the class or struct, including field initializers and other members.
``` c#
    public class MyClass(int id, string name)
    {
        // 'id' and 'name' are accessible here
        private readonly int _id = id; 
        public string Name { get; } = name;
    }
```
Primary Constructors and additional constructors

If you define additional constructors (overloads) for the same class or struct, they must call the primary constructor using the : `this(...)` syntax, passing the necessary arguments to satisfy the primary constructor's parameters.
``` c#
    public class MyClass(int id, string name)
    {
        private readonly int _id = id; // id and name are valid with the declaration context, and used for initialization !
        public string Name { get; } = name;

        // Another constructor chaining to the primary constructor
        public MyClass(string name) : this(0, name) // Calls the primary constructor with a default 'id' - so is it really an id anymore ?
        {
            // Additional logic specific to this constructor, if any
        }
    }
```



1. Building a class general conventions
- members  whether static or non-static, all fields should be private, this implements the basic tenet of Object Oriented Programming, encapsulation
  - Encapsulation : the discipline of restricting access to internal data and methods except through an intentional public interface
  - See how allowing access to internal data members may corrupt the class state
- methods  class design should be considered when specifying access modifiers.
  - methods that allow the user to interact with the class should be public
  - methods used internally by the class that may be dangerous for the user to invoke should be private/protected
- Given that encapsulation is encouraged(enforced), it is a hassle to force the user to only access class state information strictly through method calls  ie. GetReal(), C# helps here with the inclusion of properties.
