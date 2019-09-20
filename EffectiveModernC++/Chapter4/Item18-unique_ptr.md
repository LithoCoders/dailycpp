# Smart Pointer

## Why raw pointers are bad:

- When you see `TYPE* ptr;`. Does `ptr` points to a single object or to an array? Remember that an array is a pointer to its first element
- Who owns the memory pointerd by `ptr`? Am I reposible for destroying it? If yes, how? `delete, free, delete []` or using a custom deleter?
- You have to ensure that in no cases you destroy the pointer more than once. This is UB.
- You have to ensure that in no cases you destroy the pointer less than one time. This is a memory leak.
- How do you know that the value of `ptr` is not dangling i.e. pointing to a destroyed memory location?

Smart pointers address all those issues in an elegant manner. There are many smart pointers flavors each with its own pros and cons, uses cases and performances costs.

# `std::unique_ptr` 
`std::unique_ptr`  is meant to address the scenario where is you are using the pointer then you are responsible for its destruction. It models ownership. a `std::unique_ptr`  always owns the memory it points. This translates to the fact that `std::unique_ptr` are not copyable but only movable.

```cpp
template<typename T1>
unique_pointer(const unique_pointer<T1>&) = delete;

template<typename T1>
unique_pointer& operator=(const unique_pointer<T1>&) = delete; 
```

The way you want the object to be destructed can be specified during the construction of the `unique_ptr`, by configuring it with a custom deleter. A custom deleter it's useful in all those scenarios where you do not simply want the object to be deleted but also other operation to be performed e.g. a logging operation.

```cpp
#include <iostream>
#include <memory>
#include <cstdlib>
using namespace std;

struct Foo
{
    Foo(const string& a) : name(a){};
    string name;
};

void FooDestructor(Foo* ptr)
{
   cout<<__PRETTY_FUNCTION__<<std::endl;
    delete ptr;
}

struct FooDestructorFunctor
{
    void operator()(Foo* ptr)
    {
        cout<<__PRETTY_FUNCTION__<<std::endl;
       
        delete(ptr);
        cout<<endl;
    }
};


int main()
{
    std::unique_ptr<Foo> bar(new Foo("1")); //default behaviour
    std::unique_ptr<Foo, decltype(&FooDestructor)> bar1(new Foo("2"), FooDestructor); //custom deleter
    std::unique_ptr<Foo, FooDestructorFunctor> bar2(new Foo("3"), FooDestructorFunctor()); //custom deleter
    
    cout<<"sizeof(bar) "<<sizeof(bar)<<endl;
    cout<<"sizeof(bar1) "<<sizeof(bar1)<<endl;
    cout<<"sizeof(bar2) "<<sizeof(bar2)<<endl;
    return 0;
}

```
Notice anything about the output?

```bash
I am a stupid functor
destroying *ptr = 3

destroying *ptr = 2
```

It is the lightest of the smart pointers as it hase the same size of a raw pointer (so no size penalty =) ) and in most cases same time performance.

You incur in a size and time penalty when you use custom deleters because the pointer to the deleters might need to be stored in the smart pointer.



# `unique_ptr` implementation

The following is a minimalistic unique_ptr implementation (not supporting custom deleters)
```cpp
#include <cassert>
#include <iostream>


#define DEBUG (TRUE)
#define TRACE_FUNCTION {std::cout<<__PRETTY_FUNCTION__<<std::endl;}

//---------------------
namespace learning::memory
{
 
 template<typename T>
 class unique_pointer
 {  
    typedef T* pointer_type;
    typedef T& referece_type;
    typedef T  type;
 
    public:
    unique_pointer() : ptr(pointer_type())
    {TRACE_FUNCTION}
    
    unique_pointer(pointer_type p) : ptr(p)
    {TRACE_FUNCTION}
    
    unique_pointer(unique_pointer&& up) : ptr(up.release_pointer()) {TRACE_FUNCTION}   
    ~unique_pointer()
    {
        TRACE_FUNCTION
        reset();
    }

    pointer_type get() const { TRACE_FUNCTION; return this->ptr;}
    void reset(pointer_type p = nullptr)
    {
    TRACE_FUNCTION;
      if(p!=get()) //self assignment
      {
        delete this->ptr;
        this->ptr = p;
      }
    }
         

    pointer_type release_pointer()
    {
        TRACE_FUNCTION
        const pointer_type p = this->ptr;
        this->ptr = nullptr;
        return p;
    }
    
    operator bool() const{
        return get()!=nullptr;
    }
    
    referece_type operator*() const 
    {
        return *get();
    }
 
    
    //disable copy

    template<typename T1>
    unique_pointer(const unique_pointer<T1>&) = delete;
    
    template<typename T1>
    unique_pointer& operator=(const unique_pointer<T1>&) = delete; 

   private:
     T* ptr;
    
 };
 
    template<typename T1, typename T2>
    bool operator==(const unique_pointer<T1>& p1 , const unique_pointer<T2>& p2)
    {
      TRACE_FUNCTION;
      return p1.get()== p2.get(); 
    }
    
    /*template<typename T1>
    bool operator==(const unique_pointer<T1>& p1 , const unique_pointer<T1>& p2)
    {
    TRACE_FUNCTION;
      return p1.get()== p2.get(); 
    }*/
 
}

using namespace learning::memory;

template<typename T>
void/*unique_pointer<T>*/ foo(unique_pointer<T> p)
{
    TRACE_FUNCTION
    if(p)
    {
        (*p)++;
    }
   // return std::move(p);
}

template<typename T, typename T1>
bool/*unique_pointer<T>*/ foo_compare(const unique_pointer<T>& p, const unique_pointer<T1>& p1)
{
   return p==p1;
}


struct A{
    int a;
};
int main(){
    unique_pointer<int> up1 (new int(4));
    //unique_pointer<double> up2 (new double(4));
    unique_pointer<A> up2 (new A);
    
    //error
    //unique_pointer<double> up3 (up2);
    
    //error
    //unique_pointer<long> up4 (up1);
    
    //foo(std::move(up1));
    //foo_compare(up1,up1);
    //foo_compare(up1,up2);//error
    
    if(up1==up2)
        std::cout<<"yesah";
    

}
```
