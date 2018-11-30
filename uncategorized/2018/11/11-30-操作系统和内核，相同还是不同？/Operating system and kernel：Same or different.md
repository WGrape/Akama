Operating system and kernel : Same or different?

This is the topic I have heard in which people get confused inspite of knowing the answer. It's time to draw the clear line between the two keywords, "Operating System" and "Kernel"

In simple terms, one line answer will be, kernel is a part of an operating system.

Explanation:
Let’s first define what an operating system is. We all have heard of the common definition that it acts as an interface between the application programs and the system hardware. It is also responsible for resource allocation, assigning CPU to the processes running, manages the hardware etc. But to point out, we have no universally accepted definition of what an operating system is. The fundamental goal of computer systems is to execute user programs and to make problem solving easier. Computer hardware is constructed towards this goal. Since bare hardware alone is not sufficient and easy to use, application programs are developed on top of them to provide users to interact with. These programs now may need various common operations such as memory, IO devices etc. These common operations are brought together into one piece of software known as operating system. A simple viewpoint is that it includes everything a vendor ships when you order operating system.

Thus, Operating System is a generic name given to all of the elements (user interface, libraries, resources) which make up the system as a whole.

The kernel is "brain" of the operating system, which controls everything from access to the hard disk to memory management.

Whenever you want to do anything, it goes though the kernel. It provides low level services like device management, process management, memory management i.e. it provides all the core system calls to accomplish any task. OS includes this kernel, along with this, it also includes GUI components like shells, command interpreter, browser, compilers, text editors, windowing systems etc via which user asks the kernel to perform a certain job.
Thus, when extra utilities and useful applications are added over the kernel, it becomes a package. So, it can easily be said that the OS consists of a user space and a kernel space.

As a quick test, ask yourself now if LINUX is a kernel or an operating system?

The answer will be Kernel. Linux is a kernel as it does not include any extra applications with it. That's why companies like ubuntu, redHat have added some extra utilities and interfaces and made that as an OS.
