## Microsoft Edge’s JavaScript engine to go open-source

Today at JSConf US Last Call in Florida, we announced that we will open-source the core components of Chakra as ChakraCore, which will include all the key components of the JavaScript engine powering Microsoft Edge. The ChakraCore sources will be made available on GitHub under the MIT license next month.

<img src="./images/1.jpg">

Chakra offers best-in-class JavaScript execution with the broadest set of ES2015 feature coverage and dependable performance, reliability, and scalability. We expect ChakraCore to be used wherever these factors are important, ranging from cloud-based services to the Internet of Things and beyond.

We’re investing more than ever in improving Chakra, and are excited to team up with our community to drive further improvements. In addition to the public, several organizations have already expressed interest in contributing to ChakraCore—among many others, we look forward to working with Intel, AMD, and NodeSource as we develop this community.

## Chakra: A modern JavaScript engine

In 2008, we created a new JavaScript engine, codenamed Chakra, from a clean slate. Our founding principles were to ensure that Chakra had the performance characteristics needed for the modern web and could easily adapt to other potentially emerging scenarios, across a range of hardware profiles. In a nutshell, this means that Chakra needed to start fast, run fast, and deliver a great user experience, while utilizing the full potential of the underlying hardware. Chakra achieved these goals via a unique multi-tiered pipeline that supports an interpreter, a multi-tiered background JIT compiler, and a traditional mark and sweep garbage collector that can do concurrent and partial collections.

<img src="./images/2.png">

Since Chakra’s inception, JavaScript has expanded from a language that primarily powered the web browser experience to a technology that supports apps in stores, server side applications, cloud based services, NoSQL databases, game engines, front-end tools and most recently, the Internet of Things. Over time, Chakra evolved to fit many of these contexts and has been optimized to deliver great experiences across them all. This meant that apart from throughput, Chakra had to support native interoperability, great scalability and the ability to throttle resource consumption to execute code within constrained resource environments. Chakra’s interpreter played a key role in easy portability of the technology across platform architectures.

Today, outside of the Microsoft Edge browser, Chakra powers Universal Windows applications across all form factors where Windows 10 is supported—whether it’s on an Xbox, a phone, or a traditional PC. It powers services such Azure DocumentDB, Cortana and Outlook.com. It is used by (and optimized for) TypeScript. And with Windows 10, we enabled Node.js to run with Chakra, to help advance the reach of Node.js ecosystem and make Node.js available on a new IoT platform: Windows 10 IoT Core.

With the release of Windows 10 earlier this year, Chakra was not only optimized to run the web faster, but more than doubled its performance on some key JavaScript benchmarks owned by other browser vendors.

Additionally, Chakra supports most of the ECMAScript 2015 (aka ES6) features and has support for some of the future ECMAScript proposals like Async Functions and SIMD. It supports asm.js and the team is a key participant in helping evolve WebAssembly and its associated infrastructure.

Since its introduction in 2008, Chakra has grown to be a perfect choice for the web, cloud services, and the Internet of Things. With today’s announcement, we’re taking the next step by giving developers a fully supported and fully open-source JavaScript engine available to embed in their projects, innovate on top of, and contribute back to: ChakraCore.

## What’s in ChakraCore?
ChakraCore is a fully fledged, self-contained JavaScript virtual machine that can be embedded in derivative products and power applications that need scriptability such as NoSQL databases, productivity software, and game engines. ChakraCore can be used to extend the reach of JavaScript on the server with platforms such as Node.js and cloud-based services. It includes everything that is needed to parse, interpret, compile and execute JavaScript code without any dependencies on Microsoft Edge internals.

ChakraCore shares the same set of capabilities that are supported by Chakra in Microsoft Edge, with two key differences. First, it does not expose Chakra’s private bindings to the browser or the Universal Windows Platform, both of which constrain it to a very specific use case. Second, instead of exposing the COM based diagnostic APIs that are currently available in Chakra, ChakraCore will support a new set of modern diagnostic APIs, which will be platform agnostic and could be standardized or made interoperable across different implementations in the long run. As we make progress on these new diagnostics APIs, we plan to make them available in Chakra as well.

<img src="./images/3.png">

## What’s next for ChakraCore?
Any modern JavaScript Engine must deliver on a performance envelope that goes beyond browser scenarios, encompassing everything from small-footprint devices for IoT applications, all the way up to high-throughput, massively parallel server applications based on cloud technologies.

ChakraCore is already designed to fit into any application stack that calls for a fast, scalable, and lightweight engine. We intend to make it even more versatile over time, both within and beyond the Windows ecosystem. While the initial January release will be Windows-only, we are committed to bringing ChakraCore to other platforms in the future. We’d invite developers to help us in this pursuit by letting us know which other platforms they’d like to see ChakraCore supported on to help us prioritize future investments, or even by helping port it to the platform of their choice.

## Contributing to ChakraCore
Starting in January, we will open our public GitHub repository for community contributions. At that time, we will provide more detail on our initial priorities and guidance on how to contribute effectively to the project. The community is at the heart of any open source project, so we look forward to the community cloning the repository, inspecting the code, building it, and contributing everything from new functionality to tests or bug fixes. We also welcome suggestions on how to improve ChakraCore for particular scenarios that are important to you or your business.

We are committed to making Microsoft Edge and its associated ecosystem a benchmark for collaborative innovation, interoperability, and developer productivity. This commitment led to initiatives like the new Microsoft Edge Dev site, Platform Status, and User Voice to foster a two-way dialog between the Microsoft Edge team and the community. Open-sourcing ChakraCore is a natural complement to that effort, and is inspired by the same principles of openness and transparency.

We’re excited about this milestone, and are hopeful that developing in the open will allow us to collaborate even more deeply with more developers around the world, resulting in better products for everyone. If you have any questions or if there is something we didn’t cover, let us know @MSEdgeDev on Twitter or in the comments section below! We look forward to sharing more soon.