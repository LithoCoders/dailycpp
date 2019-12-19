```c++

#include <cstdio>
#include <unordered_map>
#include <list>
#include <memory> //for unique_ptr

typedef
std::unique_ptr<std::unordered_map<std::string, std::string>> UPtrMapSS;

using UPtrMapSS =
std::unique_ptr<std::unordered_map<std::string, std::string>>;

template <typename T>
class MyAlloc {
};
class tst {
};
//Clarity
/*
// FP is a synonym for a pointer to a function taking an int and a const std::string& and returning nothing
typedef void (*FP)(int, const std::string&); // typedef same meaning as above
using FP = void (*)(int, const std::string&); // alias declaration
*/
//Easier for template
template<typename T> 
struct MyAllocList {                      
typedef std::list<T, MyAlloc<T>> type; //MyAllocList<T>::type is synonym for std::list<T,MyAlloc<T>> ==>dependent Type
}; 

//typename is needed for declaration!
template<typename T>
class Widget { 
private: 
typename MyAllocList<T>::type list; // Widget<T> contains a MyAllocList<T> as a data member

};

template<typename T> 
using MyAllocList1 = std::list<T, MyAlloc<T>>; // MyAllocList<T> is synonym for std::list<T,MyAlloc<T>>

template<typename T>
class Widget1 {
private:
MyAllocList1<T> list; // no "typename", no "::type" as MyAllocList is an alias template
};

MyAllocList<tst>::type lw; // client code

/*
When compilers see MyAllocList<T>::type (i.e., use of the nested typedef) in the
Widget template, on the other hand, they can’t know for sure that it names a type,
because there might be a specialization of MyAllocList that they haven’t yet seen
where MyAllocList<T>::type refers to something other than a type. T
*/
class Wine { … };
template<> // MyAllocList specialization
class MyAllocList<Wine> { // for when T is Wine
private:
enum class WineType // see Item 10 for info on "enum class"
{ White, Red, Rose }; 
WineType type; // in this class, type is
… // a data member!
};

// MyAllocList<T>::type inside the Widget template would refer to a data member, not a type.

- typedefs don’t support templatization, but alias declarations do.
- Alias templates avoid the “::type” suffix and, in templates, the “typename”
prefix often required to refer to typedefs.
```
