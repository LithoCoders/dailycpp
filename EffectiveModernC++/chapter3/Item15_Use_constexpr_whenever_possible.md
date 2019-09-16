There are two types of `constexpr` in C++11, one for objects, the other one for functions.
The `constexpr` objects are constants and known during compilation. Whereas, `constexpr` functions are a bit tricky but there are rules.

# `constexpr` objects
Let us try to create some `constexpr` variables

```c++
int main()
{
    int sz = 10;
    constexpr auto arraySize = sz;
       
    return 0;
}
Start

prog.cc: In function 'int main()':
prog.cc:5:32: error: the value of 'sz' is not usable in a constant expression
    5 |     constexpr auto arraySize = sz;
      |                                ^~
prog.cc:4:9: note: 'int sz' is not const
    4 |     int sz = 10;
      |         ^~
prog.cc:5:20: warning: unused variable 'arraySize' [-Wunused-variable]
    5 |     constexpr auto arraySize = sz;
      |                    ^~~~~~~~~

1

Finish
```
The code fails because `sz` is not a constant. So, let make it constant.

```c++
#include <array>
int main()
{
    const int sz = 10;
    constexpr auto arraySize = sz;

    std::array<int, arraySize> data;
    
    return 0;
}
```
It looks like a `constexpr` is extra thing added to a constant variable. That extra thing seems to be 'known during complation' so that there is some optimization from compiler.

## Why `constexpr` ?

Objects are constants and known during compilation are **privileged** because of two reasons:
1. They can be placed in read-only memory, an advantage in embeded programming.
2. They can be used in integral constant expression such as:
  * arraysize
  * integral template arguments
  * enummerator values
  * alignment specifiers
  * and more

Example for enummerators, alignment specifiers: todo

## `const` vs. `constexpr`

`constexpr` objects are `const`s but not the other way around. It means, `constexpr` objects are constant and known during complation but `const` can be not known during compilation

The previous example but `arraySize` is a `const`
```c++
int main()
{
    int sz;
    const auto arraySize = sz;    
    return 0;
}
```
This compiles OK-ish but two variables `sz` and `arraySize` have no initial values, take `0`s

Certainly, you can not use `arraySize` to specify size of an array
```c++
#include <array>
int main()
{
    int sz;
    const auto arraySize = sz;    
    std::array<int, arraySize> data;
    return 0;
}

Start

prog.cc: In function 'int main()':
prog.cc:6:30: error: the value of 'arraySize' is not usable in a constant expression
    6 |     std::array<int, arraySize> data;
      |                              ^
prog.cc:5:16: note: 'arraySize' was not initialized with a constant expression
    5 |     const auto arraySize = sz;
      |                ^~~~~~~~~
prog.cc:6:30: note: in template argument for type 'long unsigned int'
    6 |     std::array<int, arraySize> data;
      |                              ^
prog.cc:6:32: warning: unused variable 'data' [-Wunused-variable]
    6 |     std::array<int, arraySize> data;
      |                                ^~~~

1

Finish
```

# `constexpr` functions



