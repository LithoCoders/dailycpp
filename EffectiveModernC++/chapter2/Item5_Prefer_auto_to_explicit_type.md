# Item 5: Prefer `auto` to explicit type declarations

Note: Contains code snippets from Effective Modern C++ by Scott Meyers.

Today, we we'll be discussing Item 5, which describes why the keyword `auto` should be used in place of explicit type declarations. The advantages of `auto` are as follows:

1. Avoid uninitialize variables
2. Readability 
3. Hold closures
4. Avoid *type shortcuts*
5. Prevent bugs due to unintentional type mismatches
6. Facilitate refactory

## 1. Avoid uninitialize variables
```
int x1;      // potentially uninitialized
auto x2;     // error! initializer required
auto x3 = 0; // fine, x3's value is well-defined
```

`auto` forces you to initialize the variables.

## 2. Readability
Consider an algorithm taking two iterations (begin and end) as inputs and apply something to the current value.
```
//before C++11
template<typename It> // algorithm to dwim ("do what I mean")
void dwim(It b, It e) // for all elements in range from b to e
{ 
	while (b != e) {
	typename std::iterator_traits<It>::value_type currValue = *b; // who wants to write something 
								      // like this to express the type?
	//do something with currValue...
	}
}
```
Using C++11 with `auto` make the types more readable.
```
//after C++11
template<typename It> // algorithm to dwim ("do what I mean")
void dwim(It b, It e) // for all elements in range from b to e.
{
	while (b != e) {
		auto currValue = *b; // Lot more easier.
		//do something here with currValue
	}
}
```
## 3. Hold closures
It is always a good idea to declare lambda type with `auto`. If not, you might have to use `std::function` like this:
```
std::function<bool(const std::unique_ptr<Widget>&, 
                   const std::unique_ptr<Widget>&)> derefUPLess = [](const std::unique_ptr<Widget>& p1, 
		                                                     const std::unique_ptr<Widget>& p2) 
			                                            { return *p1 < *p2; }; 
```
The above lambda is to compare two `Widget` objects pointed by `std::unique_ptrs` pointers.
The author argues that using `std::function` takes more memory and often slower compared to `auto`.

With C++11, it is much shorter.
```
//C++11
auto derefUPLess = [](const std::unique_ptr<Widget>& p1, const std::unique_ptr<Widget>& p2) 
			{ return *p1 < *p2; }; 
```
In C++14, parameters to lambda can use `auto`.
```
//C++14
auto derefLess = [](const auto& p1, const auto& p2)  // to compare two values pointed by
			{ return *p1 < *p2; };       // anything pointer-like

```
## 4. Avoid *type shortcuts*
Many developers write this
```c++
std::vector<int> vi; 
unsigned sz = vi.size(); // it is a *type shortcut*
```
`vi.size()` returns `std::vector<int>::size_type`, an unsigned integral type. Therefore, many developers *use a shortcut to type it* `unsigned`.
* On 32-bit Windows, both `std::vector<int>::size_type` and `unsigned` have the same size.
* On 64-bit Windows, `std::vector<int>::size_type` is 64 bits and `unsigned` is 32 bits.
So,
```c++
	auto sz = v.size() // evaluates to std::vector<int>::size_type sz, correct type.
	                   // prevent portability issues
```

## 5. Prevent bugs due to unintentional type mismatches

```
std::unordered_map<std::string, int> m;
for (const std::pair<std::string, int>& p : m)
{
 // do something with p
}
````

```
// The key is actually const so the type is std::pair
<const std::string, int> not const std::pair<std::string, int>

for (const auto& p : m)
{
 // as before
}
```

## 6. Facilitate refactory 
"`auto` types automatically change if the type of their initializing expression changes". This helps with refactoring code.

## Question from ISLA

Can auto be used in class declaration? The below snippet does not compile.

```
#include<iostream>

class A
{
    public:
         auto i = 0;
};


int main()
{
    A a1;
    std::cout<<a1.i<<std::endl;
}
```
Yes, but only `static` and `const`

```c++
class A
{
    public:
         static const auto i = 0;
};
```
