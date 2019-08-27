Given a function template like this:
```c++
template<typename T>
void f(ParamType param)
```
At call side, when calling f(x) compiler will deduce type for:
* T
* ParamType

Note that T and ParamType can be different (due to const, volatile , & or * specifier)

# Case 1: ParamType is a reference or a pointer but not a universal reference
Given a function template with ParamType is a reference: 
```c++
template<typename T>
void f(T & param);  // ParamType is T &
```
At call side:
```c++
int x = 27;  
f(x);	         // T: int, ParamType: int &    
```
At call side:
```c++
const int cx = x; 
f(cx);              // T: const int, ParamType: const int &
```
At call side:
```c++
const int & rx = x; 
f(rx);              // T: const int, ParamType: const int &
                    // Note that T is not a reference due to patern matching
```
Putting all together, output on C++Insights shows compiler deduce T and ParamType in two ways. 
```c++
template<typename T>
void func(T& param)
{
  //do a thing
}

#ifdef INSIGHTS_USE_TEMPLATE
template<>
void func<int>(int & param)   //for f(x)
{
}
#endif


#ifdef INSIGHTS_USE_TEMPLATE
template<>
void func<const int>(const int & param) //for f(cx) and f(rx)
{
}
#endif


int main()
{
  int x = 1;
  func(x);
  const int cx = 2;
  func(cx);
  const int & rx = 3;
  func(rx);
  return 1;
}

```
Given:
```c++
template<typename T>
void f(const T & param);   // ParamType is const T &
```
```c++
f(x);               // T: int, ParamType: const int &

f(cx);              // T: int, ParamType: const int &

f(rx);              // T: int, ParamType: const int &
```

Output on C++ Insights shows compiler always deduce to one form:

```c++
template<typename T>
void func(const T& param)
{
  //do a thing
}

#ifdef INSIGHTS_USE_TEMPLATE
template<>
void func<int>(const int & param)   // T: int, ParamType: const int & for all three cases
{
}
#endif


int main()
{
  int x = 1;
  func(x);
  const int cx = 2;
  func(cx);
  const int & rx = 3;
  func(rx);
  return 1;
}

```

Given:
```c++
template<typename T>
void f(T * param);  // ParamType is T*
```
```c++
int x = 27; 
f(&x);              // T: int, ParamType: int *

const int * px = &x; 
f(px);              // T: const int, ParamType: const int *
```
Output on C++ Insight:

```c++
template<typename T>
void func(T* param)
{
  //do a thing
}

#ifdef INSIGHTS_USE_TEMPLATE
template<>
void func<int>(int * param)  // for f(&x) above
{
}
#endif


#ifdef INSIGHTS_USE_TEMPLATE
template<>
void func<const int>(const int * param) // for f(px) above
{
}
#endif


int main()
{
  int x = 1;
  func(&x);
  const int * px = &x;
  func(px);
  return 1;
}

```
RULEs are:
1. If expr’s type is a reference, ignore the reference part
2. Then pattern-match expr’s type against ParamType to determine T


# Case 2: ParamType is universal reference
```c++
template<typename T>
void f(T && param); // ParamType is T&&
```
At call side:
```c++
int x = 27; f(x);   // T: int &, ParamType: int &

const int cx = x; 
f(cx);              // T: const int &, ParamType: const int &

const int & rx = x; 
f(rx);              // T: const int &, ParamType: const int &

f(27);              // T: int, ParamType: int &&
```
Output from CppInsight
```c++
#ifdef INSIGHTS_USE_TEMPLATE
template<>
void func<int &>(int & param) // for f(x)  - deduced to be lvalue reference
{
}
#endif

#ifdef INSIGHTS_USE_TEMPLATE
template<>
void func<const int &>(const int & param) // for f(cx) and f(rx) - deduced to be lvalue reference
{
}
#endif

template<>
void func<int>(int && param)  // for f(27) - rvalue, pattern-matching
{
}
#endif
```

Rules are:
1. If expr is an lvalue, both T and ParamType are deduced to be lvalue references. 
2. If expr is an rvalue, rules from case 1 apply

# Case 3: ParamType is neither a pointer nor a reference
```c++
template<typename T>
void func(T param)   //ParamType is T
{
  //do a thing
}

int main()
{
  int x = 1;
  func(x);
  
  const int cx = 2;
  func(cx);
  
  const int& rx = 3;
  func(rx);
}
```
Output from C++ Insights:
```c++
template<typename T>
void func(T param)
{
  //do a thing
}

/* First instantiated from: insights.cpp:10 */
#ifdef INSIGHTS_USE_TEMPLATE
template<>
void func<int>(int param)
{
}
#endif


int main()
{
  int x = 1;
  func(x);
  const int cx = 2;
  func(cx);
  const int & rx = 3;
  func(rx);
}
```
Rules are:
1. If expr’s type is a reference, ignore the reference part. 
2. After that, if expr is const, ignore that too. If it is volatile, also ignore that.

## Array arguments & function arguments
In this case, we consider function template with ParamType is an array/ reference to an array

```c++

template<typename T>
void func(T param)
{
  //do a thing
}

template<typename T>
void func2(T& param)
{
  //do other thing
}

int main()
{
  const char myname[] = "ABC DEF";
  func(myname);
  
  func2(myname);
}

```
An array decays into a pointer to its first element

```c++

template<>
void func<const char *>(const char * param)
{
}

template<>
void func2<char const[8]>(char const (&param)[8])
{
}

```
We consider function template with ParamType is a function/ reference to a function

```c++
template<typename T>
void func(T param)
{
  //do a thing
}

template<typename T>
void func2(T& param)
{
  //do other thing
}

int makeMagics(double d);

int main()
{
  
  func(makeMagics);
  
  func2(makeMagics);
}
```

Function types can decay into function pointers. Output from C++ Insight:

```c++
template<>
void func<int (*)(double)>(int (*param)(double))
{
}

template<>
void func2<int (double)>(int (&param)(double))
{
}
```

More on type deduction with function template taking array as parameter

```c++
template<class T>
void f(T param[])
```
compiles is that there is no such a thing as array function parameters in C++. So that is equivalent to

```c++
template<class T>
void f(T* param)
```

`const char name[] = "Hello world";`
When you call `f(name)`, name is convertible to char* hence the `param` is deduced to be `char*` because an array type is trated like a pointer.

But the niche case is when we can declare reference to array (and yes that is possible). Only in that case the type is deduced to be `const char[]`

Example:

```c++

#include <cstdio>
#include<iostream>
using namespace std;

template<typename T>
void f_ptr(T* param){
  return; 
}

template<typename T>
void f_array(T param[]){
  return; 
}


template<typename T>
void f_reference(T& param){
  return; 
}

int main()
{
	const char name[] = "hello world";
  f_array(name);
  f_reference(name);
  f_ptr(name);
  return 0;
}
```
becomes

```c++
#include <cstdio>
#include<iostream>
using namespace std;

template<typename T>
void f_ptr(T* param){
  return; 
}

/* First instantiated from: insights.cpp:26 */
template<>
void f_ptr<const char>(const char * param)
{
  return;
}



template<typename T>
void f_array(T param[]){
  return; 
}

/* First instantiated from: insights.cpp:24 */

template<>
void f_array<const char>(const char * param)
{
  return;
}




template<typename T>
void f_reference(T& param){
  return; 
}

/* First instantiated from: insights.cpp:25 */

template<>
void f_reference<char const[12]>(char const (&param)[12])
{
  return;
}



int main()
{
  const char name[12] = "hello world";
  f_array(name);
  f_reference(name);
  f_ptr(name);
  return 0;
}
```

