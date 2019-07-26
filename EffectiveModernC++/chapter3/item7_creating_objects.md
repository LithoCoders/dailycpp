Initializing variables of C++ built-in types:
```c++
int i = 0;
int i{0};
int i = {0};
int i(0);
```

Initialization of std containers using curly braces:
```c++
vector<int> vv{4, 5, 6};
```

Some useful knowledge:
```c++
Tuna on; // default ctor is called
Tuna on(); // error (it is a function returning Tuna ob)
Tuna on{}; // possible using curly braces
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


Be aware about implicit conversions:
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
Tuna ob{4, 6.7}; // ! 3rd ctor is called (implicit conversion is taking place)
Tuna ob{4, true}; // ! 3rd ctor is called (implicit conversion is taking place)
```

```c++
struct Tuna
{
	Tuna(int i, bool b);
	Tuna(int i, double b);
	Tuna(initializer_list<long double> ii);
	operator float() const; // conversion of ob to float
	operator()(); // this is overloading of ()
}
```

With that conversion, it is possible to have:

```c++
// the function which requires a float input is defined:
void func(float in1);

Tuna ob(2, 4.5);
// we can then call:
func(ob);

Tuna ob2(ob); //copy ctor is called
Tuna ob2{ob}; // 3rd ctor is called (conversion from Tuna to float and then to long double is taking place)
```

