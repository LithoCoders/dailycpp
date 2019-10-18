```c++
#include <utility>    
#include <iostream>    
#include <set>
#include <string>

std::multiset<std::string> names;

template<typename T>
void logAndAddToSet(T&& name)
{
    // do logging
    names.emplace(std::forward<T>(name));
    std::cout << __PRETTY_FUNCTION__ << std::endl;
}

void logAndAddToSet(int index)
{
    // do logging
    //find name from index
    std::string a = "foo";
    names.emplace(a);
    std::cout << __PRETTY_FUNCTION__ << std::endl;
}

int main () {
    std::string a_lvalue("lvalue");
    logAndAddToSet(a_lvalue);
    
    logAndAddToSet(std::string("rvalue"));
    
    logAndAddToSet("string literal");

    int i = 1;
    logAndAddToSet(i);
    
    short s = 1;
    logAndAddToSet(s);
    
    std::cout << "Set contains: ";
    for (auto i : names)
        std::cout << i << "-";

    std::cout << std::endl;
  return 0;
}
class Person
{
	public:
		template<typename T>
		explicit person(T && name):_name(std::forward<T>(name)) {
			std::cout<<"The template"<<std::endl;
		}
		
		explicit person(int ind):_name("FOO") {
			std::cout<<"The specific int"<<std::endl;
		}
		
	private:
		std::string _name;
	
};
class SpecialPerson : public Person 
{
};

int main () 
{
	Person p("ISLAM");
	//const Person P_c("ISLAM2");
	int i = 1000
	short s = 12;
	Person p2(i);
	Person p3(s);
	
	Person my_new(p);
}
```
