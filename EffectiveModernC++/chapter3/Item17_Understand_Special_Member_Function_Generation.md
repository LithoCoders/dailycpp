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

A consequence of the rule of three is that, if you declare the destructor of the class, then the copy assignment operators would not be
automatically generated.





