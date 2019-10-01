# Item 22: When using the Pimpl idiom, define special member functions in the implementation file.

Pimpl = **p**ointer to **impl**ementation. 

As the name of this item mentions, when you implement pimpl idiom, all special member functions need to be put in the implementation file (cpp); header file contains only function declarations.

## Pimpl ideas (copied from text book):
* Replace data member of a class with a pointer to an implementation class or struct.
* Put the data members that used to be in the primary class into the implementation class
* Access those data member indirectly through pointer

## Why pimpl ?
 Assume we have `Widget` class has `Gaget`, normal implementation like this
 ```c++
 #include "Gaget.h"
 struct Widget
 {
  Gaget g;
 }
 ```
 Client of this Widget
 ```c++
 #include "Widget.h"
 //using Widget
 ```
As seen, *Gaget.h* is indirectly included in client code. A chain of header including is *Gaget.h* -> *Widget.h* -> client code. As *Gaget.h* is frequently changed, compiler takes more time to build whole program.
Pimpl idiom reduces the compilation time.

## How to implement pimpl ?
Pimpl idiom makes use of *incomplete type* concept, whereas a type has been declared but not defined. `size of` and `delete` to an incomplet type is NOK.
However, declaring a pointer to an *incomplete type* is OK.
```c++
//main.cpp
#include "widget.hpp"

int main()
{
   Widget w1;

   return 0;
}

//widget.hpp
#ifndef WIDGET_H__
#define WIDGET_H__

#include <memory>              //for unique_ptr

using namespace std;


class Widget
{
        struct Impl;             //incomplete type
        unique_ptr<Impl> pimpl;  //pointer to an incomplete type

public:
        Widget();
        ~Widget();
};
#endif

//widget.cpp
#include "widget.hpp"
#include <iostream>
#include <vector>
#include <memory>
#include <string>

using namespace std;

struct Widget::Impl
{
    std::vector<int> data;
    std::string id = "";
};

Widget::Widget() : pimpl(make_unique<Impl>()){}
Widget::~Widget()
{
    cout << __PRETTY_FUNCTION__ << endl;
}

//Makefile
all: main widgetlib

main: widget.o
        g++ -std=c++14 -Wall main.cpp -o main.out widget.o

widgetlib: widget.cpp widget.hpp
        g++ -std=c++14 -Wall widget.cpp -c

clean:
        rm -f widget.o main.out

```
As we see, `widget.hpp` no longer includes Gaget headers (i.e., `vector` and `string` - just for example purpose. It can can be any user define type)
Client code `main.cpp` includes `widget.hpp`. 
If you remove destructor, compiler will try to generate one and unique pointer complains does not know `sizeof` incomplete type `Impl` struct to free this resource.

```c++
unique_ptr.h:76:22: error: invalid application of ‘sizeof’ to incomplete type ‘Widget::Impl’
  static_assert(sizeof(_Tp)>0,
```
Following is a complete example of implementing pimpl with `std::unique_ptr`
```c++
//widget.hpp
#ifndef WIDGET_H__
#define WIDGET_H__

#include <memory>              //for unique_ptr

using namespace std;


class Widget
{
        struct Impl;             //incomplete type
        unique_ptr<Impl> pimpl;

public:
        Widget();
        ~Widget();
        Widget(Widget&& rhs);
        Widget& operator=(Widget&& rhs);
        Widget(const Widget& rhs);
        Widget& operator=(const Widget& rhs);

        void setId(std::string s);
        void getId();
};
#endif

//widget.cpp
#include "widget.hpp"
#include <iostream>
#include <vector>
#include <memory>
#include <string>

using namespace std;

struct Widget::Impl
{
    std::vector<int> data;
    std::string id = "";
};

Widget::Widget() : pimpl(make_unique<Impl>()){}
Widget::~Widget()
{
    cout << __PRETTY_FUNCTION__ << endl;
}
Widget::Widget(Widget&& rhs) = default;
Widget& Widget::operator=(Widget&& rhs) = default;
Widget::Widget(const Widget& rhs) : pimpl(nullptr)
{
   if(rhs.pimpl)
       pimpl = std::make_unique<Impl>(*rhs.pimpl);
}
Widget& Widget::operator=(const Widget& rhs)
{
   if(!rhs.pimpl)
      pimpl.reset();
   else if(!pimpl)
      pimpl = std::make_unique<Impl>(*rhs.pimpl);
   else
      *pimpl = *rhs.pimpl;
   return *this;
}
void Widget::setId(std::string s)
{
    (this->pimpl)->id = s;
}

//main.cpp
#include "widget.hpp"

int main()
{
   Widget w1;
   w1.setId("w1");
   w1.getId();

   Widget w2 = std::move(w1);
   w2.getId();

   Widget w3;
   w3 = std::move(w2);
   w3.getId();

   Widget w4(w3);
   w4.getId();

   Widget w5;
   w5 = w4;
   w5.getId();

   return 0;
}

//Makefile
all: main widgetlib

main: widget.o
        g++ -std=c++14 -Wall main.cpp -o main.out widget.o

widgetlib: widget.cpp widget.hpp
        g++ -std=c++14 -Wall widget.cpp -c

clean:
        rm -f widget.o main.out
```
As we learn from Item  17, when we explicitly define a destructor, move operators are not generated. Thus, we need to make it explicit to compiler to generate two default move operators.
When we define move operators, compiler will not generate copy operators. Thus, we need to define them. Copy operators are deep-copy.

Note that all these special functions declared in header but only defined in cpp file.





