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
The code fails because `sz` is not a constant. So, let's make it constant.

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

??? Just a question: why only in *integral constant expression* ? No doubles, no floats, no booleans?

Example using `constexpr` integer to start enummerators

```
#include <iostream>

int main()
{
    constexpr int start = 4;
    enum class Color {red = start, blue, green};    
    Color c = Color::green;
    std::cout << "start " << start << "\n" << "green " << static_cast<int>(c) ;    
    return 0;
}
Start

start 4
green 6

0

Finish
```

Example using `constexpr` with alignment specifiers: todo

## `const` vs. `constexpr` ?

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
## Rules: 
* `constexpr` functions can be used in contexts that demand compile-time constants. If the values of the arguments you pass to a `constexpr` function in such a context are known during compilation, the result will be computed during compilation. If any of the arguments’ values is not known during compilation, your code will be rejected. 

* When a `constexpr` function is called with one or more values that are not known during compilation, it acts like a normal function, computing its result at runtime. This means you don’t need two functions to perform the same operation, one for compile-time constants and one for all other values. The `constexpr` function does it all.

```c++
#include <array>

constexpr int pow11(int base, unsigned exp) noexcept
{
    return (exp == 0 ? 1: base * pow11(base, exp -1)); //C++11 allows no more than a 'return' statement
}

//C++14 allows more statements
constexpr int pow14(int base, unsigned exp) noexcept
{
    auto result = 1;
    for(unsigned i = 0; i < exp; ++i)
            result *= base;
    return result;
}

int main()
{
    std::array<int, pow11(2, 5)> result11;
    
    std::array<int, pow14(2, 5)> result14;
    
    return 0;
}

```
In above example, `pow11` and `pow14` return compile-time results when called with compile-time values.

## `constexpr` ctor, setter and object

```c++
#include <array>

class Point
{
    public:
        constexpr Point(double xVal = 0, double yVal = 0) noexcept : x(xVal), y(yVal) {}    
    
        constexpr double xValue() const noexcept { return x; }
        constexpr double yValue() const noexcept { return y; }
    
        void setX(double newX) noexcept { x = newX; }
        void setY(double newY) noexcept { y = newY; }
    
    private:
        double x, y;
};

constexpr Point midpoint(const Point& p1, const Point& p2) noexcept
{
    return { (p1.xValue() + p2.xValue())/2, (p1.yValue() + p2.yValue())/2 }; // interesting, no need to invoke ctor ?
}

int main()
{
    constexpr Point p1(1.2, 3.4);
    constexpr Point p2(5.6, 7.8);
    
    constexpr auto mid = midpoint(p1, p2);
    
    std::array<int, static_cast<int>(mid.xValue()*2)> amazing_array;
    
    return 0;
}
```
This shows a nice example from moving some computations done at runtime can be done at compile time.
* At compile time: constructor -> object -> setter -> midpoint function -> midpoint object
* At runtime: amazing_array

In the above example, setters can not be `constexpr` functions in C++11 because it modifies object, and they have `void` return type (`void` is not a literal type in C++11) but this is lifted up in C++14.

```c++
#include <array>

class Point
{
    public:
        constexpr Point(double xVal = 0, double yVal = 0) noexcept : x(xVal), y(yVal) {}    
    
        constexpr double xValue() const noexcept { return x; }
        constexpr double yValue() const noexcept { return y; }
    
        constexpr void setX(double newX) noexcept { x = newX; } //only C++14
        constexpr void setY(double newY) noexcept { y = newY; } //only C++14
    
    private:
        double x, y;
};

constexpr Point midpoint(const Point& p1, const Point& p2) noexcept
{
    return { (p1.xValue() + p2.xValue())/2, (p1.yValue() + p2.yValue())/2 }; // interesting, no need to invoke ctor ?
}

constexpr Point reflection(const Point& p) noexcept
{
    Point result;                   //non-const Point
    
    result.setX(-p.xValue());
    result.setY(-p.yValue());
    
    return result;                  //return a copy of it
}

int main()
{
    constexpr Point p1(1.2, 3.4);
    constexpr Point p2(5.6, 7.8);
    
    constexpr auto mid = midpoint(p1, p2);
    constexpr auto reflectedMid = reflection(mid);
    
    std::array<int, static_cast<int>(mid.xValue()*2)> amazing_array;
    
    return 0;
}
```

The strange thing is only `reflectedMid` is complained to not used, but not other points. Is it a compile-time object ?
```c++
Start

prog.cc: In function 'int main()':
prog.cc:39:20: warning: variable 'reflectedMid' set but not used [-Wunused-but-set-variable]
   39 |     constexpr auto reflectedMid = reflection(mid);
      |                    ^~~~~~~~~~~~
prog.cc:41:55: warning: unused variable 'amazing_array' [-Wunused-variable]
   41 |     std::array<int, static_cast<int>(mid.xValue()*2)> amazing_array;
      |                                                       ^~~~~~~~~~~~~

0

Finish
```
