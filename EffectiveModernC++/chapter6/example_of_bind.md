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

Item 34 and two more resources[1] suggest to replace lambda with `std::bind`. 

The lambda version of the above code is below. We need to capture `this` and call private function `subcription_callback()` inside lambda.
```c++
  pub.subscribe([this](){subscription_callback();});
```

[1] C++ Weekly - Ep 16 Avoiding `std::bind`  https://www.youtube.com/watch?v=ZlHi8txU4aQ
