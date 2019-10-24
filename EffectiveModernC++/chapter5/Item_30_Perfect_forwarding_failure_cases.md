# Terminology

Before we look into the details, let us be familiar with the terms author uses in his book.
- *forwarding*: one function A passes - *forward* - its parameters to another function B. Function B receives the same object C that A receives.
- *perfect forwarding*: not only object but also salient characteristics:
	* type
	* lvalueness or rvalueness
	* constness
	* volatile-ness
- *forwarding function*: it is A above
- *forwarded-to function*: it is B above
- *originally passed-in objects*: it is C above
- *by-value parameters*: Rule out perfect forwarding. Object are copies of what the original caller passes in.
Argument `i` has different memory address because it is not the same in two functions `forwardingFunc` and `forwardedToFunc`
```c++
#include<iostream> 

void forwardingFunc(int i);
void forwardedToFunc(int i);

void forwardingFunc(int i)
{
    std::cout << __PRETTY_FUNCTION__ << std::endl;
    std::cout << &i << std::endl;
    forwardedToFunc(i);
}

void forwardedToFunc(int i)
{
    std::cout << __PRETTY_FUNCTION__ << std::endl;
    std::cout << &i;
}
int main()
{
  	int i = 1;
    forwardingFunc(i);
    return 0;
}
Start

void forwardingFunc(int)
0x7ffcdfdeda2c
void forwardedToFunc(int)
0x7ffcdfdeda0c

0

Finish
```
- *pointer parameters* ?? "Pointer parameters are also ruled out, because we don't want to force callers to pass pointers." ??

# Perfect-forwarding in variadic template
Universal references normal case 
```c++
template<typename T>
void forwardedToFunc(T param)
{    
}

template<typename T>
void func(T param)
{    forwardedToFunc(std::forward<T>(param));
}
```
Universal references in variadic template. You can also perfect-forward all parameters in variadic template.
```c++
#include<iostream> 
#include<utility> 

template<typename... T>
void forwardedToFunc(T... params)
{
    bool b = ((params == 0) || ...);
    std::cout << std::boolalpha << b << std::endl; //would be nice to print out all params
}

template<typename... Ts>
void func(Ts&&... params)
{
    forwardedToFunc(std::forward<Ts>(params)...);
}

int main()
{
    func(100, 'c', "blah");
    func(0,0,0,0);
    return 0;
}
\\Output:
false
true
```
# When perfect-forwarding fails ?
There are five cases that *perfect forwarding* can go wrong but they can be generalized like this:
* forwardingFunc(exp): does something
* forwardedToFunc(exp): does something else.

## Case 1: Braced initializers
forwardingFunc() fails to deduce to std::initializer_list. Fix with `auto`
```c++
#include<iostream> 
#include<utility> 
#include<vector> 

void forwardedToFunc(const std::vector<int>& v)
{
    for (int i : v)
        std::cout << i << std::endl;
}

template<typename T>
void fwd(T param)
{
    forwardedToFunc(std::forward<T>(param));
}

int main()
{
    fwd({1, 2, 3, 4});    
    return 0;
}
prog.cc: In function 'int main()':
prog.cc:19:20: error: no matching function for call to 'fwd(<brace-enclosed initializer list>)'
   19 |    fwd({1, 2, 3, 4});
      |                    ^
prog.cc:12:6: note: candidate: 'template<class T> void fwd(T)'
   12 | void fwd(T param)
      |      ^~~
prog.cc:12:6: note:   template argument deduction/substitution failed:
prog.cc:19:20: note:   couldn't deduce template parameter 'T'
   19 |    fwd({1, 2, 3, 4});
      |                    ^
```
Item 2: type deduction for function template with brace initializers **fails**, whereas, *auto* type deduction has no problem. 
Fix this with auto type deduction to `std::initializer_list`, and then pass `std::initializer_list` to function template. 
```c++
#include<iostream> 
#include<utility> 
#include<vector> 

void forwardedToFunc(const std::vector<int>& v) //convert std::initializer_list to std::vector
{
    for (int i : v)
        std::cout << i << std::endl;
}

template<typename T>
void fwd(T param)
{
    forwardedToFunc(std::forward<T>(param));
}

int main()
{
    auto il = {1, 2, 3};
    fwd(il);    
    return 0;
}
//Output:
1
2
3
```
## Case 2: 0 or NULL as null pointer @ template
`0` or `NULL` deduced to integer, not forwarded as null pointer. Please use `nullptr`. Item 8.
```c++
#include<iostream> 
#include<utility> 
#include<vector> 

template<typename T1>
void forwardedToFunc(T1 param)
{
    std::cout << param << std::endl;
}

template<typename T>
void fwd(T param)
{
    forwardedToFunc(std::forward<T>(param));
}

int main()
{
    fwd(NULL);    //T deduced to long int
    fwd(0);       //T deduced to int
    fwd(nullptr); //correct way to perfect forward a null pointer
    return 0;
}
//Output:
0
0
nullptr
```

