```c++
template<typename T>
void f(ParamType param)
```
At call side, when calling f(x) compiler will deduce type for:
* T
* ParamType

Note that T and ParamType can be different (due to const, volatile etc)


# Case 1: ParamType is reference or pointer
Given: 
```c++
template<typename T>
void f(T & param);
```
At call side:
```c++
int x = 27;  
f(x);	         // T: int, ParamType: int &    
```
At call side:
```c++
const int cx = x; 
f(cx);            // T: const int, ParamType: const int &
```
At call side:
```c++
const int & rx = x; 
f(rx);              // T: const int, ParamType: const int &
```

Given:
```c++
template<typename T>
void f(const T & param);
```
```c++
f(x);               // T: int, ParamType: const int &

f(cx);              // T: int, ParamType: const int &

f(rx);              // T: int, ParamType: const int &
```

Given:
```c++
template<typename T>
void f(T * param);
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
void f(T && param);
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
