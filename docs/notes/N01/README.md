
# c#- Review
## Precedence Table
   A reminder of the precedence table, which impacts the execution of our ever growing expression base syntax.

   |**Category**|**Operators**|
   | :- | :- |
   |Primary|`x.y  f(x)  a[x]  x++  x--  new typeof checked  unchecked`|
   |Unary|`+  -  !  ~  ++x  --x  (T)x`|
   |Multiplicative|`\*  /  %`|
   |Additive|`+  -`|
   |Shift|`<<  >>`|
   |Relational and type testing|`<  >  <=  >=  is  as`|
   |Equality|``==  !=`|
   |Logical AND|`&`|
   |Logical XOR|`^`|
   |Logical OR|`\|`|
   |Conditional AND|`&&`|
   |Conditional OR|`\|\|`|
   |Conditional|`?:`|
   |Assignment|`=  \*=  /=  %=  +=  -=  <<=  >>=  &=  ^=  \|=`|

## Data Types in C#
By now you should be familiar with the fundamental data types available in C#. All C# fundamental data types are value types (except string), and have a corresponding system type behind them. The following table shows the C# aliases, data types, and ranges:

Integral Types

   |**Alias**|**Type**|**Range**|
   | :- | :- | :- |
   |sbyte|System.SByte|-27 to 27-1|
   |byte|System.Byte|0 to 28-1|
   |char|System.Char|U+0000 to U+FFFF|
   |short|System.Int16|-215 to 215-1|
   |ushort|System.UInt16|0 to 216-1|
   |int|System.Int32|-231 to 231-1|
   |uint|System.UInt32|0 to 232-1|
   |long|System.Int64|-263 to 263-1|
   |ulong|System.UInt64|0 to 264-1|




Floating-Point Types

   |**Alias**|**Type**|**Range**|
   | :- | :- | :- |
   |float|System.Single (7 digs)|±1.5\*10-45 to 3.4\*1038|
   |double|System.Double (15-16 digs)|±5.0\*10-324 to 1.7\*10308|


Other Types

   |**Alias**|**Type**|**Range**|
   | :- | :- | :- |
   |bool|System.Boolean|`true or false`|
   |decimal|System.Decimal|±1.0\*10-28 to ±7.9\*1028|
   |string|System.String|Any set of Unicode chars|
   |object|System.Object|Any type|

All of the underlying types support, as a minimum, static functions that can indicate the min and max value for the type. Many of the types support other static functions that can do a variety of useful things. The System.Char type, for example, has character classification functions:

``` c#
char c = 'A';
outStr = char.IsLetter(c).ToString(); // true, false
```
The .NET data types also provide the ability to parse values from strings. This means that any text representation of a numeric type can be interpreted as the actual numeric type. Consider the following example:

``` c#
    double d = double.Parse("3.141\_59");// note _ placeholder
    outStr = d.ToString();
```
You should never wrap Parse(s) in try/catch/finally blocks if there is a programmatic way to perform the conversion that will not throw exceptions. Most types have a .TryParse() version that returns a bool and takes an out parameter type.

``` c#
double d;
if (double.TryParse(s, out d))
  answer = d \* 1.05;
```
With automatic out parameter declarations, this becomes easier. As this is not specific to just TryParse(), it will be covered later in this document.


## Arrays
The most primitive ( but still powerful ) base collection object. Arrays are reference types ( more later ) that form a fixed container of any single type.
 ``` c#
string [] sA = new string[] { “one”, “two”, “three” };
string [] sB = { “one”, “two”, “three” };
string [] sC = new string[4] { “one”, “two”, “three” };// last is null
```
Or multi-dimensional arrays :
 ``` c#
bool[,] bStatus = new bool[4, 3]; // 4 rows, 3 columns
//When multi-dimensional arrays are used, traversal requires the retrieval of the length of each dimension.

for (int i = 0; i < bStatus.GetLength(0); i++) //0th/1st dimension is rows
  for (int j = 0; j < bStatus.GetLength(1); j++) //1th/2nd dimension is columns
    bStatus[i, j] = false;
