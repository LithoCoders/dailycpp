# How to view deduced types
The acutal types deduced by the compiler when using generic code (templates) or auto is not visible to the developer. But how can we force to see what type the compiler actually choose for us? What tool can we use?
There are three possibilities:

1. Editors
2. Compiler time 
3. At runtime

## Editors
Some editor (like eclipse) shows hints on the type that the compiler is choosing. The IDE does that by using a compiler hence if the code is not in a state close to being compilable you get not hints. Also note that for complex generic code the decuded type can be very complex to read and hence the info given by the ide might not be super helpful.

## Compiler diagnostics 

### Through errors

The compiler internally knows what type it is using, so why don't force the compiler to show us all those info? We can achieve that by forcing the compilation to fail and as a side effect the compiler will show us all the deduced types in the error messages.
One easy way to achieve it is by **only declaring** a template structure and then trying to instantiate it with the **decltype of a variable** we would like to know the type.

```c++
#include <cassert>
#include <iostream>

using namespace std;
template<class T>
class TYPEOF;

int main(){

  const auto* p = &cout;
  TYPEOF<decltype(p)> a;
  return 0;
}
```
results in the following error in `gcc 8.2.0`:

```bash
prog.cc: In function 'int main()':
prog.cc:13:23: error: aggregate 'TYPEOF<const std::basic_ostream<char>*> a' has incomplete type and cannot be defined
   TYPEOF<decltype(p)> a;
```
results in the following error in `clang 9.0`:

```bash
prog.cc:13:23: error: implicit instantiation of undefined template 'TYPEOF<const std::__1::basic_ostream<char> *>'
  TYPEOF<decltype(p)> a;
                      ^
prog.cc:6:7: note: template is declared here
class TYPEOF;
      ^
1 error generated.
```

Another way of achieveing it is by using function templates:
```c++
template<class T>
void type_info_f(T type);

#define TYPEOF_ERROR(var) (type_info_f<decltype(var)>(var))

int main(){

  const auto&& p = 5;
  TYPEOF_ERROR(p);
  
  return 0;
}
```
which results in 

```bash
/tmp/ccRol0So.o: In function `main':
prog.cc:(.text+0x18): undefined reference to `void type_info_f<std::ostream const*>(std::ostream const*)'
collect2: error: ld returned 1 exit status
```

## Runtime  Diagnostic
## printf + typeid
C++ offers runtime type facilities such as `std::typeid`.
One can do 
```c++
#include <cassert>
#include <iostream>
using namespace std;

template<class T, class T1>
class A{
    T x;
    T1 y;
};

int main(){
  const auto& p = cout;
  auto&& r1 = 60.3;
  cout<<typeid(p).name()<<endl;
  cout<<typeid(r1).name()<<endl;
  cout<<typeid(4l).name()<<endl;
  cout<<typeid(A<decltype(r1), decltype(cin)>).name()<<endl;
    
  return 0;
}
```

which results in `gcc 8.2.0` 
```
So
d
l
1AIOdSiE
```

while in `clang 9.0.0`:
```bash
NSt3__113basic_ostreamIcNS_11char_traitsIcEEEE
d
l
1AIOdNSt3__113basic_istreamIcNS1_11char_traitsIcEEEEE
```
The implementation is free to choose whatever output they want for typeid... so do not expect or rely on anything super super usefule here.

### Through compiler specific macros
Another way to do it is force to use compiler's specific macros such as `__PRETTY_PRINT__` in `gcc`

```c++
#include <cassert>
#include <unordered_set>
#include <iostream>
#include <vector>
using namespace std;

#define PRINT_TYPE_INFO std::cout<<__PRETTY_FUNCTION__<<endl;

template<class T>
decltype(auto) f(T&& par){
  PRINT_TYPE_INFO;
  return std::forward<T>(par);
}

int main(){
  int x = 6;
  f(5);
  f(x);
  f(std::unordered_set<int>());
  f(std::vector<std::vector<int>>());
  return 0;
}
```
which results in `clang 9.0.0`
```bash
decltype(auto) f(T &&) [T = int]
decltype(auto) f(T &&) [T = int &]
decltype(auto) f(T &&) [T = std::__1::unordered_set<int, std::__1::hash<int>, std::__1::equal_to<int>, std::__1::allocator<int> >]
decltype(auto) f(T &&) [T = std::__1::vector<std::__1::vector<int, std::__1::allocator<int> >, std::__1::allocator<std::__1::vector<int, std::__1::allocator<int> > > >]
```

and in `gcc 8.2.0`
```bash
decltype(auto) f(T&&) [with T = int]
decltype(auto) f(T&&) [with T = int&]
decltype(auto) f(T&&) [with T = std::unordered_set<int>]
decltype(auto) f(T&&) [with T = std::vector<std::vector<int> >]
```

