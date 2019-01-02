# CALLBACKS IN C++ USING TEMPLATE FUNCTORS
**Copyright 1994 Rich Hickey**

**INTRODUCTION**

- One of the many promises of Object-Oriented programming is that it will allow for plug-and-play software design with re-usable components. Designers will pull objects from their library 'shelves' and hook them together to make software. In C++, this hooking together of components can be tricky, particulary if they are separately designed. We are still a long way from interoperable libraries and application components. Callbacks provide a mechanism whereby independently developed objects may be connected together. They are vital for plug and play programming, since the likelihood of Vendor A implementing their library in terms of Vendor B's classes, or your home-brewed classes, is nil.

- Callbacks are in wide use, however current implementations differ and most suffer from shortcomings, not the least of which is their lack of generality. This article describes what callbacks are, how they are used, and the criteria for a good callback mechanism. It summarizes current callback methods and their weaknesses. It then describes a flexible, powerful and easy-to-use callback technique based on template functors - objects that behave like functions.

**CALLBACK FUNDAMENTALS**

*What Are Callbacks?*

- When designing application or sub-system specific components we often know all of the classes with which the component will interact and thus explicity code interfaces in terms of those classes. When designing general purpose or library components however, it is often necessary or desirable to put in hooks for calling unknown objects. What is required is a way for one component to call another without having been written in terms of, or with knowledge of, the other component's type. Such a 'type-blind' call mechanism is often referred to as a callback.

- A callback might be used for simple notification, two-way communication, or to distribute work in a process. For instance an application developer might want to have a Button component in a GUI library call an application-specific object when clicked upon. The designer of a data entry component might want to offer the capability to call application objects for input validation. Collection classes often offer an apply() function, which 'applies' a member function of an application object to the items they contain.

- A callback, then, is a way for a component designer to offer a generic connection point which developers can use to establish communication with application objects. At some subsequent point, the component 'calls back' the application object. The communication takes the form of a function call, since this is the way objects interact in C++.

- Callbacks are useful in many contexts. If you use any commercial class libraries you have probably seen at least one mechanism for providing callbacks. All callback implementations must address a fundamental problem posed by the C++ type system: How can you build a component such that it can call a member function of another object whose type is unknown at the time the component is designed? C++'s type system requires that we know something of the type of any object whose member functions we wish to call, and is often criticized by fans of other OO languages as being too inflexible to support true component-based design, since all the components have to 'know' about each other. C++'s strong typing has too many advantages to abandon, but addressing this apparent lack of flexibility may encourage the proliferation of robust and interoperable class libraries.

- C++ is in fact quite flexible, and the mechanism presented here leverages its flexibility to provide this functionality without language extension. In particular, templates supply a powerful tool for solving problems such as this. If you thought templates were only for container classes, read on!

*Callback Terminology*

- There are three elements in any callback mechanism - the caller, the callback function, and the callee.

- The caller is usually an instance of some class, for instance a library component (although it could be a function, like qsort()), that provides or requires the callback; i.e. it can, or must, call some third party code to perform its work, and uses the callback mechanism to do so. As far as the designer of the caller is concerned, the callback is just a way to invoke a process, referred to here as the callback function. The caller determines the signature of the callback function i.e. its argument(s) and return types. This makes sense, because it is the caller that has the work to do, or the information to convey. For instance, in the examples above, the Button class may want a callback function with no arguments and no return. It is a simple notification function used by the Button to indicate it has been clicked upon. The DataEntryField component might want to pass a String to the callback function and get a Boolean return.

- A caller may require the callback for just the duration of one function, as with ANSI C's qsort(), or may want to hold on to the callback in order to call back at some later time, as with the Button class.

- The callee is usually a member function of an object of some class, but it can also be a stand-alone function or static member function, that the application designer wishes to be called by the caller component. Note that in the case of a non-static member function a particular object/member-function pair is the callee. The function to be called must be compatible with the signature of the callback function specified by the caller.

*Criteria for a Good Callback Mechanism*

- A callback mechanism in the object oriented model should support both component and application design. Component designers should have a standard, off-the-shelf way of providing callback services, requiring no invention on their part. Flexibility in specifying the number and types of argument and return values should be provided. Since the component may be designed for use in as-yet-unthought-of applications, the component designer should neither need to know, nor dictate, the types of the objects which may be 'called back' by the component.

- Application developers, given a component with this standard callback mechanism and some instance of a class with a member function compatible with the callback function signature, should have to do no custom 'glue' coding in order to connect the two together. Nor should they have to modify the callee class or hand-derive a new class. If they want to have the callback invoke a stand-alone, non-member function, that should be supported as well.