```
Helpers for arrays

Static :

- `Array.Sort( arr );` // if objects can be compared\*
- `int Array.BinarySearch( arr, value )`; // index of found value  it Must be sorted first
- `int Array.IndexOf( arr, value );` // index of found value \*\* No sort requirement, iterative search penalty
- `int Array.LastIndexOf( arr, value );` // as above but last one found
- `Array.Resize( ref arr, newSize );` // resize the array
- `Array.Reverse( arr );` // reverse the order of elements
- This is by no means comprehensive as most have overloads for start, length, etc
## List
List provides a collection with extensive functionality capable of holding any datatype. The type must be specified.

``` c#
List<int> iList; // reference to a List of int, no list exists, is null
List<int> jList = new List<int>(); // declare reference and instantiate a list
List<int> kList = new List<int>(9); // capacity is set, Count is still 0
List<int> mList = new List<int>() {1, 2, 3, 4}; // declare and initialize
List<int> dupList = mList; // dupList now refers to same list as mList
List<int> copyList = (List<int>)mList.Clone();// copy of mList but content type of value/ref extremely important
```
This course will investigate most generic collection types, exercising many of the capabilities, and evaluating the strengths and weaknesses of each.
## Strings in C#
As with just about every other library, .NET provides a type that allows storage and manipulation of strings. The System.String type has methods to do all the things you commonly need to do with strings.

Oddly enough, strings in .NET can’t be modified once their values are set. This type is known as an immutable type. All operations result in new strings being created. This makes strings in .NET rather inefficient. The System.Text.StringBuilder class offers a way around this, but is beyond the scope of this course ( but feel free to have a look ).

While strings in C# may contain escape characters, C# offers ‘verbatim strings’ which are strings that are taken verbatim. If a string is prefixed with a ‘@’ character, the string will not be processed for escape characters. Verbatim strings will also preserve white space characters. This is especially handy when dealing with file pathing.
``` c#
string sNonVerbatim = "C:\\Utils\\SomeDir\\myFile.txt";
string sVerbatim = @"C:\Utils\SomeDir\myFile.txt";
```
A welcome addition to string processing and initialization is interpolated strings, whereby variables or expressions may be embedded within the string, and evaluated upon string assignment. The interpolated string must be preceded by a “$”.
``` c#
int x = 1, y = 2;
// Variable and expression substitution within string expressions
string sExpression = $"Evaluated {x} times {y} = { x * y }";
```
## Type conversion and compatibility
For value types TryParse() is the preferred method of conversion primarily due to no exception processing requirements. Casting works, but can throw exceptions.

For reference types, an alternative exists, which can be structured to eliminate exceptions processing as well, the 'is' and 'as' operators.

## is
`is` returns a boolean indicating that the expression “is” that type ( conversion would be successful )
``` c#
String myThing = "Hi";
if (myThing is String)
  Console.WriteLine("myThing is a String");
```

There is a new extension to the is operator, which allows for automatic assignment to a supplied identifier if the “is” is true.
``` c#
object r = new Random();
if( r is Random rnd )
  Console.WriteLine($"Random num is { rnd.Next()}");
```
The scope of the new variable exists within the scope that the if() construct exists. This allows for the not case use to be just as useful.
``` c#
object r = GetThing();
if (!(r is Random rnd)) // return if not Random, continue with r if it is.
  return;
Console.WriteLine($"Random num is { rnd.Next()}");
```

## as
'as' returns a reference to target type, will be `null` if unsuccessful; this allows a test to determine if it should be used
``` c#
Object testMe = myThing;
String refMe = testMe as String;
if (refMe != null)
  Console.WriteLine("testMe was a String");
```

## Automatic variable declaration for out parameters
When using TryParse() or other functions that use out parameter modifiers, it is troublesome to have to pre-declare the out parameters prior to use. A handy addition to c# 6.0 has added a short-cut to using these functions. Slightly modified syntax now declares the variable within the call expression.
``` c#
if (!int.TryParse("1", out int iVal))
  return; // failure to convert
