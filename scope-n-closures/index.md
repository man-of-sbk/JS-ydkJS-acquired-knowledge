# Foreword
# Preface
# Chapter 1: What is Scope?
* a well-defined set of rules for storing variables in some location, and for finding those variables at a later time. We'll call that set of rules: `Scope`.

## Compiler Theory
* (me: Js)(it) is in fact a `compiled language`.

* In a `traditional compiled-language process`, a chunk of source code, your program, will undergo typically `three steps` `before` it is executed, roughly called "`compilation`":
  1. `Tokenizing/Lexing`: [...] consider the program: `var a = 2;`. This program would likely be `broken up into` the following tokens: `var`, `a`, `=`, `2`, and `;`. `Whitespace` `may or may not` be persisted as a `token`, `depending on` whether it's meaningful or not.

  2. `Parsing`: taking a `stream` (`array`) `of tokens` [...] turning it into a `tree` of nested elements [...] This `tree` is called an "`AST`" (<b>A</b>bstract <b>S</b>yntax <b>T</b>ree).
    * The tree for `var a = 2;` `might` start with a [...] node called `VariableDeclaration`, with a child node called `Identifier` (whose value is `a`), and another child called `AssignmentExpression` [...] has a child called `NumericLiteral` (whose value is `2`).

  3. `Code-Generation`: [...] taking an `AST` and turning it into `executable code`. This part [...] depending on the language.
    * take [...] `AST` (me: of) `var a = 2;` [...] turn it into a set of `machine instructions` to `actually` create a variable called `a` [...] then store a value into `a`.

* JavaScript engines don't [...] having plenty of time to optimize, because [...] compilation doesn't happen in a `build step` `ahead of time`.

* the compilation [...] happens [...] `microseconds` (or less!) before the code is executed. To ensure the fastest performance, JS engines use all kinds of tricks (like `JITs` [...] etc.).

* `any snippet` of JavaScript has to be compiled [...] `right before` [...] it's executed. So, the `JS compiler` will take [...] `var a = 2;` and compile it `first` [...] then [...] execute it.

## Understanding Scope
### The Cast
* Let's meet the `cast of characters` that `interact` `to process` [...] `var a = 2;`.
  1. `Engine`: responsible for start-to-finish compilation and execution of our JavaScript program.

  2. `Compiler`: one of `Engine`'s friends [...] parsing and code-generation.

  3. `Scope`: [...] friend of `Engine`; collects and maintains a `look-up list` of all [...] variables [...] and enforces a strict set of rules as to how these are accessible to currently executing code.

### Back & Forth
* the program `var a = 2;` [...] likely think of [...] `one statement` [...] `Engine` sees `two` distinct statements, one which `Compiler` will handle `during compilation`, and one which `Engine` will handle `during execution`.

* The first thing `Compiler` will do [...] is perform `lexing` to break it down into `tokens`, which it will then parse into a `tree`. [...] when `Compiler` gets to `code-generation`, it will:
  1. Encountering `var a`, `Compiler` asks `Scope` to see if a variable `a` already exists for [...] particular `scope collection`. If so, `Compiler` ignores this declaration and moves on. `Otherwise`, `Compiler` asks `Scope` to declare a `new variable` called `a` for that `scope collection`.

  2. `Compiler` then produces code for `Engine` to `later execute`, to handle the `a = 2` assignment [...] `Engine` runs will first ask `Scope` if there is a variable called `a` accessible in the current `scope collection`. If so, `Engine` uses that variable. If not, `Engine` looks `elsewhere` (see `nested Scope` section below).
    * If `Engine` eventually finds a variable, it assigns the value `2` to it. If not, `Engine` will raise [...] an error!

### Compiler Speak
* When `Engine` executes the code that `Compiler` produced for step (`2`), it has to look-up the variable `a` to see if it has been declared [...] But the `type of look-up` `Engine` `performs` affects the `outcome` of `the look-up`.

* In our case [...] `Engine` would be performing an "`LHS`" `look-up` for the variable `a`. The other type of look-up is called "`RHS`".

* `LHS` and `RHS` `meaning` "`left/right-hand side of an assignment`" (me: `but`) `doesn't` necessarily literally mean "left/right side of the `=` assignment operator". There are `several other ways` [...] `assignments happen` [...] it's `better` to [...] think about it as: "`who's the target of the assignment` `(LHS)`" and "`who's the source of the assignment` `(RHS)`".

