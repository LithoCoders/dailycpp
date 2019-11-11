# Subscriber & Publisher using `std::bind`

```c++
#include <iostream>
#include <cstdlib>
#include <vector>
#include <memory>
#include <string>
#include <typeinfo>
#include <functional>


class publisher
{
    public:
        
        void subscribe(std::function<void()> cb)
        {
            subscriber_cbs.push_back(cb);
        }
   
        void raise_event()
        {
            std::cout<< " call of " << __PRETTY_FUNCTION__ << std::endl;
            for (auto callback : subscriber_cbs)
                callback();
        }
    private:
        std::vector<std::function<void()>> subscriber_cbs;
};

class subscriber
{
    public:
        subscriber(std::string name) : subscriber_name(name)
        {
            std::cout<< " call of " << __PRETTY_FUNCTION__ << " from " << subscriber_name << std::endl;
        }
   
       
        ~subscriber()
        {
            std::cout<< " call of " << __PRETTY_FUNCTION__ << " from " << subscriber_name << std::endl;
        }
   
        void subscribe_to_publisher(publisher& pub)
        {
            pub.subscribe(std::bind(&subscriber::subscription_callback, this));
        }
   
    private:
   
        std::string subscriber_name;
        void subscription_callback()
        {
            std::cout<< " call of " << __PRETTY_FUNCTION__ << " from " << subscriber_name << std::endl;
        }
};

int main()
{
    publisher sv;
    subscriber clt_1("client 1");
    subscriber clt_2("client 2");
    clt_1.subscribe_to_publisher(sv);
    clt_2.subscribe_to_publisher(sv);
    sv.raise_event();
    return 0;
}
```
# Subscriber & Publisher using lambda

Item 34 and one more resource[1] suggest to replace lambda with `std::bind`. 

The lambda version of the above code is below. We need to capture `this` and call private function `subcription_callback()` inside lambda.
```c++
  pub.subscribe([this](){subscription_callback();});
```

[1] C++ Weekly - Ep 16 Avoiding `std::bind`  https://www.youtube.com/watch?v=ZlHi8txU4aQ

# Benchmarking
Lambda version seems to be ~80 times faster (which needs to understand why?) than `std::bind`

```c++
#include <iostream>
#include <functional>

static void lambda_version(benchmark::State& state) {
  // Code inside this loop is measured repeatedly
  for (auto _ : state) {
    int lowVal = 0;
    int highVal = 10;
    
    auto betweenL = [lowVal, highVal] (const auto& val) {return lowVal <= val && val <= highVal; };


    benchmark::DoNotOptimize(highVal);
    benchmark::DoNotOptimize(lowVal);
  }
}
// Register the function as a benchmark
BENCHMARK(lambda_version);

static void bind_version(benchmark::State& state) {
  // Code before the loop is not measured
  
  for (auto _ : state) {
    int lowVal = 0;
    int highVal = 10;
    using namespace std::placeholders;
    auto betweenB = std::bind(std::logical_and<>(),
                               std::bind(std::less_equal<>(), lowVal, _1),
                               std::bind(std::less_equal<>(), _1, highVal)
                             );

    benchmark::DoNotOptimize(highVal);
    benchmark::DoNotOptimize(lowVal);
  }
}
BENCHMARK(bind_version);

```
