Given a function template like this:
```c++
template<typename T>
void f(ParamType param)
```
At call side, when calling f(x) compiler will deduce type for:
* T
* ParamType

Note that T and ParamType can be different (due to const, volatile , & or * specifier)

# Case 1: ParamType is a reference or a pointer
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
Output from C++ Insights shows no supprise when ParamType is deduced to T
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