// successful parsing into automatically declared and assigned variable
string sAns = $"Yes, parsed to {iVal}";
```
## Nullable types
Value types like int will always have a value : -2 billion to +2 billion, or bool will be true or false. It is poor practice to designate arbitrary default “bad” or illegal values unless they are appropriate.

Say an int is used for a “length” type value, then -1 would make sense, it can't have negative length. Say an int represents a guess, perhaps any guess is technically valid, then picking 0 or -1 or whatever other constant value would be problematic and must be sure not to conflict with proper processing.

Value types may be designated “nullable” by appending a “?” to the end of the type. These types may now be checked against null to determine their state.
``` c#
void Nullable()
{
  int? iX = null;
  // Can then be checked for validity
  if (iX == null) // not set yet
    iX = 0;       // use 0 for default processing
  bool? bState = null; // now we technically have a tri-state bool
  // bState can be null or true or false
  switch (bState)
  {
    case true : // its true !!
      break;
    case false: // its false !!
      break;
    case null : // no one set it yet
      bState = false; // pick a state
      break;
    default:
      break;
  }
}
```

## Default Values / Optional parameters
Default values may be added to the method definition argument list.. The defaults must be known at compile time ( constants ), and must reside at the end of the list ( no mandatory may be to the right of the optional values )

Given :
``` c#
void Test(int iX, int iY, double dZ = 0.1, int iWidth = 5, string sWord = "Underdog" )
{ }

void Tester()
{
  Test(1, 2); // uses default dZ = 0.1, iWidth = 5, sWord = "Underdog"
  Test(1, 2, 0.2); // uses 0.2 for dZ, default iWidth = 5, sWord = "Underdog"
  // without named arguments must specify all values to "reach" string replacement
  Test(1, 2, 0.3, 10, "Overcat");
}

//What about useful types like Color which are not constant values  ? How about null ?

void Test2(int iX, int iY, Color cBck = Color.Black) // Error, not constant...

// Well, how about nullable types ? A little extra work, but worth it...

void Test2(int iX, int iY, Color? cBck = null) // now OK, use with a check
{
  if (cBck == null) // if it is null, use Black
  cBck = Color.Black;
  // default set...

}
```
## Named Parameters
Combined with optional parameters, the named parameter calling convention may be useful with large argument lists where many defaults are used but a selective view utilized.

The syntax is simple, to invoke a specific method using named parameters,  just prefix the value with the argument name and colon ( : ). ie. ArgName : argValue.
``` c#
void Test(int iX, int iY, double dZ = 0.1, int iWidth = 5)
{ }

void Tester2()
{
  Test(iX: 1, iY: 2); // using named parameters
  Test(iY: 2, iX: 1); // order irrelevant, but must supply all mandatory arguments
  Test(1, 2, iWidth: 10); // mixed use, using named for optional arguments
  Test(iWidth: 10, 2, 3); // Error, named arguments must follow fixed arguments
  Test(1, iY:2, iWidth:10); // OK, named arguments used after iY:2
}
```

Note this is caller only syntax, no changes to existing/new methods are required.


## Structs
Structs are of type System.ValueType and constitute a UDT ( User Defined dataType ). While structures may have any type as members, by convention, structures are value-types and should only have value-types as members.

When declared on stack, memory is allocated, but uninitialized. When declared with new(), memory allocated on stack and the constructor ( CTOR ) is invoked.

All struct types have an non-replaceable default CTOR that initializes the members. Additional overloaded non-default CTORs are allowed.
``` c#
struct STest
{
  public int iX;
  public int iY;
  public STest( int iVal )
  {
    iX = iVal;
    this.iY = iVal; // or iY = iVal, but required to initialize all members
  }
}

STest s; // on stack

//Console.WriteLine("{0}, {1}", s.iX, s.iY); // Will not compile, uninitialized

s.iX = s.iY = 1;

Console.WriteLine("{0}, {1}", s.iX, s.iY); // Ok now, all members initialized

STest sn = new STest(); // default CTOR invoked

Console.WriteLine("{0}, {1}", s.iX, s.iY); // Ok, default initialized