#### To support this behavior the callback mechanism should be:

**Object Oriented** - Our applications are built with objects. In a C++ application most functionality is contained in member functions, which cannot be invoked via normal ptr-to-functions. Non-static member functions operate upon objects, which have state. Calling such functions is more than just invoking a process, it is operating upon a particular object, thus an object-oriented callback must contain information about which object to call.

**Type Safe** - Type safety is a fundamental feature and benefit of C++ and any robust C++ callback mechanism must be type safe. That means we must ensure that objects are used in compliance with their specified interfaces, and that type rules are enforced for arguments, return values, and conversions. The best way to ensure this is to have the compiler do the work at compile time.

**Non-Coupling** - This is the fundamental goal of callbacks - to allow components designed in ignorance of each other to be connected together. If the mechanism somehow introduces a dependancy between caller and callee it has failed in its basic mission.

**Non-Type-Intrusive** - Some mechanisms for doing callbacks require a modification to, or derivation of, the caller or callee types. The fact that an object is connected to another object in a particular application often has nothing to do with its type. As we'll see below, mechanisms that are type intrusive can reduce the flexibility and increase the complexity of application code.

**Generic** - The primary differences between different callback situations are the types involved. This suggests that the callback mechanism should be parameterized using templates. Templates insure consistent interfaces and names in all callback situations, and provide a way to have any necessary support code be generated by the compiler, not the user.

**Flexible** - Experience has shown that callback systems that require an exact match between callback function and callee function signatures are too rigid for real-world use. For instance you may encounter a callback that passes a Derived * that you want to connect to a callee function that takes a Base *.

#### CURRENT MECHANISMS

**Function Model**

- The simplest callback mechanism is a pointer-to-function, a la ANSI C's qsort(). Getting a stand-alone function to act upon a particular object, however, usually involves kludges like using static or global pointers to indicate the target object, or having the callback function take an extra parameter (usually a pointer to the object to act upon). The static/global pointer method breaks down when the callback relationship exists across calls, i.e. 'I want to connect this Button to this X and this other Button to this other X, for the duration of the app'. The extra paramter method, if done type-safely, introduces undesirable coupling between the caller and callee types.

- <font color=gray size=2>qsort()</font> achieves its genericity by foregoing type safety. i.e., in order for it to be ignorant of the types it is manipulating it takes untyped <font color=gray size=2>(void *) </font> arguments. There is nothing to prevent someone from calling <font color=gray size=2>qsort()</font> on an array of apples and passing a pointer to a function that compares oranges!

- An example of this typeless mechanism you'll frequently see is the 'apply' function in collections. The purpose of an apply function is to allow a developer to pass a callback to a collection and have it be 'applied' to (called on) each item in the collection. Unfortunately it often looks like this:

```
 void apply(void (*func)(T &theItem,void *extraStuff),void *theStuff);
```
                        
- Chances are really good you don't have a function like func sitting around, so you'll have to write one (lots of casting required). And make sure you pass it the right stuff. Ugh.

*Single Rooted Hierarchy*

- Beware of callback mechanisms that appear type safe but are in fact not. These mechanisms usually involve some base-of-all-classes like Object or EventHandler, and utilize casts from ptr-to-member-of-derived to ptr-to-member-of-base. Experience has indicated that single-rooted systems are unworkable if components are to come from multiple sources.

*Parameterize the Caller*

- The component designer could parameterize the component on the type of the callee. Such parameterization is inappropriate in many situations and callbacks are one of them. Consider:

```

	class Button{
	public:
		virtual void click();
		//...
	};

	template <class T>
	class ButtonThatCallsBack:public class Button{
	public:
		ButtonThatCalls(T *who,void (T::*func)(void)):
			callee(who),callback(func){}
		void click(){
			(callee->*callback)();
		}
	private:
		T *callee;
	        void (T::*callback)(void);
	};
	
	class CDPlayer{
	public:
		void play();
		//...
	};
	
	//Connect a CDPlayer and a Button
	CDPlayer cd;
	ButtonThatCallsBack<CDPlayer> button(&cd,&CDPlayer::play);
	button.click();	//calls cd.play()

```

- A <font color=gray size=2>ButtonThatCallsBack<CDPlayer></font> would thus 'know' about <font color=gray size=2>CDPlayer</font> and provides an interface explicitly based on it. The problem is that this introduces rigidity in the system in that the callee type becomes part of the caller type, i.e. it is 'type-intrusive'. All code that creates <font color=gray size=2>ButtonThatCallsBack</font> objects must be made aware of the callee relationship, increasing coupling in the system. A <font color=gray size=2>ButtonThatCallsBack<X></font> is of a different type than a<font color=gray size=2> ButtonThatCallsBack<Y></font> , thus preventing by-value manipulation.

