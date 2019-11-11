# Prefer lambdas to `std::bind`
There are five reasons why we should prefer lambdas instead of `std::bind` even with C++11. With C++14, it is even better to use lambdas instead of `std::bind`
## Readability
Consider the following lambda
```c++
#include <chrono>

using Time = std::chrono::steady_clock::time_point;
enum class Sound { Beep, Siren, Whistle };

using Duration = std::chrono::steady_clock::duration;

//at time t, make sound s for duration d
void setAlarm(Time t, Sound s, Duration d) {};

auto setSoundLambda =   [] (Sound s) 
                        {
                            using namespace std::chrono;
                            using namespace std::literals;      // for 1h, 30s

                            setAlarm(steady_clock::now() + 1h,  //only at C++14
                                    s,
                                    30s);
                        };
int main()
{
    Sound s = Sound::Beep;
    setSoundLambda(s);
    return 0;    
}
```
The above code means, turning alarm on after one hour from now for 30 seconds with beep sound.

The equivalent `std::bind` version is
```c++
#include <chrono>
#include <functional>

using Time = std::chrono::steady_clock::time_point;
enum class Sound { Beep, Siren, Whistle };
using Duration = std::chrono::steady_clock::duration;

//at time t, make sound s for duration d
void setAlarm(Time t, Sound s, Duration d) {};

using namespace std::chrono;
using namespace std::literals;      // for 1h, 30s
using namespace std::placeholders;  // for _1

auto setSoundBind = std::bind(setAlarm, 
                                steady_clock::now() + 1h, 
                                _1, //map to 1st argument of setSoundBind function
                                30s);

int main()
{
    Sound s = Sound::Beep;
    setSoundBind(s);
    return 0;    
}
```
Readers have to mentally remember `_1` maps to the first argument of `setSoundBind()` function, namely, `s`.
It is not expressive.
For this example, the `std::bind` version is also not precise. Because, `steady_clock::now() + 1h` is calculated from the moment `std:bind` is invoked, not `setAlarm` is invoked.

At C++11, fixing this problem is more complex
```c++
#include <chrono>
#include <functional>

using Time = std::chrono::steady_clock::time_point;
enum class Sound { Beep, Siren, Whistle };
using Duration = std::chrono::steady_clock::duration;

//at time t, make sound s for duration d
void setAlarm(Time t, Sound s, Duration d) {};

using namespace std::chrono;
using namespace std::literals;      // for 1h, 30s
using namespace std::placeholders;  // for _1

struct genericAdder {
    template<typename T1, typename T2> auto operator()(T1&& param1, T2&& param2) 
                                            -> decltype(std::forward<T1>(param1) + std::forward<T2>(param2))
    {
        return std::forward<T1>(param1) + std::forward<T2>(param2);
    }
};

auto setSoundBind = std::bind(  setAlarm, 
                                std::bind( genericAdder(),
                                           std::bind(steady_clock::now), 
                                           hours(1)), 
                                _1, //map to 1st argument of setSoundBind function
                                30s);


int main()
{
    Sound s = Sound::Beep;
    setSoundBind(s);
    return 0;    
}
```
## Overloading functions
When we overload function `setAlarm` with four parameters, lambda version still works but `std::bind` version not.
```c++
#include <chrono>

using Time = std::chrono::steady_clock::time_point;
using Duration = std::chrono::steady_clock::duration;
enum class Sound { Beep, Siren, Whistle };
enum class Volume { Normal, Lound, VeryLound };

void setAlarm(Time t, Sound s, Duration d) {};
void setAlarm(Time t, Sound s, Duration d, Volume v) {};

auto setSoundLambda =   [] (Sound s) 
                        {
                            using namespace std::chrono;
                            using namespace std::literals;      // for 1h, 30s

                            setAlarm(steady_clock::now() + 1h,  //only at C++14
                                    s,
                                    30s);
                        };
int main()
{
    Sound s = Sound::Beep;
    setSoundLambda(s);
    return 0;    
}
```
To make `std::bind` version work, we need to cast `setAlarm` to the correct one, i.e, the one takes three parameters.
```c++
#include <chrono>
#include <functional>

using Time = std::chrono::steady_clock::time_point;
enum class Sound { Beep, Siren, Whistle };
using Duration = std::chrono::steady_clock::duration;
enum class Volume { Normal, Lound, VeryLound };

//at time t, make sound s for duration d
void setAlarm(Time t, Sound s, Duration d) {};
void setAlarm(Time t, Sound s, Duration d, Volume v) {};

using namespace std::chrono;
using namespace std::literals;      // for 1h, 30s
using namespace std::placeholders;  // for _1

struct genericAdder {
    template<typename T1, typename T2> auto operator()(T1&& param1, T2&& param2) 
                                            -> decltype(std::forward<T1>(param1) + std::forward<T2>(param2))
    {
        return std::forward<T1>(param1) + std::forward<T2>(param2);
    }
};

using SetAlarm3ParamType = void(*)(Time t, Sound s, Duration d);
auto setSoundBind = std::bind(  static_cast<SetAlarm3ParamType>(setAlarm), 
                                std::bind( genericAdder(),
                                           std::bind(steady_clock::now), 
                                           hours(1)), 
                                _1, //map to 1st argument of setSoundBind function
                                30s);


int main()
{
    Sound s = Sound::Beep;
    setSoundBind(s);
    return 0;    
}
```
## Inline functions
Function `setSoundLambda` inline `setAlarm` whereas, `setSoundBind` not inline `setAlarm` because `std::bind` takes a function pointer pointing to `setAlarm`.
So, lambda version generates a faster code compared to `std::bind` version.
One example shows lambda version is 20% faster compared to `std::bind` version. Source: C++ Weekly - Ep 16 Avoiding `std::bind` https://www.youtube.com/watch?v=ZlHi8txU4aQ

