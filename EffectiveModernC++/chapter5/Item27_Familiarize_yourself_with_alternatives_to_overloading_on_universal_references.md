template<typename T>
void logAndAdd(T&& name)
{
	auto now = std::chrono::system_clock::now();
	log(now, "logAndAdd");
	names.emplace(std::forward<T>(name));

	// if type is int we will be doing logAndAdd(int idx)
}

void logAndAdd(int idx) 
{
	auto now = std::chrono::system_clock::now();
	log(now, "logAndAdd");
	names.emplace(nameFromIdx(idx));
}

/* ---------------------------------------------------- */
cout << is_integral<char>() << endl; 
cout << is_integral<int>() << endl; 
cout << is_integral<uint32_t>() << endl; 
cout << is_integral<uint64_t>() << endl; 
cout << is_integral<double>() << endl; 
cout << is_integral<float>() << endl; 

cout << is_integral<char&>() << endl; 
cout << is_integral<int&>() << endl; 
cout << is_integral<uint32_t&>() << endl; 
cout << is_integral<uint64_t&>() << endl; 
cout << is_integral<double&>() << endl;
cout << is_integral<float&>() << endl; 

cout << is_integral<remove_reference<int&>::type>() << endl; 

/* ---------------------------------------------------- */


// tag dispatch
template<typename T> 
void logAndAddImpl(T&& name, std::false_type) 
{ 
	// T is double 
    names.emplace(std::forward<T>(name));
}

void logAndAddImpl(int idx, std::true_type)
{ 
	logAndAdd(nameFromIdx(idx)); 
}

template<typename T>
void logAndAdd(T&& name)
{
    logAndAddImpl(forward<T>(name), 
                  is_integral<typename remove_reference<T>::type>());
}

/* ---------------------------------------------------- */
