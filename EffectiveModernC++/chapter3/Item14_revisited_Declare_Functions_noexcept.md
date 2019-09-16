Item 14 - Declare functions noexcept if they won't emit exceptions

The author claims that noexcept is useful in the following ways:

- It could provide info that is useful for the caller.
- It allows the compiler to generate better object code.
```c++
int f(int x) throw(); // no exceptions from f: C++98 style 
int f(int x) noexcept; // no exceptions from f: C++11 style 
```
If, at runtime, an exception leaves f, f’s exception specification is violated. With the C++98 exception specification, 
the call stack is unwound to f’s caller, and, after some actions not relevant here, program execution is terminated. 
With the C++11 exception specification, runtime behavior is slightly different: the stack is only possibly unwound before program execution is terminated.

----------------------------
In a noexcept function, optimizers need
not keep the runtime stack in an unwindable state if an exception would propagate
out of the function, nor must they ensure that objects in a noexcept function are
destroyed in the inverse order of construction should an exception leave the function.
------------------------------------

```c++
RetType function(params) noexcept; // most optimizable
RetType function(params) throw(); // less optimizable
RetType function(params); // less optimizable
```

The Move Operation
push_back member function of std::vector is exception safe. In C++11, move has been introduced. In C++98, when a vector's size is equal to it's capacit, push_back allocates new memory and transfers all existing chunks into new memory. This allowed it to be exception safe because if it fails during copy, the vector remains unchanged. In C++11, std::vector::push_back takes advantage of this “move if you can, but copy if you must” strategy.

std::swap
The code below describes how conditional noexcept is implemented in std::swap.

template <class T, size_t N>
void swap(T (&a)[N], // see
T (&b)[N]) noexcept(noexcept(swap(*a, *b))); // below
template <class T1, class T2>
struct pair {
…
void swap(pair& p) noexcept(noexcept(swap(first, p.first)) &&
noexcept(swap(second, p.second)));
…
};
Legacy
The code snippet declares noexcept even though setup and cleanup is not declared noexcept. setup and cleanup could be legacy code that is written in C.

void setup(); // functions defined elsewhere
void cleanup();
void doWork() noexcept
{
setup(); // set up work to be done
… // do the actual work
cleanup(); // perform cleanup actions
}
Be careful about usage of noexcept because the callers may depend on this behavior.

By default, all memory deallocation functions and all destructors.
The only time a destructor
is not implicitly noexcept is when a data member of the class (including inherited
members and those contained inside other data members) is of a type that expressly
states that its destructor may emit exceptions
