C++98 style enums, called unscoped enums vs. C++11 style enums, scoped enums
# Advantage 1: Scoping
```C++
    enum Color { red, blue, green }; //C++98 unscoped enum
    auto blue = 10.0;
    
    prog.cc: In function 'int main()':
    prog.cc:9:10: error: 'auto blue' redeclared as different kind of entity
    9 |     auto blue = 10.0;
      |          ^~~~
    prog.cc:8:23: note: previous declaration 'main()::Color blue'
    8 |     enum Color { red, blue, green };
      | 
```
C++11 enumerators have scope inside braces
```C++
    enum class Color { red, blue, green }; //C++11 scoped enum
    auto blue = 10.0;
    Color c = Color::blue;
    std::cout << blue << std::endl;
    //Using c
    
    Start
    10
    1
    Finish
```
# Advantage 2: C++11 enumerators are much more strongly typed
```C++
void func(std::size_t x)
{
    //do something
    std::cout << x ;
    return;
}

int main()
{
    enum Color { red, blue, green };
    Color c = blue;
    
    if( c < 10.0) { //implicitly convert to doulbe
        func(c); //implicitly convert to size_t
    }
    
    return 1;
}
```
C++11 enumerators are strongly typed
```C++11
void func(std::size_t x)
{
    //do something
    std::cout << x ;
    return;
}

int main()
{
    enum class Color { red, blue, green };
    Color c = Color::blue;
    
    if( c < 10.0) {
        func(c);
    }
    
    return 1;
}
prog.cc: In function 'int main()':
prog.cc:14:11: error: no match for 'operator<' (operand types are 'main()::Color' and 'double')
   14 |     if( c < 10.0) {
      |         ~ ^ ~~~~
      |         |   |
      |         |   double
      |         main()::Color
prog.cc:15:14: error: cannot convert 'main()::Color' to 'std::size_t' {aka 'long unsigned int'}
   15 |         func(c);
      |              ^
      |              |
      |              main()::Color
prog.cc:2:23: note:   initializing argument 1 of 'void func(std::size_t)'
    2 | void func(std::size_t x)
      | ```
Fix with static_cast
```C++
int main()
{
    enum class Color { red, blue, green };
    Color c = Color::blue;
    
    if( static_cast<double>(c) < 10.0) {
        func(static_cast<size_t>(c));
    }
    
    return 1;
}
```
# Advantage 3: C++11 enumerators may be forward-declared
Why forward-declaration ?
Changes in C++11 enum do not need to recompile code where headers are included but only to header file itself and functions using changed enumerators

```C++
enum Color; //Error! C++98 does not allow forward-declaration
prog.cc: In function 'int main()':
prog.cc:6:10: error: use of enum 'Color' without previous declaration
    6 |     enum Color;
      |          ^~~~~
```
```C++
enum class Color; // OK. C++11
```

forward-declaration for C++98 style enum when explicitly specify the underlying type of enumerators

```C++
enum Color: std::uint8_t;

```
#Underlying types

