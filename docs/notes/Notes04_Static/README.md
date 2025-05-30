# Static – Fields, Methods, and Properties

## Static – When Applied, Implicitly Implies “Instance Independent”

In this scenario, all instances of a class (say a `Ball` class) could utilize a single field representing the ball size – in other words – the same size for all. Where else might this be handy? (… perhaps a file stream object for logging, a communication connection, … many others). It is said the static members are *instance independent*.

## Static Fields

If a field is static, then ALL class instances “share” a single instance of that static field. This means the shared static field does not get allocated with instances of the class; rather, the static field is created separately and each class is “linked” to the static data.

Static fields are NOT serialized – they do NOT participate in object state or memory allocation (this is actually good, see sample). It turns out you can't have a `const static`, because if a field is declared `const`, it is static by default (shared + constant). But if the static is `readonly`, the same rules apply, except it must be initialized in a *static* constructor or initializer.

Since static fields are instance-independent, they are accessed by class users by the class name rather than an instance, provided they are publicly accessible – as per standing orders, we normally will provide either a method or property to access all our class fields.

Static field access within a class is permitted, regardless of access specifiers, but since they are instance-independent, they can't be prefixed with `this`.

Static fields (if publicly accessible) may be accessed by users of the class by using the `class_name` as the prefix, rather than the `class_instance` (used for non-static fields).

ie. `Ball.my_static_x = 9;`

The shared aspect of the static field changes the way static fields are constructed. Static fields are not guaranteed to be initialized at any specific time, but know that they will be instantiated prior to any access. This has the strange side effect that a static member, if never accessed during program execution, may actually never be instantiated.

Generally, prefer initializers for initialization if possible:
- They execute in the order declared in the class.
- Guaranteed to be complete prior to the class static constructor executing.

## Static Methods

A method is made static by placing the `static` keyword modifier in front of the field definition. This usually occurs after the access modifier but may appear anywhere prior to the return type.

A static method, provided accessible, may be invoked without a class instance (unlike instance methods with their implicit `this`). Prefix the class name rather than the class instance (like in field access).

So it looks like: `class-name.static-method();`, rather than `instance-name.nonstatic-method();`.

Example: `string.Format()`, a static method of the `string` class which returns a formatted string.

All static methods can only implicitly access other static fields and methods, or `const` fields. This makes sense since the method is invoked without an object (no `this` - it's instance-independent). Of course, method parameters may be supplied of any type.

## Static Properties

Properties can be made static as well by the inclusion of the `static` keyword. Since they are really system-created “wrappers” around methods to create a “field”, they act like methods. Remember that if you create a property that is bound to a static field – make the property static as well, otherwise it will be an instance property, and it will require the use of an instance of the class to access the property.

```csharp
class Test
{
    // Random generators should be shared, multiple instances potentially cause issues
    public static Random r = new Random(); // all Tests will share this Random
    private static int iStaticValue = 20;  // private static field, shared by all
    // static property bound to iStaticValue
    public static int Value => iStaticValue;

    public static int GetRandomNumber(short upperLimit) // static too
    {
        return r.Next(upperLimit); // use static reference to random here
    }

    public static string Excuse() // and again
    {
        string[] messages = { "BS was down..", "Github was down...", "Teams was down..." };
        return messages[GetRandomNumber(messages.Length)]; // use static method here
    }
}

//Then, without ANY Test instances, accessible static elements accessed via class ( Test ): 

int iVal = Test.Value; // access static property to get static field value
Trace.WriteLine( Test.Excuse() ); // Use class name, not instance**
Trace.WriteLine( Test.GetRandomNumber(7) ); // public static again
```

## Static Constructors

Constructors may be static as well as non-static (which are called instance constructors), but have different usage. To create a static constructor, declare as static, but do not provide an access specifier (i.e., no public) and no arguments are allowed. This obviously means only a single static constructor may exist for any class.

The static constructor provides an opportunity to initialize static members when their values are not known at compile time, thereby precluding the use of an initializer. Only static fields are available within the constructor body, but code may access any other static methods available for use within the assembly. The system only guarantees that the static constructor will be invoked prior to the use of any static members; there is no way to determine if it will be invoked before or after other static constructors within the application – don't make assumptions on when it will be run.

** Although likely not relevant, if you fail to provide a static constructor for a class, even with static field initializers, those initializers may NOT actually be invoked/assigned until just prior to static field access. This means that class instances may come and go and the static initializers will not have executed! This really is only relevant if you use a static method as the initializer source and that method introduces a side-effect you thought would be guaranteed to be complete before ANY class instance is created. If you do want this action, completing a static constructor will ensure all initializers complete prior to the static constructor, which is guaranteed to complete before the instance constructor.

Now consider the special consequence of `static` for use with objects like `Random`.

If each class instance has an instance `Random` (initialized at creation time), then if a group of class instances are created in rapid succession (i.e., fill an array or container), then the potential for ALL instances to be initialized with the SAME seed value is likely. The side effect is that all instances will create the same stream of random values... probably not what was desired. Obviously, it is inefficient for each object to have its own instance of a `Random` object. If, however, all instances share a static `Random` object, then all retrievals of `Next()` access the same object thereby (in all probability) not generating duplicate sequences, and saving resources and field space.

```csharp
class StaticSample
{
    // static fields
    private static Random rnd = null; // explicit null, initialized in CTOR
    private static int iStaticRand;   // private static field
    
    // Careful! If you are binding a property to a static member, both
    // should be static! What happens if the property isn't?
    public static int CurrentRand => iStaticRand;

    // instance property, utilizing static members
    public int InstanceRand => rnd.Next(0, StaticSample.CurrentRand);

    // Static CTOR - runs before instance CTOR..
    static StaticSample()
    {
        rnd = new Random(DateTime.Now.Millisecond); // initialize Random object
        iStaticRand = rnd.Next(100, 200); // set static random value, won't change
    }
}
{
    // Access static random property, sometime before this line, the
    // static CTOR fired and initialized the random object and Current Rand value
    WriteLine($"Static property rand = {StaticSample.CurrentRand}");
    // Now make some instances, they all use the same static Random object
    StaticSample a = new StaticSample();
    WriteLine($"Instance a has rand = {a.InstanceRand}");
    StaticSample b = new StaticSample();
    WriteLine($"Instance b has rand = {b.InstanceRand}");
    // Access through class name, static value unchanged
    WriteLine($"Static property rand = {StaticSample.CurrentRand}");
    ReadLine();
}
```
```
Static property rand = 138
Instance a has rand = 101
Instance b has rand = 56
Static property rand = 138
```
