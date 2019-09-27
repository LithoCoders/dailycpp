# Item 21 - Prefer `std::make_unique` and `std::make_shared` to direct use of `new`

Note: Contains snippets from Effective Mordern C++ by Scott Meyers

`std::make_shared` is a part of C++11, but `std::make_unique` is part of C++14 and not C++11.

Implementing `std::make_unique` in C++11 is pretty striaght forward as shown below:

```c++
template<typename T, typename... Ts>
std::unique_ptr<T> make_unique(Ts&&... params)
{
return std::unique_ptr<T>(new T(std::forward<Ts>(params)...));
}
```
However note that the above implementation does not support custom deleters or arrays. There are a total of three make functions that take
in arguments, forward them to the constructor of a new object and return a smart pointer to that object. The other make object is
`std::allocate_shared`. It is same as `std::make_shared` except that it takes an allocated object as an input.

## Why should you prefer make functions

Have a look at the code snippet below:

```c++
auto upw1(std::make_unique<Widget>()); // with make func
std::unique_ptr<Widget> upw2(new Widget); // without make func
```

If you look at the `new` version, it repeats the type to be created multiple times, but that is not the case with the make version. So, 
in short it prevents code duplication.

The second reason is to do with exception safety. Suppose you have a function `computePriority()` that is called by the `new` version of 
`processWidget`.
```
void processWidget(std::shared_ptr<Widget> spw, int priority);
processWidget(std::shared_ptr<Widget>(new Widget), // potential
computePriority()); // resource
// leak!
```
As you can see above, the `new` version could lead to potential memory leaks in your code. Why does this happen? The following things 
need to happen if the `new` version of `processWidget` can be executed: `new Widget` needs to be evaluated, the shared pointer needs to 
be constructed and `processWidget` needs to be run. `new Widget` needs to be called before the shared pointer can be constructed, but there
is a possibility that the compiler may generate object code such that `computePriority` is called before the constructor. If an 
exception is thrown by the compiler during the execution of `computePriority`, then there would be a memory leak because
the constructor to the shared pointer has not been called yet. Using the implementation below would prevent such memory leaks:
```c++
processWidget(std::make_shared<Widget>(), // no potential
computePriority()); // resource leak
```
The advantage of using the snippet above is that `make_shared` or `computePriority` could be called first but even if an exception is
thrown by `computePriority`, we can be sure that the memory is deallocated or not allocated in the first place. The same is the case
with `std::make_unique`.



