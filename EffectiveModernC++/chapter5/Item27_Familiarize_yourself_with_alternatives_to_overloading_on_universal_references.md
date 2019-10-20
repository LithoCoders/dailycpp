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
