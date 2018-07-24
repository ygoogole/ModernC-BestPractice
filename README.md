# Modern C++ Best Practice
A list of best practise and idioms in C++ 11/14

## Target

This checklist will NOT talk about coding conventions like “4 spaces as default indent” or “no extra space/tab at end of line”, etc. It aims at reviewing best practice and idioms of using C++11/14 for better compliance, performance and robustness to improve general code quality.

A lightweight c++11 code linter is implemented [here](https://github.com/ygoogole/cpp11lint) out of this checklist to help statically identify defect pattern, which can be used before/in decoder peer review process. However, due to complexity of identifying specific code pattern, not all rules can be covered in code linter. Each rule check will have a confidence score (5 the highest) to indicate how likely this check is supposed to be correct.



Rule of Thumb of C++11/14

Item 1:
Use std features, not boost alternatives

A few examples:

boost
std
unordered_map
unordered_map
shared_ptr
shared_ptr
scoped_ptr
unique_ptr
BOOST_FOREACH
for
thread
thread
mutex
mutex


Item 2:
Use “auto”, not explicit type declarations, with variable initialized, unless of undesired type deduction

std::vector<Foo>::const_iterator iter = vec.begin();// too much typing
auto iter = vec.begin(); // good
auto x; // bad, x not initialised
auto x2 = 0; // good
auto x3 = static_cast<int>(double_val);

std::function<bool (const std::unique_ptr<Widget>&, const
               std::unique_ptr<Widget>&)> // too much typing
f = [](const std::unique_ptr<Widget>& p1,
const std::unique_ptr<Widget>& p2)
 	{ return *p1 < *p2; };

Item 3:
Prefer {} when creating objects but be aware of different behavior to ()
{} can also prevent “most vexing parse” problem.

double x,y;
int s1 = x+y; // bad, double truncated to int, no error
int s2(x+y); // same
int s3{x+y}; // error, narrow conversion spotted
std::vector<int> v(5,3); // [3,3,3,3,3]
std::vector<int> v{5,3}; // [5,3]

Item 4:
Use nullptr not NULL or 0

void f(int);
void f(bool);
void f(void*); 	// three overloads of f
f(0); 		// calls f(int), not f(void*)
f(NULL); 		// might not compile, but typically calls
// f(int). Never calls f(void*)
f(nullptr);		// calls f(void*) overload

Item 5:
Use type alias “using” not typedef

typedef does not support templatization, but alias declarations do.
consider below example that a list with custom allocator and widget class with such list as a member:

//using type alias:
template<typename T>
using MyAllocList = std::list<T, MyAlloc<T>>;

MyAllocList<Widget> lw; // templatization

template<typename T>
class Widget {
private:
    MyAllocList<T> list;
};

// using typedef
template<typename T>
struct MyAllocList {
    typedef std::list<T, MyAlloc<T>> type;
};

MyAllocList<Widget>::type lw;

template<typename T>
class Widget {
private:
    typename MyAllocList<T>::type list;
};

Item 6:
Use “enum class” not “enum”

Item 7:
Use “delete” for not defined private functions

class Foo {
public:
    Foo(const Foo &) = delete; // good

private:
    Foo(const Foo &); // bad
};

Item 8:
Declare overriding functions as “override”, for each derived function. “virtual” keyword is only required for top base class function, not for derived ones.

Item 9:
Use “noexcept” not “throw()” if function does not throw exception. Always use “noexcept” for destructor.

Item 10:
Use “constexpr” whenever possible, no “inline” needed if used.

Item 11: ??
Explicitly declare “default” member functions 

class Foo {
public:
    Foo() = default;
    Foo(const Foo &) = default;
    Foo& operator=(const Foo &) = default;
    Foo(Foo &&) = default;
    Foo& operator=(Foo &&) = default;
    ~Foo() = default;
};

Item 12:
Use “std::make_shared/make_unique” not direct “new”
*std::make_unique is available since C++14

auto bar = std::shared_ptr<Widget>(new Widget) // less efficient, two  
                                               // times memory allocation
                                               // not exception safe
auto foo = std::make_shared<Widget>(); // good


Item 13:
Use lambdas not std::bind/boost::bind

Item 14:
Use task-based std::async, not std::thread for simple work. Use std::promise & std::future for one-shot simple event communication.

Item 15:
Use std::atomic for concurrency

Item 16:
Consider using emplacement functions instead of push/insert in STL contains like vector, stack, queue, map, etc.

Item 17:
Use constructor delegation (call another ctor in current ctor), not “init” function called in all ctor.

class Widget {
public:
    Widget()
        : foo()
        , bar() {}

    Widget(int i)
        : Widget() // calling default ctor
        , obj(i) {}
       
private:
    Foo foo;
    Bar bar;
    Object obj;
};

Item 18:
Don’t define ctor that only initialise data members, use in-class member initializer, with default ctor

// bad
class Foo {
public:
    Foo()
        : x(0) {}

private:
    int x;
};

// good
class Foo {
public:
    Foo() = default; // more efficient than you write it
private:
    int x = 0;
};

Item 19:
Use static local variable in singleton, not double checked locking idiom.

// old
// double check null for better performance
Singleton* Singleton::getInstance() {
 	if (m_instance == NULL) {
    	    Lock lock;
    	    if (m_instance == NULL) {
        	m_instance = new Singleton;
    	    }
	}
	return m_instance;
}

// c++ 11
// local static variable will be guaranteed to be initialized completely
// before concurrent access

Singleton& Singleton::getInstance() {
	static Singleton m_instance;
	return m_instance;
}


Item 20:
Use std::unique_ptr by default unless std::shared_ptr is needed

Item 21:
Use “extern” to avoid redundant template instantiation
