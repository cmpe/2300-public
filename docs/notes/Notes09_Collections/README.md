- [Collections ( sans List )](#collections--sans-list-)
  - [Dictionary -- an associative container](#dictionary----an-associative-container)
    - [Common Functions](#common-functions)
    - [`Add`](#add)
  - [Predicates and `KeyValuePair`](#predicates-and-keyvaluepair)
  - [Predicates and `KeyValuePair`](#predicates-and-keyvaluepair-1)
  - [Implementation Considerations](#implementation-considerations)
  - [Just for Fun](#just-for-fun)
  - [Extension Methods and `Dictionary`](#extension-methods-and-dictionary)
  - [`HashSet`](#hashset)
  - [LinkedList](#linkedlist)
  - [`Queue`](#queue)
  - [Stack](#stack)
  - [Concurrent Containers](#concurrent-containers)
  - [Misc Collections](#misc-collections)
# Collections ( sans List )

## Dictionary -- an associative container

The `Dictionary` class associates keys and values. Values are accessed
in the `Dictionary` through the key, and access is typically very fast
as the `Dictionary` is implemented with a hash table. It is important
that the hashing algorithm in the key be first-rate, as the
performance of this collection greatly depends on it. Keys within the
dictionary must be unique. Dictionary collections are not sorted. 
Hash codes are of consideration when using a user-defined type ( class )
as a Key.

Each logical element in a `Dictionary` is considered a `KeyValuePair`.
The `KeyValuePair` structure represents the key and its associated
value. Enumerating a `Dictionary` returns these elements in no
particular order, but traversal typically follows order of addition.

Like the other `KeyValuePair` containers, when the collection is
indexed using a key the existence of the entry the use determines
functionality. If the operation is used as a :

l-value ( assignable, left-hand side ) object, then if the entry
already exists, it is returned, if NOT, then a default instance of both
key/value is created, the key is set to the desired key and the new
key/value pair is returned. No indication that this is a new instance is
apparent.

r-value ( used as source for expression, right hand side ), if the
entry already exists, it is returned, otherwise if the key does not
currently reside in the collection, an KeyNotFoundException is thrown.
 Be careful here.

### Common Functions

| Member Name | Description | 
| :--- | :--- |
| `Add(TKey key, TValue value)` | Adds the specified key and value to the dictionary. Throws an exception if the key already exists. | 
| `dictionary[key] = value` | Adds a new key-value pair, or updates the value if the key already exists (used as a setter). | 
| `dictionary[key]` | Gets the value associated with the specified key. **Throws `KeyNotFoundException` if the key does not exist** (used as a getter). | 
| `TryGetValue(TKey key, out TValue value)` | Safely tries to get the value associated with the specified key. Returns `true` if the key is found; otherwise, returns `false`. | 
| `Keys` | Gets a collection containing all the keys in the dictionary. | 
| `Values` | Gets a collection containing all the values in the dictionary. | 
| `ContainsKey(TKey key)` | Determines whether the dictionary contains the specified key. | 
| `ContainsValue(TValue value)` | Determines whether the dictionary contains the specified value. | 
| `Count` | Gets the total number of key/value pairs contained in the dictionary. | 
| `Remove(TKey key)` | Removes the value with the specified key from the dictionary. Returns `true` if the removal was successful; otherwise, `false`. | 
| `Clear()` | Removes all keys and values from the dictionary, resetting it to an empty state. |

### `Add`

There are multiple ways to add elements to a dictionary. The most
obvious is to use `Add`:
```c#
// create a dictionary where the key is a string and the value is a
double

Dictionary<string, double> dic = new Dictionary<string,
double>();

dic.Add("Wombats", 4.99); // key/value pair : but Key ("Wombats") must not exist or exception thrown

foreach (KeyValuePair<string, double> kvp in dic) // enumerate through using KeyValuePair
  WriteLine($"{kvp.Key} cost ${kvp.Value.ToString("f2")} each.");

//Wombats cost $4.99 each.
```

A dictionary can use any type as a key, as long as it has a suitable
hashing and equality implementation.

The `Dictionary` type provides handy shorthand - the use of indexers for
setting/getting values. The key is used as the index. If you attempt to
write to a key that does not exist, the value is added to the
dictionary! But if you attempt to read from a key that does not exist
an exception is thrown:
```c#
// create a dictionary where the key is a string and the value is a double
Dictionary<string, double> dic = new Dictionary<string, double>();

// explicit add
dic.Add("Wombats", 4.99);

// implicit (indexer) if Mice does not exist,
// a Key/Value pair is made of Mice and Value 1.95 is written
dic["Mice"] = 1.95;

// But if it does not exist on read, its different, no free entry, just an exception thrown
try
{
  WriteLine($"Rats cost : {dic["Rats"].ToString("f2")}"); // no "Rats", throw here
}
catch (Exception err)
{
  WriteLine($"Error : {err.Message}");
}

foreach (KeyValuePair<string, double> kvp in dic)
  WriteLine($" {kvp.Key} cost ${kvp.Value.ToString("f2")} each.");

//Error : The given key was not present in the dictionary.
//Wombats cost $4.99 each.
//Mice cost $1.95 each.
```

The major operations the `Dictionary` supports are: indexing, Add,
Clear, ContainsKey, ContainsValue, and Remove. There are also
properties for Keys and Values which return collections of those
items.
```c#
// create a dictionary where the key is an int and value is UDT
Dictionary<int, CInt> dic = new Dictionary<int, CInt>();

// hammer add 10 elements (some values will be replaced...)
// would not work with Add(), since Add() will throw on a duplicate, where [] will over-write
while (dic.Count < 10)
  dic[rnd.Next(0, 10)] = new CInt(rnd.Next(0, 100));

// show rough dic
foreach (KeyValuePair<int, CInt> kvp in dic)
  WriteLine(kvp.Key + " : " + kvp.Value.ToString());

// show by key, using Keys collection and ToList()
List<int> skeys = dic.Keys.ToList();
skeys.Sort();
foreach (int i in skeys)
  WriteLine("Key : " + i + ", Value : " + dic[i].ToString());

// Or use the Keys collection as enumerable, we can't Sort() in this version, but OrderBy
foreach (int i in dic.Keys.OrderBy( k => k ))
  WriteLine("Key : " + i + ", Value : " + dic[i].ToString());

// Key : 0, Value : 16
// Key : 1, Value : 94
// Key : 2, Value : 32
// Key : 3, Value : 64
//... details omitted

// Key : 6, Value : 92
// Key : 9, Value : 33
// Key : 2, Value : 32
// Key : 1, Value : 94
// .. details omitted
```

## Predicates and `KeyValuePair`

The regular operations and extension methods for the `Dictionary` work
much as they do for other containers, except the elements are presented
as instances of `KeyValuePair`. If, for example, you wanted to write a
predicate function for removing dictionary entries that had odd numbers,
it might look like this:
```c#
internal static bool HasOddValue(KeyValuePair<string, int> kvp)
{
  return (kvp.Value % 2 == 1); // true if Value is odd, could be kvp.Key > 10, etc
}
```
Use:
```c#
// dictionary of string, int
Dictionary<string, int> dic = new Dictionary<string, int>();
dic["Rat"] = 4;
dic["Elephant"] = 15;
dic["Goat"] = 9;
dic["Monkey"] = 3;
dic["Dog"] = 2;

// does not work (container modified, so can't enumerate - due to deferred execution)
foreach (KeyValuePair<string, int> kvp in dic.Where(HasOddValue))
  dic.Remove(kvp.Key);

// remove all odd key values from the dictionary, ToList() is used to create a new collection,
// which is actually being used by foreach(), this allows the dictionary to be altered...
foreach (KeyValuePair<string, int> kvp in dic.Where(HasOddValue).ToList())
  dic.Remove(kvp.Key);

// show contents
foreach (KeyValuePair<string, int> kvp in dic)
  Console.WriteLine(kvp.Key + " : " + kvp.Value.ToString());

// Rat : 4
// Dog : 2
```
Yet more but with a different KeyValuePair
```c#
Dictionary<string, int> sDict = new Dictionary<string, int>();
sDict.Add("five", 5);
sDict.Add("two", 2);
sDict.Add("nine", 9);

//... Indexer may be used to insert / modify, but only as l-value (write value )

sDict["eight"] = 8; // key=8, did not exist so if setting then create and stored with value
sDict["two"] = 2; // modify existing
sDict["four"] = 4; // new one added, same as Add()
Console.WriteLine("Valid retrieve of Key=eight : {0}", sDict["eight"]);

// Careful, if used as a r-value ( value source, right-hand side ) and the key does NOT exist,
// rather than constructing a key/value pair like the l-value access, it will just throw an KeyNotFoundException

//int iExc = sDict["ten"]; // Throws a KeyNotFoundException! not same as write

//WriteLine("Exception if invalid key = 10 : {0}", sDict[10]); // Get with invalid key will throw

if (sDict.ContainsKey("four"))
  Console.WriteLine("Key = 'four' exists, Value = {0}", sDict["four"]); // exception safe checked

if (sDict.ContainsValue(8))
  Console.WriteLine("Found value 8 in dictionary");

// Get Value without exception
if (sDict.TryGetValue("four", out int iValue)) // returns true if Key "four" exists, iValue populated

Console.WriteLine("Found key = 'four' with value : {0}", iValue);

// Just using keys
foreach (string s in sDict.Keys)
  Console.WriteLine("Key = " + s);

// Just using values
foreach (int i in sDict.Values)
  Console.WriteLine("Value = " + i.ToString());

// Use KeyValuePair<> for access to both
foreach (KeyValuePair<string,int> p in sDict)
  Console.WriteLine("Key = " + p.Key + " Value = " + p.Value);
```

`Equals()` and `GetHashCode()` are used differently.

`GetHashCode()` is used for object placement in the collection ( ie the
'Primary' key value )

`Equals()` ( or other equality check ) is used to determine if an existing
( duplicate ) key exists
```c#
// Uses GetHashCode() for object placement in collection,
// but uses Equals() ( or other equality check ) to determine
// if duplicate keys are present

Dictionary<Thing,int> d = new Dictionary<Thing,int>(); // Dictionary of Things mapped to ints
d.Add(new Thing(1), 1);
d.Add(new Thing(2), 1);
d.Add(new Thing(3), 1); // Although GetHashCode() is invoked, Equals() is as well and determines duplicate key conditions

// Because Thing overrides Equals() using value semantics, the Thing(2) objects are considered equal

d[new Thing(2)] += 4;

// Same iteration loop, but Key = a Thing
foreach (KeyValuePair<Thing, int> p in d)
  Console.WriteLine("Key = {0} Value = {1}", p.Key.X, p.Value);

// Output : Things ToString() outputs CTOR value
//Key = 1 Value = 1
//Key = 2 Value = 5 // Thing(2) was Equal(), so sum was done on second instance
//Key = 3 Value = 1
```

## Predicates and `KeyValuePair`

The use of lambda expressions are supported for use with dictionary
types as well.

The lambda argument in this case will be the KeyValuePair<> of the
Dictionary type. The parameter will contain both the Key and Value
properties with which you may base your predicate criteria on.
```c#
// Dictionary and Lambdas instead of predicates...
// Get something to experiment on
Dictionary<Thing, int> d = new Dictionary<Thing, int>(); // Dictionary of Things mapped to ints
d.Add(new Thing(1), 11);
d.Add(new Thing(2), 12);
d.Add(new Thing(3), 21);
d.Add(new Thing(4), 22);
d.Add(new Thing(5), 31);
d.Add(new Thing(6), 32);
Console.WriteLine("Lambda for 'odd' keys");

// grab entries for Key Things with ODD X values

foreach (KeyValuePair<Thing, int> p in d.Where( entry => entry.Key.X % 2 == 1))
  Console.WriteLine("Key = {0} Value = {1}", p.Key.X, p.Value);

int iCapturedValue = 21; // could be random, etc.
Console.WriteLine("Lambda for entries whose Value is > 21");
// grab entries where int Values are > 21
foreach (KeyValuePair<Thing, int> p in d.Where( entry => entry.Value >  CapturedValue ))

Console.WriteLine("Key = {0} Value = {1}", p.Key.X, p.Value);
// Output below
//Lambda for 'odd' keys
// Key = 1 Value = 11
// Key = 3 Value = 21
// Key = 5 Value = 31

// Lambda for entries whose Value is > 21
// Key = 4 Value = 22
// Key = 5 Value = 31
// Key = 6 Value = 32

// Create a Dictionary<int,Thing>() by selecting key with lambda as X member ( int )

Dictionary<int,Thing> tmp = l.ToDictionary(thing => thing.X);

// Create a Dictionary<Thing,double>() by selecting key and value,
// here key is the Thing, value is X member  double

Dictionary<Thing, double> td = l.ToDictionary(thing => thing, thing => thing.X  1.0);
```

## Implementation Considerations

`Dictionary` is implemented as a hashtable, ( an array like structure ).
As such, retrieval is done via the hash code of the key. Our previous
generic ignorance of the `GetHashCode()` method and its generic return 1;
coding is not ideal in this situation. Placement into the hashtable is
guided by the hash code returned by the key. If our key is an
user-defined object without proper hash code generation, then the
hashtable degrades to a List<> where each element entered collides
with the default value and is appended after the collision. If a proper
hash algorithm is used then provided the hash algorithm is fast -- the
retrieval is O(1) time.

Keys may not be modified such that their hash code is changed, also as
in SortedList, the key will always be unique. Equality must be supported
by the key, but the container itself is NOT sorted. No ordering is
enforced and when iterating through the container, no assumptions can be
made. The key used should have appropriate GetHashCode() support, and as
such, good key choices are dictate the usefulness and speed.

As a general rule of thumb, if a reference-type is used for the key (
like a user-defined class ), you want to seriously consider whether
overriding Equals() is appropriate. Usually, Equals() is not supplied
and default identity-semantics is used.

## Just for Fun

When a dictionary is used to hold a collection that can be indexed
(especially another dictionary), the syntax begins to eerily look like
the collection set is a 2D array:
```c#
// create a dictionary where the key is an int and the value is another dictionary <int, int>
Dictionary<int, Dictionary<int, int>> dic = new Dictionary<int,Dictionary<int, int>>();

// must add minor 'array' as new dictionary
dic[4] = new Dictionary<int, int>();

// set values : [4] is a read in the major, [3] is a set in the minor
dic[4][3] = 42;
dic[4][-4] = 17;

foreach (KeyValuePair<int, Dictionary<int, int>> okvp in dic)
  foreach (KeyValuePair<int, int> ikvp in okvp.Value)
    Trace.WriteLine( "Major : " + okvp.Key.ToString() +
      " Minor : " + ikvp.Key.ToString() +
      " Value : " + ikvp.Value.ToString());

// ```Major : 4 Minor : 3 Value : 42
// Major : 4 Minor : -4 Value : 17
```
This has the advantage of using array elements that are arbitrary (even
negative), without the requirement for a backing 2D array of matching
dimensions.

## Extension Methods and `Dictionary`

Not all extension methods are all just hugs and puppies. There is some
serious power at your disposal. The Aggregate extension method, for
example, is a very flexible method that can apply a System.Func
delegate to each element of a container while passing cumulative values
to the delegate function:
```c#
// System.Func delegate target
internal static int SumValues(int iValue, KeyValuePair<string, int> kvp)
{
  return (iValue + kvp.Value);
}

//Used by:
// dictionary of string, int
Dictionary<string, int> dic = new Dictionary<string, int>();
dic["Rat"] = 4;
dic["Elephant"] = 15;
dic["Goat"] = 9;
dic["Monkey"] = 3;
dic["Dog"] = 2;

// show contents
foreach (KeyValuePair<string, int> kvp in dic)
  Console.WriteLine(kvp.Key + " : " + kvp.Value.ToString());

// form of System.Func that targets a function that takes an int (accumulator),

// a KeyValuePair<string, int> (source for op), and returns an int (modified accumulator)

System.Func<int, KeyValuePair<string, int>, int> ShownForClairityOnly = SumValues;

Console.WriteLine("Sum of values is " + dic.Aggregate(0, SumValues).ToString());

// Rat : 4
// Elephant : 15
// Goat : 9
// Monkey : 3
// Dog : 2
// Sum of values is 33
```

## `HashSet`

`HashSet` is a collection that stores unique elements and is optimized for fast lookups, additions, and deletions. Unlike List or LinkedList, HashSet is unordered and does not allow duplicate values. Internally, it uses a hash table to manage entries, which allows for O(1) average-time complexity for most operations (such as Add and Contains).

You can consider a HashSet a Dictionary without any values.

Key Characteristics:

- Unique Elements: HashSet does not allow duplicate values. Any attempt to add a duplicate will be ignored.
- Unordered: HashSet does not maintain any particular order of elements.
- Efficient Lookup: Optimized for quick checks to see if an element exists.
- No Indexing: Since it's unordered, there is no index-based access like a List or LinkedList.

Unsupported List-Like Methods: Since a HashSet is not index-based, it lacks methods that rely on ordering or indexing:
- No `AddAt()`, `IndexOf()`, `Insert()`, or `Sort()`
- No `RemoveAt()` or `RemoveRange()` 

Instead, a `HashSet` focuses on fast membership tests and set-based operations.

Common Methods for HashSet:
- `Add(item)` - Adds an item to the HashSet. If the item is already present, no action occurs. Returns true if added, false if it was a duplicate.
- `Remove(item)` - Removes an item from the HashSet. Returns true if successful, false if item wasn't found.
- `Contains(item)` - Checks if an item is in the HashSet, returning true or false.
- `UnionWith(collection)` - Adds all elements of the given collection that are not already in the HashSet.
- `IntersectWith(collection)` - Modifies the HashSet to keep only elements that are present in both the HashSet and the specified collection.
- `ExceptWith(collection)` - Removes all elements from the HashSet that are also present in the specified collection.
- `SymmetricExceptWith(collection)` - Keeps only elements that are in either the HashSet or the specified collection, but not both.
- `Clear()` - Removes all elements from the HashSet.

Usage Examples:
```c#
HashSet<int> numbers = new HashSet<int>(); // Initialize a new HashSet
numbers.Add(1); // Add element to HashSet
numbers.Add(2);
numbers.Add(3);
Console.WriteLine(numbers.Contains(2)); // True, since 2 is in the set

// Attempting to add a duplicate element
bool added = numbers.Add(2); // Returns false, as 2 already exists in the set

// Removing an element
numbers.Remove(1); // Removes 1 from the set
Console.WriteLine(numbers.Contains(1)); // False, as 1 was removed

//Using Set Operations:
HashSet<int> setA = new HashSet<int> { 1, 2, 3, 4 };
HashSet<int> setB = new HashSet<int> { 3, 4, 5, 6 };

// Union of setA and setB (adds only unique elements from both sets to setA)
setA.UnionWith(setB); // setA now contains {1, 2, 3, 4, 5, 6}

// Intersection of setA and setB (modifies setA to only keep elements in both sets)
setA.IntersectWith(setB); // setA now contains {3, 4}

// Difference (Except) of setA and setB (removes elements in setB from setA)
setA.ExceptWith(setB); // setA now contains {1, 2}

// Symmetric Difference (keeps unique elements only in one of the sets)
setA.SymmetricExceptWith(setB); // setA now contains {1, 2, 5, 6}
```

Creating a HashSet from Other Collections

You can initialize a HashSet with an existing collection by passing it as a constructor argument:
```c#
List<int> numberList = new List<int> { 1, 2, 3, 3, 4, 5 };
HashSet<int> numberSet = new HashSet<int>(numberList); // Duplicate 3 will be removed
```
HashSet Properties and Performance:
- `Count`: Returns the number of unique elements in the set.
- `IsSubsetOf` and `IsSupersetOf`: Useful for comparing two collections without iterating manually.
- Performance: The hash table implementation allows average O(1) time complexity for adding, removing, and checking membership of elements.

Its lack of ordering and inability to access elements by index is a trade-off for faster lookups and uniqueness enforcement, which makes it particularly useful for situations where duplicates are not allowed and order doesn't matter.

## LinkedList

`LinkedList` -- effectively the same as List, has different
optimizations for insert/delete ( List is actually an array, whereas
LinkedList is just that -- a Linked List )

Due to list architecture some methods differ :

There is NO => Add(), BinarySearch(), GetRange(), IndexOf(), Insert(),
RemoveAt(), RemoveRange(), Sort(!!!)

This is due to the lack of Random Access ( indexing / subscript ) for
the LinkedList collection.

Replacements differ also in the use of the concept of a
`LinkedListNode<>` which is often a return type or argument type,
that acts as your "index" into the LinkedList -- ie. the objects in your
list don't live at an `[index]`, rather they exist as a node in the
list

The LinkedList has :

`AddFirst(object)` - add object as first node, returns LinkedListNode
reference to new node ( can be ignored )

`AddLast(object)` - add object as last node, returns LinkedListNode
reference to new node ( can be ignored )

`AddBefore(node, object)` - add object before LinkedListNode argument,
returns LinkedListNode reference to new node

`AddAfter(node, object)` - add object after LinkedListNode argument,
returns LinkedListNode reference to new node

`LinkedListNode<>` types are often returned after an operation but some
exist for your use within the LinkedList class itself.

`First`, `Last` are of type LinkedListNode and can be used as
arguments :

ie. `linkedlist.AddLast(value)` would be the same as `linkedlist.AddAfter(
linkedlist.Last, value );`

`LinkedList` supports many of the generic methods seen in the List section

Some of the generic methods like : `Distinct()`, `Intersect()`, `Union()`
return an IEnumerable object BUT there is NOT a ToLinkedList()
helper... ) so you must use the constructor version of creation:

`LinkedList<int> InterLink = new
LinkedList<int>(bLinked.Intersect(cLinked));`

Not quite as convenient, but perhaps this makes the adage "Choose the
most appropriate container" much more relevant.

Some example usage of LinkedLists
```c#
LinkedList<int> aLinked = new LinkedList<int>(); // Make a new LinkedList
aLinked.AddFirst(0); // Add value 0 to front of list
aLinked.AddLast(99); // Add value 99 to end of list

// Using AddAfter(), specifying First will put value 2 in 2nd node in the list,
// then the return of AddAfter is a LinkedListNode referring to the node just added
LinkedListNode<int> node = aLinked.AddAfter(aLinked.First, 2);

// reuse node here rather than First/Last, etc, we can use returned nodes
// Here we use the previously returned node and value 5 into new node using AddBefore()
node = aLinked.AddBefore(node, 5); // here the argument node is ourlast add
foreach (int i in aLinked) // Traverse through LinkedList, automatic with int
  Write($"{i},");

WriteLine(Environment.NewLine);
node = aLinked.First.Next; // assign node = to second node

for (int i = 0; i < 10; ++i)
  node = aLinked.AddAfter(node, rnd.Next(10)); // add all just after second node

// make new list as a copy, not same as mm = ll; where both reference the same list
LinkedList<int> bLinked = new LinkedList<int>(aLinked);

// Examine LinkedList and LinkedListNode elements
// Value property returns the "object" ( ie. Type ) of the list, in this case int of first node

bLinked.First.Value = 8; // Has set/get, here we set the first nodes value to 8

// Also Has properties : Next, Previous
// these refer to the adjacent nodes in list, and
// the List property is the "List" object where this node belongs

bLinked.First.Next.Next.Previous.Next.Previous.Value = -1; // Goofy, but legal and dangerous

LinkedList<int> cLinked = new LinkedList<int>();// Create a new LinkedList with some values
for (int i = 0; i < 10; ++i)
  cLinked.AddLast(rnd.Next(10)); // put some random values to end

// because there is no ToLinkedList() helper for this, but can build a new list
LinkedList<int> InterLink = new LinkedList<int>(bLinked.Intersect(cLinked));
WriteLine("Intersect of Linked Lists :");
foreach (int i in InterLink)
  Write($"{i},");
  WriteLine(Environment.NewLine);

// While foreach() makes collection iteration easy, it's failing for 
// LinkedList's is that it will populate the "Value" member of each node into 
// your foreach() value, stripping the ability to access the entire node.
// We can iterate through the list via an alternate means : the LinkedListNode<> type

// By accessing nodes directly, we can iterate through the list nodes directly,
// perhaps accessing adjacent nodes as we go

// Single node iteration example
LinkedListNode<int> traverseNode = cLinked.First; // attain a reference to the first node
while (traverseNode != null) // loop until we have no nodes left, ie. node is null
{
  Write(traverseNode.Value.ToString()); // output node Value, in this case <int>
  traverseNode = traverseNode.Next; // move to next node, repeat
}

WriteLine(Environment.NewLine);
// Node traversal, but with adjacent node access, output of node and its Next sibling
// attain reference to first node.. same as previous

LinkedListNode<int> traverseTwo = cLinked.First;
// loop provided we have a valid node AND it has a valid adjacent node  NOTE ORDER OF CHECK
while (traverseTwo != null && traverseTwo.Next != null)
{
  // output node and nexts nodes values together
  Write($"{traverseTwo.Value} : {traverseTwo.Next.Value}n");
  // Move to next node
  traverseTwo = traverseTwo.Next;
}

// Or use for() rather than while iteration :

for (LinkedListNode<int> iter = cLinked.First; iter != null; iter = iter.Next)
  Write($"{iter.Value}");

// Or perhaps backwards
for (LinkedListNode<int> iter = cLinked.Last; iter != null; iter = iter.Previous)
  Write($"{iter.Value}");
```

## `Queue`

`Queue` - Represents a C# collection that operates as a First In First
Out ( FIFO ) queue.

A `Queue` is implemented as an array or List, but with simplified
functionality.

This mimics the functionality of "lineups" or queues, where the "first
come -- first served" functionality is desired. Often used to manage
service requests -- like the windows message queue.

Supports a greatly reduced method set including :

- `Count [ readonly property ]` - returns the number of items in the queue
- `Clear()` - remove all elements in the queue, leaving it empty ( Count = 0 )
- `Enqueue( object )` - add the object to the end of the queue
- `Dequeue()` - returns the object at the front of the queue, removing it from the queue
- `Peek()` - returns the object at the front of the queue, but does not remove the element or modify the stack

Dequeue and Peek will throw an
`InvalidOperationException` if the stack is empty

Many of the extension methods are available, as they apply to most
collections

The collection does not support Sort(), and as such BinarySearch() - if
you want these, this is probably the wrong collection choice.

## Stack

`Stack` - Represents a C# collection that operates as a Last In First Out
queue ( LIFO ) or "Stack"

A stack is implemented as an array or List, but with simplified
functionality.

This mimics the stack of objects where the last one added is the first
one removed, as in stacks of cards, program execution stack, etc.

Supports a reduced method set including :
- `Count [ readonly property ]` -- returns the number of elements in the stack
- `Clear()` - Remove all elements from stack, leaving it empty ( Count = 0 )
- `Push(object)` -- Add an element to the stack
- `Pop()` - return the topmost element on the stack, removing it from the stack
- `Peek()` - return the topmost element on the stack, but does not remove it from the stack or modify the stack

`Pop` and `Peek` will throw an `InvalidOperationException` if the stack is empty

Many of the extension methods are available, as they apply to most
collections

The collection does not support Sort(), and as such BinarySearch() - if
you want these, this is probably the wrong collection choice.

Queue and Stack Examples.
```c#
Queue<int> q = new Queue<int>();

for( int i = 0; i < 10 ; ++i )
  q.Enqueue(i); // push an element into the queue

foreach( int i in q )
  Write( i.ToString() + ","); // order is 1,2,3,.. ie. First-In, in Queue order

int j = q.Dequeue(); // remove and return 1st item ( oldest ), 0 removed
int k = q.Peek(); // return ( but not remove ) the 1st item
while( q.Count > 0 ) // exception thrown if empty
  WriteLine($"Extracted :{ q.Dequeue() }");


Stack<int> s = new Stack<int>();
for (int i = 0; i < 10; ++i)
  s.Push(i);

foreach( int i in s )
  Write( i.ToString() + ","); // order is 9,8,7,.. ie. Last-In, in Stack order

int j = s.Peek(); // return top element of stack without removing it
int k = s.Pop(); // remove and return top element of stack
while( s.Count > 0)
  WriteLine("Popped : {s.Pop()}");
```
## Concurrent Containers

In the `System.Collections.Concurrent` namespace there are a number of
thread safe collections that can be used to facilitate concurrent
programming ( threads, async/await ).

[Concurrent in
MSDocs](https://docs.microsoft.com/en-us/dotnet/api/system.collections.concurrent?view=net-6.0)

Some of these include :

- `BlockingCollection<T>` - Provides blocking and bounding capabilities for thread-safe collections that implement `IProducerConsumerCollection<T>`.
- `ConcurrentBag<T>` - Represents a thread-safe, unordered collection of   objects.
- `ConcurrentDictionary<TKey,TValue>` - Represents a thread-safe collection of key/value pairs that can be accessed by multiple threads concurrently.
- `ConcurrentQueue<T>` - Represents a thread-safe first in-first out (FIFO) collection.
- `ConcurrentStack<T>` - Represents a thread-safe last in-first out (LIFO) collection.

With proper use of other concurrent constructs ( lock, Interlocked,
semaphore.. ) concurrent programming could use the standard containers,
there are no built-in compile time safeguards to ensure the
implementation is properly constructed.

The implementation of these containers differ from the traditional
collections in that some operations may not be valid ( due to other
thread/task contexts doing operations concurrently ), and thus use
TryNNN() patterns to ensure proper operation.

For example, if a consumer of a Queue first checks for items prior to
dequeueing them, their context could be interrupted, at which time
another consumer empties the queue, then upon execution the previous
check is no longer valid -- and any dequeue() attempt will be an illegal
operation. Since the check + consume operation must be atomic and
non-interruptable, the dequeue is replaced with a TryDequeue(). An
attempt to dequeue is made, and if successful, returns true with an out
variable populated with the dequeued item, otherwise false is returned.

The most appropriate collection should always be used to ensure
valid/correct operation, but manual concurrent constructs might be
required for educational and operational reasons.

While these containers are great for collection management in
multi-threaded applications, not all operations that need to be
thread-safe are collections. You can use lock() to manage these
operations, but another helper class is useful here Interlocked.

[Interlocked in MS
Docs](https://docs.microsoft.com/en-us/dotnet/api/system.threading.interlocked?view=net-6.0)

There are a number of methods available, but the most common are Add()
and Increment(). These methods ensure that operations on variables are
executed atomically, and will not be corrupted by multi-threaded
operations.

## Misc Collections

- `SortedDictionary`
- `SortedList`
- [Collections ( sans List )](#collections--sans-list-)
  - [Dictionary -- an associative container](#dictionary----an-associative-container)
    - [`Add`](#add)
  - [Predicates and `KeyValuePair`](#predicates-and-keyvaluepair)
  - [Predicates and `KeyValuePair`](#predicates-and-keyvaluepair-1)
  - [Implementation Considerations](#implementation-considerations)
  - [Just for Fun](#just-for-fun)
  - [Extension Methods and `Dictionary`](#extension-methods-and-dictionary)
  - [`HashSet`](#hashset)
  - [LinkedList](#linkedlist)
  - [`Queue`](#queue)
  - [Stack](#stack)
  - [Concurrent Containers](#concurrent-containers)
  - [Misc Collections](#misc-collections)
