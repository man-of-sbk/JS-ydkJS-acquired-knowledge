# Foreword
# Preface
# Chapter 1: What is Scope?
* a well-defined set of rules for storing variables in some location, and for finding those variables at a later time. We'll call that set of rules: `Scope`.

## Compiler Theory
* (me: Js)(it) is in fact a `compiled language`.

* In a `traditional compiled-language process`, a chunk of source code, your program, will undergo typically `three steps` `before` it is executed, roughly called "`compilation`":
  1. `Tokenizing/Lexing`: [...] consider the program: `var a = 2;`. This program would likely be `broken up into` the following tokens: `var`, `a`, `=`, `2`, and `;`. `Whitespace` `may or may not` be persisted as a `token`, `depending on` whether it's meaningful or not.
    * The difference between `tokenizing` and `lexing` is subtle [...] if the tokenizer were to invoke `stateful parsing rules` `to figure out` whether `a` should be considered a `distinct token` or just `part of another token`, that would be `lexing`.
    * (me: A `tokenizer` breaks a stream of text into tokens [...] A `lexer` is basically a `tokenizer`, but it usually attaches extra context to the tokens -- this token is a number, that token is a string literal [...]. [Learn more](https://stackoverflow.com/questions/380455/looking-for-a-clear-definition-of-what-a-tokenizer-parser-and-lexers-are))

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
* When `Engine` executes the code that `Compiler` produced for step (`2`), it has to look-up the variable `a` to see if it has been declared
* the `type of look-up` `Engine` `performs` affects the outcome of `the look-up`.

* In our case [...] `Engine` would be performing an "`LHS`" `look-up` for the variable `a`. The other type of look-up is called "`RHS`".

* a little more precise. An `RHS look-up` is [...] a look-up of the `value of some variable`
* whereas the `LHS look-up` is trying to find the `variable` [...] `so that it can assign`.

```js
function foo(a) {
	console.log( a ); // 2
}

foo( 2 );
```

* `foo(..)` [...] requires an `RHS` reference to `foo`, meaning, "go look-up the value of `foo`, and give it to me.".

* You may have missed the `implied` `a = 2` [...] happens when the value `2` is passed [...] to the `foo(..)` function [...] To (`implicitly`) assign to parameter `a`, an `LHS look-up` is performed).

* There's also an `RHS reference` for the value of `a`, and that [...] value is passed to `console.log(..)`.

* `console.log(..)` needs a reference to execute. It's an `RHS look-up` for the `console` object, then a `property-resolution` occurs to see if it has a method called `log`.

* Finally [...] Inside of the native implementation of `log(..)`, we can assume it has parameters, the first of which [...] has an `LHS reference look-up`, `before assigning` `2` to it.

* You might [...] conceptualize [...] `function foo(a) {...` as [...] `var foo` and `foo = function(a){...` [...] However [...] `Compiler` handles both the `declaration` and the `value definition` during code-generation (me: for `function declarations` like the one above), such that when `Engine` is executing code, there's no [...] "assign" a function value to `foo`. Thus, it's `not` really appropriate to think of a `function declaration` as an `LHS look-up` assignment.

### Engine/Scope Conversation
### Quiz
* (CL)

## Nested Scope
* `scopes` are nested inside other `scopes` [...] if a variable cannot be found in the `immediate scope`, `Engine` consults the next `outer containing scope`, `continuing` until found or until the `outermost` (aka, `global`) `scope` has been reached.

```js
function foo(a) {
	console.log( a + b );
}

var b = 2;

foo( 2 ); // 4
```

* `Engine` starts at the currently executing `Scope`, looks for the variable there, then if not found, keeps `going up` one level, and so on. If the `outermost global scope` is reached, the search `stops`, whether it finds the variable or not.

### Building on Metaphors
## Errors
* Why does it matter whether we call it `LHS` or `RHS`?
* Because these [...] behave differently in the circumstance where the variable [...] is not found in any [...] `Scope`.

* If an `RHS look-up` fails to ever find a variable, anywhere in the nested `Scopes`, this results in a `ReferenceError` [...] It's important to note that the error is [...] `ReferenceError`.

* if the `Engine` is performing an `LHS look-up` and `arrives at` the [...] `global Scope` [...] without finding it, `and if` [...] `not` running in "`Strict Mode`" [...] then [...] a new variable of that name (me: is created) `in the global scope`.

* "`Strict Mode`" [...] added in `ES5`, has [...] `different behaviors` from `normal`/`relaxed`/`lazy` `mode`. One such behavior is that it disallows the [...] implicit global variable creation. In that case (me: if the loop-up could not find the variable in the global scope) [...] `Engine` would throw a `ReferenceError` similarly to the `RHS case`.

* if a variable is found for an `RHS look-up`, but you try to do something with its value `that is impossible`, such as [...] reference a property on a `null` or `undefined` value [...] `Engine` throws [...] `TypeError`.

* `ReferenceError` is `Scope` `resolution-failure` related, whereas `TypeError` implies that `Scope` resolution was `successful`, but that there was an `illegal/impossible` action attempted against the result.

## Review (TL;DR)
### Quiz Answers

# Chapter 2: Lexical Scope
* There are two predominant models for how scope works. The first [...] used by the vast majority of programming languages [...] `Lexical Scope` [...] other model [...] `Dynamic Scope` [...] (me: which is) covered in `Appendix A`.

## Lex-time
* the first traditional phase of a standard language compiler is called `lexing` (aka, `tokenizing`).

* `lexical scope` is scope that is defined at `lexing time`. In other words, `lexical scope` is based on where variables and blocks of scope are authored, `by you`, `at write time`, and thus is (mostly) (me: an idiom `impossible to change`)(`set in stone`) by the time the `lexer` processes your code.

* We will see [...] some ways to cheat `lexical scope`, thereby modifying it after the `lexer` has passed by, but [...] It is considered best practice to treat `lexical scope` as [...] lexical-only, and thus entirely `author-time` in nature.

```js
function foo(a) {

	var b = a * 2;

	function bar(c) {
		console.log( a, b, c );
	}

	bar(b * 3);
}

foo( 2 ); // 2 4 12
```

* There are `three nested scopes` [...] in this code example [...] think [...] as `bubbles` `inside of each other`.
  * `Bubble 1` [...] the global scope [...] has just `one identifier` [...]: `foo`.
  * `Bubble 2` [...] the scope of `foo` [...] includes the `three identifiers`: `a`, `bar` and `b`.
  * `Bubble 3` [...] the scope of `bar` [...] includes just `one identifier`: `c`.

### Look-ups
* In the above code snippet, the `Engine` executes the `console.log(..)` statement and goes looking for [...] `a`, `b`, and `c` [...] first starts with the `innermost scope bubble`, the scope of the `bar(..)` function. It won't find `a` there, so it goes up [...] to the next `nearest scope bubble`, the scope of `foo(..)`. It finds `a` there, and [...] uses that `a`. Same thing for `b`. But `c`, it does find inside of `bar(..)`.

* `Scope look-up stops once it finds the first match`.
* same `identifier name` can be specified at `multiple layers` of `nested scope` [...] called "shadowing" (the `inner identifier` "`shadows`" the `outer identifier`).

* `Global` variables are [...] automatically properties of the global object (`window` in browsers, etc.).

```js
// (me)
var a = 2

window.a
```

* This technique gives `access` to a `global variable` which would [...] be inaccessible due to it being `shadowed`.

* The `lexical scope look-up` `process` only applies to `first-class` identifiers, such as the `a`, `b`, and `c`. If you had a reference to `foo.bar.baz` [...] the [...] look-up would apply to finding the `foo` identifier [...] once it locates that variable, `object property-access` `rules` take over to resolve the `bar` and `baz` properties.

## Cheating Lexical
If lexical scope is defined only by where a function is declared, which is entirely an author-time decision, how could there possibly be a way to "modify" (aka, cheat) lexical scope at run-time?

JavaScript has two such mechanisms. Both of them are equally frowned-upon in the wider community as bad practices to use in your code. But the typical arguments against them are often missing the most important point: **cheating lexical scope leads to poorer performance.**

Before I explain the performance issue, though, let's look at how these two mechanisms work.

### `eval`

The `eval(..)` function in JavaScript takes a string as an argument, and treats the contents of the string as if it had actually been authored code at that point in the program. In other words, you can programmatically generate code inside of your authored code, and run the generated code as if it had been there at author time.

Evaluating `eval(..)` (pun intended) in that light, it should be clear how `eval(..)` allows you to modify the lexical scope environment by cheating and pretending that author-time (aka, lexical) code was there all along.

On subsequent lines of code after an `eval(..)` has executed, the *Engine* will not "know" or "care" that the previous code in question was dynamically interpreted and thus modified the lexical scope environment. The *Engine* will simply perform its lexical scope look-ups as it always does.

Consider the following code:

```js
function foo(str, a) {
	eval( str ); // cheating!
	console.log( a, b );
}

var b = 2;

foo( "var b = 3;", 1 ); // 1 3
```

The string `"var b = 3;"` is treated, at the point of the `eval(..)` call, as code that was there all along. Because that code happens to declare a new variable `b`, it modifies the existing lexical scope of `foo(..)`. In fact, as mentioned above, this code actually creates variable `b` inside of `foo(..)` that shadows the `b` that was declared in the outer (global) scope.

When the `console.log(..)` call occurs, it finds both `a` and `b` in the scope of `foo(..)`, and never finds the outer `b`. Thus, we print out "1 3" instead of "1 2" as would have normally been the case.

**Note:** In this example, for simplicity's sake, the string of "code" we pass in was a fixed literal. But it could easily have been programmatically created by adding characters together based on your program's logic. `eval(..)` is usually used to execute dynamically created code, as dynamically evaluating essentially static code from a string literal would provide no real benefit to just authoring the code directly.

By default, if a string of code that `eval(..)` executes contains one or more declarations (either variables or functions), this action modifies the existing lexical scope in which the `eval(..)` resides. Technically, `eval(..)` can be invoked "indirectly", through various tricks (beyond our discussion here), which causes it to instead execute in the context of the global scope, thus modifying it. But in either case, `eval(..)` can at runtime modify an author-time lexical scope.

**Note:** `eval(..)` when used in a strict-mode program operates in its own lexical scope, which means declarations made inside of the `eval()` do not actually modify the enclosing scope.

```js
function foo(str) {
   "use strict";
   eval( str );
   console.log( a ); // ReferenceError: a is not defined
}

foo( "var a = 2" );
```

There are other facilities in JavaScript which amount to a very similar effect to `eval(..)`. `setTimeout(..)` and `setInterval(..)` *can* take a string for their respective first argument, the contents of which are `eval`uated as the code of a dynamically-generated function. This is old, legacy behavior and long-since deprecated. Don't do it!

The `new Function(..)` function constructor similarly takes a string of code in its **last** argument to turn into a dynamically-generated function (the first argument(s), if any, are the named parameters for the new function). This function-constructor syntax is slightly safer than `eval(..)`, but it should still be avoided in your code.

The use-cases for dynamically generating code inside your program are incredibly rare, as the performance degradations are almost never worth the capability.

### `with`

The other frowned-upon (and now deprecated!) feature in JavaScript which cheats lexical scope is the `with` keyword. There are multiple valid ways that `with` can be explained, but I will choose here to explain it from the perspective of how it interacts with and affects lexical scope.

`with` is typically explained as a short-hand for making multiple property references against an object *without* repeating the object reference itself each time.

For example:

```js
var obj = {
	a: 1,
	b: 2,
	c: 3
};

// more "tedious" to repeat "obj"
obj.a = 2;
obj.b = 3;
obj.c = 4;

// "easier" short-hand
with (obj) {
	a = 3;
	b = 4;
	c = 5;
}
```

However, there's much more going on here than just a convenient short-hand for object property access. Consider:

```js
function foo(obj) {
	with (obj) {
		a = 2;
	}
}

var o1 = {
	a: 3
};

var o2 = {
	b: 3
};

foo( o1 );
console.log( o1.a ); // 2

foo( o2 );
console.log( o2.a ); // undefined
console.log( a ); // 2 -- Oops, leaked global!
```

In this code example, two objects `o1` and `o2` are created. One has an `a` property, and the other does not. The `foo(..)` function takes an object reference `obj` as an argument, and calls `with (obj) { .. }` on the reference. Inside the `with` block, we make what appears to be a normal lexical reference to a variable `a`, an LHS reference in fact (see Chapter 1), to assign to it the value of `2`.

When we pass in `o1`, the `a = 2` assignment finds the property `o1.a` and assigns it the value `2`, as reflected in the subsequent `console.log(o1.a)` statement. However, when we pass in `o2`, since it does not have an `a` property, no such property is created, and `o2.a` remains `undefined`.

But then we note a peculiar side-effect, the fact that a global variable `a` was created by the `a = 2` assignment. How can this be?

The `with` statement takes an object, one which has zero or more properties, and **treats that object as if *it* is a wholly separate lexical scope**, and thus the object's properties are treated as lexically defined identifiers in that "scope".

**Note:** Even though a `with` block treats an object like a lexical scope, a normal `var` declaration inside that `with` block will not be scoped to that `with` block, but instead the containing function scope.

While the `eval(..)` function can modify existing lexical scope if it takes a string of code with one or more declarations in it, the `with` statement actually creates a **whole new lexical scope** out of thin air, from the object you pass to it.

Understood in this way, the "scope" declared by the `with` statement when we passed in `o1` was `o1`, and that "scope" had an "identifier" in it which corresponds to the `o1.a` property. But when we used `o2` as the "scope", it had no such `a` "identifier" in it, and so the normal rules of LHS identifier look-up (see Chapter 1) occurred.

Neither the "scope" of `o2`, nor the scope of `foo(..)`, nor the global scope even, has an `a` identifier to be found, so when `a = 2` is executed, it results in the automatic-global being created (since we're in non-strict mode).

It is a strange sort of mind-bending thought to see `with` turning, at runtime, an object and its properties into a "scope" *with* "identifiers". But that is the clearest explanation I can give for the results we see.

**Note:** In addition to being a bad idea to use, both `eval(..)` and `with` are affected (restricted) by Strict Mode. `with` is outright disallowed, whereas various forms of indirect or unsafe `eval(..)` are disallowed while retaining the core functionality.

### Performance

Both `eval(..)` and `with` cheat the otherwise author-time defined lexical scope by modifying or creating new lexical scope at runtime.

So, what's the big deal, you ask? If they offer more sophisticated functionality and coding flexibility, aren't these *good* features? **No.**

The JavaScript *Engine* has a number of performance optimizations that it performs during the compilation phase. Some of these boil down to being able to essentially statically analyze the code as it lexes, and pre-determine where all the variable and function declarations are, so that it takes less effort to resolve identifiers during execution.

But if the *Engine* finds an `eval(..)` or `with` in the code, it essentially has to *assume* that all its awareness of identifier location may be invalid, because it cannot know at lexing time exactly what code you may pass to `eval(..)` to modify the lexical scope, or the contents of the object you may pass to `with` to create a new lexical scope to be consulted.

In other words, in the pessimistic sense, most of those optimizations it *would* make are pointless if `eval(..)` or `with` are present, so it simply doesn't perform the optimizations *at all*.

Your code will almost certainly tend to run slower simply by the fact that you include an `eval(..)` or `with` anywhere in the code. No matter how smart the *Engine* may be about trying to limit the side-effects of these pessimistic assumptions, **there's no getting around the fact that without the optimizations, code runs slower.**

## Review (TL;DR)

Lexical scope means that scope is defined by author-time decisions of where functions are declared. The lexing phase of compilation is essentially able to know where and how all identifiers are declared, and thus predict how they will be looked-up during execution.

Two mechanisms in JavaScript can "cheat" lexical scope: `eval(..)` and `with`. The former can modify existing lexical scope (at runtime) by evaluating a string of "code" which has one or more declarations in it. The latter essentially creates a whole new lexical scope (again, at runtime) by treating an object reference *as* a "scope" and that object's properties as scoped identifiers.

The downside to these mechanisms is that it defeats the *Engine*'s ability to perform compile-time optimizations regarding scope look-up, because the *Engine* has to assume pessimistically that such optimizations will be invalid. Code *will* run slower as a result of using either feature. **Don't use them.**