- If a component has many callback relationships it quickly becomes unworkable to parameterize them all. Consider a Button that wants to maintain a dynamic list of callees to be notified upon a click event. Since the callee type is built into the Button class type, this list must be either homogeneous or typeless.

- Library code cannot even create ButtonThatCallsBack objects because their instantiation depends on application types. This is a severe constraint. Consider GUI library code that reads a dialog description from a resource file and creates a Dialog object. How can it know that you want the Buttons in that Dialog to call back CDPlayers? It can't, therefore it can't create the Buttons for you.

*Callee Mix-In*

- The caller component designer can invent an abstract base class to be the target of the callback, and indicate to application developers that they mix-in this base in order to connect their class with the component. I call this the "callee mix-in."

- Here the designer of the <font color=gray size=2>Button</font> class wants to offer a click notification callback, and so defines a nested class <font color=gray size=2>Notifiable</font> with a pure virtual function <font color=gray size=2>notify()</font> that has the desired signature. Clients of the <font color=gray size=2>Button</font> class will have to pass to its constructor a pointer to a <font color=gray size=2>Notifiable</font>, which the <font color=gray size=2>Button</font> will use (at some point later on) for notification of clicks:

```

	class Button{
	public:
		class Notifiable{
		public:
			virtual void notify()=0;
			};
		Button(Notifiable *who):callee(who){}
		void click()
			{callee->notify();}
	private:
		Notifiable *callee;
	};
	
	Given :
	
	class CDPlayer{
	public:
		void play();
		//...
	};
```

- an application developer wishing to have a Button call back a CDPlayer would have to derive a new class from both CDPlayer and Button::Notifiable, overriding the pure virtual function to do the desired work:

```
	
	class MyCDPlayer:public CDPlayer,public Button::Notifiable{
	public:
		void notify()
			{play();}
	};
```

- and use this class rather than CDPlayer in the application:

```
	
	MyCDPlayer cd;
	Button button(&cd);
	button.click();	//calls cd.play()
```

- This mechanism is type safe, achieves the decoupling of Button and CDPlayer, and is good magazine article fodder. It is almost useless in practice, however.

- The problem with the callee mix-in is that it, too, is type-intrusive, i.e. it impacts the type of the callee, in this case by forcing derivation. This has three major flaws. First, the use of multiple inheritance, particularly if the callee is a callee of multiple components, is problematic due to name clashes etc. Second, derivation may be impossible, for instance if the application designer gets CDPlayers from an unchangeable, untouchable API (library designers note: this is a big problem with mix-in based mechanisms in general). The third problem is best demonstrated. Consider this version of <font color=gray size=2>CDPlayer</font>:

```
	
	class CDPlayer{
	public:
		void play();
		void stop();
		//...
	};
```

- It doesn't seem unreasonable to have an application where one Button calls CDPlayer::play() and another CDPlayer::stop(). The mix-in mechanism fails completely here, since it can only support a single mapping between caller/callee/member-function, i.e. MyCDPlayer can have only one notify().

**CALLBACKS USING TEMPLATE FUNCTORS**

- When I first thought about the inter-component callback problem I decided that what was needed was a language extension to support 'bound-pointers', special pointers representing information about an object and a member function of that object, storable and callable much like regular pointers to functions. ARM 5.5 commentary has a brief explanation of why bound pointers were left out.

- How would bound pointers work? Ideally you would initialize them with either a regular pointer-to-function or a reference to an object and a pointer-to-member-function. Once initialized, they would behave like normal pointer-to-functions. You could apply the function call operator() to them to invoke the function. In order to be suitable for a callback mechanism, the information about the type of the callee would _not_ be part of the type of the bound-pointer. It might look something like this:

```

	// Warning - NOT C++
	
	class Fred{
	public:
		void foo();
	};
	Fred fred;
	void (* __bound fptr)() = &fred.foo;
```

- Here <font color=gray size=2>fptr</font> is a bound-pointer to a function that takes no arguments and returns <font color=gray size=2>void</font>. Note that Fred is not part of <font color=gray size=2>fptr's</font> type. It is initialized with the object fred and a pointer-to-member-function-of-Fred, <font color=gray size=2>foo</font> . Saying:

<font color=gray size=2>fptr();</font>
would invoke <font color=gray size=2>foo</font> on <font color=gray size=2>fred</font>.

