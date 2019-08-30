# `decltype` almost always yields the type of a variable or expression without any modifications
Given a name or an expression, decltype tells you the name’s or the expression’s type.

decltype(name) = *decl*ared *type* of name

Given this:
```c++
#include<vector>
#include<deque>

struct Tuna
{
	int a, b;
};

int main()
{
  const int ci = 0; 
  decltype(ci) dci = 0; 	 // decltype(ci) is const int
  				 // require initialization because of deducing dci as a const  
  Tuna t; 
  decltype(t) dt;  
  bool f(const Tuna & ob);  	 // decltype(ob) const Tuna &                            
                            	 // decltype(f) bool(const Tuna &)  
  decltype(Tuna::a) dta = 10;
  
  std::vector<int> v = {1, 2, 3}; // decltype(vv) is vector<int>
  decltype(v) dv;
  decltype(v[0]) dv0 = t.a;       // decltype(v[0]) is int&
  
  std::deque<int> d;  		  // decltype(aa[0]) is int&
  decltype(d) dd;
}
```

Output on C++ Insight shows:
```c++
  const int ci = 0;
  const int dci = 0;
  
  Tuna t = Tuna();
  Tuna dt = Tuna();
  bool f(const Tuna & ob);  
  int dta = 10;
  
  std::vector<int> v 
        = std::vector<int, std::allocator<int> >{std::initializer_list<int>{1, 2, 3}, std::allocator<int>()};
  decltype(v) dv = std::vector<int, std::allocator<int> >();
  __gnu_cxx::__alloc_traits<std::allocator<int> >::value_type & dv0 = t.a;
  std::deque<int> d = std::deque<int, std::allocator<int> >();
  decltype(d) dd = std::deque<int, std::allocator<int> >();
```
But exception with vector of bool:
```c++
  std::vector<bool> vb = {true, false}; 		  //decltype(vb[0]) is bool (!not bool&)
  decltype(vb[0]) dvb = false;
  
  
  error: no viable conversion from 'bool' to 'decltype(vb[0])' (aka 'std::_Bit_reference')
  decltype(vb[0]) dvb = false;
                  ^     ~~~~~
```
This is because decltype of  vb[0] is a brand new object bool not a reference to bool so that you can not assign `dvb` to false as above.
This lead to one problem we try to solve a bit later.
Problem: write a function that takes a container and an index and return the result of the indexing operation. The return type of the function should be the same as the type returned by the indexing operation, can be T or T&
This is where decltype takes place. In C++11, the primary use for decltype is declaring function templates where the functions’ return type depends on its parameter types.

todo: where to put this
• For lvalue expressions of type T other than names, decltype always reports a type of T&. 


// The use of auto before the function name has nothing
// to do with type deduction.
// Rather, it indicates that C++11’s trailing return type
// syntax is being used (func return type will be declared
// after ->)
auto f() -> int
{
	//...
}

template<typename Container, typename Index>
auto f(Container & c, Index i) -> decltype(c[i])
{
	return c[i];
}


deque<int> dd;
f(dd, 5) = 10; // Won't compile as auto return type deduction
// will strip the reference. To make it compile in C++14:

template<typename Container, typename Index>
decltype(auto) f(Container & c, Index i)
{
	retrun c[i];
}

Things to Remmember:
• decltype almost always yields the type of a variable or expression without any modifications. 
• For lvalue expressions of type T other than names, decltype always reports a type of T&. 
• C++14 supports decltype(auto), which, like auto, deduces a type from its initializer, but it performs the type deduction using the decltype rules
