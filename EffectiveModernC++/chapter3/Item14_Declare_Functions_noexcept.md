# Item 14 - Declare functions `noexcept` if they won't emit exceptions

This item is not very useful. I really don't understand the point the author is trying to make.

The author claims that `noexcept` is useful in the following ways:
1. It could provide info that is useful for the caller.
2. It allows the compiler to generate better object code. 


## std::vector
`push_back` member function of `std::vector` is exception safe. In C++11, move has been introduced. In C++98, when a vector's size is
equal to it's capacit, push_back allocates new memory and transfers all existing chunks into new memory. This allowed it to be 
exception safe because if it fails during copy, the vector remains unchanged. In C++11, std::vector::push_back takes advantage of this
“move if you can, but copy if you must” strategy.

## std::swap

```c++
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

```

The code snippet declares `noexcept` even though `setup` and `cleanup` is not declared `noexcept`. `setup` and `cleanup` could be 
legacy code that ex

```c++
void setup(); // functions defined elsewhere
void cleanup();
void doWork() noexcept
{
setup(); // set up work to be done
… // do the actual work
cleanup(); // perform cleanup actions
}

```
Be careful about usage of `noexcept` because the callers may depend on this behavior.
