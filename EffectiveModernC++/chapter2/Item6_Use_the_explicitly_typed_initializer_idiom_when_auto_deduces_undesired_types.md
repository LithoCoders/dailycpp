# Item 6: Use the explicitly typed initializer idiom when `auto` deduces undesired types

## *Invisible* proxy types with `std::vector<bool>`
`std::vector::operator[]` returns a reference to an element of the container **except bool**. 
Indexing operator `[]` of `std::vector<bool>` returns an object of type `std::vector<bool>::reference`, a nested class inside `std::vector<bool>`.

```c++
#include <iostream>
#include <vector>
using namespace std;
int main()
{
  std::vector<int> vi ={0, 1};
  std::vector<bool> vb ={true, false};
  auto a = vb[0];
  bool b = vb[0];
  int i = vi[0];
  return 0;
}
```
Output from C++Insight shows the types are difference between `a` and `b` and `i`.
```c++
#include <iostream>
#include <vector>
using namespace std;
int main()
{
  std::vector<int> vi = std::vector<int, std::allocator<int> >
                            {std::initializer_list<int>{0, 1}, std::allocator<int>()};
  std::vector<bool> vb = std::vector<bool, std::allocator<bool> >
                            {std::initializer_list<bool>{true, false}, std::allocator<bool>()};
  std::_Bit_reference a = vb.operator[](static_cast<unsigned long>(0));
  bool b = static_cast<bool>(vb.operator[](static_cast<unsigned long>(0)).operator bool());
  int i = vi.operator[](static_cast<unsigned long>(0));
  return 0;
}
```
Another example to make use of bit reference to set element in vector of bools. This is different to vector of integers.
```c++
#include <iostream>
#include <vector>
using namespace std;
int main()
{
  std::vector<bool> vb = {true};   
  auto a = vb[0];
  std::cout << std::boolalpha << a << std::endl;
  a = false;  
  std::cout << std::boolalpha << vb[0];
  return 0;
}
/*Output:
true
false
*/
```
Therefore, the following code has problem when using `auto` instead of `bool`.
```c++
std::vector<bool> features(const Widget& w);
Widget w;
auto highPrio = features(w)[5]; // highPrio is not a bool but std::_Bit_reference 
processWidget(w, highPrio);     // undefined behavior if highPrio is dangling.
```
The better way to do this is either explicitly say `highPrio` is a bool or we need to cast return of indexing operator to bool.
```c++
bool highPrio = features(w)[5];
//Or
auto highPrio = static_cast<bool>(features(w)[5]);
```
The author seems like the second approach since it preserve using `auto`.

The author calls `std::vector<bool>::reference` a proxy types as it refers to the `bool` underlying. Other examples of proxy types are smart pointers, where they wrap the actual resource we want to use.
In any case with invisible proxy types, author suggests to use the "explicitly typed initializer idiom", i.e., `static_cast<>()`.