STest st = new STest( 23 ); // user defined CTOR invoked

Console.WriteLine("{0}, {1}", s.iX, s.iY); // Ok, user initialized
```

Structs as value types require use of ref modifier to allow invoked methods to modify existing instance, otherwise a copy of the struct is made and this is why struct members are normally only value-types.

ie. a struct with a value-type and reference member copy will result in a copy where the value-type is a copy of the original instance, but both structs will be holding a reference to the same reference object !

## break/continue/return
- `return; // no value, immediately stop execution returning no value to the caller`
- `return someValue; // immediately stop execution returning someValue to the caller`
- `break; // in looping context, will stop execution of the most local looping scope, continuing execution at the next statement after the loop scope`
- `continue; // in looping context, will stop execution of the current loop iteration and proceed to the conditional loop continuance check.` If it is a for loop, the iteration increment step is performed as well.

## Value vs Reference types
### Creation

struct declared or new(ed) on the stack, derived from System.ValueType reference-types ( arrays, classes, etc ) are  new() only on the managed heap, derived from System.Object

### Assignment

struct – if all fields are value-types, assignment performs as expected, assigning the left hand side fields the value of the right hand side fields, maintaining variable independence, BUT – if the struct contains any reference types then references of left hand side are rebound to “point to” the right hand reference (now they refer to the SAME object)

reference – the left hand reference is abandoned and is rebound to refer to the right hand object. Simple one step operation ( unlike value assignment ), but consequences must be considered, you have 2 reference that now refer to the same object ( if one reference changes a field, that will be visible in the other )

### Comparison

struct – System.ValueType provides an implementation of Equals(), which, among other things, is optimized to compare value types, known as Value Equality

if the objects are of identical value types, and the fields are decomposable value-types, then the comparison is optimized into a bitwise equality check. This becomes more complex if the struct contains reference types ( NOT good ), then the equality check becomes a mix of value equality and identity ( reference ) equality.

reference – `System.Object` provides an implementation of `Equals()` too, but for reference types, its functionality is completely different. It implements Identity Equality, which basically means that unless the 2 references refer to the same object, it will return false, even if both reference refer to their own instance of an object which contain identically matched fields ! This effectively throws a wrench into things when you are looking to compare reference types based on Value Equality rather than Identity Equality. This can be overcome by overloading the `Equals()` method for your reference class, this however can open additional cans (  of worms ) and one must then consider additional responsibilities that Equals() handles ( more to come, just be aware ! )

Use of the static `==` and `!=` operators should only be used when the underlying structure and mechanics of the types used is known. By default the `==`. `!=` operators utilize the member Equals() method to determine equality – but only are available for reference types ( no `==/!=` for user defined value types - structs – use Equals()). These operators are only meant to be used as a short hand, and will ultimately invoke Equals() anyway.
## Boxing/Unboxing
Boxing – implicit conversion of a value type to a reference type

This occurs when a value type is used as an argument for a method whose parameter type is based on Object.

Note : boxing constructs the wrapper reference class instance around a copy of the value, not the actual value, thus modifications to the boxed object are NOT persistent, compiler alerts you when you attempt to modify the boxed value indicating that it is valid only for nullable types ( boxed value types are not nullable )

Occurs when :

- convert from value type to object reference
- convert from value type to System.ValueType reference
- convert from value type to reference to interface implemented by value type
- convert from enum type to System.Enum reference ( not mentioned )

Unboxing – exact opposite, value in box is copied out and assigned to a instance of the value on the stack. This is an entirely new variable that just happens to hold the same value as the boxed object's value
``` c#
int iVal = 9;
PrintMod(iVal);
static public void PrintMod(object obj) //<< boxing occurs here when passed
{
  Console.WriteLine("I could be anything : {0}", obj.ToString());
  // Unboxing attempt, cast and attempt to modify
  //  (int)obj += 10; // X - Nope, LHS of = must be var, property or indexer
  int iVal = (int)obj; // Unboxing – cast to int, make copy of obj's value
  iVal += 10; // no problem, but not modifying obj anyway
}
```

