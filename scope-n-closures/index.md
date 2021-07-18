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
* `two` such mechanisms. Both of them are [...] bad practices [...] most important point: `cheating lexical scope leads to poorer performance.`.

### eval
* `eval(..)` function [...] takes a string [...] and treats the contents of the string as if it had actually been authored code at that point in the program.

```js
function foo(str, a) {
	eval( str ); // cheating!
	console.log( a, b );
}

var b = 2;

foo( "var b = 3;", 1 ); // 1 3 (me: read more on why a static string of "code" is passed on the second note below)
```

* `"var b = 3;"` is treated, `at the point` of the `eval(..)` call [...] Because that code [...] declare a new variable `b`, it modifies the `existing lexical scope` of `foo(..)` [...] (me: and) `shadows` the `b` that was declared in the outer scope.

* In this example [...] the string of "code" we pass in was a `fixed literal`. But [...] `eval(..)` is usually used to execute `dynamically created code` [...] dynamically evaluating [...] static code from a string literal [...] provide no real benefit to just authoring the code directly.

* if a string of code that `eval(..)` executes contains [...] declarations [...] this action modifies the `existing lexical scope` in which the `eval(..)` resides.

* `eval(..)` when used in a `strict-mode` program `operates` `in its own lexical scope`, which means `declarations` made inside of the `eval()` do `not` actually modify the `enclosing scope`.

```js
function foo(str) {
   "use strict";
   eval( str );
   console.log( a ); // ReferenceError: a is not defined
}

foo( "var a = 2" );
```

* similar [...] to `eval(..)`. `setTimeout(..)` and `setInterval(..)` `can` take a string for their [...] `first argument`, the contents of which are `eval`uated as the code of a `dynamically-generated function`.

* `new Function(..)` function constructor similarly takes a string of code in its `last` argument to turn into a `dynamically-generated function` (the `first argument(s)`, `if any`, are the `named parameters` `for the new function`). [...] syntax is `slightly safer` than `eval(..)`.

* The use-cases for `dynamically generating code` [...] are incredibly rare, as the `performance degradations` are almost `never worth` the capability.

### `with`
* The other [...] feature [...] cheats `lexical scope` is the `with` keyword.

* `with` is [...] a `short-hand` for `making` `multiple property references` against an object `without repeating` the `object reference` [...] each time.

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

* However, there's much more going on:

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

* The `with` statement takes an object [...] and treats `that object` as if `it` is a `wholly separate lexical scope`, and thus the `object's properties` are [...] defined identifiers `in that` "`scope`".

* Even though a `with` block treats an object like a `lexical scope`, a normal `var` declaration `inside` that `with` block will `not be scoped` to that `with` block, but instead the `containing function scope`.

