# Item 5: Prefer `auto` to explicit type declarations

Note: Contains code snippets from Effective Modern C++ by Scott Meyers
Skipped the part on std::function as it goes out of point.

Today, we we'll be discussing Item 5, which describes why the keyword `auto` should be used in place of explicit type declarations. The advantages of `auto` are as follows:

1. Avoid uninitialize variables
2. Readability 
3. Hold closures
4. Prevent bugs due to unintentional type mismatches
5. Facilitate refactory

## Avoid uninitialize variables
```
int x1;      // potentially uninitialized
auto x2;     // error! initializer required
auto x3 = 0; // fine, x3's value is well-defined
```

`auto` forces you to initialize the variables.

## Readability
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
Because `auto` uses type deduction, it can represent types known only to compilers. Let us consider a comparison function to compare two `Widget` objects pointed by `std::unique_ptrs` pointers.
```
//C++11
auto derefUPLess = [](const std::unique_ptr<Widget>& p1, const std::unique_ptr<Widget>& p2) 
			{ return *p1 < *p2; }; 
```
In C++14, parameters to lambda can use `auto`
```
//C++14
auto derefLess = [](const auto& p1, const auto& p2)  // to compare two values pointed by
			{ return *p1 < *p2; };       // anything pointer-like

```
## Hold closures

## Prevent bugs due to unintentional type mismatches

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

```
std::vector<int> v;
/*unsigned */
unsigned sz = v.size(); // v.size() => std::vector<int>::size_type = unsigned on 32 bit Windows
                        // unsigned(32 bits) != std::vector<int>::size_type(64 bits) on 64 bit windows

auto sz = v.size() // evaluates to std::vector<int>::size_type sz, correct type, prevent portability issues


```

## Question from ISLA

Can auto be used in class declrations? The below snippet does not compile.

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