Such bound-pointers would be ideal for callbacks:

```
	
	// Warning - NOT C++
	
	class Button{
	public:
		Button(void (* __bound uponClickDoThis)() )
			:notify(uponClickDoThis)
			{}
		void click()
			{
			notify();
			}
	private:
		void (* __bound notify)();
	};
	
	class CDPlayer{
	public:
		void play();
	};
	
	CDPlayer cd;
	Button button(&cd.play);
	button.click();	    //calls cd.play()
```

- Bound-pointers would require a non-trivial language extension and some tricky compiler support. Given the extreme undesirability of any new language features I'd hardly propose bound-pointers now. Nevertheless I still consider the bound-pointer concept to be the correct solution for callbacks, and set out to see how close I could get in the current and proposed language. The result is the Callback library described below. As it turns out, the library solution can not only deliver the functionality shown above (albeit with different syntax), it proved more flexible than the language extension would have been!

- Returning from the fantasy world of language extension, the library must provide two things for the user. The first is some construct to play the role of the 'bound-pointer'. The second is some method for creating these 'bound-pointers' from either a regular pointer-to-function or an object and a pointer-to-member-function.

- In the 'bound-pointer' role we need an object that behaves like a function. Coplien has used the term functor to describe such objects. For our purposes a functor is simply an object that behaves like a pointer-to-function. It has an <font color=gray size=2>operator()</font> (the function call operator) which can be used to invoke the function to which it points. The library provides a set of template <font color=gray size=2>Functor</font> classes. They hold any necessary callee data and provide pointer-to-function like behavior. Most important, their type has no connection whatsoever to the callee type. Components define their callback interface using the <font color=gray size=2>Functor</font> classes.

- The construct provided by the library for creating functors is an overloaded template function, <font color=gray size=2>makeFunctor()</font>, which takes as arguments the callee information (either an object and a ptr-to-member-function, or a ptr-to-function) and returns something suitable for initializing a <font color=gray size=2>Functor</font> object.

The resulting mechanism is very easy to use. A complete example:

```
	#include <callback.h>	//include the callback library header
	#include <iostream.h>
	
	class Button{
	public:
		Button(const Functor0 &uponClickDoThis)
			:notify(uponClickDoThis)
			{}
		void click()
			{
			notify();	//a call to operator()
			}
	private:
		Functor0 notify;	//note - held by value
	};
	
	//Some application stuff we'd like to connect to Button:
	
	class CDPlayer{ public:
		void play(){cout<<"Playing"<<endl;}
		void stop(){cout<<"Stopped"<<endl;}
	};
	
	void wow()
		{cout<<"Wow!"<<endl;}
	
	void main()
		{
		CDPlayer cd;
	
		//makeFunctor from object and ptr-to-member-function
	
		Button playButton(makeFunctor(cd,&CDPlayer::play));
		Button stopButton(makeFunctor(cd,&CDPlayer::stop));
	
		//makeFunctor from pointer-to-function
	
		Button wowButton(makeFunctor(&wow));
	
		playButton.click();	//calls cd.play()
		stopButton.click();	//calls cd.stop()
		wowButton.click();	//calls wow()
		}
```

- Voila! A component (Button) has been connected to application objects and functions it knows nothing about and that know nothing about Button, without any custom coding, derivation or modification of the objects involved. And it's type safe.

- The Button class designer specifies the callback interface in terms of Functor0, a functor that takes no arguments and returns void. It stores the functor away in its member notify. When it comes time to call back, it simply calls operator() on the functor. This looks and feels just like a call via a pointer-to-function.

- Connecting something to a component that uses callbacks is simple. You can just initialize a Functor with the result of an appropriate call to makeFunctor(). There are two flavors of <font color=gray size=2>makeFunctor()</font>. You can call it with a ptr-to-stand-alone function:

```
	makeFunctor(&wow)
```

- OR with an object and a pointer-to-member function:

```
	makeFunctor(cd,&CDPlayer::play)
```

- I must come clean at this point, and point out that the syntax above for <font color=gray size=2>makeFunctor()</font> is possible only in the proposed language, because it requires template members (specifically, the <font color=gray size=2>Functor</font> constructors would have to be templates). In the current language the same result can be achieved by passing to <font color=gray size=2>makeFunctor()</font> a dummy parameter of type ptr-to-the-Functor-type-you-want-to-create. This iteration of the callback library requires you pass <font color=gray size=2>makeFunctor()</font> the dummy as the first parameter. Simply cast 0 to provide this argument:

```

	makeFunctor((Functor0 *)0,&wow)

	makeFunctor((Functor0 *)0,cd,&CDPlayer::play);
```