* the "`scope`" declared by the `with` [...] when we passed in `o1` [...] had an "`identifier`" in it which corresponds to the `o1.a` [...] when we used `o2` as the "`scope`", it had `no` such `a` "identifier"
* `Neither` the "scope" of `o2`, nor the scope of `foo(..)`, nor the `global scope` [...] has an `a` identifier [...] so when `a = 2` is executed [...] the `automatic-global` being created ([...] we're in `non-strict mode`).

* both `eval(..)` and `with` are affected [...] by `Strict Mode`. `with` is [...] disallowed, `whereas` various forms of [...] unsafe `eval(..)` are disallowed `while` retaining the `core functionality`.

### Performance
* The JavaScript `Engine` has a number of `performance optimizations` [...] during the `compilation phase`. Some of these boil down to being able to [...] `statically` analyze the code [...] and `pre-determine` where all the variable and function declarations are, so that it takes `less effort` to resolve identifiers during execution.

* because it cannot know at `lexing time` exactly `what code` you may pass to `eval(..)` to `modify the lexical scope`, or the contents of the object you may pass to `with` to create a new lexical scope [...] most of those optimizations [...] are pointless if `eval(..)` or `with` are present [...] it simply doesn't perform the optimizations `at all`.

## Review (TL;DR)

# Chapter 3: Function vs. Block Scope
## Scope From Functions
## Hiding In Plain Scope
* "`Principle of Least Privilege`" [...] sometimes called "`Least Authority`" or "`Least Exposure`" [...] states that in the design of software, such as the `API` for a `module/object`, you should expose only what is [...] necessary, and "`hide`" everything else.

```js
function doSomething(a) {
	// (me: "doSomethingElse" & "b" are "private" details of how `doSomething(..)` does its job)
	function doSomethingElse(a) {
		return a - 1;
	}

	var b;

	b = a + doSomethingElse( a * 2 );

	console.log( b * 3 );
}

doSomething( 2 ); // 15
```

* `b` and `doSomethingElse(..)` are not accessible to any outside influence, instead controlled only by `doSomething(..)`.

### Collision Avoidance
Another benefit of "hiding" variables and functions inside a scope is to avoid unintended collision between two different identifiers with the same name but different intended usages. Collision results often in unexpected overwriting of values.

For example:

```js
function foo() {
	function bar(a) {
		i = 3; // changing the `i` in the enclosing scope's for-loop
		console.log( a + i );
	}

	for (var i=0; i<10; i++) {
		bar( i * 2 ); // oops, infinite loop ahead!
	}
}

foo();
```

* The `i = 3` assignment inside of `bar(..)` overwrites [...] the `i` that was declared in `foo(..)` at the for-loop [...] result in an infinite loop, because `i` is set to a fixed value of `3` and that will forever remain `< 10`.

* `var i = 3;` would fix the problem (and would create [...] "`shadowed variable`" declaration for `i`). An [...] alternate, option is to pick another identifier name entirely, such as `var j = 3;`.

#### Global "Namespaces"
* `Multiple libraries` loaded into your program can [...] collide with each other if they don't [...] hide their internal [...] functions and variables.

* libraries [...] will create [...] an object, with a [...] unique name, in the global scope [...] where [...] exposures of functionality are made as properties of that object (`namespace`).

#### Module Management
* using [...] `dependency managers` [...] no libraries ever add any identifiers to the global scope, but [...] required to have their identifier(s) be `explicitly imported` `into another specific scope`.

## Functions As Scopes
* we can take any snippet of code and wrap a function around it [...] "hides" any enclosed variable or function declarations.

```js
var a = 2;

function foo() { // <-- insert this

	var a = 3;
	console.log( a ); // 3

} // <-- and this
foo(); // <-- and this

console.log( a ); // 2
```

* While this technique "works" [...] the identifier name `foo` itself "pollutes" the enclosing scope [...] We also have to explicitly call [...] `foo()`.

* solution:

```js
var a = 2;

(function foo(){ // <-- insert this

	var a = 3;
	console.log( a ); // 3

})(); // <-- and this

console.log( a ); // 2
```

* If "`function`" is the very `first thing` in the statement, then it's a `function declaration`. Otherwise, it's a `function expression` [...] The key difference [...] between a `function declaration` and a `function expression` relates to where its name is bound as an identifier.

* In other words, `(function foo(){ .. })` [...] means the identifier `foo` is found `only` in the scope where the `..` indicates, not in the outer scope.

### Anonymous vs. Named
```js
setTimeout( function(){
	console.log("I waited 1 second!");
}, 1000 );
```

* This is called an "`anonymous function expression`", because `function()...` has `no name identifier` on it. `Function expressions` can be anonymous, but `function declarations` `cannot` omit the name.

* `Anonymous function` [...] have several `draw-backs`:
  * no useful name to display in `stack traces` [...] make debugging more difficult.

  * if the function needs to refer to itself, for `recursion`, etc., the `deprecated` `arguments.callee` reference is [...] required.

  * A descriptive name helps `self-document` the code.

* `Inline function expressions`:

```js
setTimeout( function timeoutHandler(){ // <-- Look, I have a name! (me: an "Inline function expressions")
	console.log( "I waited 1 second!" );
}, 1000 );

// (me)
timeoutHandler() // errors: ReferenceError: timeoutHandler is not defined
```

### Invoking Function Expressions Immediately
* the most common form of IIFE is to use an `anonymous function expression`. While [...] less common, naming an IIFE has all the aforementioned benefits over anonymous function expressions.

* There's a slight variation on the traditional IIFE form [...] `(function(){ .. }())` [...] two forms are identical in functionality.

* Another variation on IIFE's which [...] `pass in argument(s)`.

```js
var a = 2;

(function IIFE( global ){

	var a = 3;
	console.log( a ); // 3
	console.log( global.a ); // 2

})( window );

console.log( a ); // 2
```

* We pass in the `window` object reference, but we name the parameter `global`, so that we have a clear stylistic delineation for global vs. non-global references [...] This is mostly `just stylistic choice`.

* another variation of the IIFE [...] used in the `UMD` (`Universal Module Definition`) project.

```js
var a = 2;

(function IIFE( def ){
	def( window );
})(function def( global ){

	var a = 3;
	console.log( a ); // 3
	console.log( global.a ); // 2

});
```

## Blocks As Scopes
```js
for (var i=0; i<10; i++) {
	console.log( i );
}
```

* We [...] `intent` is to use `i` only within the context of that `for-loop`, and [...] ignore the fact that the variable [...] scopes itself to the `enclosing scope`.

* `Block-scoping` (if it were possible) for the `i` variable would make `i` available `only` for the `for-loop` [...] But [...] JavaScript has no facility for block scope [...] That is, until you dig a little further.

### `with`
* `with` [...] is an example of (`a form of`) block scope, in that the scope that is created from the object only exists [...] (me:)(in) that `with` statement, and `not` in the `enclosing scope`.

### `try/catch`
* It's a `very little known` [...] that [...] `ES3` specified the variable declaration in the `catch` clause of a `try/catch` to be `block-scoped`.

For instance:

```js
try {
	undefined(); // illegal operation to force an exception!
}
catch (err) {
	console.log( err ); // works!
}

console.log( err ); // ReferenceError: `err` not found
```

* many `linters` seem to still complain if you have two or more `catch` clauses in the same scope which each declare their `error variable` with the `same identifier name`.

### `let`
* `ES6` [...] introduces a new keyword `let` [...] as another way to declare variables.

* `let` keyword attaches the `variable declaration` to the `scope` of [...] `block` (commonly a `{ .. }` pair).

* Creating `explicit blocks` `for block-scoping` [...] making it more obvious where variables are attached and not.

```js
var foo = true;

if (foo) {
	{ // <-- explicit block
		let bar = foo * 2;
		bar = something( bar );
		console.log( bar );
	}
}

console.log( bar ); // ReferenceError
```

* declarations made with `let` will `not` `hoist`.

#### Garbage Collection
```js
function process(data) {
	// do something interesting
}

var someReallyBigData = { .. };

process( someReallyBigData );

var btn = document.getElementById( "my_button" );

btn.addEventListener( "click", function click(evt){
	console.log("button clicked");
}, /*capturingPhase=*/false );
```

* `click` [...] `callback` `doesn't need` the `someReallyBigData` [...] after `process(..)` runs, the `big memory-heavy data` [...] could be garbage collected. `However`, it's [...] likely (though `implementation dependent`) that the `JS engine` will still have to `keep the structure around`, since the `click` function has a `closure` `over the entire scope`.

* `Block-scoping` can [...] making it clearer to the engine that it does not need to keep `someReallyBigData`.

```js
function process(data) {
	// do something interesting
}

// anything declared inside this block can go away after!
{
	let someReallyBigData = { .. };

	process( someReallyBigData );
}

var btn = document.getElementById( "my_button" );

btn.addEventListener( "click", function click(evt){
	console.log("button clicked");
}, /*capturingPhase=*/false );
```

#### `let` Loops
* `let` [...] `re-binds it` to each `iteration` of the loop.

```js
{
	let j;
	for (j=0; j<10; j++) {
		let i = j; // re-bound for each iteration!
		console.log( i );
	}
}
```

### `const`
* `ES6` introduces `const` [...] also creates a `block-scoped variable`, but whose value is `fixed` (`constant`).

## Review (TL;DR)

# Chapter 4: Hoisting
## Chicken Or The Egg?

```js
a = 2;

var a;

console.log( a );
```

* Many [...] would expect `undefined`[...] `However`, the `output` will be `2`.

```js
console.log( a );

var a = 2;
```

* `undefined` is the `output`.

## The Compiler Strikes Again
* `all` declarations, both `variables` and `functions`, are `processed first`, `before` any part of your code is executed.

* `var a = 2;` [...] JavaScript [...] thinks of it as two statements: `var a;` and `a = 2;`
  * the `declaration`, is processed `during` the `compilation phase`.
  * the `assignment`, is left [...] for the `execution phase`.

* `first snippet` then should be thought of as being handled like this:

```js
var a;
```
```js
a = 2;

console.log( a );
```

* `second` snippet is actually processed as:

```js
var a;
```
```js
console.log( a );

a = 2;
```

* `Only` the `declarations` [...] are `hoisted`, while any `assignments` or other `executable` logic are left `in place`.

```js
foo();

function foo() {
	console.log( a ); // undefined

	var a = 2;
}
```

* The function `foo`'s declaration [...] `includes` the `implied value` of it as an actual `function` [...] is hoisted (me: read the `Compiler Speak` title inside the `Chapter 1` for more detail).

* `Function declarations` are hoisted, as we just saw. `But` `function expressions` are not.

```js
foo(); // not ReferenceError, but TypeError!

var foo = function bar() {
	// ...
};
```

* The variable identifier `foo` is hoisted [...] so `foo()` doesn't fail as a `ReferenceError`. But `foo` has `no value yet` [...] `foo()` is attempting to invoke the `undefined` value, which is a `TypeError` illegal operation.

* Also `recall` that even though it's a `named function expression`, the `name identifier` is `not available` in the `enclosing scope`:

```js
foo(); // TypeError
bar(); // ReferenceError

var foo = function bar() {
	// ...
};
```

* This snippet is `more accurately interpreted` (with hoisting) as:

```js
var foo;

foo(); // TypeError
bar(); // ReferenceError

foo = function() {
	var bar = ...self...
	// ...
}
```

## Functions First
* Both `function declarations` and `variable declarations` are hoisted. But a subtle detail (that `can` show up in code with multiple "`duplicate`" `declarations`) is that `functions` are `hoisted first`, and then `variables`.

Consider:

```js
foo(); // 1

var foo;

function foo() {
	console.log( 1 );
}

foo = function() {
	console.log( 2 );
};
```

* `1` is printed instead of `2`! This snippet is interpreted by the `Engine` as:

```js
function foo() {
	console.log( 1 );
}

foo(); // 1

foo = function() {
	console.log( 2 );
};
```

* `var foo` was the `duplicate` (and thus `ignored`) `declaration`.

* `While` `multiple/duplicate` `var` declarations are effectively `ignored`, `subsequent` `function declarations` [...] `override` previous ones.

```js
foo(); // 3

function foo() {
	console.log( 1 );
}

var foo = function() {
	console.log( 2 );
};

function foo() {
	console.log( 3 );
}
```

* `Function declarations` that appear inside of normal `blocks` typically `hoist` to the `enclosing scope`.

```js
foo(); // "b"

var a = true;
if (a) {
   function foo() { console.log( "a" ); }
}
else {
   function foo() { console.log( "b" ); }
}
```

* this behavior is `not reliable` and [...] change in future versions of JavaScript.

## Review (TL;DR)

# Chapter 5: Scope Closure
## Enlightenment
## Nitty Gritty
* `Closure` is when a function is able to remember and access its `lexical scope` `even when` that function is `executing` `outside its lexical scope`.

```js
function foo() {
	var a = 2;

	function bar() {
		console.log( a ); // 2
	}

	bar();
}

foo();
```

* Is this "`closure`"?

* `technically`... `perhaps`. But by our [...] definition `above`... `not exactly`. I think the most accurate way to explain `bar()` referencing `a` is via `lexical scope look-up` rules, and those rules are `only` [...] `part` of what `closure` is.

* (me: `has a closure over the scope of ...` means to have a `closure` which `closes over` (covers) a `scope` of ...)

* function `bar()` `has a closure over` the `scope` of `foo()` (and [...] over the [...] the `global scope` in our case). Put slightly differently, it's said that `bar()` (me: `covers`)(`closes over`) the scope of `foo()`.

```js
function foo() {
	var a = 2;

	function bar() {
		console.log( a );
	}

	return bar;
}

var baz = foo();

baz(); // 2 -- Whoa, closure was just observed, man.
```
* `bar()` is executed, for sure. But [...] it's executed `outside` of its `declared lexical scope`.

* After `foo()` executed [...] we would expect that the entirety of the `inner scope` of `foo()` would go away [...] a `Garbage Collector` [...] frees up memory once it's no longer in use [...] But [...] `That inner scope` is in fact `still` "`in use`", and `thus` `does not go away`. Who's using it [...] function `bar()`.

* `bar()` has a `lexical scope closure` `over` that `inner scope` of `foo()`, which keeps that scope alive for `bar()` to reference at any later time.

* **`bar()` `still has` a `reference to that scope`, and `that reference` is called `closure` [...] lets the function continue to access the `lexical scope` it was defined in `at author-time`**.

## Now I Can See
* it is often said that `IIFE` [...] is an example of observed closure, I would somewhat disagree, by our definition above.

```js
var a = 2;

(function IIFE(){
	console.log( a );
})();
```

* This code "works", but it's not strictly an observation of `closure` [...] Because the function [...] `IIFE` [...] is not executed outside its `lexical scope`. It's still invoked [...] in the same scope as it was declared [...] `a` is found via normal `lexical scope look-up`, not really via `closure`.

## Loops + Closure
```js
for (var i=1; i<=5; i++) {
	setTimeout( function timer(){
		console.log( i );
	}, i*1000 );
}
```

* if you run this code, you get "`6`" printed out `5 times`.

* The timeout function callbacks are all running well after the completion of the loop [...] and thus print `6` each time.

* we are trying to `imply` that each iteration of the loop "captures" its own copy of `i`, at the time of the iteration. But [...] all 5 of those functions, though they are defined separately in each loop iteration, all **are closed over the same shared global scope**, which has, in fact, only one `i` in it.

* we need a `new closured scope` for each iteration of the loop.

```js
for (var i=1; i<=5; i++) {
	(function(j){
		setTimeout( function timer(){
			console.log( j );
		}, j*1000 );
	})( i );
}
```

* The use of an IIFE inside each iteration [...] gave our `timeout function callbacks` the opportunity to `close over` a `new scope` for `each iteration` [...] which had a `variable` with the [...] `per-iteration` `value` in it for us to access.

### Block Scoping Revisited
* There's a `special behavior` defined for `let` declarations used `in the head of` a `for-loop` [...] says that the variable will be `declared` not just once for the loop, `but each iteration`.

```js
for (let i=1; i<=5; i++) {
	setTimeout( function timer(){
		console.log( i );
	}, i*1000 );
}
```

## Modules
```js
function CoolModule() {
	var something = "cool";
	var another = [1, 2, 3];

	function doSomething() {
		console.log( something );
	}

	function doAnother() {
		console.log( another.join( " ! " ) );
	}

	return {
		doSomething: doSomething,
		doAnother: doAnother
	};
}

var foo = CoolModule();

foo.doSomething(); // cool
foo.doAnother(); // 1 ! 2 ! 3
```

* This is the pattern [...] call `module`. The most common way of implementing the `module` pattern is often called `Revealing Module`, and it's the variation we present here.

* not required [...] return an [...] object [...] We could just return back an `inner function` [...] `jQuery` [...] a good example of this. The `jQuery` and `$` identifiers are the `public API` for the `jQuery "module"`, but they are [...] just a function.

* there are two "`requirements`" for the `module pattern` to be exercised:
  1. There must be an outer enclosing function.
  2. The `enclosing function` `must` return back at least one `inner function`.

* A slight `variation` on this pattern is when you `only` care to have `one instance`, a "`singleton`" of sorts:

```js
var foo = (function CoolModule() {
	var something = "cool";
	var another = [1, 2, 3];

	function doSomething() {
		console.log( something );
	}

	function doAnother() {
		console.log( another.join( " ! " ) );
	}

	return {
		doSomething: doSomething,
		doAnother: doAnother
	};
})();

foo.doSomething(); // cool
foo.doAnother(); // 1 ! 2 ! 3
```

### Modern Modules
* Various `module dependency loaders/managers` essentially `wrap up` this pattern of `module definition` into a `friendly API` [...] let me present a `very simple` proof of concept `for illustration purposes (only)`:

```js
var MyModules = (function Manager() {
	var modules = {};

	function define(name, deps, impl) {
		for (var i=0; i<deps.length; i++) {
			deps[i] = modules[deps[i]];
		}
		modules[name] = impl.apply( impl, deps );
	}

	function get(name) {
		return modules[name];
	}

	return {
		define: define,
		get: get
	};
})();
```

* here's how I might use it to define some modules:

```js
MyModules.define( "bar", [], function(){
	function hello(who) {
		return "Let me introduce: " + who;
	}

	return {
		hello: hello
	};
} );

MyModules.define( "foo", ["bar"], function(bar){
	var hungry = "hippo";

	function awesome() {
		console.log( bar.hello( hungry ).toUpperCase() );
	}

	return {
		awesome: awesome
	};
} );

var bar = MyModules.get( "bar" );
var foo = MyModules.get( "foo" );

console.log(
	bar.hello( "hippo" )
); // Let me introduce: hippo

foo.awesome(); // LET ME INTRODUCE: HIPPO
```

### Future Modules
* modules. When loaded via the module system, `ES6` treats a file as a separate module. Each module can both import other modules or specific API members, as well export their own public API members.

* ES6 modules `do not` have an "`inline`" format, they must be defined in `separate files` (`one per module`). The `browsers/engines` have a `default "module loader"` [...] `overridable` [...] which `synchronously` loads a module file when it's imported.

**bar.js**
```js
function hello(who) {
	return "Let me introduce: " + who;
}

export hello;
```

**foo.js**
```js
// import only `hello()` from the "bar" module
import hello from "bar";

var hungry = "hippo";

function awesome() {
	console.log(
		hello( hungry ).toUpperCase()
	);
}

export awesome;
```

```js
// import the entire "foo" and "bar" modules
module foo from "foo";
module bar from "bar";

console.log(
	bar.hello( "rhino" )
); // Let me introduce: rhino

foo.awesome(); // LET ME INTRODUCE: HIPPO
```

* `import` imports `one or more members` from a `module's API` into the current scope, each to a `bound variable` (`hello` in our case).
* `module` imports an `entire module API` to a `bound variable` (`foo`, `bar` in our case).
* `export` exports an identifier.

## Review (TL;DR)

# Appendix A: Dynamic Scope