Boxed values can be modified, but only if implemented via a user defined structure, where the struct implements an interface containing a modifiable property, whereby the object is cast to the interface type and the property is modified via this mechanism ( not for us to use )

Boxing ( invisible, no explicit mechanism required ), and unboxing ( usually via explicit cast ) are handy for the necessity of mixing reference and value types in c#, while the auto-boxing is a time saver, try to avoid situations where this will occur as a performance penalty will apply. ie. an array of Object type, which will happily accept “boxed” value types, \*\* you can usually detect boxing/unboxing occasionally when your code exhibits seemingly unrequired casts.

The inclusion of generics ( like List<> ) provide a mechanism to avoid boxing/unboxing when used with value types but where the power of the generic collections are desired. While other collections exist, they will, by nature, box and unbox the elements when used resulting in an efficiency penalty.

## Pass by [<>] conventions.
The argument passing modifiers [none], out, ref and params all have functionality based on the type being passed.

## [default ] Pass-by-value
The default mechanism, no decoration or keyword is required. Also known as pass-by-copy. The argument variable is assigned a copy of the passing argument. When the argument is a value-type, the argument variable has its own copy of the value and can not affect the value of the calling argument. This often is desirable, as the caller ( invoker ) should be protected from having passed data modified unless explicitly specified.

While this is relevant to normal value types ( int, bool, etc ) , it is also relevant to structures. Passing a structure object will result in a complete copy of the structure being made, and modifications to individual member fields will only impact the temporary copy, not the original.

When the argument is a reference-type, the 'reference' is copied to the argument variable. This is an important distinction, the passing reference can't be modified, but, the copied reference still is attached to the source reference object ! The method can't change the reference ( ie. Make it refer to a different object ), but using the reference can modify object being referred to. An example would be passing an array, whereby the array elements modified are actually changing the same array object that was passed.

## `ref`
Pass-by-reference : use the ref keyword modifier prior to the type in the argument list. This modifier will result in the caller argument and receiving argument variable actually referring to the same object, whether it be value or reference-type. Since both refer to the same object, modifications are persistent.

While no real distinction is necessary between value-type and reference-types, an extra caveat is necessary for reference-types. All reference-types must be initialized prior to being pass-by-reference(d).

## `out`
Pass-by-out : use of the out keyword modifier is identical to the pass-by-reference ( ref ) modifier, except that the initialization requirement is waived, and the invoked method must assign the out modified argument. Obviously, this mechanism is specifically designed to be used for methods whose specific functionality is to programmatically determine an appropriate value and assign it to the passed argument.

## Params
Provides a way to allow for an unknown number of same type arguments to be passed to a method. The most obvious use of this is in using Command Line Arguments to execute your programs. This obviously has a limited and specific use within the framework. Only the last argument in a function may be decorated with the params modifier. The benefit of this is a single argument name representing an unknown number of multiple arguments. The arguments appear as an array in the receiving function. Consider the following example:
``` c#
// using this implementation of Sumem
private int Sumem( params int [] args)
{
  int iSum = 0;
  for (int i = 0; i < args.Length; ++i)
  {
    iSum += args[i];
  }
  return iSum;
}

// use a function to add up a bunch of ints
int a = Sumem(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// see what Sumem returned
Trace.WriteLine("a after Sumem : " + a.ToString());
```

## Dialogs
Dialogs are just Windows forms that are spawned from an application in either a modal or modeless state.

A modal form takes focus from the spawning form and will not allow interaction with other application components until it is dismissed ( closed ).

A modeless form is shown and the user may interact with any application components.
``` c#
InfoDialog dlg = new InfoDialog();
// Show modally and determine result, execution stops while dialog is open
if( dlg.ShowDialog() == DialogResult.OK )
  status = “OK”;
```
In the modal dialog the user may programatically close it by setting the DialogResult property. This could be done in a timer callback, button callback or any executed code.

`DialogResult = DialogResult.OK;`

To show the dialog modelessly, where the dialog is shown but main form processing continues, the `Show()` method is used. Any number of forms can be “Show()” ed. All processing is still performed by the main form thread, processing messages and dispatching to the appropriate form/method.