- I will use this current-language syntax from here on.

- The <font color=gray size=2>Button</font> class above only needs a callback function with no arguments that returns <font color=gray size=2>void</font>. Other components may want to pass data to the callback or get a return back. The only things distinguishing one functor from another are the number and types of the arguments to <font color=gray size=2>operator()</font> and its return type, if any. This indicates that functors can be represented in the library by (a set of) templates:

```


	//Functor classes provided by the Callback library:
	
	Functor0	//not a template - nothing to parameterize
	Functor1<P1>
	Functor2<P1,P2>
	Functor3<P1,P2,P3>
	Functor4<P1,P2,P3,P4>
	Functor0wRet<RT>
	Functor1wRet<P1,RT>
	Functor2wRet<P1,P2,RT>
	Functor3wRet<P1,P2,P3,RT>
	Functor4wRet<P1,P2,P3,P4,RT>
```

- These are parameterized by the types of their arguments (P1 etc) and return value (RT) if any. The numbering is necessary because we can't overload template class names on number of parameters. 'wRet' is appended to distinguish those with return values. Each has an operator() with the corresponding signature, for example:


```

	template <class P1>
	class Functor1{
	public:
		void operator()(P1 p1)const;
		//...
	};
	
	template <class P1,class P2,class RT>
	class Functor2wRet{
	public:
		RT operator()(P1 p1,P2 p2)const;
		//...
	};
```

- These <font color=gray size=2>Functor</font> classes are sufficient to meet the callback needs of component designers, as they offer a standard and consistent way to offer callback services, and a simple mechanism for invoking the callback function. Given these templates in the library, a component designer need only pick one with the correct number of arguments and specify the desired types as parameters. Here's the <font color=gray size=2>DataEntryField</font> that wants a validation callback that takes a <font color=gray size=2>const String &</font> and returns a <font color=gray size=2>Boolean</font>:

```
	#include <callback.h>
	
	class DataEntryField{
	public:
		DataEntryField(const Functor1wRet<const String &,Boolean> &v):
			validate(v){}
		void keyHit(const String & stringSoFar)
			{
			if(validate(stringSoFar))
				// process it etc...
			}
	private:
		Functor1wRet<const String &,Boolean> validate;
		//validate has a
		//Boolean operator()(const String &)
	};
```

- These trivial examples just scratch the surface of what you can do given a general purpose callback library such as this. Consider their application to state machines, dispatch tables etc.

- The callback library is 100% compile-time type safe. (Where compile time includes template-instantiation time). If you try to make a functor out of something that is not compatible with the functor type you will get a compiler error. All correct virtual function behavior is preserved.

- The system is also type flexible. You'll note that throughout this article I have said 'type compatible' rather than 'exactly-matching' when talking about the relationship between the callback function and the callee function. Experience has shown that requiring an exact match makes callbacks too rigid for practical use. If you have done much work with pointer-to-function based interfaces you've probably experienced the frustration of having a pointer to a function 'that would work' yet was not of the exact type required for a match.

- To provide flexibility the library supports building a functor out of a callee function that is 'type compatible' with the target functor - it need not have an exactly matching signature. By type compatible I mean a function with the same number of arguments, of types reachable from the functor's argument types by implicit conversion. The return type of the function must be implicitly convertible to the return type of the functor. A functor with no return can be built from a function with a return - the return value is safely ignored.

```

	//assumes Derived publicly derived from Base
	void foo(Base &);
	long bar(Derived &);
	
	Functor1<Derived&> f1 =
	        makeFunctor((Functor1<Derived&> *)0,&foo);
		//ok - will implicitly convert
	
	f1 = makeFunctor((Functor1<Derived&> *)0,&bar);
		//ok - ignores return
```

- Any necessary argument conversions or ignoring of returns is done by the compiler, i.e. there is no coercion done inside the mechanism or by the user. If the compiler can't get from the arguments passed to the functor to the arguments required by the callee function, the code is rejected at compile time. By allowing the compiler to do the work we get all of the normal conversions of arguments - derived to base, promotion and conversion of built-in types, and user-defined conversions.

- The type-flexibility of the library is something that would not have been available in a language extension rendition of bound pointers.

- Rounding out the functionality of the Functor classes are a default constructor that will also accept 0 as an initializer, which puts the Functor in a known 'unset' state, and a conversion to Boolean which can be used to test whether the Functor is 'set'. The Functor classes do not rely on any virtual function behavior to work, thus they can be held and copied by-value. Thus a Functor has the same ease-of-use as a regular pointer-to-function.

