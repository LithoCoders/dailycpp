# Make const member functions thread safe in a concurrent context

```c++
#include<vector>
#include<mutex>

class Polynomial {
    public:
        using RootsType = std::vector<double>;
    
        RootsType computeRoots() const  // this function is expensive, we want it to be run one time, store results to rootValues.
                                        // it does not modify roots, so, we make it 'const'
        {
            std::lock_guard<std::mutex> g(m);  // to avoid data race
            if(!rootsAreValid){
                //expensive calculation here
                //stores values in rootValues
                rootValues = RootsType{1.2, 3.4};
                rootsAreValid = true; // set the flag, roots are stored at cache                
            }
            return rootValues;
        }// implicitly unlock m
    
    private:
        mutable std::mutex m;                 // using mutex to avoid undefined results due to multiple threads call on one object
        mutable bool rootsAreValid {false} ;  // using 'mutable' so that it can be modified in a 'const' function
        mutable RootsType rootValues{};       // using 'mutable' together with mutex is so called, 'M&M' rule
};
    

int main()
{
    Polynomial poly_obj;
    std::vector<double> roots = poly_obj.computeRoots();    
    return 1;
}

```
std::mutex is a move-only type, it can not be copied. Therefore, an object of class Polynomial is also a move-only type, not be copiable.

# Use of std::atomic variables is suitable for manipulation of only a single variable or memory location.

std::atomic is less expensive, less overkill compared to std::mutex. std::atomic is also a move-only type. !!! cppreference.com: "std::atomic is neither copyable nor movable. "

Example: Counting how many time a function is called with std::atomic
```c++
class Point
{
    public:
        double calDistance() const noexcept          // this function should not modify data member
        {
            ++callCount;                            // increase number of count
            return std::sqrt((x*x) + (y*y));
        }
        
    private:
        mutable std::atomic<unsigned> callCount{0}; // count is atomic
        double x, y;
};
```
Problem with two atomic variables
```c++
class Widget{
{
    public:
        int calMagicValue() const
        {
            if(cacheValid) return cachedValue;
            else
            {
                auto value1 = expensiveComputation1();
                auto value2 = expensiveComputation2();
                cachedValue = value1 + value2;
                cachedValid = true;
                return cachedValue;                
            }
        }
    private:
        mutable std::atomic<bool> cacheValid {false};
        mutable std::atomic<int> cachedValue;
};
```
std::atomic does not work if there are two values
Try mutex
```c++
class Widget{
{
    public:
        int calMagicValue() const
        {
            std::lock_guard<std::mutex> g(m);
            if(cacheValid) return cachedValue;
            else
            {
                auto value1 = expensiveComputation1();
                auto value2 = expensiveComputation2();
                cachedValue = value1 + value2;
                cachedValid = true;
                return cachedValue;                
            }
        }
    private:
        mutable std::mutex m;
        mutable std::atomic<bool> cacheValid {false};
        mutable std::atomic<int> cachedValue;
};
```
Things to remember
* Make const member functions thread safe unless you’re certain they’ll never be used in a concurrent context.
* Use of std::atomic variables may offer better performance than a mutex, but they’re suited for manipulation of only a single variable or memory location.
