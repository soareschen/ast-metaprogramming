# AST Metaprogramming and Implementing JavaScript inside JavaScript

There have been many religious debate in programming paradigms, such as static vs dynamic typing, functional vs object oriented programming, etc. While each camp of supporters have their valid reasons to love or hate some language features, there is really no black-and-white distinction of which paradigm is good or bad. In reality, each programming paradigms have their own strengths and weaknesses, and we often sacrifice many tradeoffs by restricting ourselves to just follow certain paradigms.

But what if there is a new programming paradigm that combines the best of each opposing paradigms and make them work together as one? In this article, I will share about a new programming paradigm that I have either invented or rediscovered, and I would call it _AST metaprogramming_.

## AST Metaprogramming

In short, _AST metaprogramming is functional composition in steroid using object-oriented programming_. At the core, AST metaprogramming is made of composable functions that are mostly side-effect free. But instead of composing these functions manually, they are encapsulated inside objects and are linked together to form AST-like data structures.

For those of you who don’t know, AST stands for [Abstract Syntax Tree](http://en.wikipedia.org/wiki/Abstract_syntax_tree). AST is commonly used to represent the code that you have written after they are parsed by compilers and interpreters. In other words, all code are parsed into AST before they are eventually transformed into machine code.

The AST metaprogramming paradigm can be applied in most programming languages, but here I will mostly focus on applying it in JavaScript (ES6). AST metaprogramming also make heavy use of dynamic typed language features, although static typed languages can still take advantage of some of its basic concepts.

## Macro Programming

The concept of AST manipulation itself is not new. The most well known example is called macro programming in the [Lisp](http://en.wikipedia.org/wiki/Lisp_%28programming_language%29) family of programming languages. Lisp is special in that the AST representation of its source code are simply lists, which can easily be created inside Lisp programs. With this it is possible to write Lisp programs that generate Lisp code, and these are often called Lisp macros.

AST metaprogramming is similar but not exactly the same as macro programming. The key difference is that the AST can be structured in any way and not related to the source language.

Put it in another way - macros are written as functions in a host language, accepting an AST of that host language and transform it into a different AST of the same host language; AST Metaprogramming is written on a host language to construct AST of a target meta-language, which then generates programs as functions inside the host language or in some other form.

## Domain Specific Languages

The concept of AST metalanguages is similar to domain specific languages (DSL) but there are some major differences. A DSL is typically created by using its host language features to create new syntax for the DSL.

In contrast AST metaprogramming there is no syntactic representation like typical languages. The parsing phase is skipped entirely and we are instead programmatically building the AST of a meta-language.

Put it in another way - AST metaprogramming can be viewed as _syntaxless DSL_. An _AST metalanguage_ exists only in the form of AST constructed programmatically in a host language.

## AST Meta-Language

Without the syntax, an AST metalanguage is arguably harder to use than a domain specific language. A user would need to be proficient in both the host language as well as the metalanguage in order to build an application.

But unlike DSL, the goal of AST metaprogramming is not to be used by non-coder to write some “pseudocode” in specific domains. Rather it is for experienced programmers to build powerful applications. 

As we will see later, the ease of programmatically construct an AST also means that it is as easy to generate new AST or manipulate existing AST by essentially doing metaprogramming in the host language.

## Functional Programming

The concept of AST programming was first developed when I was designing a component system for my project [Quiver.js](http://quiverjs.org). The way to create a web application in Quiver is by writing many small and composable functions that are then combined into a single function as the result application.

```javascript
function handler(request, response)
function builder(config) => handler
function middleware(config, builder) => handler

var combineBuilderMiddleware = (middleware, builder) =>
  config =>
    middleware(config, builder)

var combineMiddlewares = (middleware1, middleware2) =>
  (config, builder) =>
    middleware2(config, middleware1(config, builder))
```

Above shows some simplified function definitions in Quiver. The `builder` and `middleware` functions both can generate handler functions based on their arguments. While we can implement all builder and middleware functions by hand, we can also generate new builder and middleware functions by combining existing functions.

This functional programming technique of generating new functions by composing existing functions can can be very powerful. But in practice, manually composing many small functions together quickly become tedious and introduce a lot of boilerplate code. When functions are composed directly, it also becomes hard to modify or inspect the composition from the outside.

## Object Oriented Programming

On the other hand, object oriented programming is well suited for building data structures that describe the relationship between composable functions. The structure of functional composition also resembles the AST structures of programming languages.

Generalizing that, we are in fact building AST metalanguages that generate new programs based on the AST we constructed.

```javascript
class HandlerComponent {
  constructor(builder)
  loadHandler(config) => handler
  toBuilder() => builder
  addMiddleware(middlewareComponent) => this
}

class MiddlewareComponent {
  constructor(middleware)
  toMiddleware() => middleware
  addMiddleware(middlewareComponent) => this
}

let cacheMiddleware = new MiddlewareComponent(
  (config, builder) => { ... })

let mainApp = new HandlerComponent(
  config => { ... })
  .addMiddleware(cacheMiddleware)

let config = { ... }
let handler = mainApp.loadHandler(config)

http.createServer(handler)
```

Above shows a simplified overview of the Quiver component system. We define `HandlerComponent` and `MiddlewareComponent` classes that wrap around the builder and middleware functions we defined earlier. 

Both component classes have an `.addMiddleware()` function that adds a middleware component to the current component. When a handler component’s `.toBuilder()` method is called, it will combine the original builder function provided in its constructor with the middleware functions generated by its middleware components. Similarly the middleware function generated by a middleware component is combined in the similar way.

In Quiver, the handler and middleware component classes are essentially the AST for building a Quiver web application. The result component AST would have hundreds of nodes, each containing small and composable functions. Using that new applications can be easily created by simply rearranging the components and construct different component AST.

## AST Manipulation

AST metaprogramming combines the best concepts in macro programming and DSL and creates a new way of metaprogramming. Because AST are just ordinary objects inside a host language, it becomes very easy to manipulate the AST by simply writing code in the host language.

This technique of manipulating an AST metalanguage is the reason why this paradigm is called AST _metaprogramming_. In typical programming languages, metaprogramming is a powerful and advanced technique to manipulate a program’s structure before it is executed or compiled. But metaprogramming is usually hard to implement, because they are restricted to use only subset of a language feature.

By using a host language as a platform to do metaprogramming on AST metalanguages, we are given the full power of the host language to do metaprogramming in any way we like.

It is beyond our scope to discuss here what kind of metaprogramming techniques we can use to create applications. But to demonstrate that, the following is an example of metaprogramming in Quiver to debug Quiver applications by injecting a debug middleware to each of the subcomponents.

```javascript
let debugMiddleware = new MiddlewareComponent(
  (config, builder) => { ... })

let debuggableComponent = (component, mapTable={}) =>
  component.map(debuggableComponent, mapTable)
  .addMiddleware(debugMiddleware(component))
```

Similar to mapping lists, `component.map()` maps a component’s subcomponents by calling a map function provided. Because each Quiver subcomponents would in turn have their own subcomponents, the function above is also recursive so that all nested subcomponents can be debugged.

From the example, we can see that AST can be treated like ordinary data structure in a host language. With that we can apply functional programming techniques such as `map()` to these data structures and not even think of them as programs.

## Meta-Language Extension

An AST metalanguage is designed based on the AST structure instead of syntax. Because of that it is also much easier to extend an AST metalanguage than a DSL.

By defining the AST structure as abstract interfaces, it is possible to define custom implementation of an AST node and remain interoperable with each others. Using object-oriented programming, we can also easily define new subclasses that extend the feature of an AST node implementation.

Quiver uses this technique to create new handler component types such as simple handler. And because they are all just subclasses, that means anyone can define new handler component types and effectively “extending” the Quiver metalanguage.

## Function Generation

At the simplest level, an AST node generates new functions by composing the functions generated by its subnodes. The generated functions would have specific signature that can easily be further composed.

This is a simpler version of AST metaprogramming because the program generated by AST can simply be functions implemented in the host language. But AST metaprogramming don’t necessarily need to generate programs running on the host language.

## Code Generation

At more advanced levels, we could also use AST metaprograming to design AST that generates code directly. The generated code can be in any target language, including C, JVM byte code, LLVM, or assembly language. The generated code can then be fed into respective compilers to generate the final program binary.

It is at this level that AST metaprogramming shows its similarity with AST of programming languages in typical compilers. The main difference between AST metaprogramming and typical compilers is that AST metaprogramming skips the syntax and parsing of a source language and goes directly to constructing the source AST in a host language.

Put it in other words - A compiler runs on a host language, accepts code written in a source language, parse it into AST of a host language, and generate code written in a target language; AST metaprogramming runs on a host language, programmatically constructs AST of a source metalanguage, executes the AST methods to generate code written in a target language.

The main advantage of the AST metaprogramming is that it loosely defines a AST metalanguage that can be easily extensible and manipulated. By retaining the AST structure, an AST metalanguage can be as expressive and powerful as any real programming language. By eliminating the syntactic barrier, an AST metalanguage is much easier to be extended and be manipulated by doing metaprogramming on the host language.

## Static Typed AST Meta-Language in JavaScript

By applying compiler design knowledge to AST metaprogramming, we can indeed implement a static typed AST metalanguage in JavaScript that is capable of generating machine code.

There are many advantages of writing in a static typed metalanguage on JavaScript over coding in a static typed language directly, which we will explore slowly. But the first reason is the same as using AST metaprogramming in other use cases - it allows us to manipulate the AST _in JavaScript_ before generating the program.

```javascript
function codeGenerator(env) => bytecode

class Expr {
  toCodeGenerator() => codeGenerator
}

class IfExpr extends Expr {
  constructor(conditionExpr, thenExpr, elseExpr)
}
```

A very rough concept of the classes in this AST metalanguage would be something like above. It would have an expression base class that generates function that generates [LLVM](http://llvm.org/) byte code base on given environment information such as base memory address.

A traditional if expression could then be implemented by defining an `IfExpr` class with constructor taking three expression AST. The `IfExpr` code generator would then simply wire up the code generated by its branch expressions and return the combined byte code.

With AST metaprogramming, our static metalanguage becomes extensible and we can easily define new expression types beyond just classical expressions in typical programming languages.

In fact, AST metaprogramming would allow us to extend any language feature in this static metalanguage. We can for example implement multiple generic type systems, or even write generic JavaScript functions that generate AST based on given types.

## Compiler Optimization

AST metaprogramming can not only generate machine code from static metalanguages but also perform optimization on it. Because the constructed AST are just ordinary objects, compiler optimization can be done by simply writing JavaScript functions that manipulate the AST.

The compiler-related functions can also be shared across multiple AST metalanguages by defining general interfaces that the metalanguages can implement. For example, one can define a generic [control flow graph](http://en.wikipedia.org/wiki/Control_flow_graph) interface and then create control flow analysis functions to perform optimization on any AST classes that implement that interface.

Using this method of optimization, we can not only easily write compiler optimization code in JavaScript, but also easily add in new optimization techniques to the metalanguages.

## ASM.js

JavaScript has another unique feature that makes it particularly suited AST metaprogramming. With [Emscripten](http://emscripten.org) and [asm.js](http://asmjs.org/), it has been possible to run native programs compiled from other languages to run on JavaScript at near-native speed. 

This is made possible by compiling LLVM byte code into a restricted version of JavaScript called asm.js. It allows the JavaScript engine to statically analyze the types in the JavaScript code and perform ahead-of-time (AOT) compilation to convert it into machine code.

Asm.js has the performance advantage of running JavaScript at near-native speed. But in practice it is actually not feasible to code in asm.js by hand. This is because asm.js is as low level as assembly languages despite its JavaScript-like syntax. As a result today’s asm.js programs are typically written in source languages other than JavaScript itself.

But with AST metaprogramming, we can harness the power of asm.js and construct static typed AST metalanguages that generate asm.js code. In other words, we can indirectly write static typed programs in JavaScript that run at near-native speed.

## Dynamic Code Generation

There is another reason why JavaScript as a dynamic typed language is much more suited for AST metaprogramming. Without constraint of a formal type system, it means that we could even construct AST and generate asm.js code on the fly and execute it at runtime.

In other words we can effectively perform manual just-in-time (JIT) compilation on performance-critical sections of our programs based on real-time requirements.

## Network Data Parsing

One of the example use cases for dynamic code generation is to write JavaScript code that process network data. For instance it is possible to build an AST metalanguage similar to [Protocol Buffers](https://code.google.com/p/protobuf/). The AST of this metalanguage will generate asm.js functions that can parse structured network data efficiently.

With the dynamic nature of JavaScript, such network application can also easily be configured at runtime to accept new data formats, all without having to restart the application.

## Higher Order AST Metaprogramming

Metaprogramming is great, but what’s even greater if we could do meta-metaprogramming on our meta programs? The dynamic type system in JavaScript also enables this type of indefinite meta-level of indirection in writing programs.

Once we have AST metalanguages defined, we can furthermore create new AST metalanguages that generate AST of other metalanguages. For the earlier example, we could for instance create an AST metalanguage that defines network data format, which generates AST of the static typed metalanguage, which in turns generates asm.js code.

On top of our static typed AST metalanguage, we can also create higher order type-theory AST metalanguage that defines and generates new types. Because types are still just objects in JavaScript, we can even perform advanced type-level programming like the ones in Haskell, except with much less restrictions.

Regardless of how deep the metaprogramming goes, the host language remains to be JavaScript. As such it is no more complicated to write a meta-metaprogram than a metaprogram. Because of that, a fully implemented static typed metalanguage in JavaScript could have type system as powerful as the one in Haskell, but with much more flexibility of metaprogramming in JavaScript.

## JS Implemented in JS

Combining all advanced concepts in AST metaprogramming, one of the ultimate projects made possible is to implement JavaScript in JavaScript.

It is not easy to [self host](http://en.wikipedia.org/wiki/Self-hosting) a dynamic typed programming language on itself. The lack of type information makes it hard to write a language implementation that implements its own runtime. Nevertheless there are successful examples of dynamic typed languages self hosting, notably [PyPy](http://pypy.org/) and [Rubinius](http://rubini.us/).

Consider the case of PyPy - PyPy defines a language subset of Python called RPython and use it to implement Python. The restricted nature of RPython allows PyPy to statically analyze the RPython program and generate code from it. And because RPython is subset of Python, it is possible to run PyPy on both Python and bare metals.

Using AST metaprogramming, we could develop an alternative approach to self host JavaScript. At the base level, the JavaScript engine can be implemented on top of a static typed AST metalanguage. The metalanguage would generate the JavaScript engine code in multiple formats, including LLVM, asm.js, and JavaScript itself. As such the implemented JavaScript engine can also run on JavaScript.

But a static AST metalanguage alone is not much more productive than any typical static typed programming language. This is where the power of metaprogramming techniques come in. By utilizing JavaScript itself as a host language, we can write JavaScript functions that manipulate and generate ASTs. We can even write higher order AST metalanguages that generate the base AST, such as parser generators.

## Just The Beginning

Currently I only have the rough idea written here on possible ways to implement JavaScript in JavaScript using AST metaprogramming. I am not sure if it is feasible in practice, but it is worth the effort to explore the possibility.

If the technique is proven to be feasible, AST metaprogramming might be the key to our dream of implement everything in JavaScript - Not just JavaScript engines, but also perhaps browsers, rendering engines, and even operating system kernels.

This just the beginning of a journey for a new programming paradigm. Let us all take this concept of metaprogramming to the next level and see what we can do with it!
