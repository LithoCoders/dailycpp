```c++
class Base {
public:
virtual void doWork(); // base class virtual function
…
};
class Derived: public Base {
public:
virtual void doWork(); // overrides Base::doWork
… // ("virtual" is optional here)
}; 
std::unique_ptr<Base> upb = // create base class pointer
std::make_unique<Derived>(); // to derived class object;
// see Item 21 for info on
… // std::make_unique
upb->doWork(); // call doWork through base class ptr; derived class function is invoked
----------------------------------------------------------------------------------------------------------
For overriding to occur, several requirements must be met:
• The base class function must be virtual.
• The base and derived function names must be identical (except in the case of
destructors).
• The parameter types of the base and derived functions must be identical.
• The constness of the base and derived functions must be identical.
• The return types and exception specifications of the base and derived functions
must be compatible.
------------------------------                       --------------------------------------------------------
                              -----------------------
For C++11:
-The functions’ reference qualifiers must be identical
class Widget {
public:
…
void doWork() &; // this version of doWork applies only when *this is an lvalue
void doWork() &&; // this version of doWork applies only when *this is an rvalue
}; 
…
Widget makeWidget(); // factory function (returns rvalue)
Widget w; // normal object (an lvalue)
…
w.doWork(); // calls Widget::doWork for lvalues
// (i.e., Widget::doWork &)
makeWidget().doWork(); // calls Widget::doWork for rvalues (i.e., Widget::doWork &&)

if a virtual function in a base class has a reference qualifier, derived class overrides of that function must have exactly 
the same reference qualifier. If they don’t, the declared functions will still exist in the derived class,
but they won’t override anything in the base class.

//How overriding could be mess(i)Y
class Base {
public:
virtual void mf1() const;
virtual void mf2(int x);
virtual void mf3() &;
void mf4() const;
};
class Derived: public Base {
public:
virtual void mf1();
virtual void mf2(unsigned int x);
virtual void mf3() &&;
void mf4() const;
};
---------------------------------------------- Hint-----------------------------------------------------------------------------
- mf1 is declared const in Base, but not in Derived.
• mf2 takes an int in Base, but an unsigned int in Derived.
• mf3 is lvalue-qualified in Base, but rvalue-qualified in Derived.
• mf4 isn’t declared virtual in Base.
C++11 gives you a way to make explicit that a derived class function is supposed to override a base class version: declare it override. 


//Solution using override!
class Derived: public Base {
public:
virtual void mf1() override;
virtual void mf2(unsigned int x) override;
virtual void mf3() && override;
virtual void mf4() const override;
};
---->> Compilation Error

-----------------------------------------------Solution-------------------------------------------------------------------------------
class Base {
public:
virtual void mf1() const;
virtual void mf2(int x);
virtual void mf3() &;
virtual void mf4() const;
};
class Derived: public Base {
public:
virtual void mf1() const override;
virtual void mf2(int x) override;
virtual void mf3() & override;
void mf4() const override; // adding "virtual" is OK,
};
```
