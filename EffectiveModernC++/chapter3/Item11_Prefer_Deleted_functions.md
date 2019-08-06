# Item 11 - Prefer deleted functions to private undefined ones.

The author suggests to use deleted functions(C++11) in place of private undefined ones(C++98) for the following reasons:

* Error caught at compile time instead of link time.
* Can use it for all functions instead of only class member functions
* Can be used also in template instantiations.

## Catch error at Compile Time

To make a copy assignment operator to be not usable, C++98 declares those functions as private. If any friends or member functions, try to
access this operator, then the linking will fail.
```c++
//C++98
template <class charT, class traits = char_traits<charT> >
class basic_ios : public ios_base {
public:
…
private:
basic_ios(const basic_ios& ); // not defined
basic_ios& operator=(const basic_ios&); // not defined
};
```

In C++11, this is acheived by using `= delete`. In this case, the compilation will fail. This is an improvement as we catch the problem during
compilation and not linking like in the case of C++98.

```c++

template <class charT, class traits = char_traits<charT> >
class basic_ios : public ios_base {
public:
…
basic_ios(const basic_ios& ) = delete;
basic_ios& operator=(const basic_ios&) = delete;
…
};

```

## Delete all functions

By convention, in C++11, deleted functions are declared `public` as the author claims that it leads to better error messages in most compilers.

Advantage of using delete function is that any function can be deleted.

```c++
//non-sensical
if (isLucky('a')) … 
if (isLucky(true)) … 
if (isLucky(3.5)) … 
```

The above lines of code may compile even though they don't make sense  because C++ tries to convert everything to `int`. To prevent this, we can
use delete.

```
bool isLucky(int number); // original function
bool isLucky(char) = delete; // reject chars
bool isLucky(bool) = delete; // reject bools
bool isLucky(double) = delete; // reject doubles and
// floats
if (isLucky('a')) … // error! call to deleted function
if (isLucky(true)) … // error!
if (isLucky(3.5f)) … // error!
```

## Use in template instantiations
You can also use it to prevent template instantiations to un-intended types(`void*` and `char*` for instance).

```c++

template<typename T>
void processPointer(T* ptr);

template<>
void processPointer<void>(void*) = delete;
template<>
void processPointer<char>(char*) = delete;
template<>
void processPointer<const void>(const void*) = delete;
template<>
void processPointer<const char>(const char*) = delete;

```

If we want to do it for function templates inside a class, in C++ 98, you would do it the following way.
```c++
//C++98 way,
class Widget {
public:
…
template<typename T>
void processPointer(T* ptr)
{ … }
private:
template<> // error!
void processPointer<void>(void*);
};
```
The above code snippet does not compile because different access level for specialization not allowed.

In C++11, you would do it the follwing way:

```c++
class Widget {
public:
…
template<typename T>
void processPointer(T* ptr)
{ … }
…
};
template<> // still
void Widget::processPointer<void>(void*) = delete; // public,
// but
// deleted
```

In this case, the compiler does not throw an error because in this case, you don't need a different access level for a specialization.