Consider this program, which has both `LHS` and `RHS` references:

```js
function foo(a) {
	console.log( a ); // 2
}

foo( 2 );
```

* `foo(..)` [...] requires an `RHS` reference to `foo`, meaning, "go look-up the value of `foo`, and give it to me.".

* You may have missed the `implied` `a = 2` [...] happens when the value `2` is passed [...] to the `foo(..)` function [...] To (`implicitly`) assign to parameter `a`, an `LHS look-up` is performed (me: `Engine` look for the variable in the current scope).

* There's also an `RHS reference` for the value of `a`, and that [...] value is passed to `console.log(..)`.

* `console.log(..)` needs a reference to execute. It's an `RHS look-up` for the `console` object, then a `property-resolution` occurs to see if it has a method called `log`.

* Finally [...] Inside of the native implementation of `log(..)`, we can assume it has parameters, the first of which [...] has an `LHS reference look-up`, `before assigning` `2` to it.

* You might [...] conceptualize the function declaration `function foo(a) {...` as [...] `var foo` and `foo = function(a){...`. In so doing, it would be tempting to think of this function declaration as involving an `LHS look-up`
* However [...] `Compiler` handles both the `declaration` and the `value definition` `during code-generation`, such that when `Engine` is executing code, there's no [...] necessary to "assign" a function value to `foo`. Thus, it's `not` really appropriate to think of a `function declaration` as an `LHS look-up` assignment.
* (me: `a = 2` does not contain the declaration of `a`, it is only tempting to make an assignment, `a = 2`. Thus, an `LHS loop-up` is performed).

### Engine/Scope Conversation

```js
function foo(a) {
	console.log( a ); // 2
}

foo( 2 );
```

Let's imagine the above exchange (which processes this code snippet) as a conversation. The conversation would go a little something like this:

> ***Engine***: Hey *Scope*, I have an RHS reference for `foo`. Ever heard of it?

> ***Scope***: Why yes, I have. *Compiler* declared it just a second ago. He's a function. Here you go.

> ***Engine***: Great, thanks! OK, I'm executing `foo`.

> ***Engine***: Hey, *Scope*, I've got an LHS reference for `a`, ever heard of it?

> ***Scope***: Why yes, I have. *Compiler* declared it as a formal parameter to `foo` just recently. Here you go.

> ***Engine***: Helpful as always, *Scope*. Thanks again. Now, time to assign `2` to `a`.

> ***Engine***: Hey, *Scope*, sorry to bother you again. I need an RHS look-up for `console`. Ever heard of it?

> ***Scope***: No problem, *Engine*, this is what I do all day. Yes, I've got `console`. He's built-in. Here ya go.

> ***Engine***: Perfect. Looking up `log(..)`. OK, great, it's a function.

> ***Engine***: Yo, *Scope*. Can you help me out with an RHS reference to `a`. I think I remember it, but just want to double-check.

> ***Scope***: You're right, *Engine*. Same guy, hasn't changed. Here ya go.

> ***Engine***: Cool. Passing the value of `a`, which is `2`, into `log(..)`.

> ...

### Quiz

Check your understanding so far. Make sure to play the part of *Engine* and have a "conversation" with the *Scope*:

```js
function foo(a) {
	var b = a;
	return a + b;
}

var c = foo( 2 );
```

1. Identify all the LHS look-ups (there are 3!).

2. Identify all the RHS look-ups (there are 4!).

**Note:** See the chapter review for the quiz answers!

## Nested Scope

We said that *Scope* is a set of rules for looking up variables by their identifier name. There's usually more than one *Scope* to consider, however.

Just as a block or function is nested inside another block or function, scopes are nested inside other scopes. So, if a variable cannot be found in the immediate scope, *Engine* consults the next outer containing scope, continuing until found or until the outermost (aka, global) scope has been reached.

Consider:

```js
function foo(a) {
	console.log( a + b );
}

var b = 2;

foo( 2 ); // 4
```

The RHS reference for `b` cannot be resolved inside the function `foo`, but it can be resolved in the *Scope* surrounding it (in this case, the global).

So, revisiting the conversations between *Engine* and *Scope*, we'd overhear:

> ***Engine***: "Hey, *Scope* of `foo`, ever heard of `b`? Got an RHS reference for it."

> ***Scope***: "Nope, never heard of it. Go fish."

