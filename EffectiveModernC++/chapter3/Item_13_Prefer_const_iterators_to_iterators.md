# const_iterators
+ const_iterators are the STL equivalent of pointers-to-const
+ use const whenever possible
+ use const_iterators any time you need an iterator, no need to modify what the iterator points to

# C++98 iterators
```c++
#include<iostream>
#include<vector>
#include<algorithm>

int main()
{
  std::vector<int> values = {60, 61, 62, 63, 64, 65, 60};
  std::vector<int>::iterator it = std::find(values.begin(), values.end(), 60);
  values.insert(it, 06);
  for(auto i: values)
  {
    std::cout << i << std::endl;
  }
  return 1;
}
Start

6
60
61
62
63
64
65
60

1

Finish
```
# C++98 using const_iterators !!! NOT WORKING!!!
```c++
using IterT = std::vector<int>::iterator;
using ConstIterT = std::vector<int>::const_iterator;

std::vector<int> values;
ConstIterT ci = std::find(static_cast<ConstIterT>(values.begin()), 
                          static_cast<ConstIterT>(value.end()), 
                          60);
values.insert(static_cast<IterT>(ci), 06); //casting does not work here.
```

# C++11 using const_iterators
```c++
int main()
{
	std::vector<int> values = {60, 61, 62, 63, 64, 65, 60};
  auto it = std::find(values.cbegin(), values.cend(), 60);
	values.insert(it, 06);    
  for(auto i: values)
  {
    std::cout << i << std::endl;
  }
  return 1;
}
Start

6
60
61
62
63
64
65
60

1

Finish
```

# C++14 write generic library to handle container. 
Only C++14: cbegin, cend, rbegin, rend, crbegin, crend are truly working.

```c++
#include<iostream>
#include<vector>
#include<algorithm>

template<typename C, typename V>
void findAndInsert(C& container, const V& targetValue, const V& insertValue)
{
  using std::cbegin; //non-member cbegin
  using std::cend; // also non-member cend
  auto it = std::find(cbegin(container), cend(container), targetValue);
  container.insert(it, insertValue);
}

int main()
{
	std::vector<int> values = {60, 61, 62, 63, 64, 65, 60};
  findAndInsert(values, 60, 06);
	for(auto i: values)
  {
    std::cout << i << std::endl;
  }
  return 1;
}
Start

6
60
61
62
63
64
65
60

1

Finish
```
# C++11 write generic library to handle container. 
You need to define your own cbegin, cend.

```c++
#include<iostream>
#include<vector>
#include<algorithm>

template<class C>
auto cbegin(const C& container) ->decltype(std::begin(container))
{
  return std::begin(container);  //yield a const-iterator
}
template<class C>
auto cend(const C& container) ->decltype(std::end(container))
{
  return std::end(container);
}


template<typename C, typename V>
void findAndInsert(C& container, const V& targetValue, const V& insertValue)
{
  auto it = std::find(cbegin(container), cend(container), targetValue);
  container.insert(it, insertValue);
}

int main()
{
	std::vector<int> values = {60, 61, 62, 63, 64, 65, 60};    
  findAndInsert(values, 60, 06);
	for(auto i: values)
  {
    std::cout << i << std::endl;
  }
  return 1;
}
Start

6
60
61
62
63
64
65
60

1

Finish
```
# Things to remember:
+ Prefer const_iterators to iterators
+ In maximally generic code, prefer non-member versions of begin, end, rbegin, etc, over their member function counterparts.
