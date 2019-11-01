```c++
int test(int val)
{
    return val;
}

int main()
{
    int res = test(5);
    cout << res << endl;
    return 0;
}
// ----------------------------------- //

int & test(int val)
{
    return val;
}

int main()
{
    int res = test(5);
    cout << res << endl;
    return 0;
}
// ----------------------------------- //
auto test(int val) -> decltype(val)
{
    return val;
}

int main()
{
    int res = test(5);
    cout << res << endl;
    return 0;
}
// ----------------------------------- //
auto test(int val) -> decltype((val))
{
    return val;
}

int main()
{
    int res = test(5);
    cout << res << endl;
    return 0;
}
// ----------------------------------- //
auto test(int val) -> decltype(5)
{
    return val;
}

int main()
{
    int res = test(5);
    cout << res << endl;
    return 0;
}
// ----------------------------------- //
auto test(int val) -> decltype((5))
{
    return val;
}

int main()
{
    int res = test(5);
    cout << res << endl;
    return 0;
}
// ----------------------------------- //
#include <iostream>
#include <utility>
#include <map>
#include <string>
#include <algorithm>
#include <type_traits>

using namespace std;

using DBxItem = map<string, int>;


int main()
{
    DBxItem data;
    data.insert(make_pair("tuna", 14));
    data.insert(make_pair("salmon", 12));
    data.insert(make_pair("apple", 10));
    
    for (auto & it : data) // what is auto type ?
    {
        cout << it.first << endl;
    }
}
// ----------------------------------- //
#include <iostream>
#include <utility>
#include <map>
#include <string>
#include <algorithm>
#include <type_traits>

using namespace std;

using DBxItem = map<string, int>;


int main()
{
    DBxItem data;
    data.insert(make_pair("tuna", 14));
    data.insert(make_pair("salmon", 12));
    data.insert(make_pair("apple", 10));
    
    auto res = min_element(data.begin(), data.end(),
                            [](DBxItem::value_type & item1, DBxItem::value_type &item2) 
                            -> bool { return item1.second < item2.second; });
    cout << res->first << " " << res->second << endl;
    return 0;
}

// ----------------------------------- //

#include <iostream>
#include <utility>
#include <map>
#include <string>
#include <algorithm>
#include <type_traits>

using namespace std;

using DBxItem = map<string, int> ;


int main()
{
    DBxItem data;
    data.insert(make_pair("tuna", 14));
    data.insert(make_pair("salmon", 12));
    data.insert(make_pair("apple", 10));
    
    auto res = min_element(data.begin(), data.end(),
                            [](decltype(data)::value_type & item1, decltype(data)::value_type &item2) 
                            -> bool { return item1.second < item2.second; });
    cout << res->first << " " << res->second << endl;
    return 0;
}
```
