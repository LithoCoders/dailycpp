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
