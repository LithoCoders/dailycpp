Initializing variables of C++ built-in types:
```c++
int i = 0;
int i{0};
int i = {0}; // same as int i{0};
int i(0);
```

Initialization of std containers using curly braces:
```c++
vector<int> vv{4, 5, 6};
```

Some useful knowledge:
```c++
Tuna on; // default ctor is called
Tuna on(); // error (it is a function returning Tuna object, so-called 'most C++ vexing parse')
Tuna on{}; // possible using curly braces, still default ctor
Tuna ob2 = ob; // copy ctor is called
Tuna ob3(ob); // copy ctor
ob = ob2; // copy assignment operator
```

```c++
struct Tuna
{
	Tuna(){}
	Tuna(const Tuna & ob){} // it is copy ctor
	Tuna & operator=(const Tuna & ob); // copy assignemnt
}
```


Be aware about implicit narrowing conversions:
```c++
double x, y;
int sum = x + y;
int sum{x + y}; // error (but not for all compilers. Modern versions accept it)
```

```c++
struct Tuna
{
	Tuna(int i, bool b); // 1st ctor
	Tuna(int i, double b); // 2nd ctor
}

Tuna ob(4, 6.7); // 2nd is called
Tuna ob{4, 6.7}; // 2nd is called
Tuna ob(4, true); // 1st is called
Tuna ob{4, true}; // 1st is called
```

Let's extend our class with ctor which has initializer_list:
```c++
struct Tuna
{
	Tuna(int i, bool b);
	Tuna(int i, double b);
	Tuna(initializer_list<long double> ii);
}

Tuna ob(4, 6.7); // 2nd ctor is called
Tuna ob{4, 6.7}; // ! 3rd ctor is called (implicit conversion is taking place: int and double to long double)
Tuna ob{4, true}; // ! 3rd ctor is called (implicit conversion is taking place: int and bool to long double)
```

```c++
struct Tuna
{
	Tuna(int i, bool b);
	Tuna(int i, double b);
	Tuna(initializer_list<long double> ii);
	operator float() const {return 3.0f;}; // conversion of Tuna object to a float of value 3.0 
	operator()(); // this is overloading of ()
}
```

With that conversion, it is possible to have:

```c++
// the function which requires a float input is defined:
void func(float para);

Tuna ob(2, 4.5);
// we can then call func() because ob can be represented as a float
func(ob); 

Tuna ob2(ob); //copy ctor is called
Tuna ob2{ob}; // 3rd ctor is called (conversion from Tuna to float and then to std::initializer_list<long double> is taking place. std::initializer_list has only one value of 3.0 long double)
```

std::initializer_list<bool> constructor
```c++
struct Tuna
{
	Tuna(int i, bool b);
	Tuna(int i, double b);
	Tuna(initializer_list<bool> ii);	
}

Tuna t{10, 5.0}; // does not work since braced initialization does not allow implict narrowing conversion from int(10) and double(5.0) to bools. !!! Compiler insists to use braced initialization if there is such a conversion. Conversion allowed or not is other problem!!! 
```

std::initializer_list<string> constructor
```c++
struct Tuna
{
	Tuna(int i, bool b);
	Tuna(int i, double b);
	Tuna(initializer_list<string> ii);	
}	
Tuna t{10, 5.0}; Call 2nd ctor. There’s no way to convert the types of the arguments in a braced initializer to the type in a std::initializer_list, compilers fall back on normal overload resolution.
```

```c++
struct Tuna
{
	Tuna();
	Tuna(initializer_list<string> ii);	
}
Tuna t; // default ctor
Tuna t2{}; //default ctor too
Tuna t3(); // vexing, not allowed!!! declaring a function
Tuna t4({}); //call std::initializer_list ctor with empty list
Tuna t5{{}}; this too
```

Exception with std::vector initialization. Documented in std document.
```c++
std::vector<int>	 vi (1,2);  // a vector of one element has default value of 2
std::vector<int> 	 vil{1, 2}; // a vector of two elements have value of 1 and 2
```

variadict template
```c++
tbd
```