`dlg.Show(); // show the supplement form, continue processing`

In both modal and modeless dialogs, information may be “pushed” in by use of public members ( never ), public methods ( in a pinch ), or preferably with properties.

Information retrieval for modal dialogs occur after the dialog has closed and the same methodology can be used to “pull” information as to the final state of the modal dialog.

Information retrieval from modeless dialogs is “possible” the same way, the difference being that the spawning ( main ) form has no idea when the user has changed the form information or even if  the user has closed and/or hidden it. While polling can provide a feeble solution, in this case, the best practice is to use delegates.

## Null Conditional Operator ( The Elvis Operator )
To streamline null reference checking and/or copying code, the Null Conditional Operator was introduced. It uses the ? character either followed by a dot( . ) or indexer [ ].

When the `?` is encountered in this context, the expression up to the ? is evaluated and if it is null the expression terminates evaluation, returning null if required, otherwise evaluation continues.
``` c#
string s = null;
if( s != null ) // null check prior to access
  s = s.Substring(0, s.Length / 2);
//OR implicit check
s = s?.Substring(0, s.Length / 2);
```
While handy, the strength for our use lies in the null check and implicit atomicity of the invocation operation for delegates or events. Recall the pattern normally used for delegate/event firing :
``` c#
public delegate void dVoidBool(bool b); // delegate TYPE
void DelegateTarget(bool b) { } // intended target method

dVoidBool _delSample = null; // instance of delegate type, null == no target
public Sample()
{
  _delSample = DelegateTarget; // Assign delegate instance a target method
}

void Show()
{
  // The "old" way, make a copy, validate for null, invoke
  dVoidBool localDelegate = \_delSample;
  if (localDelegate != null)
  localDelegate.Invoke(true);
  // OR using Elvis
  // null check with referential guarantee that
  // _delSample won't change during the check/invoke step
  _delSample?.Invoke(true);

}
```
In summary, the null-conditional operator will:

- Return `null` if the operand is `null`
- Short-circuit additional invocations in the call chain if the operand is `null`
- Return a nullable type (`System.Nullable<T>`) if the target member returns a value type
- Support delegate invocation in a thread safe manner
- Is available as both a member operator (`?.`) and an index operator (`?[…]`)

## Null Coalescing operator ??
The null coalescing operator is similar to the conditional operator ( expr ? true : false ), but instead of checking the expression for true or false, the null coalescing operator explicitly tests the expression for null and substitutes the supplied expression.
``` c#
string firstname = null;
string fName = firstname == null ? "Unknown" : firstname; // conditional operator
string fNamed = firstname ?? "Unknown"; // null coalescing operator
```
This operator becomes more useful when combined with class objects and a chain of references is evaluated.
``` c#
string NameDept = Corp?.Dept?.Name ?? "Unknown"; // null coalescing operator
```
In this case if any step along the way evaluates to null, evaluation is discontinued with a null which is then coalesced into “Unknown”, Saving an extended, nested multi-step evaluation.

## Delegates
Delegates in the broadest sense can be thought of as function/method objects that may be dynamically populated with a callback method and invoked when desired.

Delegate use requires a delegate definition, a delegate instance, a callback method assignment, and a delegate invocation.

The delegate definition describes what method signature the delegate will be registering.

The delegate instance is the creation of an actual delegate object of the desired delegate definition.

The calllback method assignment instructs the delegate instance what method to call when it is invoked.

The invocation of the delegate will sequentially invoke all methods to which it has been assigned. The ability of invoke a collection of methods rather than just a singleton is known as multi-cast.

Delegates are the foundation of the event system of the Windows forms environment.

Delegate definition : Usually resides in the class that will declare the delegate instance ( like the modeless dialog ), ensure it is visible and signature reflects the types required for intended functionality.
``` c#
public delegate bool _delBoolIntString(int i, string s);
```

