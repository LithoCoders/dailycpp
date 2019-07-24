

# Item 5

Note: Contains code snippets from Effective Modern C++ by Scott Meyers
Skipped the part on std::function as it goes out of point.

Today, we we'll be discussing Item 5, which describes why the keyword `auto` should be used in place of explicit type declarations. The advantages of auto are as follows:

1. Readability 
2. Prevent bugs due to unintentional type mismatches

```
int x1; // potentially uninitialized
auto x2; // error! initializer required
auto x3 = 0; // fine, x's value is well-defined
```

`auto` forces to initialize the variables.


## Readability

```
//before C++11
template<typename It> // algorithm to dwim ("do what I mean")
void dwim(It b, It e) // for all elements in range from
{ // b to e
	while (b != e) {
	typename std::iterator_traits<It>::value_type currValue = *b; // who wants to write something like this to express the type?
	}
}
```

```
//after C++11
template<typename It> // algorithm to dwim ("do what I mean")
void dwim(It b, It e) // for all elements in range from
{ // b to e
	while (b != e) {
	auto currValue = *b; // Lot more easier.
	}
}
```

```
//C++11

auto derefUPLess = // comparison func.
[](const std::unique_ptr<Widget>& p1, // for Widgets
const std::unique_ptr<Widget>& p2) // pointed to by
{ return *p1 < *p2; }; // std::unique_ptrs
```

```
//C++14 - parameters to lambda can use auto

auto derefLess = // C++14 comparison
[](const auto& p1, // function for
const auto& p2) // values pointed
{ return *p1 < *p2; }; // to by anything pointer-like

```
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
