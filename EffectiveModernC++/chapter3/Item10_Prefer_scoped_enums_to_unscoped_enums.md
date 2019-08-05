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


    
