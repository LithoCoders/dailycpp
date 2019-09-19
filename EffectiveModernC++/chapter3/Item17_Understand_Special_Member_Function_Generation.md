# Understand Special Member Function Generation

In C++98, the compiler automatically generates four special member functions: the default constructor, the destructor, the copy constructor
and the copy assignment operator.

In C++11, we have two more such functions, namely, the move constructor and the move assignment operator. Their signatures are as 
shown below:

```c++
class Widget {
public:
…
Widget(Widget&& rhs); // move constructor
Widget& operator=(Widget&& rhs); // move assignment operator
…
};
```

These are generated only if needed. If they are generated, then they perform "member-wise" moves on non-static data members of the class. 
Move constructing or Move assigning do not guarantee that moves will be performed. For types that are not move enabled, copy is used.

Move operations are not generated if you declare them yourself. There is a subtle difference between copy operators and move operators.
If you declare a copy constructor, but do not generate a copy assignment operator, the compiler would generate an assignment operator for 
you. However, that is not the case for move operators, the operators are dependent on each other.

Move operations won't be generated for classes that explicitly declare copy operations. This is because, if the default generated copy
operator(i.e. memberwise copy), then the default generated move is also not valid. The same is valid in the other direction also.
If move operation is explicitly declared, then copy operator will no be generated.


## Rule of Three

The rule of three states that if you declare any copy constructor, destructor or assignment operator, you should declare all three.
The logic is that if you want to explicilty declare the copy constructor, this means that you want to manually manage resources.
SO, all other copy operations and the destructor should comply to the same set of rules.

A consequence of the rule of three is that, if you declare the destructor of the class, then the copy assignment operators shpuld not be
automatically generated.However, that is not how C++98 and C++11 generate the code for copy operators. However, C++11 does not generate
move operations for classes which explicitly declare the constructor.

In summary, the move operations are generated only in the followig three cases:
1. No copy operations declared 
2. No move operations declared
3. No destructor declared

You should consider upgrading your code which depends on automatic generation of copy operators, incase you already declared the
destructor using the `default` keyword:
```c++
//easy, because C++11’s “= default” lets you say that explicitly:
class Widget {
public:
…
~Widget(); // user-declared dtor
… // default copy ctor
Widget(const Widget&) = default; // behavior is OK
Widget& // default copy assign
operator=(const Widget&) = default; // behavior is OK
…
};
```
Using the keyword `default` makes your intention clear and can prevent subtle bugs. AT times, when you perform a move, and if the class
had declared a destructor, the move operator is not generated. And instead, a copy would be performed(which is significantly slower
than move). Using the keyword `default` prevents such bugs.

```c++
class Base {
public:
virtual ~Base() = default; // make dtor virtual
Base(Base&&) = default; // support moving
Base& operator=(Base&&) = default;
Base(const Base&) = default; // support copying
Base& operator=(const Base&) = default;
…
};
```

The C++11 rules for these special member functions are:
1. Default Constructor: Same as C++98
2. Destructor : Same as C++98, except that it is `noexcept` by default.
3. Copy Operators: Runtime behavior is same, generation rules mentioned above, has dependency on move.
4. Move operators: New to C++11, generated only if destructors, other copy and move operators are undeclared.

Further Readings: https://stackoverflow.com/questions/3279543/what-is-the-copy-and-swap-idiom

