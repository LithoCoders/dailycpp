Previously, we learn that *moves* are faster than *copies* but this is not the case for few scenarios. Many types fail to support *move semantics*. 

# No *move operations*
C++11 generates *move operations* for class that lacks of them. Only for class with no copy operations or move operations or destructors. Item 17.

# *Move operations* are not cheap
All containers in standard C++11 support *moving*s but not all of them are cheap because container element does not fully support moving.

- consider `std::vector` in C++11: 
```c++
	std::vector<Widget> v1;  // v1 points to heap
	auto v2 = std::move(v1); // set v2 points to heap
	                         // set v1 to null
                           // moving takes constant time
```
							
- consider `std::array` in C++11
```c++
	std::array<Widget, 10000> a1;  // a1 stores on stack ? approved by P.
	auto a2 = std::move(a1);       // really a move from a1 to a2
                                 // moving takes linear time. So that, it is not cheap.
```
								  
- consider `std::string` in C++11, constant-time moves and linear-time copies.
	What is SSO ? *small string optimization*. With string less than 15 characters, it is stored in buffer within `std::string` object, so it is not stored in heap. -> Moving is not faster than copying. 
	
# *Move operations*  are not usable
Some types have move operators emit exceptions. Item 17. Then moves are not used.

In short, some scenarios *moving* is not good:
* No move operations. Object does not support moves thus, copies are invoked
* Move not faster. Object does support moves but not faster than copies. For example, `std::array`
* Move not usable. Move operations emit no exceptions but they are not declared with `noexcept`
* Source object is lvalue. Only rvalues can be moved. Item 25 mentions few exceptions. 
