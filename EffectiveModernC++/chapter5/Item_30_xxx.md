Item 30 - perfect forwarding failure cases
Before we look into the details, let us be familiar with the terms author uses in his book.
- *forwarding*: one function A passes - *forward* - its parameters to another function B. Function B receives the same object C that A receives.
- *perfect forwarding*: not only object but also salient characteristics:
	* type
	* lvalueness or rvalueness
	* constness
	* volatile-ness
- *forwarding function*: it is A above
- *forwarded-to function*: it is B above
- *originally passed-in objects*: it is C above
- *by-value parameters*: they are copies of what the original caller passes in.
Argument `i` has different memory address because it is not the same in two functions `forwardingFunc` and `forwardedToFunc`
```c++
#include<iostream> 

void forwardingFunc(int i);
void forwardedToFunc(int i);

void forwardingFunc(int i)
{
    std::cout << __PRETTY_FUNCTION__ << std::endl;
    std::cout << &i << std::endl;
    forwardedToFunc(i);
}

void forwardedToFunc(int i)
{
    std::cout << __PRETTY_FUNCTION__ << std::endl;
    std::cout << &i;
}
int main()
{
  	int i = 1;
    forwardingFunc(i);
    return 0;
}
Start

void forwardingFunc(int)
0x7ffcdfdeda2c
void forwardedToFunc(int)
0x7ffcdfdeda0c

0

Finish
```
- *pointer parameters* ?? To be re-checked

Universal references normal case vs. Universal references in variadic template

There are xxx case that *perfect forwarding* can go wrong but it can be generalized:
forwardingFunc(exp): does something
forwardedToFunc(exp): does something else.

Case 1: Braced initializers
forwardingFunc() fails to deduce to std::initializer_list. Fix with `auto`

Case 2: 0 or NULL as null pointer @ template
Deduced to integer. Don't. Item xxx
