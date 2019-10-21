As in Item 26, we see problems with overloading on universal references. To fix the problem, we should not abandon overloading or universal reference. 

# Using Tag dispatch
Basically, this approach weaken overloading on universal reference by having another parameter which is not universal reference. This parameter relies on `T`. By this, universal reference parameter no longer produces an exact match.

```c+++
template<typename T>
void logAndAdd(T&& name)
{
	auto now = std::chrono::system_clock::now();
	log(now, "logAndAdd");
	names.emplace(std::forward<T>(name));

	// if type is int we will be doing logAndAdd(int idx)
}

void logAndAdd(int idx) 
{
	auto now = std::chrono::system_clock::now();
	log(now, "logAndAdd");
	names.emplace(nameFromIdx(idx));
}

```

/* ---------------------------------------------------- */
cout << is_integral<char>() << endl; 
cout << is_integral<int>() << endl; 
cout << is_integral<uint32_t>() << endl; 
cout << is_integral<uint64_t>() << endl; 
cout << is_integral<double>() << endl; 
cout << is_integral<float>() << endl; 

cout << is_integral<char&>() << endl; 
cout << is_integral<int&>() << endl; 
cout << is_integral<uint32_t&>() << endl; 
cout << is_integral<uint64_t&>() << endl; 
cout << is_integral<double&>() << endl;
cout << is_integral<float&>() << endl; 

cout << is_integral<remove_reference<int&>::type>() << endl; 

/* ---------------------------------------------------- */


// tag dispatch
template<typename T> 
void logAndAddImpl(T&& name, std::false_type) 
{ 
	// T is double 
    names.emplace(std::forward<T>(name));
}

void logAndAddImpl(int idx, std::true_type)
{ 
	logAndAdd(nameFromIdx(idx)); 
}

template<typename T>
void logAndAdd(T&& name)
{
    logAndAddImpl(forward<T>(name), 
                  is_integral<typename remove_reference<T>::type>());
}
	
	
```c++
#include <utility>    
#include <iostream>    
#include <set>
#include <string>

std::multiset<std::string> names;

template<typename T>
void logAndAddToSet(T&& name)
{
    logAndAddToSetImpl(forward<T>(name), 
                  is_integral<typename remove_reference<T>::type>());
}

template<typename T>
void logAndAddToSetImpl(T&& name, std::false_type)
{
    // do logging
    names.emplace(std::forward<T>(name));
    std::cout << __PRETTY_FUNCTION__ << std::endl;
}

void logAndAddToSetImpl(int index, std::true_type)
{
    // do logging
    //find name from index
    std::string a = "foo";
    names.emplace(a);
    std::cout << __PRETTY_FUNCTION__ << std::endl;
}

int main () {
    std::string a_lvalue("lvalue");
    logAndAddToSet(a_lvalue);
    
    logAndAddToSet(std::string("rvalue"));
    
    logAndAddToSet("string literal");

    int i = 1;
    logAndAddToSet(i);   
    
    std::cout << "Set contains: ";
    for (auto i : names)
        std::cout << i << "-";

    std::cout << std::endl;
    return 0;
}

```

/* ---------------------------------------------------- */

# Constraining templates that take universal references

`std::enable_if` gives you a way to force compilers to behave as though the template did not exist. Let's take a look at the boilerplate 
code for `std::enable_if` used in constructors for a class.

```c++
class Person {
public:
template<typename T,
typename = typename std::enable_if<condition>::type>
explicit Person(T&& n);
};
```
To specify the condition that the type `T` is not same as the person, you can use the type trait `std::is_same`. However, `std::is_same`
would evaluate to false if the type is of `T&`, even though that is not the intention. So, `Person` and `Person&` would not be
considered as the same type by `std::is_same`. To get rid of this, we can use the other type trait `std::decay`. `std::decay` would 
strip the type of references, const and volatiles.

```c++
class Person {
public:
template<typename T,
typename = typename std::enable_if<!std::is_same<Person,
typename std::decay<T>::type>::value>::type
>
explicit Person(T&& n);
…
};
```
Suppose that we have a class deriving from `Person` and it forwards it's arguments to the base class, std::is_same would not work because the base class and the derived class would not be the same.

```c++
class SpecialPerson: public Person {
public:
SpecialPerson(const SpecialPerson& rhs) // copy ctor; calls
: Person(rhs) // base class
{ … } // forwarding ctor!
SpecialPerson(SpecialPerson&& rhs) // move ctor; calls
: Person(std::move(rhs)) // base class
{ … } // forwarding ctor!
…
};
```

To overcome this, we need to use another type trait `std::is_base_of`. std::is_base_of<T1, T2>::value is true if T2 is derived from T1 or if T2 is same as T1.

```c++
class Person {
public:
template<typename T,
typename = typename std::enable_if<!std::is_base_of<Person,typename std::decay<T>::type>::value>::type
>
explicit Person(T&& n);
…
};
```
In C++14, you can get rid of typename keyword by using `std::enable_if_t` and `std::decay_t` as shown below:

```c++
class Person { // C++14
public:
template<
typename T,
typename = std::enable_if_t< // less code here
!std::is_base_of<Person,
std::decay_t<T> // and here
>::value
> // and here
>
explicit Person(T&& n);
…
};
```
In addition, if we want to disable the constructor for integral types, you can use `std::is_integral` in addition.
```c++
class Person {
public:
template<
typename T,
typename = std::enable_if_t<
!std::is_base_of<Person, std::decay_t<T>>::value
&&
!std::is_integral<std::remove_reference_t<T>>::value
>
>
explicit Person(T&& n) // ctor for std::strings and
: name(std::forward<T>(n)) // args convertible to
{ … } // std::strings
explicit Person(int idx) // ctor for integral args
: name(nameFromIdx(idx))
{ … }
… // copy and move ctors, etc.
private:
std::string name;
};
```

Another type trait that can be used to limit what is passed to the constructor is `std::is_constructible<T1,T2>`. It can be used to 
determinte whether T1 can be constructed from T2.

```c++
class Person {
public:
template< // as before
typename T,
typename = std::enable_if_t<
!std::is_base_of<Person, std::decay_t<T>>::value
&&
!std::is_integral<std::remove_reference_t<T>>::value
>
>
explicit Person(T&& n)
: name(std::forward<T>(n))
{
// assert that a std::string can be created from a T object
static_assert(
std::is_constructible<std::string, T>::value,
"Parameter n can't be used to construct a std::string"
);
… // the usual ctor work goes here
}
… // remainder of Person class (as before)
};
```

