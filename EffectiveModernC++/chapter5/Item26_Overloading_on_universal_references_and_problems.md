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
The author gave a clear explanation: "There are two overloads. The one taking a universal reference can deduce `T` to be `short&`, thus yielding an exact match (item 28 explains why `short&` not `short`). The overload with an `int` parameter can match the `short` argument only with a promotion. Per the normal overload resolution rules, an exact match beats a match with a promotion, so the universal reference overload is invoked."

As compliation log shows, there is no string ctor taking a `short int&`, `long int&` or `long unsigned int&` to create a string, inside `multiset::emplace()`.

"Functions taking universal references are the greediest functions in C++. They instantiate to create exact matches for almost any type of argument (except few types in Item 30). The universal overload vacuums up far more argument types than expected."

## Perfect-forwarding constructors
*Credit to ISLA*

```c++
class Person
{
	public:
		template<typename T>
		explicit person(T && name):_name(std::forward<T>(name)) {
			std::cout<<"The template"<<std::endl;
		}
		
		explicit person(int ind):_name("FOO") {
			std::cout<<"The specific int"<<std::endl;
		}
		
	private:
		std::string _name;
	
};
class SpecialPerson : public Person 
{
};

int main () 
{
	Person p("ISLA");
	//const Person P_c("ISLA");
	int i = 1000
	short s = 12;
	Person p2(i);
	Person p3(s);
	
	Person my_new(p);
}
```
