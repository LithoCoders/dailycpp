template<typename T>
void f(PT param)
PT is ParamType

f(x) -> compiler will deduce type for:
	- T
	- ParamType
	Note that T and ParamType can be different (due to const, volatile etc)


Case 1: ParamType is reference or pointer

template<typename T>
void f(T & param);

int x = 27;  f(x);	
	T: int
	PT: int &

const int cx = x; f(cx);
	T: const int
	PT: const int &

const int & rx = x; f(rx);
	T: const int
	PT: const int &


template<typename T>
void f(const T & param);

f(x);
	T: int
	PT: const int &

f(cx);
	T: int
	PT: const int &

f(rx);
	T: int
	PT: const int &


template<typename T>
void f(T * param);

int x = 27; f(&x);
	T: int
	PT: int *

const int * px = &x; f(px);
	T: const int
	PT: const int *


Case 2: ParamType is universal reference

template<typename T>
void f(T && param);

int x = 27; f(x);
	T: int &
	PT: int &

const int cx = x; f(cx);
	T: const int &
	PT: const int &

const int & rx = x; f(rx);
	T: const int &
	PT: const int &

f(27);
	T: int
	PT: int &&