Delegate instance :  Usually declared where the delegate will be invoked ( like the modeless dialog ), in the simplest case it is made public to allow  method assignment by the interested party ( the main form ).
``` c#
public _delBoolIntString myDelegate = null;
```
Callback method assignment/registration. Usually done by the interested party ( the main form ), registering an existing method with a matching signature to the delegate definition. Can be done as a single assignment ( `=` ), or using the overloaded ( `+=` ) operator to add additional callbacks to the existing collection.
``` c#
MyModelessDialog.myDelegate = myCallbackMethod;
```
Finally, the delegate may be invoked thereby sequentially invoking all registered callbacks.
``` c#
myDelegate?.Invoke( 2, “string arg” ); // Null checked, Blocking Invoke
myDelegate( 2, “string arg” ); // Blocking, could throw on null
myDelegate?.BeginInvoke( 2, “string arg” ); // checked, NON-Blocking
```
Refrain from using `BeginInvoke()` as a default over `Invoke()`. `BeginInvoke()` is primarily used under specific circumstances, often where threads might be entangled in a deadlock scenario. `BeginInvoke()` is actually the first component in a full asynchronous Begin/End scenario and must be used properly to be able to capture a return value if the delegate signature is expecting a return value.

## Threads
While ThreadPool threads have a simplified and distinct role in multi-threaded programming, generally more explicit control over thread operation is desirable.

The Thread object provides this level of control and is included into your project via the `System.Threading` namespace.
``` c#
Thread _threadGenerate = null; // member thread object, or collection of
//_threadGenerate = new Thread( new ParameterizedThreadStart(DoGenerate)); // Long form
_threadGenerate = new Thread(DoGenerate); // short-cut without delegate, compiler deduces it

_threadGenerate.IsBackground = true; // NOT the default, so set it

_threadGenerate.Start(ClientSize); // Start thread with single parameter, any type allowed
```
Now within the thread method, which has a signature of `void fnd ( object obj )`, while the thread's context is as a member of the form, and therefore has access to all form members, care must be taken to not attempt operations which “step” on main thread UI access.

### Thread Communication and Cross-Thread Violations

For the thread to communicate back into the main form, 2 main options exist :
- using separate data marshaling operations ( like `lock()` ) and non-UI data elements
- using a delegate and the form or control's `Invoke()` method !! NOT the delegate invoke !!

The choice made here depends on whether the main form needs to know WHEN the data has been changed – if the main form needs to immediately show updated data then, REALLY, you must be periodically ( timer ) be checking for this “updated” data. Sometimes a combination of both is required – the thread might update numerous elements ( properly ) and via a single parameterless delegate POKE the main thread to process/display the data.

### Invoke()

2 Examples are shown, below is an explicit use of Invoke() in a thread method to perform a bitmap set operation
on the main form. The second example is a better, best practice version, where the Invoke() is performed in the
method itself.

Explicit Example

Invoke() is a control/form method ( not delegate Invoke()) and provides a mechanism to request that the form perform the invocation of the method in its thread.

Using the delegate method, first a delegate type and instance are required :
``` c#
delegate void delVoidBitmap( Bitmap b ); // delegate type
delVoidBitmap _SetBackground = null;
```
Then the instance should be assigned :
``` c#
// _SetBackground = new delVoidBitmap(SetBackground); // Explicit long form
// OR short form :
_SetBackground = SetBackground; // Register our Invoke() callback for completion
```

Then the delegate may be used within the thread to “request” the main form thread execute the delegate :
``` c#
try
{
  Invoke(_SetBackground, bm);// BLOCKING !, use BeginInvoke() for non-blocking
  // Beware, this version will throw if _SetBackground is null
}
catch // catch all, if it didn't work, not much to "undo" this,
{
  System.Diagnostics.Trace.WriteLine("Failed to set background");
  //MessageBox.Show("Failed to set background"); // Not best practice
}
```

In this example, the bitmap is sent back through the delegate invoke, but this could have been modified to allow the thread to modify the bitmap as a form member, and when ready, notify the main form via the delegate invoke  that the  bitmap is now ready for use... The combination of these operations is a design decision based on your specific problem.

### Best Practice Example

