# Properties

1. Properties
    - Properties help encourage encapsulation because their use removes direct access to member data/fields
    - Properties act like member fields, but do NOT allocate any space
    - Properties are actually shorthand notation for creating methods used to read and write to fields, while still acting ( ie. `X.y = 7;` `iVal = X.z;` ) like member fields requiring no explicit invoke ()
    - Properties do NOT have to map to an actual member field
        - ie. an Area property could perform a calculation based on other fields returning the result, it would, however, just be a readonly property – how would you set an area ?
    - Properties are defined similar to fields in that both have an access modifier ( public ? ) and a data type and name
    - Where a field would end there with either an initializer or semicolon, a property will instead specify the :
        - get method steps involved if the property was accessed for a read, and/or
        - set method steps involved if the property was accessed for a write
    - When implemented, the get/set properties will actually create hidden methods get_XXX, and set_XXX that are part of the class but not visible ( not even through intellisense )

## Manual properties

```csharp
class Box
{
  private int _iWidth;       // member always private
  public int Width           // expose a public property 
  {
    get { return _iWidth; }  // provide get code, may provide either or both
    set { _iWidth = value; } // provide set code, user value arrives via “value”
  }

  // Constructors
  public Box(int width)
  {
    _iWidth = Width;  // or you may use property perhaps if it is checking
    Width = width;    // DON'T do both, pick member unless property value checks the entry
  }

  public void Display()
  {
    Console.WriteLine("Width: {0}", _iWidth); // or Width property is available too..
    Console.WriteLine("Width: {0}", Width);   // NOT as efficient, invokes the hidden getter()
  }
}

Box A = new Box(12); // Now normally you would have to the accessor/mutator combo to increment

// Property acts like an instance member, but data is not “technically” exposed
int iVal = A.Width; // fires the “get” version of the property Width
A.Width = iVal + 2; // once right side evaluated, the “set” property is invoked
A.Width++; // magically fires both getter and setter to make this happen, better for user
```
## Convention within the libraries is to favor properties over traditional get and set accessors.
Using this manual method you are able to supply :
- just get – property and underlying field effectively become readonly
- just set – property and underlying field effectively become writeonly
- both
- or apply protection to either ( get{}, protected set {} ) where readonly to users, but available within class

Example of a Manual property with data validation and a “fake” ( calculated field )

```csharp
class BoxB
{
  private int _iWidth;      // actual member field
  public int Width
  {
    get { return _iWidth; } // expose publicly
    private set             // DATA VALIDATION : only use in class, code it however needed
    {
      if (value < 0)
        _iWidth = -value;   // value keyword available as user's assignment value
      else
        _iWidth = value;
    }
  }

  private int _iHeight;
  // Could provide similar property as with Width

  public int Area           // expose a public property to calculated(fake) field
  {
    get { return _iWidth * _iHeight; }  // provide get code, may provide either or both
    // set { ?? }           // not possible to provide set code.. is it ?
  }

  // alternate get/set syntax using expression bodied methods
  private int _depth;           // private member only field, used in property below
  public int Depth             // expose a public property to calculated(fake) field
  {
    get => _depth * 10; // implied return, expression is value returned
    set => _depth = value / 10; // single expression only - one-liner
  }

  // alternate shortcut syntax using expression bodied property
  public int Area3 => _iWidth * _iHeight; // with only expression, get-only is implied

  // Constructors
  public BoxB(int iWidth, int iHeight)
  {
    Width = iWidth;      // use property to protect against bad values
    _iHeight = iHeight;  // no protection here... should fix that
  }
}

BoxB box = new BoxB(-12, 12); // Width argument will be corrected

int iArea = box.Area;         // Use calculated property as if it is a field
```

## Automatic Properties
- You can imagine the additional typing required for a populous class with many fields that should be available to the user.... automatic properties to the rescue
- New to .NET 3.0, now better in 6.0, you can let the compiler do ( most ) of the job

```csharp
public class Employee
{
  private static Random _rnd = new Random(); // helper member
  public string FullName { get; set; } // note the property name is specified, but not
  // the member field – compiler makes it for us...
  public string Id { get; private set; } // public get for the user, setter only in class
  public int iNum { get; } //  readonly, can only be set/initialized in body of CTOR
  public int iNumA { get; } = 9; //  readonly, use initializer
  public int iNumB { get; } = Environment.TickCount; // readonly, initializer version
  public List<int> myList { get; } = new List<int>(); // initializer allowed
  public int iNumNo { set; } // ?? Doesn't make sense, does it ? You've made a garbage can...

  // Another expression bodied example, not automatic
  private int _secret = _rnd.Next(); // my secret number
  public int Secret // If your properties can implemented as a single expression :
    {
        get => _secret; // expression-bodied properties – note implied “return”
        set => _secret = value; // typical use of value, but single expression
    }

  public Employee(string Name, string Id)
  {
    FullName = Name;                  // MUST use the property to access, actual field hidden
    iNum = _rnd.Next();               // only able to initialize here
    this.Id = Id;                     // OOPS, must use this. to resolve name conflict!
  }
}
```

## New c# properties enhancements.

There are numerous new features relevant to properties that have been added to c#, most are not available when using framework 4.8.1 and less. Expression-bodied members are allowed and can shorten the code for simple methods.

Here is a contrived example from MSDN
``` c#
public class Person
{
    private string? _firstName; // nullable string
    public string? FirstName // property
    {
        get => _firstName; // shortcut, provide the value returned
        set
        {   // no shortcut, multiple lines required
            _firstName = value;
            _fullName = null;
        }
    }

    private string? _lastName; // as above
    public string? LastName
    {
        get => _lastName;
        set
        {
            _lastName = value;
            _fullName = null;
        }
    }

    private string? _fullName; // rebuilt and saved on modifications
    public string FullName
    {
        // NO set - a computed field
        get
        {
            if (_fullName is null)
                _fullName = $"{FirstName} {LastName}";
            return _fullName;
        }
    }
}
```