- At this point you know everything you need to use the callback library. All of the code is in one file, callback.h. To use a callback in a component class, simply instantiate a Functor with the desired argument types. To connect some stuff to a component that uses Functors for callbacks, simply call makeFunctor() on the stuff. Easy.

*Power Templates*

- As usual, what is easy for the user is often tricky for the implementor. Given the black-box descriptions above of the Functor classes and makeFunctor() it may be hard to swallow the claims of type-safety, transparent conversions, correct virtual function behavior etc. A look behind the curtain reveals not only how it works, but also some neat template techniques. Warning: most people find the pointer-to-member and template syntax used in the implementation daunting at first.

- Obviously some sort of magic is going on. How can the Functor class, with no knowledge of the type or signature of the callee, ensure a type safe call to it, possibly with implicit conversions of the arguments? It can't, so it doesn't. The actual work must be performed by some code that knows both the functor callback signature and everything about the callee. The trick is to get the compiler to generate that code, and have the Functor to point to it. Templates can help out all around.

- The mechanism is spread over three components - the Functor class, a Translator class, and the makeFunctor() function. All are templates.

- The Functor class is parameterized on the types of the callback function signature, holds the callee data in a typeless manner, and defines a typed operator() but doesn't actually perform the work of calling back. Instead it holds a pointer to the actual callback code. When it comes time to call back, it passes the typeless data (itself actually), as well as the callback arguments, to this pointed-to function.

- The Translator class is derived from Functor but is parameterized on both the Functor type _and_ the callee types. It knows about everything, and is thus able to define a fully type-safe static 'thunk' function that takes the typeless Functor data and the callback arguments. It constructs its Functor base class with a pointer to this static function. The thunk function does the work of calling back, turning the typeless Functor data back into a typed callee and calling the callee. Since the Translator does the work of converting the callee data to and from untyped data the conversions are considered 'safe'. The Translator isA Functor, so it can be used to initialize a Functor.

- The makeFunctor() function takes the callee data, creates a Translator out of it and returns the Translator. Thus the Translator object exists only briefly as the return value of makeFunctor(), but its creation is enough to cause the compiler to lay down the static 'thunk' function, the address of which is carried in the Functor that has been initialized with the Translator.

- All of this will become clearer with the details.

- For each of the 10 Functor classes there are 2 Translator classes and 3 versions of makeFunctor(). We'll examine a slice of the library here, Functor1 and its associated Translators and makeFunctors. The other Functors differ only in the number of args and return values.

*The Functors*

- Since the Functor objects are the only entities held by the caller, they must contain the data about the callee. With some care we can design a base class which can hold, in a typeless manner, the callee data, regardless of whether the callee is a ptr-to-function or object/ptr-to-member-function combo:

```
	
	//typeless representation of a function or object/mem-func
	
	class FunctorBase{
	public:
		typedef void (FunctorBase::*_MemFunc)();
		typedef void (*_Func)();
		FunctorBase():callee(0),func(0){}
		FunctorBase(const void *c,const void *f,size_t sz)
			{
			if(c)	//must be callee/memfunc
				{
				callee = (void *)c;
				memcpy(memFunc,f,sz);
				}
			else	//must be ptr-to-func
				{
				func = f;
				}
			}
		//for evaluation in conditions
		//will be changed to bool when bool exists
		operator int()const{return func||callee;}
	
		class DummyInit{
		};
	////////////////////////////////////////////////////////////////
	// Note: this code depends on all ptr-to-mem-funcs being same size
	// If that is not the case then make memFunc as large as largest
	////////////////////////////////////////////////////////////////
	
		union{
		const void *func;
		char memFunc[sizeof(_MemFunc)];
		};
		void *callee;
	};
```

- All Functors are derived (protected) from this base. FunctorBase provides a constructor from typeless args, where if c is 0 the callee is a pointer-to-function and f is that pointer, else c is pointer to the callee object and f is a pointer to a pointer-to-member function and sz is that ptr-to-member-function's size (in case an implementation has pointer-to-members of differing sizes). It has a default constructor which inits to an 'unset' state, and an operator int to allow for testing the state (set or unset).

- The Functor class is a template. It has a default constructor and the required operator() corresponding to its template parameters. It uses the generated copy constructor and assignment operators.

```

	/************************* one arg - no return *******************/
	template <class P1>
	class Functor1:protected FunctorBase{
	public:
		Functor1(DummyInit * = 0){}
		void operator()(P1 p1)const
			{
			thunk(*this,p1);
			}
		FunctorBase::operator int;
	protected:
		typedef void (*Thunk)(const FunctorBase &,P1);
		Functor1(Thunk t,const void *c,const void *f,size_t sz):
			FunctorBase(c,f,sz),thunk(t){}
	private:
		Thunk thunk;
	};
```

