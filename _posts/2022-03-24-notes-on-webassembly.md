---
layout: post
title: Notes on WebAssembly
categories: [web]
excerpt: WebAssembly appeared in a recent conversation, and I felt the need to formalize some of my knowledge.
last_modified_at:
---

WebAssembly appeared in a recent conversation, and I felt the need to formalize some of my knowledge. WebAssembly (wasm) is a binary instruction format for a stack-based virtual machine. It's safe to say that you do not write WebAssembly, you compile other higher-level languages to it.

> A stack-based virtual machine is an abstraction of a computer. Stack machines are used as the building blocks of complex programs, they provide the stack structure to put values on the stack and then perform operations with them. The intention of these virtual machines is to be specialized in one type of bytecode (an interpreter, translating code in real-time on the CPU). Examples are the JVM and EVM (Ethereum virtual machine).

Wasm was not created to replace JavaScript, the intention was to complement it with two main characteristics: speed and portability.

## Speed
WebAssembly translates to assembly code much faster than JavaScript. Its binaries are much smaller than textual JavaScript files, as a consequence, they are faster to download, which is especially important on slow networks. Another aspect is that parsing JavaScript involves transforming plain text to a data structure called abstract syntax tree (AST) and turning that into binary format. WebAssembly is delivered as binary, and decoding it happens much faster. It’s statically typed so, unlike with JavaScript, the engine doesn’t need to speculate during compilation about what types will be used.

When is this an advantage? Think about the cases where you need to use software outside the browser: video games, video editing, 3D rendering, or music production. These applications do a lot of calculations and require a high degree of performance. That kind of performance is hard to get from JavaScript.

Also, the cost of downloading, parsing, and compiling very large JavaScript applications can be prohibitive. Mobile and other resource-constrained platforms can further amplify these performance bottlenecks.
  - More on the topic in the post: [The Mobile Performance Inequality Gap, 2021](https://infrequently.org/2021/03/the-performance-inequality-gap/)

## Portability
Think about the software you need to use outside the browser, especially on the huge and established C/C++ applications used for image processing that can now run on the web, by using WebAssembly as its compile target.

Also, languages like Rust can have specialized tooling and support for WebAssembly, to offer a great experience for writing for the web, in languages that historically could not be run on the web. Standalone Server-side runtimes (like Wasmer, Wasmtime) allow for running WebAssembly on the server, as its own application, or embedded in a host application written in a variety of languages. Allowing you to write business logic, or computationally intensive algorithms once and have a commonly shared implementation that is run in a lightweight runtime.

## When should I generally use WebAssembly?

WebAssembly has become another tool in a developer's toolbelt. For example, a web developer's tools are HTML (Declaring UI), CSS (Styling UI), JavaScript (Adding functionality to a UI), and now WebAssembly (Processing heavy tasks, to give results back to the UI). Using the right tool for the job is what will give you the best results for your project.

A good rule of thumb is: Use WebAssembly for computationally intensive tasks, such as games, image manipulation, math, physics, audio effects, etc. More scenarios on the AssemblyScript site: https://www.assemblyscript.org/frequently-asked-questions.html#frequently-asked-questions

In practice, for web developers to start writing wasm code without using C or Rust, AssemblyScript is the best option. AssemblyScript compiles a strict variant of TypeScript to WebAssembly, allowing web developers to keep using TypeScript-compatible tooling.


## Usage on The Graph
I have used WebAssembly while developing a subgraph for web3 apps. From what I could see the wasm part is mature enough and works as expected, much of the problems I faced were caused by the early stage of subgraph development toolset.

About AssemblyScript, the advertisement states that "You are now able to write WebAssembly without learning a new language...", which is true and false. True because it does look like regular typescript code, however, multiple basic functionalities are not yet supported, such as closures, and I was often only aware of the constraints at compilation time which is annoying.

## References
- [https://en.wikipedia.org/wiki/Stack_machine](https://en.wikipedia.org/wiki/Stack_machine)
- [https://andreabergia.com/stack-based-virtual-machines/](https://andreabergia.com/stack-based-virtual-machines/)
- [https://ethereum.org/en/developers/docs/evm/](https://ethereum.org/en/developers/docs/evm/)
- [https://blog.logrocket.com/webassembly-how-and-why-559b7f96cd71/](https://blog.logrocket.com/webassembly-how-and-why-559b7f96cd71/)
- [https://madewithwebassembly.com/about/](https://madewithwebassembly.com/about/)
- [https://developer.mozilla.org/en-US/docs/WebAssembly/Concepts](https://developer.mozilla.org/en-US/docs/WebAssembly/Concepts)

