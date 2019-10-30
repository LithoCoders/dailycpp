*Credits to ISLA*

--- TO BE CLEANED---
using FilterContainer = // see Item 9 for
std::vector<std::function<bool(int)>>;
FilterContainer filters;
void addDivisorFilter()
{
auto calc1 = computeSomeValue1();
auto calc2 = computeSomeValue2();
auto divisor = computeDivisor(calc1, calc2);
filters.emplace_back( // danger!
[&divisor](int value) { return value % divisor == 0; } // ref to
); // divisor
}


----------------------------------------------------------------------------
template<typename C>
void workWithContainer(const C& container)
{
                auto calc1 = computeSomeValue1(); // as above
                auto calc2 = computeSomeValue2(); // as above
                auto divisor = computeDivisor(calc1, calc2); // as above
                using ContElemT = typename C::value_type; // type of
                // elements in
                // container
                using std::begin; // for
                using std::end; // genericity;
                // see Item 13
                if (std::all_of( // if all values
                                                                                begin(container), end(container), // in container
                                                                                [&](const ContElemT& value) // are multiples
                                                                                { return value % divisor == 0; }) // of divisor...
                ) {
                … // they are...
} else {
… // at least one
} // isn't...
}

//
// Long-term, it’s simply better software engineering to explicitly list the local variables
// and parameters that a lambda depends on.
//
if (std::all_of(begin(container), end(container),
[&](const auto& value) // C++14
{ return value % divisor == 0; }))

filters.emplace_back( // now
[=](int value) { return value % divisor == 0; } // divisor
); // can't
// dangle
-------------------------------------------------------------------------------------------------
///Pass by value
class Widget {
                public:
                … // ctors, etc.
                void addFilter() const; // add an entry to filters
                private:
                                int divisor; // used in Widget's filter
};

void Widget::addFilter() const
{
filters.emplace_back(
[=](int value) { return value % divisor == 0; }
);
}
void Widget::addFilter() const
{
filters.emplace_back(
[divisor](int value) // error! no local
{ return value % divisor == 0; } // divisor to capture
);
}
void Widget::addFilter() const
{
auto currentObjectPtr = this;
filters.emplace_back(
[currentObjectPtr](int value)
{ return value % currentObjectPtr->divisor == 0; }
);
}
using FilterContainer = // as before
std::vector<std::function<bool(int)>>;
FilterContainer filters; // as before
void doSomeWork()
{
auto pw = // create Widget; see
std::make_unique<Widget>(); // Item 21 for
// std::make_unique
pw->addFilter(); // add filter that uses
// Widget::divisor
…
} // destroy Widget; filters
// now holds dangling pointer!

void Widget::addFilter() const
{
filters.emplace_back( // C++14:
[divisor = divisor](int value) // copy divisor to closure
{ return value % divisor == 0; } // use the copy
);
}
void addDivisorFilter()
{
static auto calc1 = computeSomeValue1(); // now static
static auto calc2 = computeSomeValue2(); // now static
static auto divisor = // now static
computeDivisor(calc1, calc2);
filters.emplace_back(
[=](int value) // captures nothing!
{ return value % divisor == 0; } // refers to above static
);
++divisor; // modify divisor
}