- The Functor class has a protected constructor that takes the same typeless args as FunctorBase, plus an additional first argument. This argument is a pointer to function (the thunk function) that takes the same arguments as the operator(), plus an additional first argument of type const FunctorBase &. The Functor stores this away (in thunk) and implements operator() by calling thunk(), passing itself and the other arguments. Thus it is this thunk() function that does the work of 'calling back'.

- A key issue at this point is whether operator() should be virtual. In the first iteration of my mechanism the Functor classes were abstract and the operator()'s pure virtual. To use them for callbacks a set of derived template classes parameterized on the callee type was provided. This required that functors always be passed and held by reference or pointer and never by value. It also required the caller component or the client code maintain the derived object for as long as the callback relationship existed. I found the maintenance and lifetime issues of these functor objects to be problematic, and desired by-value syntax.

- In the current mechanism the Functor classes are concrete and the operator() is non-virtual. They can be treated and used just like ptr-to-functions. In particular, they can be stored by value in the component classes.

*The Translators*

- Where does the thunk() come from? It is generated by the compiler as a static member of a template 'translator' class. For each Functor class there are two translator classes, one for stand-alone functions (FunctionTranslator) and one for member functions (MemberTranslator). The translator classes are parameterized by the type of the Functor as well as the type(s) of the callee. With this knowledge they can, in a fully type-safe manner, perform two important tasks.

- First, they can initialize the Functor data. They do this by being publicly derived from the Functor. They are constructed with typed callee information and which they pass (untyped) to the functor's protected constructor.

- Second, they have a static member function thunk(), which, when passed a FunctorBase, converts its callee data back into typed information, and executes the callback on the callee. It is a pointer to this static function which is passed to the Functor constructor.

```
	
	template <class P1,class Func>
	class FunctionTranslator1:public Functor1<P1>{
	public:
		FunctionTranslator1(Func f):Functor1<P1>(thunk,0,f,0){}
		static void thunk(const FunctorBase &ftor,P1 p1)
			{
			(Func(ftor.func))(p1);
			}
	};
```

- FunctionTranslator is the simpler of the two. It is parameterized by the argument type of the Functor and some ptr-to-function type (Func). Its constructor takes an argument of type Func and passes it and a pointer to its static thunk() function to the base class constructor. The thunk function, given a FunctorBase ftor, casts ftor's func member back to its correct type (Func) and calls it. There is an assumption here that the FunctorBase ftor is one initialized by the constructor (or a copy). There is no danger of it being otherwise, since the functors are always initialized with matching callee data and thunk functions. This is what is called a 'safe' cast, since the same entity that removed the type information also re-instates it, and can guarantee a match. If Func's signature is incompatible with the call, i.e. if it cannot be called with a single argument of type P1, then thunk() will not compile. If implicit conversions are required the compiler will perform them. Note that if func has a return it is safely ignored.

```
	
	template <class P1,class Callee, class MemFunc>
	class MemberTranslator1:public Functor1<P1>{
	public:
		MemberTranslator1(Callee &c,const MemFunc &m):
			Functor1<P1>(thunk,&c,&m,sizeof(MemFunc)){}
		static void thunk(const FunctorBase &ftor,P1 p1)
			{
			Callee *callee = (Callee *)ftor.callee;
			MemFunc &memFunc(*(MemFunc*)(void *)(ftor.memFunc));
			(callee->*memFunc)(p1);
			}
	};
```

- MemberTranslator is parameterized by the argument type of the Functor, some class type (Callee), and some ptr-to-member-function type (MemFunc). Not surprisingly it's constructor is passed 2 arguments, a Callee object (by reference) and a ptr-to-member-function, both of which are passed, along with the thunk function, to the base class constructor. Once again, the thunk function casts the typeless info back to life, and then calls the member function on the object, with the passed parameter.

- Since the Translator objects are Functor objects, and fully 'bound' ones at that, they are suitable initializers for their corresponding Functor, using the Functor's copy constructor. We needn't worry about the 'chopping' effect since the data is all in the base class portion of the Translator class and there are no virtual functions involved. Thus they are perfect candidates for the return value of makeFunctor()!

*The makeFunctor Functions*

- For each Functor class there are three versions of makeFunctor(), one for ptr-to-function and a const and non-const version for the object/ptr-to-member-function pair.