This example will perform the Invoke() from within the desired method itself. This is not recursion, rather
a way to intercept the method prior to causing a cross-thread violation. This means any call to the method
will correctly execute the code depending on the context.

``` c#
private void SetBackground(Bitmap bmp)
{
  // Check if we are in the correct thread, this.InvokeRequired, which is a 
  //  property of the form, it determines if we are in the correct thread, 
  //  allowing control based on context.
  if (InvokeRequired)
  {
    // If we made it here, this is running in a different thread than the form
    try
    {
      Invoke(new delVoidBitmap(SetBackground), bmp);
      // or
      delVoidBitmap dSetBackground = new delVoidBitmap(SetBackground);
      Invoke(dSetBackground, bmp); // Simplified way, but still requires delegate
    }
    catch // catch all, if it didn't work, not much to "undo" this,
    {
      System.Diagnostics.Trace.WriteLine("Failed to set background");
      //MessageBox.Show("Failed to set background"); // Not best practice
    }
    return; // Why return ? Because we were NOT in a UI thread, 
    // so we Invoked() it and that call will skip this block,
    // and set the background image
  }
  // We are in the correct thread, so we can safely modify the UI
  BackgroundImage = bmp;
}
```

## Data Synchronization/Marshaling with lock()
While many options exist within the .NET framework to marshal data between threaded or application context different entities, you have previously used the most common and simplest – the `lock()` command.

`lock( object key)` - the lock operation uses a special member within the object reference being chosen as the key to ensure that only one context can be executing once the lock is obtained.

The code executing within the lock context is called the critical section, and may be comprised of any block of code/method calls which the programmer deams necessary to protect from multiple contexts executing concurrently. An example here might be a single ( or many ) collections holding application data.

The only requirement for the key is that it be a reference-type. You must think of the key as a lock/key combination, and all critical sections must use this same combination when attempting access.

The key does NOT necessarily have an actual relationship with the critical section. In other words, if you are using a lock to marshal access to a List<> of data, the key need NOT be the List<> itself – it can be any object BUT all access to the List<> within the context of your code should be uniform in the use of the `lock()` construct WHATEVER object is used as the key.

The caveat here is : the compiler does NOT have your back. You must find all accesses to the critical section ( or collection ) and ensure all accesses are protected via the lock mechanism. The compiler is happy to leave the door open. Hint : “Find all References...”
``` c#
object key = new object(); // make a key specifically for this use
// The above key must be visible to all contexts ( threads ) that
//   will attempt to use it .. so it usually is a form member
lock (key)
{
  // critical section here, key is not required to be used
}

// OR somewhat commonly, the data object is used as the key
//  usually for convenience and readability, here _balls is a List defined
//  as a form member and used as the key itself
lock (_balls)
{
  // critical section using _balls here
  _balls.Add( new Ball() );
}
```
The IDE does NOT verify your correct use of `lock()`. In the above example, other access to the _balls collection is not forced to use `lock()` by the IDE - It is YOUR responsibility. Therefore whenever a critical section ( collection access ) is desired, you should highlight/findall places that the operations occur and ensure YOUR correct implementation of `lock()` to protect your data.

## C# Version 6, 7+ enhancements – new or reminders
Inclusion of static decorator in using statement to pull specific static objects ( and their handy methods ) into scope for ease of use.

``` c#
using static System.Math;
// Sin(), Cos(), etc

using static System.Diagnostics.Trace;
// Write() and WriteLine() now in local scope and are Trace methods, not console

using static GDIDrawer.RandColor;
// GetColor() is local now
```
## Literal enhancements

Binary literals and Digit separators – use underscores to help make numeric literals easier to read.
``` c#
// numbers can get long, use embedded underscore to break up numbers of any type
int lots = 0b1001_1100_1101_0011;
double d = 3_210.012_345_678; // anywhere, but be real

// binary literals and digit separators
int one = 0b0001; // instead of hex 0x01;
```
`nameof()` expressions – return string as name of variable, property or member field.
``` c#
if (String.IsNullOrWhiteSpace(lastName))
  throw new ArgumentException(message: "Cannot be blank", paramName: nameof(lastName));
```
