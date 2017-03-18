# Play with C++17

Information and Examples to experience C++17.

[Last update: March 15, 2017)

This document has been written by [Marco Arena](http://marcoarena.wordpress.com).

[toc]

### Introduction ###

* C++17 has many more "tiny" features than C++11
* Lots of library additions have been used and discussed for years in the industry
* It's a major release of the language and will act as a springboard for C++20
* Compiler support: http://en.cppreference.com/w/cpp/compiler_support
* **GCC** is language-complete: [status](https://gcc.gnu.org/projects/cxx-status.html#cxx11)
* **Clang** is language-complete: [status](https://clang.llvm.org/cxx_status.html)
* **Visual C++**: [news](https://docs.microsoft.com/en-us/cpp/cpp-conformance-improvements-2017)
* Another great presentation: https://jfbastien.github.io/what-is-cpp17/#/8/1
* Another great cheatsheet: https://github.com/AnthonyCalandra/modern-cpp-features#structured-bindings

### Online compilers to play with C++17 ###

GCC and Clang: http://melpon.org/wandbox

Visual C++: http://webcompiler.cloudapp.net/

##Language ##

### Structured bindings

Documentation: http://en.cppreference.com/w/cpp/language/declarations#Decomposition_declaration

Disappointment from the past:
```cpp
tuple<int, int, int, float> MakeRGBa() { ... }

int r, g, b; float a;
tie(r, g, b, a) = MakeRGBa();

array<int, 3> MakeRGB();
// won't compile
int r, g, b;
tie(r, g, b) = MakeRGB();

struct RGBa
{
   int r, g, b;
   float a;
};
RGBa MakeRGBa();

int r, g, b; float a;
// won't compile
tie(r, g, b, a) = MakeRGBa();
```
Types that are **Destructurable** can benefit of *Structured bindings*:

```cpp
auto [r, g, b, a] = MakeRGBa();
```

A type is **destructurable** if:

* Either all non-static data members:
    * Must be public
    * Must be direct members of the type or members of the same public base class of the type
    * Cannot be anonymous unions
* Or, it has:
    * An obj.get<>() method or an ADL-able get<>(obj) overload
    * Specializations of std::tuple_size<> and std::tuple_element<>

Let's use structured bindings to beautifully iterate on a std::map:

```cpp
map<string, int> nameToAge;
for (auto& [name, age] : nameToAge)
{
   cout << name << " has " << age << " years\n";
}
```

Structured binding is kind of syntactic sugar. This one:

```cpp
tuple<int, int, string> make();

auto [a, b, c] = make();
```

it's equivalent to:

```cpp
tuple<int, int, string> make();
auto __tmp = make();
auto& a = get<0>(__tmp);
auto& b = get<1>(__tmp);
auto& c = get<2>(__tmp);
```

At the same time:

```cpp
array<int, 2> arr = {1, 2};

auto [a, b] = arr;
```

it's equivalent to:

```cpp
auto __tmp = arr;
auto& a = __tmp [0]; // get<0>(__tmp )
auto& b = __tmp [1]; // get<1>(__tmp )
```

Instead, this one:

```cpp
array<int, 2> arr = {1, 2};

auto& [a, b] = arr;
```

does not introduce any temporary:

```cpp
auto& a = arr[0]; // get<0>(arr)
auto& b = arr[1]; // get<1>(arr)
```

Structured bindings **does not copy to values**, rather it references them.


Play: http://melpon.org/wandbox/permlink/DxWo0TiMeWG6TFSo

### Template argument deduction for class templates

Documentation: http://en.cppreference.com/w/cpp/language/class_template_deduction

Disappointment from the past:
```cpp
template<typename Func>
class LambdaVisitor : SomeVisitor 
{
   Func visitFn;
public:
   LambdaVisitor(Func f) : visitFn{f} {}
   void Visit(const Node& node) { visitFn(node); }
   // ...
};

// make function to deduce 'Func'
template<typename Func> 
LambdaVisitor<Func> MakeLambdaVisitor(Func f) 
{
   return {f};
}

auto visitor = MakeLambdaVisitor([](const Node& n) {...});
```

What if LambdaVisitor has "scoped" ownership (e.g. is not copyable nor moveable)?
```cpp
template<typenameFunc>
class LambdaVisitor: SomeVisitor 
{
   Funcvisit Fn; lock_guard<mutex> l;
   // the rest is the same as before
```

C++17 introduces "Template argument deduction for class templates":

```cpp
class LambdaVisitor; // same as before

// don't need the make-function anymore

LambdaVisitor visitor{ [](const Node& n) {...} };
```

See [here](http://en.cppreference.com/w/cpp/language/class_template_deduction#User-defined_deduction_guides) for more details on **custom deduction guides**.

Play: http://melpon.org/wandbox/permlink/yeP0bQKndcM8O6vZ

### if/switch statements with initializers

Documentation: http://en.cppreference.com/w/cpp/language/if

New versions of the if and switch statements which simplify common code patterns and help users keep scopes tight:

```cpp
map<string, int> m = { {"A", 1}, {"B", 2 };
if (auto it = m.find("B"); it!=end(m))
{
  // ...
}
```

This syntax - apart from being more compact - is useful to prevent subtle bugs like the following:

```cpp
map<string, int> m = { {"A", 1}, {"B", 2 };
auto it = m.find("B");
if (it!=end(m))
{
  // ...
}
// use *it anyway...
```

Instead, putting **it** into the same scope of the selection will make impossible to dereference such iterator when the if fails.

Play: http://melpon.org/wandbox/permlink/ptdZG2VZAGs4EGEb

### Folding expressions

Documentation: http://en.cppreference.com/w/cpp/language/fold

Reduce (*fold*) a parameter pack over a binary operator.

```cpp
template<typename... Args>
auto sum(Args... args)
{
    return (0 + ... + args);
}

template<typename F, typename... T>
void for_each(F fun, T&&... args)
{
    (fun (std::forward<T>(args)), ...);
}


cout << sum(1,2,3,4) << endl;

for_each([](auto elem) {
    cout << elem << endl;
}, 1, "hello", 10.0);
```

More on [this article by Marco Alesiani](http://www.italiancpp.org/2015/06/15/folding-expressions).

Play: http://melpon.org/wandbox/permlink/9sBbZ3JbW5PtyPDg

## Library Additions ###

C++17 merges many library components that have been in the ecosystem for a long time into the standard.
The good news is that we already have a long experience with such additions.

### string_view

What's ugly with the following code?
```cpp
vector<string> split(const string& str, const char* delims)
{
  vector<string> ret;

  string::size_type start = 0;
  auto pos = str.find_first_of(delims, start);
  while (pos != string::npos)
  {
     if (pos != start)
     {
        ret.push_back(str.substr(start, pos - start));
     }
     start = pos + 1;
     pos = str.find_first_of(delims, start);
  }
  if (start < str.length())
  ret.push_back(str.substr(start, str.length() - start));
  return ret;
}
```

Basically, I pay a new string for each token. Actually, each token is already part of the string that I'm splitting. Some offsets to the original string will suffice...

Can I do better?

Sure, I can employ **string_view**:
```cpp
vector<string_view> split(string_view str, const char* delims)
{
   vector<string_view> ret;
  
   string_view::size_type start = 0;
   auto pos = str.find_first_of(delims, start);
   while (pos != string_view::npos)
   {
      if (pos != start)
      {
         ret.push_back(str.substr(start, pos - start));
      }
      start = pos + 1;
      pos = str.find_first_of(delims, start);
   }
   if (start < str.length())
      ret.push_back(str.substr(start, str.length() - start));
   return ret;
}
```

Or, more generically:

```cpp
template<typename Action>
void on_each_token(string_view str, const char* delims, Action action)
{
   string_view::size_type start = 0;
   auto pos = str.find_first_of(delims, start);
   while (pos != string_view::npos)
   {
      if (pos != start)
      {
         action(str.substr(start, pos - start));
      }
      start = pos + 1;
      pos = str.find_first_of(delims, start);
   }
   if (start < str.length())
      action(str.substr(start, str.length() - start));
}
```

**string_view** is an object that can refer to a constant contiguous sequence of char-like objects with the first element of the sequence at position zero.

**string_view is a reference**. This means: it does not participate in the ownership of the sequence.

Typical implementations:

* const char pointer and a size;
* couple of pointers;

You can imagine string_view as a **smart const char*** which provides any const member function of std::string as well as a few handy utilities to reduce its span. You cannot enlarge a string_view until you reassign it.

First of all, string_view is an **adapter**: different string types cam be adapted into a std::string-like container through string_view:
```cpp
CString cstr = ...
string_view cstrv {cstr.GetString()};

string stdstr = ...
string_view stdstrv {stdstr};

QString qstr = ...
string_view qstrv {qstr.toLatin1().constData()};

// only one implementation
ReturnType readonly_on_string_function(string_view sv); // only one implementation
```

Whenever you can do operations that are just the matter of "moving an offset" on the original string, string_view is your best friend:

```cpp
string_view sv = name.GetString(); // GetString() returns a null-terminated const char*
sv.remove_prefix(std::min(sv.find_first_not_of(" "), sv.size())); 
sv.remove_suffix(std::min(sv.size()-sv.find_last_not_of(" ")-1, sv.size())); 
```

Does it recall you anything?

Yes, **trim**. Without creating a new string instance.

Play: http://melpon.org/wandbox/permlink/D67pp9xu1F4VVhJx

Some string_view pitfalls: https://marcoarena.wordpress.com/2017/01/03/string_view-odi-et-amo/

### associative containers additions

Note: examples and documentation consider only std::map. The same holds for other associative containers.

#### insert_or_assign

Documentation: http://en.cppreference.com/w/cpp/container/map/insert_or_assign

A way to write the classical "insert or update" idiom:
```cpp
template<typename K, typename V>
string& insert_or_assign(map<K, V>& m, K&& k, V&& v) 
{
   	auto p = m.equal_range(k);
	if (p.first != p.second)
		return p.first->second = forward<V>(v);
	return m.emplace_hint(p.first, k, forward<V>(v))->second;
}

map<int, string> m;
insert_or_assign(m, 10, "hello world C++17!"s);
insert_or_assign(m, 20, "associative containers 2.0"s);
```

Finally, C++17 makes makes this part of the container's interface:
```cpp
m.insert_or_assign(10, "hello world C++17!"s);
m.insert_or_assign(20, "associative containers 2.0"s);
```

Why this is even better than any hand-made "insert or update":
* implementers may take advantage or internal details of the container;
* unordered containers finally computes hash only once (they ignore hints).

#### try_emplace

Documentation: http://en.cppreference.com/w/cpp/container/map/try_emplace

More or less like emplace but with two main differences:
1. if the insertion does not happen, does not move from rvalue arguments (emplace/insert leave that unspecified);
2. treats the key and the arguments to the mapped_type separately (emplace requires the arguments to construct a pair).

```cpp
map<string, unique_ptr<int>> m2;
m2["A"] = nullptr;  
ptr = make_unique<int>(23);
m2.try_emplace("A", move(ptr));
assert(ptr); // will never fire (1)
    
// spot the differences (2):
map<string, string> m3;
m3.emplace(piecewise_construct, forward_as_tuple("pippo"), forward_as_tuple(5, 'A'));
cout << m3["pippo"] << "\n";
m3.try_emplace("pluto", 5, 'A'); // cute
cout << m3["pluto"] << "\n\n";
```

#### extract

Documentation: http://en.cppreference.com/w/cpp/container/map/extract

Unlinks a node from an associative container. The node can be found either by position (by passing a **valid** const_iterator) or by key. 
In either case, no elements are copied or moved, only the internal pointers of the container nodes are repointed (rebalancing may occur, as with erase()).
extract returns a **node handle** that owns the extracted element, or *empty* **node handle** in case the element is not found by key.
```cpp
map<string, string> langs { {"apple", "obj-c"}, {"ms", "c#"} };
auto elemHandle = langs.extract("ms");
assert(langs.count("ms") == 0); // the node has been physically extracted from the map
elemHandle.key() = "microsoft";
langs.insert(move(elemHandle)); // move it back
assert(!elemHandle); // the handle is not valid anymore
```

extract is the only way to change a key of a map element without reallocation:

#### merge

Documentation: http://en.cppreference.com/w/cpp/container/map/merge

Attempts to extract ("splice") each element in source and insert it into *this using the the comparison object of *this. If there is an element in *this with key equivalent to the key of an element from source, then that element is not extracted from source. No elements are copied or moved, only the internal pointers of the container nodes are repointed. All pointers and references to the transferred elements remain valid, but now refer into *this, not into source.

```cpp
map<string, string> oldContacts = { 
  {"antonio", "1234"}, {"franco", "34412"}, {"gpad", "166101010"} 
};
map<string, string> newContacts = { 
  {"marco", "23123"}, {"gpad", "8991234"} 
};
    
// nodes are physically extracted from OldContacts, duplicates are skipped
newContacts.merge(oldContacts); 
/* newContacts = 
  {"antonio", "1234"}, 
  {"franco", "34412"}, 
  {"gpad", "8991234"}, 
  {"marco", "23123"} 
*/
```

Play: http://melpon.org/wandbox/permlink/hWeMmkSDUd4bNHk3

### optional

Documentation: http://en.cppreference.com/w/cpp/utility/optional

Manages an optional contained value, i.e. a value that may or may not be present. Any instance of optional<T> at any given point in time either contains a value or does not contain a value.

```cpp
optional<Message> m = TryReceive(source);
if (m)
{
 // use *m or m.value()
}
```

optional works nicely with **chaining**. Just defining such an operator:

```cpp
template<typename T, typename F>
auto operator||(optional<T> opt, F f)
{
    return opt ? f(opt.value()) : nullopt;
}
```

(that's basically: **writing the if in a single place**) you can chain computations easily:

```cpp
auto value = ( 	parse("8.5+(21+x)*5") 
			 || optimize
			 || evaluate
			 ).value_or(std::numeric_limits<double>::quiet_NaN());
```

Play: http://melpon.org/wandbox/permlink/Ky3M1j2BJe5sCLc3

### any

Documentation: http://en.cppreference.com/w/cpp/experimental/any

Type-safe container for single values of any type.
```cpp
any a = "hello"s; // contains a string
auto& str = any_cast<const string&>(a);
a = 10; // contains int
auto i = any_cast<int>(a);
```

Play: http://melpon.org/wandbox/permlink/3DMMCjwpagDf3W00

### variant

Represents a **type-safe union**. An instance of **std::variant** at any given time either holds a value of one of its alternative types, or it holds no value.

As with unions, if a variant holds a value of some object type T, the object representation of T is allocated directly within the object representation of the variant itself. **Variant is not allowed to allocate additional (dynamic) memory**.
A variant is not permitted to hold references, arrays, or the type void.
As with unions, the default-initialized variant holds a value of its first alternative, unless that alternative is not default-constructible (in which case default constructor won't compile: the helper class *std::monostate* can be used to make such variants default-constructible).

```cpp
struct Visitor {
  template<typename... T> void operator()(T...) const {} // catch-all
  void operator()(int) const {}
  template<typename T> void operator()(string, T) const {}
};

variant<int,string,double> i{ 10 }, str{ "hi" };

// valid (or nullptr) - noexcept
auto intPtr = get_if<int>(&i); 
auto nullPtr = get_if<double>(&str); 
// valid (or throws std::bad_variant_access)
auto& strRef = get<string>(str);

Visitor v;
visit(v, i); // on int
visit(v, str); // on T

// multi-visitation
visit(v, i, str); // on T, T
visit(v, str, i); // on string, T
```
#### Static polymorphism

A great article on this topic: https://akrzemi1.wordpress.com/2016/02/27/another-polymorphism/

Disambiguation: this is not about [CRTP-based polymorphism](https://en.wikipedia.org/wiki/Curiously_recurring_template_pattern).

Suppose we have a few physical components that may fit some space. These components are well-defined classes that we may or may not modify. The types could have not a common interface.

Using variant we can easily design a flexible architecture:

```cpp
using Block =
  boost::variant<Ballast, Container, MultiContainer>;
```

Data are separated from "algorithms", as in the visitor pattern, because variants can be **visited**:

```cpp
struct Weight_allowance
{
  Weight operator()(const Container& cont) const
  {
    return cont.weight_allowance;
  }
      
  Weight operator()(const MultiContainer& multi) const
  {
    Weight wgt (0);
    for (const Container& cont : multi.containers)
      wgt += cont.weight_allowance;
    return wgt;
  }
      
  Weight operator()(const Ballast&) const
  {
    return Weight(0);
  }
};
```

The true power of the *static* visitor becomes visible when you have to dispatch on two variant types simultaneously:

```cpp
struct Fits : boost::static_visitor<bool>
{
  bool operator()(const Container& c, const Bundle& b) const
  {
    return c.weight_allowance >= b.weight
        && c.volume_allowance >= b.volume;
  }
     
  bool operator()(const Container&   c,
                  const MultiBundle& mb) const
  {
    return bool("sum of bundles fits into c");
  }
     
  bool operator()(const MultiContainer& mc,
                  const Bundle&         b) const
  {
    for (const Container& c : mc.containers)
      if (Fits{}(c, b)) // self-call
        return true;
    return false;
  }
     
  bool operator()(const MultiContainer& mc,
                  const MultiBundle&    mb) const
  {
    return bool("all bundles fit across all sub containers");
  }
      
  template <typename T>
  bool operator()(const Ballast&, const T&) const
  {
    return false;
  }
};
```

Play: http://melpon.org/wandbox/permlink/Bx1EAJNJXOFPwVEM

## Attributes

Documentation: http://en.cppreference.com/w/cpp/language/attributes

Attributes provide the **unified standard syntax for implementation-defined language extensions**. An attribute can be used almost everywhere in the C++ program, and can be applied to almost everything: to types, to variables, to functions, to names, to code blocks, to entire translation units, although each particular attribute is only valid where it is permitted by the implementation.

Besides the standard attributes, implementations may support arbitrary non-standard attributes with implementation-defined behavior. That's possibile thanks to **C++17** that **introduces the following rule**: *attributes unknown to an implementation are ignored without causing an error*.

Modern libraries like the [GSL](https://github.com/Microsoft/GSL) already started using custom attributes, like [[gsl::suppress]].

New C++17 standard attributes:

**[[fallthrough]]**

Appears in a switch statement on a line of its own (technically as an attribute of a null statement), immediately before a case label. Indicates that the fall through from the previous case label is intentional and should not be diagnosed by a compiler that warns on fallthrough:
```cpp
void f(int n)
{
  switch (n) {
    case 1:
    case 2:
      cout << "case 1\n";
      [[fallthrough]];
    case 3: // no warning on fallthrough
      cout << "case 2\n";
      //[[fallthrough]]; // decomment to silent the warning
    case 4: // compiler may warn on fallthrough
      cout << "case 4\n";
      // [[fallthrough]]; // ill formed, not before a case label
  }
}
```

**[[nodiscard]]**

Appears in a function declaration, enumeration declaration, or class declaration. If a function declared nodiscard or a function returning an enumeration or class declared nodiscard by value is called from a discarded-value expression other than a cast to void, the compiler is encouraged to issue a warning:
```cpp
struct [[nodiscard]] handle { int secret; };

handle LoadModule(string_view name)
{
    if (name == "module1")
        return {10}; 
    return {}; // suppose invalid
}

[[nodiscard]] int LastErrorCode()
{
    return 0;
}

LoadModule("module"); // warning encouraged

LastErrorCode(); // warning encouraged

```

**[[maybe_unused]]**

Appears in the declaration of a class, a typedef­, a variable, a non­static data member, a function, an enumeration, or an enumerator. If the compiler issues warnings on unused entities, that warning is suppressed for any entity declared maybe_unused:
```cpp
// v--- the function may be unused
 [[maybe_unused]] void unused([[maybe_unused]] bool thing1, [[maybe_unused]] bool thing2)
{
   [[maybe_unused]] bool b = thing1 && thing2;
   assert(b); // in release mode, assert is compiled out, and b is unused
              // no warning because it is declared [[maybe_unused]]
} // parameters thing1 and thing2 are not used, no warning
```

Play: http://melpon.org/wandbox/permlink/sIo3MzPDlUxm1OjX

## Generic Programming ##

### nested namespace declarations

Documentation: http://en.cppreference.com/w/cpp/language/namespace

Not really a "generic programming" feature, more a language feature. It's listed here because, it's more likely that this feature will be massively employed by library writers.

From C++17 you can write:

```cpp
namespace std::experimental
{
   // ...
}
```

Instead of:

```cpp
namespace std
{
	namespace experimental
	{
	   // ...
	}
}
```

### apply & invoke

Documentation: http://en.cppreference.com/w/cpp/utility/apply 
and http://en.cppreference.com/w/cpp/utility/functional/invoke

**std::apply** invokes a callable object by unpacking a tuple into its arguments:
```cpp
void print(string, int, Foo);

tuple args{"hello"s, 10, Foo{}};
std::apply(print, args);
```
**std::invoke** is the "destructured" counterpart: it invokes the callable object with the parameters passed unrolled:
```cpp
void print(string, int, Foo);

std::invoke(print, "hello"s, 10, Foo{});
```

Play: http://melpon.org/wandbox/permlink/CbiNcRsGE5YrEnPZ

### make_from_tuple

Documentation: http://en.cppreference.com/w/cpp/utility/tuple/make_tuple

std::make_from_tuple constructs an object of a certain type, using the elements of a tuple as the arguments to the constructor:
```cpp
template<typename K, typename V>
class MyContainer
{
   vector<K> keys;
   vector<V> values;
public:
   template<typename KeyTuple, typename ValueTuple>
   void emplace(std::piecewise_construct_t, KeyTuple&& kt, ValueTuple&& vt)
   {
     // ... some complex logic
     keys.push_back(make_from_tuple<K>(kt));
     values.push_back(make_from_tuple<V>(vt));
   }
};
```

Play: http://melpon.org/wandbox/permlink/OAeVL7N8NwAWlN5o

### if constexpr

Documentation: http://en.cppreference.com/w/cpp/language/if#Constexpr_If

In a *constexpr if* statement, the value of condition must be a contextually converted constant expression of type bool. If the value is true, then statement-false is discarded (if present), otherwise, statement-true is discarded.
The return statements in a discarded statement do not participate in function return type deduction.

```cpp
enum class OS { Linux, Mac, Windows };

#ifdef __linux__
  constexpr OS curr_os = OS::Linux;
#elif __APPLE__
  constexpr OS curr_os = OS::Mac;
#elif __WIN32
  constexpr OS curr_os = OS::Windows;
#endif

void do_something() {
     //do something general

     if constexpr (curr_os == OS::Linux) {
         //do something Linuxy
     }
     else if constexpr (curr_os == OS::Mac) {
         //do something Appley
     }
     else if constexpr (curr_os == OS::Windows) {
         //do something Windowsy
     }

     //do something general
}
```

Play: http://melpon.org/wandbox/permlink/g1bceK9eVAMgWvBq