```
	
	template <class P1,class TRT,class TP1>
	inline FunctionTranslator1<P1,TRT (*)(TP1)>
	makeFunctor(Functor1<P1>*,TRT (*f)(TP1))
		{
		return FunctionTranslator1<P1,TRT (*)(TP1)>(f);
		}
```

- The function version is straightforward. It uses the dummy argument to tell it the type of the functor and merely returns a corresponding FunctionTranslator. I mentioned above that the Func type parameter of FunctionTranslator was invariably a ptr-to-function type. This version of makeFunctor() ensures that by explicity specifying it as such.

```
	
	template <class P1,class Callee,class TRT,class CallType,class TP1>
	inline MemberTranslator1<P1,Callee,TRT (CallType::*)(TP1)>
	makeFunctor(Functor1<P1>*,Callee &c,TRT (CallType::* const &f)(TP1))
		{
		typedef TRT (CallType::*MemFunc)(TP1);
		return MemberTranslator1<P1,Callee,MemFunc>(c,f);
		}
```

- This is the gnarliest bit. Here makeFunctor is parameterized with the type of the argument to the Functor, the type of the callee, the type of the class of which the member-function is a member, the argument and return types of the member function. Whew! We're a long way from Stack<T> land! Like the ptr-to-function version, it uses the dummy first argument of the constructor to determine the type of the Functor. The second argument is a Callee object (by reference). The third argument is this thing:
```

	TRT (CallType::* const &f)(TP1)
```

- Here f is a reference to a constant pointer to a member function of CallType taking TP1 and returning TRT. You might notice that pointer-to-member-functions are all handled by reference in the library. On some implementations they can be expensive to pass by value and copy. The significant feature here is that the function need not be of type pointer-to-member-of-Callee. This allows makeFunctor to match on (and ultimately work with) a ptr-to-member-function of some base of Callee. It then typedefs that bit and returns an appropriate MemberTranslator.

```

	template <class P1,class Callee,class TRT,class CallType,class TP1>
	inline MemberTranslator1<P1,const Callee,TRT (CallType::*)(TP1)const>
	makeFunctor(Functor1<P1>*,const Callee &c,TRT (CallType::* const &f)(TP1)const)
		{
		typedef TRT (CallType::*MemFunc)(TP1)const;
		return MemberTranslator1<P1,const Callee,MemFunc>(c,f);
		}
```

- This last variant just ensures that if the Callee is const the member function is also (note the const at the end of the third argument to the constructor - that's where it goes!).

- That, for each of ten Functors, is the whole implementation.

**Can Your Compiler Do This?**

- The callback library has been successfully tested with IBM CSet++ 2.01, Borland C++ 4.02 (no, its not twice as good ;-), and Watcom C++32 10.0. It is ARM compliant with the exception of expecting trivial conversions of template function arguments, which is the behavior of most compilers. I am interested in feedback on how well it works with other implementations.

**Summary**

- Callbacks are a powerful and necessary tool for component based object-oriented development in C++. They can be a tremendous aid to the interoperability of libraries. The template functor system presented here meets all the stated criteria for a good callback mechanism - it is object-oriented, compile-time type-safe, generic, non-type-intrusive, flexible and easy to use. It is sufficiently general to be used in any situation calling for callbacks. It can be implemented in the current language, and somewhat more elegantly in the proposed language.

- This implementation of callbacks highlights the power of C++ templates - their type-safety, their code-generation ability and the flexibility they offer by accepting ptr-to-function and ptr-to-member-function type parameters.

- Ultimately the greatest benefit is gained when class libraries start using a standard callback system. If callbacks aren't in the components, they can't be retrofitted. Upon publication of this article I am making this Callback library freely available in the hope that it will be adopted by library authors and serve as a starting point for discussion of a standard callback system.

**References**
----
* Stroustrup, B. The Design and Evolution of C++, Addison-Wesley, Reading, MA 1994

* Coplien, J.O. Advanced C++ Programming Styles and Idioms, Addison-Wesley, Reading, MA 1992

* Ellis, M.A. and B. Stroustrup. The Annotated C++ Reference Manual, Addison-Wesley, Reading, MA 1990

* Lippman, S.B. C++ Primer 2nd Edition, Addison-Wesley, Reading, MA 1991

----
**Acknowledgments**

* Thanks to my fellow developers at RCS and to [GREG COMEAU](http://www.comeaucomputing.com/ "Greg Comeau ") for reviewing and commenting on this article.

----

**About the Author**

* Rich is Technical Design Lead at Radio Computing Services, a leading software vendor in the radio industry. He designed and teaches the Advanced C++ course at New York University's Information Technologies Institute.