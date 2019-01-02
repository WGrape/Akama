---
layout:     post
title:      【译】使用TEMPLATE FUNCTORS进行C ++中的回调
subtitle:   TEMPLATE FUNCTORS & C ++
date:       2018-12-11
author:     ChanChou
header-img: 
catalog: true
tags:
    - TEMPLATE FUNCTORS
---

#【译】使用TEMPLATE FUNCTORS进行C ++中的回调
> 原文 [《CALLBACKS IN C++ USING TEMPLATE FUNCTORS》](http://www.tutok.sk/fastgl/callback.html)<br/>
> 译者：[ChanChou](https://github.com/473278056)

# 【译】使用TEMPLATE FUNCTORS进行C ++中的回调

# 使用TEMPLATE FUNCTORS进行C ++中的回调
**版权所有1994 Rich Hickey**

**介绍**

- 面向对象编程的许多承诺之一是它允许使用可重用组件进行即插即用软件设计。设计师将从他们的图书馆“架子”中取出物品并将它们连接在一起以制作软件。在C ++中，将组件连接在一起可能很棘手，特别是如果它们是单独设计的。我们距离可互操作的库和应用程序组件还有很长的路要走。回调提供了一种机制，可以将独立开发的对象连接在一起。它们对即插即用编程至关重要，因为供应商A根据供应商B的类或您自己酿造的类实现其库的可能性为零。

- 回调被广泛使用，但是当前的实现方式不同，并且大多数都存在缺点，其中最重要的是缺乏通用性。本文介绍了回调是什么，如何使用它们以及良好回调机制的标准。它总结了当前的回调方法及其弱点。然后，它描述了一种基于模板仿函数的灵活，强大且易于使用的回调技术 - 这些对象的行为类似于函数。

**CALLBACK基础**

*什么是回调？*

- 在设计应用程序或子系统特定组件时，我们经常知道组件将与之交互的所有类，从而根据这些类明确地编写接口。然而，在设计通用或库组件时，通常需要或希望放入用于调用未知对象的钩子。所需要的是一个组件在没有根据或了解其他组件类型的情况下调用另一个组件的方法。这种“类型盲”调用机制通常被称为回调。

- 回调可用于简单通知，双向通信或在流程中分配工作。例如，应用程序开发人员可能希望Button GUI库中的组件在单击时调用特定于应用程序的对象。数据输入组件的设计者可能希望提供调用应用程序对象以进行输入验证的功能。集合类通常提供一个apply()函数，它将应用程序对象的成员函数“应用”到它们包含的项目中。

- 因此，回调是组件设计者提供通用连接点的一种方式，开发人员可以使用该连接点与应用程序对象建立通信。在某个后续点，组件“回调”应用程序对象。通信采用函数调用的形式，因为这是对象在C ++中交互的方式。

- 回调在许多情况下都很有用。如果您使用任何商业类库，您可能已经看到至少一种提供回调的机制。所有回调实现都必须解决C ++类型系统带来的基本问题：如何构建一个组件，以便它可以在设计组件时调用其类型未知的另一个对象的成员函数？C ++的类型系统要求我们知道我们希望调用其成员函数的任何对象的类型，并且经常被其他OO语言的粉丝批评为太不灵活，无法支持真正的基于组件的设计，因为所有组件都必须“了解”彼此。C ++强大的打字有很多优势可以放弃，

- 事实上，C ++非常灵活，这里介绍的机制利用其灵活性提供此功能而无需语言扩展。特别是，模板提供了一个强大的工具来解决这样的问题。如果您认为模板仅用于容器类，请继续阅读！

*回调术语*

> 任何回调机制中都有三个元素 - 调用者，回调函数和被调用者。

- 所述呼叫者通常是一些类的实例，例如库组件（尽管它可以是一个函数，像 qsort()），其提供或需要回调; 即它可以或必须调用某些第三方代码来执行其工作，并使用回调机制来执行此操作。就调用者的设计者而言，回调只是一种调用进程的方法，这里称为回调函数。调用者确定回调函数的签名，即其参数和返回类型。这是有道理的，因为调用者需要完成工作或传达信息。例如，在上面的例子中，Buttonclass可能需要一个没有参数且没有返回的回调函数。它是一个简单的通知函数，用于Button指示它已被点击。该 DataEntryField组件可能想传递一个String 回调函数，并得到了Boolean回报。

- 调用者可能只需要回调一个函数的持续时间，就像ANSI C一样qsort()，或者可能希望保留回调以便稍后回调，就像 Button类一样。

- 该被叫方通常是一些类的一个对象的成员函数，但它也可以是一个独立的函数或静态成员函数，应用程序设计者希望通过主叫组件来调用。注意，在非静态成员函数的情况下，特定对象/成员 - 函数对是被调用者。要调用的函数必须与调用者指定的回调函数的签名兼容。

*良好回调机制的标准*

- 面向对象模型中的回调机制应该支持组件和应用程序设计。组件设计人员应该有一种标准的，现成的方式来提供回调服务，而不需要他们的发明。应提供指定参数和返回值的数量和类型的灵活性。由于组件可以设计用于尚未应用的应用程序，因此组件设计者既不需要知道也不需要指示组件可以“回调”的对象类型。

- 应用程序开发人员，给定具有此标准回调机制的组件以及具有与回调函数签名兼容的成员函数的类的某个实例，应该不需要自定义“粘合”编码以将两者连接在一起。他们也不应该修改被调用者类或手工派生新类。如果他们想让回调调用一个独立的非成员函数，那么也应该支持它。

**为了支持这种行为，回调机制应该是：**

*面向对象* - 我们的应用程序是使用对象构建的。在C ++应用程序中，大多数功能都包含在成员函数中，这些函数不能通过普通的ptr-to-functions调用。非静态成员函数对具有状态的对象进行操作。调用这些函数不仅仅是调用一个进程，而是在特定对象上运行，因此面向对象的回调必须包含有关要调用的对象的信息。

*类型安全* - 类型安全是C ++的基本特性和优点，任何强大的C ++回调机制都必须是类型安全的。这意味着我们必须确保对象的使用符合其指定的接口，并且对参数，返回值和转换强制执行类型规则。确保这一点的最佳方法是让编译器在编译时完成工作。

*非耦合* - 这是回调的基本目标 - 允许彼此无知的组件连接在一起。如果机制以某种方式在调用者和被调用者之间引入了依赖性，那么它的基本任务就失败了。

*非类型侵入* - 一些执行回调的机制需要修改或推导调用者或被调用者类型。对象连接到特定应用程序中的另一个对象的事实通常与其类型无关。正如我们将在下面看到的那样，类型侵入的机制会降低灵活性并增加应用程序代码的复杂性。

*通用* - 不同回调情况之间的主要区别是涉及的类型。这表明应该使用模板对回调机制进行参数化。模板在所有回调情况下确保一致的接口和名称，并提供一种方法来由编译器而不是用户生成任何必要的支持代码。

*灵活* - 经验表明，要求回调函数和被调用函数签名完全匹配的回调系统对于实际使用而言过于僵化。例如，您可能会遇到一个回调，Derived *它将您想要连接的回调传递给需要a的被调用函数Base *。

**当前机制**

*功能模型*

- 最简单的回调机制是一个指针到功能，一拉ANSI C的 qsort()。但是，获取独立函数来处理特定对象通常涉及使用静态或全局指针来指示目标对象，或者让回调函数采用额外参数（通常是指向要处理的对象的指针）等问题。 。当调用之间存在回调关系时，静态/全局指针方法会中断，即'我想在此应用期间将此Button连接到此X而另一个按钮连接到另一个X'。额外的参数方法，如果安全地类型化，则在调用者和被调用者类型之间引入不期望的耦合。

- qsort()通过前述类型安全来实现其通用性。也就是说，为了使它不知道它正在操纵的类型，它需要无类型（void *）参数。没有什么可以阻止某人调用qsort()苹果数组并将指针传递给比较橙子的函数！

- 您经常看到的这种无类型机制的一个例子是集合中的“应用”功能。apply函数的目的是允许开发人员将回调传递给集合，并将其“应用”（调用）集合中的每个项目。不幸的是它通常看起来像这样：

> void apply(void (*func)(T &theItem,void *extraStuff),void *theStuff);
                            
- 机会真的很好你没有像func坐着这样的功能，所以你必须写一个（需要大量的演员）。并确保你传递正确的东西。啊。

*单根生成层次结构*

注意看起来类型安全但实际上不是的回调机制。这些机制通常涉及一些基类的所有类，如Object或EventHandler，并使用从ptr到成员的转换为ptr-to-base-of base。经验表明，如果组件来自多个来源，单根系统是行不通的。

*参数化调用者*

- 组件设计者可以根据被调用者的类型参数化组件。在许多情况下，这种参数化是不合适的，并且回调就是其中之一。考虑：

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

- 这样，callsback <CDPlayer>按钮就会“知道”CDPlayer，并提供基于它的显式接口。问题是，这在系统中引入了刚性，因为被调用方类型成为调用方类型的一部分，即它是“类型入侵”类型。创建ButtonThatCallsBack对象的所有代码都必须知道被调用对象之间的关系，从而增加系统中的耦合。回调<X>的ButtonThatCallsBack<Y>与回调<Y>的buttonthat回调<X>不同，因此可以防止按值操作。

- 如果组件具有许多回调关系，则很快就无法对它们进行参数化。考虑一个Button 想要维护一个动态的被调用者列表，以便在点击事件时得到通知。由于callee类型内置于Button 类类型中，因此该列表必须是同类或无类型的。

- 库代码甚至无法创建ButtonThatCallsBack 对象，因为它们的实例化取决于应用程序类型。这是一个严重的限制因素。考虑GUI库代码，该代码从资源文件中读取对话框描述并创建Dialog 对象。它怎么能知道你想Buttons在 Dialog回调CDPlayers？它不能，因此它不能Buttons为你创造。

**Callee Mix-In**

- 调用者组件设计者可以发明一个抽象基类作为回调的目标，并向应用程序开发人员指出他们将这个基础混合在一起，以便将他们的类与组件连接起来。我称之为“被调用者混合”。

- 这里Button该类的设计者想要提供一个点击通知回调，因此定义了一个嵌套类 Notifiable，notify() 其中包含具有所需签名的纯虚函数。Button 该类的客户端必须向其构造函数传递一个指向a的指针Notifiable，该指针 Button将用于（稍后在某些时候）通知点击：


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
	
	给定 :
	
	class CDPlayer{
	public:
		void play();
		//...
	};
```

- 如果应用程序开发人员希望有一个按钮回调，那么CDPlayer将不得不从CDPlayer和Button:: notify都派生一个新类，重写纯虚函数来完成所需的工作:


```
	
	class MyCDPlayer:public CDPlayer,public Button::Notifiable{
	public:
		void notify()
			{play();}
	};
```

- 并使用此类而不是CDPlayer在应用程序中：

```
	
	MyCDPlayer cd;
	Button button(&cd);
	button.click();	//calls cd.play()
```

- 这种机制是类型安全的，实现Button 和 CDPlayer，和杂志文章饲料的解耦。然而，它在实践中几乎是无用的。

- 被调用者混合的问题在于它也是类型侵入的，即它会影响被调用者的类型，在这种情况下是强制推导。这有三个主要缺陷。首先，使用多重继承，特别是如果被调用者是多个组件的被调用者，由于名称冲突等问题是有问题的。其次，推导可能是不可能的，例如，如果应用程序设计者 CDPlayers来自不可更改的，不可触及的API（库设计者）注意：这是基于混合的机制的一个大问题）。第三个问题是最好的证明。考虑以下版本CDPlayer：

```
	
	class CDPlayer{
	public:
		void play();
		void stop();
		//...
	};
```

- 它似乎没有不合理的地方之一，有一个应用程序Button 调用CDPlayer::play()和另一个CDPlayer::stop()。混入机制在这里完全失败，因为它只能支持调用者/被调用者/成员函数之间的单个映射，即MyCDPlayer 只能有一个notify()。

**使用模板功能进行回调**

- 当我第一次考虑组件间回调问题时，我决定需要的是支持“绑定指针”的语言扩展，表示对象信息和该对象的成员函数的特殊指针，可存储和可调用，就像常规一样指向函数的指针。ARM 5.5评论简要解释了为什么绑定指针被遗漏了。

- 绑定指针如何工作？理想情况下，您可以使用常规指向函数或对象的引用和指向成员的函数来初始化它们。初始化后，它们的行为就像普通的指针到函数一样。您可以将函数调用 operator()应用于它们以调用该函数。为了适用于回调机制，有关被调用者类型的信息将不是绑定指针类型的一部分。它可能看起来像这样：


```

	// Warning - NOT C++
	
	class Fred{
	public:
		void foo();
	};
	Fred fred;
	void (* __bound fptr)() = &fred.foo;
```

- 这fptr是一个绑定指针，指向不带参数和返回的函数void。请注意，这Fred 不是fptr's类型的一部分。它是用对象fred和指向成员函数的指针 初始化的foo。他说：FPTR（）;将调用foo上fred。这样的绑定指针是回调的理想选择：

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

- 绑定指针需要一个非平凡的语言扩展和一些棘手的编译器支持。鉴于任何新语言功能的极端不受欢迎，我现在很难提出绑定指针。尽管如此，我仍然认为绑定指针概念是回调的正确解决方案，并着手看看我能够在当前和提议的语言中获得多么接近。结果是下面描述的Callback库。事实证明，库解决方案不仅可以提供上面显示的功能（虽然语法不同），它证明比语言扩展更灵活！

- 从语言扩展的幻想世界回归，图书馆必须为用户提供两件事。第一个是扮演'bound-pointer'角色的构造。第二种是从常规指向函数或对象和指向成员函数的指针创建这些“绑定指针”的方法。

- 在'bound-pointer'角色中，我们需要一个行为类似于函数的对象。Coplien使用术语仿函数来描述这些对象。对于我们的目的，仿函数只是一个对象，其行为类似于指向函数的指针。它有一个operator() （函数调用运算符），可用于调用它指向的函数。该库提供了一组模板Functor 类。它们保存任何必要的被调用者数据并提供类似行为的指针。最重要的是，它们的类型与被调用者类型没有任何关系。组件使用Functor类定义其回调接口。

- 库提供的用于创建仿函数的构造是一个重载的模板函数，makeFunctor()它将被调用者信息（对象和ptr到成员函数，或者ptr-to-function）作为参数，并返回适合的函数。初始化一个 Functor对象。

- 由此产生的机制非常容易使用。一个完整的例子：

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
	
	// 一些应用程序的东西，我们想连接到按钮:
	
	class CDPlayer{ public:
		void play(){cout<<"Playing"<<endl;}
		void stop(){cout<<"Stopped"<<endl;}
	};
	
	void wow()
		{cout<<"Wow!"<<endl;}
	
	void main()
		{
		CDPlayer cd;
	
		// makeFunctor从对象和ptr到成员函数
	
		Button playButton(makeFunctor(cd,&CDPlayer::play));
		Button stopButton(makeFunctor(cd,&CDPlayer::stop));
	
		// 从pointer-to-function 中得到makeFunctor
	
		Button wowButton(makeFunctor(&wow));
	
		playButton.click();	//calls cd.play()
		stopButton.click();	//calls cd.stop()
		wowButton.click();	//calls wow()
		}
```
- 瞧！组件（Button）已经连接到应用程序对象和它不知道的任何功能的函数Button，没有任何自定义编码，派生或修改所涉及的对象。它的类型安全。

- 该Button级设计师指定的条款回调接口 Functor0，一个仿函数，它没有参数，并返回 void。它将仿函数存储在其成员中notify。当回电时，它只需要调用operator() 仿函数。这看起来和感觉就像通过指向函数的调用一样。

- 将某些东西连接到使用回调的组件很简单。您可以Functor使用适当的调用结果 初始化a makeFunctor()。有两种口味makeFunctor()。您可以使用ptr-to-stand-alone功能调用它：

	> makeFunctor(&wow)
- 或者与对象和指向成员的函数：

	> makeFunctor(cd,&CDPlayer::play)
- 我必须在这一点上说清楚，并指出上面的语法 makeFunctor()只能在提议的语言中使用，因为它需要模板成员（具体来说，Functor 构造函数必须是模板）。在当前语言中，通过传递makeFunctor()类型为ptr-to-the-Functor-type-you-want-to-create的伪参数，可以实现相同的结果。回调库的这次迭代要求您将makeFunctor() 虚拟对象作为第一个参数传递。简单地演员0提供这个论点：

	> makeFunctor((Functor0 *)0,&wow)

	> makeFunctor((Functor0 *)0,cd,&CDPlayer::play);
- 我将从这里开始使用这种当前语言语法。

- Button上面的类只需要一个没有返回参数的回调函数void。其他组件可能希望将数据传递给回调或返回。区分一个函子与另一个函子的唯一区别是参数的数量和类型operator()及其返回类型（如果有的话）。这表明仿函数可以通过（一组）模板在库中表示：


```


	//Callback库提供的Functor类：
	
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
- 这些参数由参数类型（P1 等）和返回值（RT）（如果有）参数化。编号是必要的，因为我们不能在参数数量上重载模板类名。wRet附加' '以区分具有返回值的那些。每个都有一个operator()带有相应的签名，例如：

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

- 这些Functor类足以满足组件设计者的回调需求，因为它们提供了一种标准且一致的方式来提供回调服务，以及一种用于调用回调函数的简单机制。给定库中的这些模板，组件设计者只需要选择具有正确数量的参数的模板，并将所需类型指定为参数。这 DataEntryField是需要一个验证回调，它接受一个const String &并返回一个 Boolean：

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

- 给出一个通用的回调库，这些琐碎的例子只是抓住了你可以做的表面。考虑他们对状态机，调度表等的应用。

- 回调库是100％编译时类型安全的。（编译时包括模板实例化时间）。如果您尝试使用与仿函数类型不兼容的函数创建函子，则会出现编译器错误。保留所有正确的虚函数行为。

- 该系统也是灵活的类型。你会注意到，在讨论回调函数和被调用函数之间的关系时，我在整篇文章中都说'类型兼容'而不是'完全匹配'。经验表明，要求完全匹配会使回调过于严格，无法实际使用。如果您已经使用基于指针到函数的接口做了很多工作，那么您可能会遇到一个指向函数的指针的失败，这个函数“可以工作”但不是匹配所需的确切类型。

- 为了提供灵活性，库支持使用与目标仿函数“类型兼容”的被调用函数构建仿函数 - 它不需要具有完全匹配的签名。类型兼容我指的是具有相同数量的参数的函数，可以通过隐式转换从仿函数的参数类型中获得类型。函数的返回类型必须可隐式转换为仿函数的返回类型。可以使用带返回的函数构建没有返回的仿函数 - 可以安全地忽略返回值。 

```

	//假设Derived公开派生自Base 
	void foo(Base &);
	long bar(Derived &);
	
	Functor1<Derived&> f1 =
	        makeFunctor((Functor1<Derived&> *)0,&foo);
		//ok  - 将隐式转换
	
	f1 = makeFunctor((Functor1<Derived&> *)0,&bar);
		//ok - 忽略返回
```

- 任何必要的参数转换或忽略返回都由编译器完成，即在机制内或用户内没有强制执行。如果编译器无法从传递给functor的参数获取callee函数所需的参数，则在编译时拒绝代码。通过允许编译器完成工作，我们获得了所有参数的正常转换 - 派生到基础，内置类型的提升和转换，以及用户定义的转换。

- 库的类型灵活性是绑定指针的语言扩展再现中不可用的。

- 舍入Functor 类的功能是一个默认构造0 函数，它也将作为初始化程序接受，它将 Functor处于已知的“未设置”状态，并且Boolean可以使用转换 来测试是否Functor 为“set”。这些Functor类不依赖于任何虚函数行为，因此它们可以按值保存和复制。因此，a Functor具有与常规指向函数相同的易用性。

- 此时，您知道使用回调库所需的一切。所有代码都在一个文件中callback.h。要在组件类中使用回调，只需Functor 使用所需的参数类型实例化a 。要将一些东西连接到Functors用于回调的组件，只需调用makeFunctor() 这些东西即可。简单。

**功率模板**

- 像往常一样，对于实现者来说，对用户来说容易的事情通常是棘手的。鉴于上面的Functor 类的黑盒描述，makeFunctor()可能很难接受类型安全，透明转换，正确的虚函数行为等声明。窗帘背后的外观不仅揭示了它是如何工作的，还有一些简洁的模板技术。警告：大多数人一开始发现实现中使用的指向成员的指针和模板语法令人生畏。

- 显然某种魔法正在发生。在Functor 不知道被调用者的类型或签名的情况下，类如何确保对它进行类型安全调用，可能是对参数的隐式转换？它不能，所以它没有。实际工作必须由一些知道functor回调签名和所有关于被调用者的代码执行。诀窍是让编译器生成该代码，并Functor指向它。模板可以提供帮助。

- 该机制分布在三个组件上 - Functor 类，Translator类和makeFunctor() 函数。都是模板。

- 该Functor班是在类型回调函数签名的参数，在一个无类型的方式持有被叫数据，并确定一个类型operator()，但实际上并不执行回呼的工作。相反，它拥有一个指向实际回调代码的指针。当回调时，它会将无类型数据（实际上是自己的）以及回调参数传递给这个指向函数。

- 转换器类派生自函子，但是在函子类型_和被调用类型__上都是参数化的。它知道所有的事情，因此能够定义一个完全类型安全的静态“thunk”函数，该函数接受无类型函子数据和回调参数。它用指向这个静态函数的指针构造它的函子基类。thunk函数执行回调的工作，将无类型的函子数据转换回有类型的被调用方并调用被调用方。由于转换程序负责将被调用数据转换为非类型化数据，因此转换被认为是“安全的”。转换器是一个函子，因此它可以用来初始化函子。

- 该makeFunctor()函数接受被调用者数据，创建一个Translator并从中 返回Translator。因此，Translator对象仅作为返回值暂时存在makeFunctor()，但其创建足以使编译器放置静态“thunk”函数，其地址在Functor已经初始化的地址中进行Translator。

*所有这一切都将随着细节变得更加清晰。*

- 对于10个Functor类中的每个类，有2个 Translator类和3个版本makeFunctor()。我们将在这里检查一个库的片段，Functor1以及它的关联Translators和makeFunctors。另一个Functors区别仅在于args和返回值的数量。

**Functors**

- 由于Functor对象是调用者持有的唯一实体，因此它们必须包含有关被调用者的数据。我们可以设计一个基类，它可以无类型地保存被调用者数据，无论被调用者是ptr-to-function还是object / ptr-to-member-function组合：


```
	
	//函数或对象的无类型表示/ mem-func 
	
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
		//在条件评估
		//将被改变时布尔存在为bool 
		operator int()const{return func||callee;}
	
		class DummyInit{
		};
	////////////////////////////////////////////////// ////////////// 
	//注意：这段代码依赖于所有相同大小的ptr-to-mem-func 
	//如果不是这样的话，那就让memFunc大到最大
	// ////////////////////////////////////////////////// //////////// 
	
		union{
		const void *func;
		char memFunc[sizeof(_MemFunc)];
		};
		void *callee;
	};
```

- 所有Functors这些都是从这个基础派生（保护）的。 FunctorBase提供了一个无类型args的构造函数，if if c是0被调用者是指向函数的f指针，c 是指针，否则是指向被调用者对象f的指针，是指向成员指针函数的指针，sz是ptr-to- member-function的大小（如果实现具有指向不同大小的成员的指针）。它有一个默认构造函数，它进入'unset'状态，并operator int允许测试状态（set或unset）。

- 该Functor班是一个模板。它有一个默认构造函数，并且需要operator()与其模板参数相对应。它使用生成的复制构造函数和赋值运算符。


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

- 该Functor班有一个受保护的构造函数相同的无类型ARGS作为FunctorBase，加上额外的第一个参数。这个参数是一个指向函数的指针（thunk函数），它接受与该函数相同的参数operator()，以及另一个类型的第一个参数const FunctorBase &。在Functor这个远离（在thunk的）存储和工具 operator()调用thunk()，通过自身和其他参数。因此，正是这个thunk() 功能完成了“回叫”的工作。

- 此时的一个关键问题是，是否operator() 应该是虚拟的。在我的机制的第一次迭代中，Functor 类是抽象的，而且operator()是纯虚拟的。为了将它们用于回调，提供了一组在被调用者类型上参数化的派生模板类。这要求仿函数始终通过引用或指针传递和保持，而不是通过值传递。只要回调关系存在，它还需要调用者组件或客户端代码维护派生对象。我发现这些仿函数对象的维护和生命周期问题是有问题的，并且需要按值语法。

- 在当前机制中，Functor类是具体的并且operator()是非虚拟的。它们可以像ptr-to-functions一样进行处理和使用。特别是，它们可以按值存储在组件类中。

**编译**

- 它thunk()来自哪里？它由编译器生成为模板“转换器”类的静态成员。对于每个 Functor类，有两个转换器类，一个用于独立函数（FunctionTranslator），另一个用于成员函数（MemberTranslator）。转换器类通过被调用者的类型Functor以及类型来参数化。凭借这些知识，他们可以以完全类型安全的方式执行两项重要任务。

- 首先，他们可以初始化Functor数据。他们通过公开衍生出来做到这一点Functor。它们由类型化的被调用者信息构成，并且它们传递（无类型）到仿函数的受保护构造函数。

- 其次，它们有一个静态成员函数thunk()，当传递给它时，它将FunctorBase其被调用者数据转换回类型信息，并在被调用者上执行回调。它是指向此静态函数的指针，该函数将传递给Functor 构造函数。


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

- FunctionTranslator两者中更简单。它由参数类型Functor和一些ptr-to-function类型（Func）参数化。它的构造函数接受一个类型的参数，Func并将它和一个指向其静态thunk()函数的指针传递给基类构造函数。FunctorBasethtor 函数，给定一个ftor，将ftor的func成员强制转换回正确的类型（Func）并调用它。这里假设FunctorBaseftor是由构造函数（或副本）初始化的。否则就没有危险，因为仿函数总是用匹配的被调用者数据和thunk函数初始化。这就是所谓的“安全”强制转换，因为删除类型信息的同一实体也会重新启用它，并且可以保证匹配。如果 Func的签名与调用不兼容，即如果不能使用单个参数类型调用它P1，则thunk()不会编译。如果需要隐式转换，编译器将执行它们。请注意，如果func 有退货，则会安全地忽略它。

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

- MemberTranslator由Functor一些类type（Callee）和一些ptr-to-member-function type（MemFunc）的参数类型参数化 。毫不奇怪，它的构造函数传递了2个参数，一个 Callee对象（通过引用）和一个ptr-to-member-function，它们都与thunk函数一起传递给基类构造函数。再次，该thunk函数将无类型信息转换为生命，然后使用传递的参数调用对象上的成员函数。

- 由于Translator对象是 Functor 对象，并且完全“绑定”对象，因此它们是相应的初始化器Functor，使用Functor复制构造函数。我们不必担心'斩波'效应，因为数据都在类的基类部分，Translator 并且不涉及虚函数。因此，他们是回报价值的完美候选人 makeFunctor()！

**makeFunctor函数**

- 对于每个Functor类，有三个版本 makeFunctor()，一个用于ptr-to-function，另一个用于object / ptr-to-member-function对的const和non-const版本。

```
	
	template <class P1,class TRT,class TP1>
	inline FunctionTranslator1<P1,TRT (*)(TP1)>
	makeFunctor(Functor1<P1>*,TRT (*f)(TP1))
		{
		return FunctionTranslator1<P1,TRT (*)(TP1)>(f);
		}
```

- 功能版本很简单。它使用伪参数告诉它仿函数的类型，只返回一个对应的 FunctionTranslator。我在上面提到过，Func 类型参数 FunctionTranslator总是一个ptr-to-function类型。此版本makeFunctor()确保通过明确指定它。

```
	
	template <class P1,class Callee,class TRT,class CallType,class TP1>
	inline MemberTranslator1<P1,Callee,TRT (CallType::*)(TP1)>
	makeFunctor(Functor1<P1>*,Callee &c,TRT (CallType::* const &f)(TP1))
		{
		typedef TRT (CallType::*MemFunc)(TP1);
		return MemberTranslator1<P1,Callee,MemFunc>(c,f);
		}
```

- 这是最难的一点。这里使用makeFunctor参数的类型，Functor被调用者的类型，成员函数所属的类的类型，成员函数的参数和返回类型进行参数化。呼！我们离Stack<T>土地很远 ！与ptr-to-function版本一样，它使用构造函数的虚拟第一个参数来确定类型Functor。第二个参数是一个Callee 对象（通过引用）。第三个论点就是这个：

> TRT（CallType :: * const＆f）（TP1）

- 这里f是一个常量指针的引用的成员函数 CallType取TP1和返回TRT。您可能会注意到指向成员函数的指针都是通过库中的引用来处理的。在某些实现中，通过值和复制它们可能是昂贵的。这里的重要特征是函数不必是指向Callee成员的类型。这允许 makeFunctor匹配（并最终使用）某些基础的ptr-to-member-function Callee。然后键入dede那个位并返回一个合适的MemberTranslator。
```

	template <class P1,class Callee,class TRT,class CallType,class TP1>
	inline MemberTranslator1<P1,const Callee,TRT (CallType::*)(TP1)const>
	makeFunctor(Functor1<P1>*,const Callee &c,TRT (CallType::* const &f)(TP1)const)
		{
		typedef TRT (CallType::*MemFunc)(TP1)const;
		return MemberTranslator1<P1,const Callee,MemFunc>(c,f);
		}
```

- 最后一个变量只是确保如果Callee是const，则成员函数也是（注意const构造函数的第三个参数的末尾 - 它就是它的位置！）。

**对于十个中的每一个，这Functors是整个实施。**

*你的编译器可以这样做吗？*

- 回调函数库已经成功使用IBM CSet ++ 2.01，Borland C ++ 4.02（不，它不是两倍;-)和Watcom C ++ 32 10.0进行了测试。它符合ARM，除了期望模板函数参数的微不足道的转换，这是大多数编译器的行为。我对其与其他实现的工作情况的反馈感兴趣。

**摘要**

- 回调是C ++中基于组件的面向对象开发的强大而必要的工具。它们可以极大地帮助库的互操作性。这里介绍的模板函子系统符合所有规定的良好回调机制标准 - 它是面向对象的，编译时类型安全，通用，非类型侵入，灵活且易于使用。它足够通用，可用于任何需要回调的情况。它可以用当前语言实现，并且在所提出的语言中更优雅。

- 回调的这种实现强调了C ++模板的强大功能 - 它们的类型安全性，它们的代码生成能力以及它们通过接受ptr-to-function和ptr-to-member-function类型参数所提供的灵活性。

- 最终，当类库开始使用标准回调系统时，可以获得最大的好处。如果回调不在组件中，则无法对其进行改装。在发表这篇文章之后，我正在免费提供这个Callback库，希望它可以被库作者采用，并作为讨论标准回调系统的起点。

**参考**

----
* Stroustrup, B. The Design and Evolution of C++, Addison-Wesley, Reading, MA 1994

* Coplien, J.O. Advanced C++ Programming Styles and Idioms, Addison-Wesley, Reading, MA 1992

* Ellis, M.A. and B. Stroustrup. The Annotated C++ Reference Manual, Addison-Wesley, Reading, MA 1990

* Lippman, S.B. C++ Primer 2nd Edition, Addison-Wesley, Reading, MA 1991

----

>致谢

>感谢RCS的开发人员和 Greg Comeau 对本文的审阅和评论。

*关于作者*

* Rich是Radio Computing Services的技术设计主管，Radio Computing Services是无线电行业的领先软件供应商。他在纽约大学信息技术学院设计并教授高级C ++课程。