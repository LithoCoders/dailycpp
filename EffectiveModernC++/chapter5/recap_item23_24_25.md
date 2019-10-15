Both std::move and std::forward do not generate code.  They are simply functions performing cast on their argument.

- move always cast the argument into a rvalue
- forward performs this casts only when the argument is a rvalue otherwise it does nothing.

a possible implementation for std::move in C++14 is the following:

```cpp
template<typename T>
decltype(auto) move(T&& arg)
{
	using Type = remove_reference<T>(arg)::value&&;
	return static_cast<Type>(arg);
}
```

when this function is called on:

##  rvalues 
```cpp
Object a = std::move(Object());
// Object() is temporary, which is prvalue
```
and move will be instantiated as:
```cpp
//T is deduced to be Object
Object&& move(Object&& arg)
{
	using Type = Object&&;
	return static_cast<Object&&>(arg);
}
```

## on lvalues

```cpp
Object ob1;
Object a = std::move(obj1);
```
and move will be instantiated as:
```cpp
//T is deduced to be Object&
Object&& move(Object& && arg) //& && = &
{
	using Type = Object&&;
	return static_cast<Object&&>(arg);
}
```

## Do not declare object const if you want to move from them.

The fact that you call move on a object does not mean that an actual move operation take place. 

```cpp
class Foo{
	explicit Foo(const string&& s) : 
		bar(move(s)) {};

private: 
	string bar;
}
```

You would expect that `s` is moved into `bar` but it is not because `s` is constant and the constructor for `strings` are the followings:

```cpp
string(const string& );// copy constructor
string(string&& ); //move
```
 The compiler has to choose which of these two to choose. It will go for the copy constructor because `s` in `Foo`s constructor is declared const and `string(string&& )` cannot be called with a `const string&&`.

---

## Forward 
std::forward is a conditional cast. The typical usage is a function having a universal reference as parameter that has to be passed down to some other functions.
The universal reference parameter con be both a lvalue or rvalue, depending on how the function is called.

```cpp
void process(const Widget& lvalArg); // process lvalues
void process(Widget&& rvalArg); // process rvalues
template<typename T> // template that passes
void logAndProcess(T&& param) // param to process
{
	auto now = // get current time
	std::chrono::system_clock::now();
	makeLogEntry("Calling 'process'", now);
	process(std::forward<T>(param));
}
```

```cpp
Widget w;
	logAndProcess(w); // call with lvalue
	logAndProcess(std::move(w)); // call with rvalue
```
The correct overload for process is called depending whether param is a rvalue or a lvalue.


## Rule of thumb for distinguishing rvalue references from universal references
If type deduction happens, then this is a universal reference. And the r or l valueness of the reference depends on the type of the initializer. If you initialize a universal reference with a lvalue then it will be a lvalue reference, otherwise it will be an rvalue reference. 

> **Note that the deduction must appear only in the form T&&. for instance `vector<T> && v` is not an universal reference. Qualifier are also not allowed for a variable to be a universal reference. `const T&& v` is not a universal reference**

There are basically two cases where this happens:

1.  Template . The type ot `T` in `F` needs to be deduced. 
```cpp
template<typename T>
void F(T&&)
...
```
2. auto declaration. the type of `v` needs to be deduced. It can both be deduced to be `lvalue` or `rvalue` depending on the type of `v1`
```cpp
int f(...){
 auto&& v = v1;
}
``` 

## Use forward on universal reference 
The rationale is that a universal reference can be bound to both lvalue and rvalue. If you use move on an instantiation of `f` that happens to bound its universal reference parameter to a lvalue, then you will potentially have a side effect on that parameter.

```cpp
#include <iostream>
#include <vector>
#include <cmath>
#include <unordered_map>
using namespace std;

void p(string& w)
{
	cout<<__PRETTY_FUNCTION__<<": ";
	cout<<"The value of w is "<<w<<endl;
	
    //!!!!
	string s1(w);
	
	cout<<__PRETTY_FUNCTION__<<": ";
	cout<<"The value of w is "<<w<<endl;
}

void p(string&& w)
{
	//w is rvalue reference. We always call move on those.
	cout<<__PRETTY_FUNCTION__<<": ";
	cout<<"The value of w is "<<w<<endl;
	
    // !*
	string s1(std::move(w)); 
	
	cout<<__PRETTY_FUNCTION__<<": ";
	cout<<"The value of w is "<<w<<endl;
}

template<typename T>
void ff(T&& w){ 
    //v is a universal reference
	// can be both lvalue or rvalue 
	//depending on the call site
	
    cout<<__PRETTY_FUNCTION__<<": ";
	cout<<"The value of w is "<<w<<endl;
	
    //p(std::move(w)); //!!!!
    p(std::forward<T>(w)); //!!!!
    
    cout<<__PRETTY_FUNCTION__<<": ";
	cout<<"The value of w is "<<w<<endl;
}
int main(){
	string w; //!!!!
    
	cin>>w;
	cout<<__PRETTY_FUNCTION__<<": ";
	cout<<"The value of w is "<<w<<endl;
	
    ff(w); //!!!!
    
    cout<<__PRETTY_FUNCTION__<<": ";
	cout<<"The value of w is "<<w<<endl;
}
```

if in `f` you use move then the moment you call `f` with an `lvalue` you are causing a side effect on the parameter. The value of `w` in main is not valid anymore after the call to `ff`. Is this what you really want? No! What you want is to move only if you are providing `ff` with an `rvalue` otherwise you want the old gold copy mechanism to take place.
You can always force the move from an `lvalue` by casting `w` in main to an rvalue at the call site for `ff`. 

```cpp

int main(){
	string w; //!!!!
    
	cin>>w;
	cout<<__PRETTY_FUNCTION__<<": ";
	cout<<"The value of w is "<<w<<endl;
	
    //explicitely moving
    ff(std::move(w)); 
    
    cout<<__PRETTY_FUNCTION__<<": ";
	cout<<"The value of w is "<<w<<endl;
}
```


Question: what happens if we modify `ff` such that it does this?
```cpp
// p(std::forward<T>(w)); //!!!!
 p(w); 
```

The lvalue reference will be called because `w` in `f` is of type rvalue reference but  `w` itself is an  lvalue reference. so if you pass `w` around without forwarding or moving, then it will be treated as an lvalue.


## Always apply `move` when returning rvalue references
If you are returning an rvalue reference from a function, always apply move because that avoids the cost of copying into the return object.
```cpp
Matrix operator+(Matrix&& lhs, const Matrix& rhs)
{
	lhs += rhs;
	return std::move(lhs); // move lhs into
}
```

In this case lhd is an rvalue and we are returning a brand new matrix from the operator+. Thus is it efficient to move from lhs into the return object.

Same thing when you return a rvalue reference. Always apply forward before returning becuase that would cause the move only when the universal reference is bound to a rvalue.

template<typename T>
T operator+(T&& lhs, const T& rhs)
{
	lhs += rhs;
	return std::forward(lhs); // move lhs into if lhs is rvalue
}


## Do not call move on return when returning local object 

```cpp
Matrix sum(Matrix& lhs, Matrix& rhs){
	Matrix m(lhs);
	m +=rhs;
	return m;
}
```

In this case doing `return std::move(m);` would cause the return value optimization not to be deployed. 
