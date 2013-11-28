components
==========

Motivation.

The main motivation for the Components class is handling default arguments. This implementation is designed (unlike boost.parameter) to support several sets of default arguments. It often happens that a function takes many arguments, but only some of them are really needed, the rest having default values. The other motivation is that sometimes one would like to keep all the arguments grouped together. This way arguments can be easily moved around. Another advantage of this design is that we can keep many different configurations of default arguments. An instance of the Components class represents one configuration of default arguments.
Motivating example

Assume that we'd like to write a function which takes three functors: Init, Start and Stop. We are going to introduce the interface using the Components class.

Very basic usage:

As a library developer (or a function provider) we introduce the following interface

    template <typename DoStuffComponents>
    void do_stuff(DoStuffComponents comps);

we introduce the names of the needed components

    struct Init;
    struct Start;
    struct Stop;

and specify the appropriate Components class

    template <typename... Args>
    using DoStuffComponents = Components<Init, Start, Stop>::type<Args...>;

Later on a user can run the function as follows

    DoStuffComponents<MyInitImpl, MyStartImpl, MyStopImpl> doStuffComponents;
    do_stuff(doStuffComponents);

Here we assume that all the implementations have default constructors, however this is not mandatory. There are different ways of initializing components(see section:Constructing Components for more details).

How to get the parameters back from DoStuffComponents:

    doStuffComponents.get<Init>(); //getting Init component

    MyInitImpl anotherImplementation(42);
    doStuffComponents.set<Init>(anotherImplementation); //setting Init component

    doStuffComponents.call<Start>("hello world"); // you can directly call a component if it is a functor

How to provide default parameters:

You can define DoStuffComponents as follows

    template <typename... Args>
    using DoStuffComponents = Components<Init, NameWithDefault<Start, DefaultStart>, NameWithDefault<Stop, DefaultStop>>::type<Args...>;

Later on a user can construct the DoStuffComponents as follows

    DoStuffComponents<MyInitImpl> doStuffComponents1;
    DoStuffComponents<MyInitImpl, MyStartImpl> doStuffComponents2;
    DoStuffComponents<MyInitImpl, MyStartImpl, MyStopImpl> doStuffComponents3;

This is the main motivation but the library consists of much more handy ways of manipulating the Components.
Defining Components

The type of the component can be any type including references. Default parameters can be specified for any number of last components. One can define components using template aliasing (preferred)

    //inside library:
    template <typename... Args>
    using DoStuffComponents = Components<Init, Start, Stop>::type<Args...>;

    //user:
    DoStuffComponents<MyInitImpl, MyStartImpl, MyStopImpl> doStuffComponents;

This can also be done without template aliasing

    //inside library:
    typedef Components<Init, Start, Stop> DoStuffComponents;

    //user:
    DoStuffComponents::type<MyInitImpl, MyStartImpl, MyStopImpl> doStuffComponents;

Constructing Components

There are several ways of contructing components:

    One can provide any number of arguments. The kth argument has to be convertible to the kth component.

        //inside library:
        template <typename... Args>
        using DoStuffComponents = Components<Init, Start, Stop>::type<Args...>;

        //user:
        typedef DoStuffComponents<double, int, int> MyDoStuffComponents;

        MyDoStuffComponents doStuffComponents;
        MyDoStuffComponents doStuffComponents(1,2);
        int a;
        MyDoStuffComponents doStuffComponents(a);

    One can provide any object and a CopyTag. This tag indicates, that the passed object has get<Name> member functions for some Names.

        template <typename... Args>
        using DoStuffComponents = Components<Init, Start, Stop>::type<Args...>;
        typedef DoStuffComponents<int, int, int> Big;

        template <typename... Args>
        using SmallDoStuffComponents = Components<Start, Stop>::type<Args...>;
        typedef SmallDoStuffComponents<int, int> Small;

        Small small(1,2);
        Big big(small, CopyTag());

        Small small2(big, CopyTag());

    One can make object providing some arguments by name.

        template <typename... Args>
        using DoStuffComponents = Components<Init, Start, Stop>::type<Args...>;
        typedef DoStuffComponents<int, int, int> MyDoStuffComps;

        auto m = MyDoStuffComps::make<Init, Stop>(7, 2); // The start component has default int value (actually as build in it might be uninitialized);

    The user does not have to provide any type at all.

        typedef Components<Init, Start, Stop> DoStuffComponents;

        int a;
        auto myComps = DoStuffComponents::make_components(1, a, std::ref(a));

    If the deduced type should be reference, the std::ref wraper should be used.

The important thing is that components do not have to be default constructible unless it is acctually default constructed.
Replacing Components

There is a way to replace component for given components.

    template <typename... Args>
    using DoStuffComponents = Components<Init, Start, Stop>::type<Args...>;
    typedef DoStuffComponents<int, int, int> SomeDoStuffComps;

    typedef ReplacedType<Start, double, SomeDoStuffComps>::type Replaced; // Start type is changed from int to double

    SomeDoStuffComps comps;
    double d(5);
    Replaced replaced = replace<Start>(d, comps); //replacing component

Comparison to other libraries

    Boost.Parameter

    There are two main differences between boost.parameter and Components class
        The Components, supports different sets of default arguments
        The Components is written in pure C++ (no macros).

    The Components is not designed to replacing boost.parameter but in many cases (e.g. when given arguments are functors or when given arguments can depend on each other) it might be more natural design.

    Boost.Fusion

    Although the idea of Components is very similar to boost fusion, the Components gives some functionalities that are not offered by Boost.Fusion. The example is different initialization methods (see section:Constructing Components for more details).

Supported Compilers

    clang 3.2
    g++ 4.7.2

Msvc is not supported, because of lack of the template aliasing.
Iterating Over Components class (TODO)