## Complicated functions
Let us define a function consider a value is in range from two local variables.

In C++14, The lambda version below shows understandable code:
```c++
#include <iostream>
#include <functional>

int main()
{
    int lowVal = 0;
    int highVal = 10;
    
    auto betweenL = [lowVal, highVal] (const auto& val) {return lowVal <= val && val <= highVal; };
    
    using namespace std::placeholders;
    auto betweenB = std::bind(std::logical_and<>(),
                               std::bind(std::less_equal<>(), lowVal, _1),
                               std::bind(std::less_equal<>(), _1, highVal)
                             );
    
    std::cout << std::boolalpha;
    std::cout << betweenL(5) << std::endl;
    std::cout << betweenB(5) << std::endl;
    return 0;    
}
//Output
true
true
```
In C++11, the lambda version still shows better code:
```c++
#include <iostream>
#include <functional>

int main()
{
    int lowVal = 0;
    int highVal = 10;
    
    //C++11 does not support auto parameter in parameter list
    auto betweenL = [lowVal, highVal] (int val) {return lowVal <= val && val <= highVal; }; 
    
    using namespace std::placeholders;
    auto betweenB = std::bind(std::logical_and<bool>(),          //In C++11, need to specify type to compare
                               std::bind(std::less_equal<int>(), lowVal, _1),
                               std::bind(std::less_equal<int>(), _1, highVal)
                             );
    
    std::cout << std::boolalpha;
    std::cout << betweenL(5) << std::endl;
    std::cout << betweenB(5) << std::endl;
    return 0;    
}
//Output
true
true
```
## `std::bind` always copies its arguments
In lambda, we can capture local variables by value or by reference, whereas, `std::bind` always copies its arguments. To fix this, we need `std::ref`
```c++
#include <iostream>
#include <functional>

struct Widget{};
enum class CompressLevel { Low = 0, Normal, High };
Widget compress(const Widget& w, CompressLevel lev)
{ 
    Widget tmp = w; 
    //compress tmp with level lev
    std::cout << static_cast<int>(lev) << std::endl;
    return tmp; 
};

int main()
{
    Widget w;
    
    auto compressRateLambda = [w](CompressLevel lev) { return compress(w, lev); };
    compressRateLambda(CompressLevel::Low);
    
    using namespace std::placeholders;
    auto compressRateBind = std::bind(compress, std::ref(w), _1); //Call compress() with reference to w
    compressRateBind(CompressLevel::High);
    
    return 0;    
}
//Output
0
2
```
In C++14, it is clear that lambdas is more attractive compared to `std::bind`

# When do `std::bind` shine ?
Only in C++11, there are two cases where `std::bind` is preferable.
* Move capture: C++11 does not support move capture, so we need `std::bind` to mimic this. But in C++14, with *init capture* we don't need move capture anymore. Item 32.
* Polymorphic funtion objects: `std::bind` accept arguments of any type. Consider templatized function call operator below:
!!Compiled with C++2a, but not C++11!!
```c++
#include <iostream>
#include <functional>
class PolyWidget{       //function object
    public:
        template<typename T>
        void operator()(const T& param) const 
        {
            std::cout << param << std::endl;                                             
        };
};

int main()
{
    PolyWidget pw;
    using namespace std::placeholders;
    auto boundPW = std::bind(pw, _1);
    
    boundPW(2019);    //pass int to PolyWidget::operator()
    boundPW(nullptr); //pass nullptr to PolyWidget::operator()
    boundPW("blue");  //pass string to PolyWidget::operator()
    
    return 0;    
}
//Output:
2019
nullptr
blue
```
We can do this with lambda takes `auto` param in C++14:
```c++
#include <iostream>
#include <functional>
class PolyWidget{       //function object
    public:
        template<typename T>
        void operator()(const T& param) const 
        {
            std::cout << param << std::endl;                                             
        };
};

int main()
{
    PolyWidget pw;
    using namespace std::placeholders;
    auto boundPW = std::bind(pw, _1);
    
    boundPW(2019);    //pass int to PolyWidget::operator()
    boundPW(nullptr); //pass nullptr to PolyWidget::operator()
    boundPW("blue");   //pass string to PolyWidget::operator()
    
    std::cout << std::endl;
    
    auto boundPWLambda = [pw](const auto& param){ pw(param); };
    boundPWLambda(2020);
    boundPWLambda(nullptr);
    boundPWLambda("red");
    
    return 0;    
}
//Output
2019
nullptr
blue

2020
nullptr
red
```
# Conclusion
* From C++14, prefer lambdas over `std::bind`
* C++11, `std::bind` can be useful in move capture and templatized function call operators.

