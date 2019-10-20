					*Code and text from the book*

## Overloading on universal references leads to the universal reference overload being called more frequently than expected
Without overloading the following code is inefficient.

```c++
#include <iostream>    
#include <set>
#include <string>

std::multiset<std::string> names;

void logAndAddToSet(const std::string& name)
{
    // do logging
    names.emplace(name);
    std::cout << __PRETTY_FUNCTION__ << std::endl;
}

int main () {
    std::string a_lvalue("lvalue");
    logAndAddToSet(a_lvalue);              //is copied to `names`
    
    logAndAddToSet(std::string("rvalue")); //temporary object is created, then is copied to `names`
    
    logAndAddToSet("string literal");      //temporary object is created, then is copied to `names`

    std::cout << "Set contains: ";
    for (auto i : names)
        std::cout << i << "-";

    std::cout << std::endl;
  return 0;
}
///Output:
void logAndAddToSet(const string&)
void logAndAddToSet(const string&)
void logAndAddToSet(const string&)
Set contains: lvalue-rvalue-string literal-

```
As we see the second and third calls are inefficient because a temporary object is created and then copied to `names`. The temporary object is then destroyed. We could, however in these cases, *move* the temporary object to `names`. 
We need a function that can take both lvalue and rvalue, i.e., a universal reference 

```c++
template<typename T>
void logAndAddToSet(T&& name)
{
    // do logging
    names.emplace(std::forward<T>(name));
    std::cout << __PRETTY_FUNCTION__ << std::endl;
}

////At call side

logAndAddToSet(a_lvalue);              //is copied to `names`
    
logAndAddToSet(std::string("rvalue")); //is moved to `names`
    
logAndAddToSet("string literal");      //create std::string in multiset 
                                       //instead of copying a temporary std::string
                                       //MINB: How to check?
////Output
void logAndAddToSet(T&&) [with T = std::__cxx11::basic_string<char>&]
void logAndAddToSet(T&&) [with T = std::__cxx11::basic_string<char>]
void logAndAddToSet(T&&) [with T = const char (&)[15]]
Set contains: lvalue-rvalue-string literal-

```
Problem shows up when we overload this universal reference with another function taking an integer. Suppose, we want to resolve naming via an index. 

```c++
void logAndAddToSet(T&& name);
void logAndAddToSet(int index);
```
At the call side, when we have the following calls, which function will be invoked ? The one takes universal reference or the one takes an integer ?

```c++
logAndAddToSet(a_lvalue);    
logAndAddToSet(std::string("rvalue"));    
logAndAddToSet("string literal");
logAndAddToSet(1);    
short s = 1;
logAndAddToSet(s);
```
The following code shows good weather case. 
```c++
#include <utility>    
#include <iostream>    
#include <set>
#include <string>

std::multiset<std::string> names;

template<typename T>
void logAndAddToSet(T&& name)
{
    // do logging
    names.emplace(std::forward<T>(name));
    std::cout << __PRETTY_FUNCTION__ << std::endl;
}

void logAndAddToSet(int index)
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
///Output
void logAndAddToSet(T&&) [with T = std::__cxx11::basic_string<char>&]
void logAndAddToSet(T&&) [with T = std::__cxx11::basic_string<char>]
void logAndAddToSet(T&&) [with T = const char (&)[15]]
void logAndAddToSet(int)
Set contains: foo-lvalue-rvalue-string literal-
```
The problem is when we call `logAndAddToSet()` with other integer than `int`, e.g., `short`, `long`, `std::size_t`.

```c++
short s = 1;    
logAndAddToSet(s);      

long l = 1;
logAndAddToSet(l);      

std::size_t sz = 1;
logAndAddToSet(sz);      
    
//Compilation log:
error: no matching function for call to 'std::__cxx11::basic_string<char>::basic_string(short int&)'
error: no matching function for call to 'std::__cxx11::basic_string<char>::basic_string(long int&)'
error: no matching function for call to 'std::__cxx11::basic_string<char>::basic_string(long unsigned int&)'

..
required from 'void logAndAddToSet(T&&) [with T = short int&]'  
required from 'void logAndAddToSet(T&&) [with T = long int&]'
required from 'void logAndAddToSet(T&&) [with T = long unsigned int&]'
    
```
The author gave a clear explanation. In case passing a function with `short`, "There are two overloads. The one taking a universal reference can deduce `T` to be `short&`, thus yielding an exact match (item 28 explains why `short&` not `short`). The overload with an `int` parameter can match the `short` argument only with a promotion. Per the normal overload resolution rules, an exact match beats a match with a promotion, so the universal reference overload is invoked." The other cases with `long` and `std::size_t` are the same.

