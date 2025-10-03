- [Predicates](#predicates)
    - [Example: Creating a System Type (int) Predicate for Use with `<int>` Collections](#example-creating-a-system-type-int-predicate-for-use-with-int-collections)
    - [Example: Creating a User-Defined Type (Thing) Predicate for Use with `<Thing>` Collections](#example-creating-a-user-defined-type-thing-predicate-for-use-with-thing-collections)
    - [Methods Requiring Predicates](#methods-requiring-predicates)
  - [Anonymous Methods](#anonymous-methods)
    - [Benefits of Anonymous Delegates](#benefits-of-anonymous-delegates)
  - [Lambda Expressions](#lambda-expressions)
    - [Examples](#examples)
    - [Lambda Replacement of a `Comparison<>` Method](#lambda-replacement-of-a-comparison-method)
# Predicates

Predicates allow a method to be used to filter/restrict inclusion based on user-defined criteria. Put simply, a predicate will return a bool (true/false) that is determined by the state of the argument object. For example, a method accepts an object and returns true/false if it meets some criteria.

These predicates can be used with a number of `List` methods that automatically iterate through the collection, applying the predicate to each element and acting accordingly.

```csharp
public delegate bool Predicate<in T>(T obj);
```

Predicates are a specific version of a system type 'delegate', which is basically a method reference variable type. It can bind to methods and is the underlying mechanism that allows “handlers” to be used within the C# framework. Recall that the property editor allows you to select the handler desired for a given event – here an event is another specific version of a delegate. In our case, a predicate delegate has an expected signature, and provided the supplied entry point signature “matches” the required signature – in our case `Find()` wants a signature that matches: `bool Name(type)` - then the system silently converts our entry point into the required delegate form.

Depending on the collection, predicates can check if an integer is even, a class instance has an appropriate value, etc.

- Predicates for system types should be added as internal/public static methods in the current namespace/class.
- Predicates for user-defined types (classes) should be added as internal/public static member methods within the class.

### Example: Creating a System Type (int) Predicate for Use with `<int>` Collections

```csharp
// Predicate for integer > 5 would be [internal/public] static bool IsBigger(int iVal);

public static bool IsBigger(int iVal)
{
    return iVal > 5;
}

// For this, returns first element where IsBigger() is true, 
// if none found, default value of type, in this case int = 0 returned

// Formal version, showing that IsBigger method is turned into an integer argument Predicate
WriteLine($"Found: {l.Find(new Predicate<int>(IsBigger))} first that IsBigger()");

// The short-form version, still easy to read, the int is implied by the elements of the list
WriteLine($"Found: {l.Find(IsBigger)} first that IsBigger()");
```

### Example: Creating a User-Defined Type (Thing) Predicate for Use with `<Thing>` Collections

```csharp
class Thing : IComparable
{
    public int X { get; private set; }
    public Thing(int iVal)
    {
        X = iVal;
    }
    // Other class details omitted
    // Predicate<Thing> for use with List() methods using criteria
    public static bool IsEven(Thing arg)
    {
        if (arg is null) return false; // null is not even
        return (arg.X % 2) == 0; // evaluates to true if even
    }
}

// In this case predicate added as a static member method to Thing, to determine if member value is even
Thing found = things.Find(Thing.IsEven); // Note the dot syntax to reference the method

// In this case, if not found, the Thing returned is null, not zero
Console.WriteLine($"Using Find() got {(found?.X.ToString() ?? "not")} found");
```

### Methods Requiring Predicates

- `Find(predicate)` - locate first element that meets the criteria (i.e., returns true from the predicate supplied).
- `FindLast(predicate)` - as above, but last.
- `FindIndex(predicate)` - return index of first element that meets the criteria.
- `FindLastIndex(predicate)` - as above, but last index.
- `FindAll(predicate)` - return a new subset of the invoking collection, comprised only of those elements which meet the criteria. **For reference types, the elements in the list returned refer to the same objects as the source list.
- `RemoveAll(predicate)` - remove from the invoking list, all elements that meet the criteria.
- `Exists(predicate)` - return true if any element in the invoking list meets the criteria.
- `TrueForAll(predicate)` - return true if ALL elements in the invoking list meet the criteria.

It is important to remember that predicate use does not involve invoking the method:
```csharp
Find(Thing.IsEven())   // NO! WRONG!, the predicate required is the method name
Find(Thing.IsEven);    // Yes, no invoke, just supply the name
```

## Anonymous Methods

Sometimes your code calls for predicates or delegates that are only used once in the context of an extension method operation. It is burdensome to code it as an explicit method in your class, and it pollutes your code with what arguably is an unnecessary member.

An anonymous method is a short-form version where the method code is defined in the actual context (place) where it is used. The syntax appears strange and new, but is straightforward.

Note that while the argument type(s) must be specified, the return type is deduced from the anonymous method return statement.

```csharp
// Given a List of int called l, find those less than zero
iTest = l.First(delegate(int iVal) { return iVal < 0; });

// or if spread out:
iTest = l.First(
    delegate(int iVal)
    {
        return iVal < 0;
    }
);
```

### Benefits of Anonymous Delegates

- Benefit of anonymous delegates includes "capturing” local variables which can make the delegate easier to code.

```csharp
int iBoundary = 3; // could be any number, even a random
// Now note the use of LOCAL variable within the delegate,
// to code this with a formal predicate, you must somehow set an 
// internal static class (or somehow accessible) 
// field to be tested within the static method! WooHoo!

// Notice how iBoundary can be used as a “captured” value
int iCount = l.Count(delegate(int iVal) { return iVal < iBoundary; });

// now with anonymous on our class Thing - remember that the field tested must have public access, since the scope
// of the delegate is within THIS method, not within a static method
int iBoundary = new Random().Next(-2, 2); // use random number here
int iCount = t.Count(delegate(Thing obj) { return obj.X < iBoundary; }); // use local boundary as test constraint
```

Anonymous methods have an analogy with their prolific use in JavaScript, though the minimal but required type specification can make using this approach burdensome.

## Lambda Expressions

C# introduced lambda expressions as a short-form for the useful construct. A lambda expression uses a new operator to further simplify the use of anonymous methods while still providing the valuable asset of variable capture.

It uses the `=>` pair of characters as the operator and takes the form:
```csharp
<argument identifier> => <evaluation_code>
```

Example:
```csharp
x => x < 0  // x is the identifier populated by the collection iteration, x < 0 evaluates to bool and is returned.
```

The compiler uses the context to deduce the types involved, and at its most basic use, doesn't require any explicit argument types or return statement.

To use lambdas in methods where multiple arguments are required, like `Comparison<>`, wrap the argument identifiers in parentheses:
```csharp
(left, right) => left.CompareTo(right)
```

### Examples

```csharp
// Assuming the same setup as the anonymous example, note the simplification
iCount = l.Count(x => x < iBoundary);

// Same example again, utilizing the Thing objects populating the collection
// Note: As this is NOT a formal member method, 
// only publicly accessible members/methods/properties are allowed
iCount = t.Count(tThing => tThing.X < iBoundary);

// If you wish more than a single expression term, you must scope the lambda:
iCount = l.FindAll((int x) => { int iNewB = iBoundary * 2; return x < iNewB; }).Count;

// Or spaced out:
iCount = l.FindAll((int x) => 
{ 
    int iNewB = iBoundary * 2; 
    return x < iNewB; 
}).Count;

// Now explicit type with Thing
// If you wish more than a single expression term, you must scope the lambda:
iCount = t.FindAll((Test tThing) => 
{ 
    int iNewB = iBoundary * 2; 
    return tThing.X < iNewB; 
}).Count;

// Or without the explicit type
iCount = t.FindAll(z => { int iNewB = iBoundary * 2; return z.X < iNewB; }).Count;
```

### Lambda Replacement of a `Comparison<>` Method

```csharp
// int Comparison<type>(type arg1, type arg2)
// Wrap your arguments in () - as if you were going to explicitly specify the type
t.Sort((left, right) => left.X.CompareTo(right.X) * -1); // compare and negate to reverse sort

// Or if we have a few lines to execute to properly resolve our return -1/0/1 value:
t.Sort((left, right) => 
{ 
    int iLeft = left.X;
    int iRight = right.X;
    return iLeft.CompareTo(iRight) * -1;
}); // provide a scope for the lambda, remember return REQUIRED here
```

We will consider Predicates and Lambdas again when we visit associative containers.

Lambda expressions are powerful and when used appropriately can make usually laborious things much easier.

Consider the need to invoke methods across process threads. Usually, a delegate must be defined, then an instance created, then the delegate instance must be assigned a matching method. Finally, the delegate is invoked through a `<form>.Invoke()` call:

```csharp
// Normally required delegate definition
private delegate void delVoidInt(int i);

// A UI or thread-sensitive method - Usually form-centric
private void TargetMethod(int i)
{
    Text = i.ToString();
}

// Some non-UI thread method, requires new delegate instance, then Invoke() request
private void MyThreadMethod()
{
    int iNum = 42;
    Invoke(new delVoidInt(TargetMethod), iNum);
}

// Using an anonymous function created with a lambda expression, simple operations
// can now be completed in one line.
// This could be collapsed into:
Invoke(new delVoidInt((arg) => Text = arg.ToString()), iNum); // iNum becomes arg

// Or if somewhat more complicated operations are required.. invoke something ...
Invoke(new delVoidInt((arg) => DoBigUIUpdate(arg), iNum);

// Or, if you can find a matching framework equivalent type to your delegate:
Invoke(new Action<int>((arg) => Text = arg.ToString()), iNum);

// In this case, the system Action<type[,type]> is a void returning multiuse template.
// But best of all when wrapped in variable capture

// Using empty Action(no-params), capture iNum
Invoke(new Action(() => Text = iNum.ToString())); // iNum is captured, available in lambda
```