> ***Engine***: "Hey, *Scope* outside of `foo`, oh you're the global *Scope*, ok cool. Ever heard of `b`? Got an RHS reference for it."

> ***Scope***: "Yep, sure have. Here ya go."

The simple rules for traversing nested *Scope*: *Engine* starts at the currently executing *Scope*, looks for the variable there, then if not found, keeps going up one level, and so on. If the outermost global scope is reached, the search stops, whether it finds the variable or not.

### Building on Metaphors

To visualize the process of nested *Scope* resolution, I want you to think of this tall building.

<img src="fig1.png" width="250">

The building represents our program's nested *Scope* rule set. The first floor of the building represents your currently executing *Scope*, wherever you are. The top level of the building is the global *Scope*.

You resolve LHS and RHS references by looking on your current floor, and if you don't find it, taking the elevator to the next floor, looking there, then the next, and so on. Once you get to the top floor (the global *Scope*), you either find what you're looking for, or you don't. But you have to stop regardless.

## Errors

Why does it matter whether we call it LHS or RHS?

Because these two types of look-ups behave differently in the circumstance where the variable has not yet been declared (is not found in any consulted *Scope*).

Consider:

```js
function foo(a) {
	console.log( a + b );
	b = a;
}

foo( 2 );
```

When the RHS look-up occurs for `b` the first time, it will not be found. This is said to be an "undeclared" variable, because it is not found in the scope.

If an RHS look-up fails to ever find a variable, anywhere in the nested *Scope*s, this results in a `ReferenceError` being thrown by the *Engine*. It's important to note that the error is of the type `ReferenceError`.

By contrast, if the *Engine* is performing an LHS look-up and arrives at the top floor (global *Scope*) without finding it, and if the program is not running in "Strict Mode" [^note-strictmode], then the global *Scope* will create a new variable of that name **in the global scope**, and hand it back to *Engine*.

*"No, there wasn't one before, but I was helpful and created one for you."*

"Strict Mode" [^note-strictmode], which was added in ES5, has a number of different behaviors from normal/relaxed/lazy mode. One such behavior is that it disallows the automatic/implicit global variable creation. In that case, there would be no global *Scope*'d variable to hand back from an LHS look-up, and *Engine* would throw a `ReferenceError` similarly to the RHS case.

Now, if a variable is found for an RHS look-up, but you try to do something with its value that is impossible, such as trying to execute-as-function a non-function value, or reference a property on a `null` or `undefined` value, then *Engine* throws a different kind of error, called a `TypeError`.

`ReferenceError` is *Scope* resolution-failure related, whereas `TypeError` implies that *Scope* resolution was successful, but that there was an illegal/impossible action attempted against the result.

## Review (TL;DR)

Scope is the set of rules that determines where and how a variable (identifier) can be looked-up. This look-up may be for the purposes of assigning to the variable, which is an LHS (left-hand-side) reference, or it may be for the purposes of retrieving its value, which is an RHS (right-hand-side) reference.

LHS references result from assignment operations. *Scope*-related assignments can occur either with the `=` operator or by passing arguments to (assign to) function parameters.

The JavaScript *Engine* first compiles code before it executes, and in so doing, it splits up statements like `var a = 2;` into two separate steps:

1. First, `var a` to declare it in that *Scope*. This is performed at the beginning, before code execution.

2. Later, `a = 2` to look up the variable (LHS reference) and assign to it if found.

Both LHS and RHS reference look-ups start at the currently executing *Scope*, and if need be (that is, they don't find what they're looking for there), they work their way up the nested *Scope*, one scope (floor) at a time, looking for the identifier, until they get to the global (top floor) and stop, and either find it, or don't.

Unfulfilled RHS references result in `ReferenceError`s being thrown. Unfulfilled LHS references result in an automatic, implicitly-created global of that name (if not in "Strict Mode" [^note-strictmode]), or a `ReferenceError` (if in "Strict Mode" [^note-strictmode]).

### Quiz Answers

```js
function foo(a) {
	var b = a;
	return a + b;
}

var c = foo( 2 );
```

1. Identify all the LHS look-ups (there are 3!).

	**`c = ..`, `a = 2` (implicit param assignment) and `b = ..`**

2. Identify all the RHS look-ups (there are 4!).

    **`foo(2..`, `= a;`, `a + ..` and `.. + b`**


[^note-strictmode]: MDN: [Strict Mode](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions_and_function_scope/Strict_mode)