As compliation log shows, there is no string ctor taking a `short int&`, `long int&` or `long unsigned int&` to create a string, inside `multiset::emplace()`.

"Functions taking universal references are the greediest functions in C++. They instantiate to create exact matches for almost any type of argument (except few types in Item 30). The universal overload vacuums up far more argument types than expected."

## Perfect-forwarding constructors
*Credit to ISLA*

The same problem happens when you overload on constructors taking universal reference. Consider example:

```c++
#include <string>
#include <iostream>
#include <utility>

std::string resolveName(int index)
{
    return "Foo";
}
class Person
{
    public:
    	template<typename T>
	explicit Person(T&& name): name_(std::forward<T>(name)) 
	{
		std::cout<< __PRETTY_FUNCTION__ <<std::endl;
	}
	explicit Person(int index): name_(resolveName(index)) 
	{
		std::cout<< __PRETTY_FUNCTION__ <<std::endl;
	}
		
    private:
    	std::string name_;
	
};

int main () 
{
    Person p("ISLA");
    int i = 1000;
    Person p2(i);    
}
///Output:
Person::Person(T&&) [with T = const char (&)[5]]
Person::Person(int)
```
The same problem we have above with other types than integer, `short`, `long`, `std::size_t`

```c++
    short s = 12;	
    Person p3(s);	

    ///Compilation error:    
In instantiation of 'Person::Person(T&&) [with T = short int&]':
...
explicit Person(T&& name): name_(std::forward<T>(name)) {
...
no known conversion for argument 1 from 'short int' to 'std::__cxx11::basic_string<char>&&'
```

Two more problems:
* Perfect forwarding ctor also beats default copy ctor
* Inheritance

### Perfect-forwarding ctor vs. copy ctor
Ctor with universal reference also beats copy ctor. This has nothing todo with overloading universal reference but rather a problematic when you have a perfrect-forwarding ctor. 

In the above example, default copy ctor is generated.
```c++
  public: 
  // inline Person(const Person &) = default;
```
But it is never used because perfect-forwarding ctor is a better match for the following call:
```c++

int main () 
{
	Person p("ISLA");    
	Person new_person(p);
}

//Compilation error:
In instantiation of 'Person::Person(T&&) [with T = Person&]':
   required from here
no matching function for call to 'std::__cxx11::basic_string<char>::basic_string(Person&)'
   explicit Person(T&& name): name_(std::forward<T>(name)) {
                                                         ^
```
Ther is no string constructor taking `Person&` as a parameter.

But when we declare `p` to be a `const`, then default copy constructor is invoked.
```c++
int main () 
{
    const Person p("ISLA");  // call overloading on universal reference
    Person new_person(p);    // call default copy ctor because of better match 
    		             // copy ctor: inline Person(const Person &)
}
```
### Perfect-forwarding ctor with inheritance
The problem is more serious when we involve inheritance into our code. 

```c++
class SuperMan : public Person
{
    public:
        SuperMan(const SuperMan& rhs) : Person(rhs) {};        //invoke Person' perfect-forwading ctor
        SuperMan(SuperMan&& rhs) : Person(std::move(rhs)) {};  //invoke Person' perfect-forwading ctor 
};
//Compilation error for SuperMan copy ctor:
In instantiation of 'Person::Person(T&&) [with T = const SuperMan&]':
required from here
error: no matching function for call to 'std::__cxx11::basic_string<char>::basic_string(const SuperMan&)'
   13 |   explicit Person(T&& name): name_(std::forward<T>(name)) {
      |                                                         ^
      .....
      
```
The code will not compile because no `string` ctor takes `SuperMan` as argument.
