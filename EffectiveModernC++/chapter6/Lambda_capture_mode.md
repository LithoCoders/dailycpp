Default capture modes: by-reference and by-value
```c++
#include <iostream>
#include <cstdlib>

int main()
{
    int x = 1; 
    int y = 1;
    std::cout << "Outside lambdas &x : " << &x << std::endl;
    auto lCaptureByValue = [=] () -> void
    {
        std::cout << "Inside lCaptureByValue &x : " << &x << std::endl;        
        //strange that even though x, y are captured by value
        //we can not modify x & y here
    };    
    lCaptureByValue();
    std::cout << x << "  " << y << std::endl;
    
    auto lCaptureByRef = [&] () -> bool    //capture-by-ref for both x and y
    {        
        std::cout << "Inside lCaptureByRef &x : " << &x << std::endl;        
        x--;        
        y--;        
        return (x + y) > 0;
    };
    std::cout << std::boolalpha << lCaptureByRef() << " " << x << "  " << y << std::endl;
    
    auto lCaptureByRefAndValue = [&x, y] () -> void  //[&x, =] does not work!!
    {    
        std::cout << "Inside lCaptureByRefAndValue &x : " << &x << std::endl;        
        x--;
    };
    lCaptureByRefAndValue();
    std::cout << x << "  " << y << std::endl;
    
    return 0;
}

//Output:
Outside lambdas &x : 0x7ffdbcb206fc
Inside lCaptureByValue &x : 0x7ffdbcb206f4
1  1
Inside lCaptureByRef &x : 0x7ffdbcb206fc
false 0  0
Inside lCaptureByRefAndValue &x : 0x7ffdbcb206fc
-1  0
```
