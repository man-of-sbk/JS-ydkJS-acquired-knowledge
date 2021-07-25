# Foreword
# Preface
# Chapter 1: this Or That?
## Why `this`?
```js
function identify() {
	return this.name.toUpperCase();
}

function speak() {
	var greeting = "Hello, I'm " + identify.call( this );
	console.log( greeting );
}

var me = {
	name: "Kyle"
};

var you = {
	name: "Reader"
};

identify.call( me ); // KYLE
identify.call( you ); // READER

speak.call( me ); // Hello, I'm KYLE
speak.call( you ); // Hello, I'm READER
```

* This code snippet allows the `identify()` and `speak()` functions to be re-used against `multiple` `context` (`me` and `you`) `objects`, `rather than` needing a `separate version of the function` `for each object`.

* Instead of [...] `this`, you could have `explicitly passed` in a `context object` to both `identify()` and `speak()`.

```js
function identify(context) {
	return context.name.toUpperCase();
}

function speak(context) {
	var greeting = "Hello, I'm " + identify( context );
	console.log( greeting );
}

identify( you ); // READER
speak( me ); // Hello, I'm KYLE
```

* However, the `this` mechanism provides a more `elegant` way of `implicitly "passing along"` an object reference, leading to `cleaner API design`.

## Confusions
### Itself
* The first common temptation is to assume `this` refers to the function itself.

* to illustrate how `this` doesn't let a function get a reference to itself [...] Consider the following code.

```js
function foo(num) {
	console.log( "foo: " + num );

	// keep track of how many times `foo` is called
	this.count++;
}

foo.count = 0;

var i;

for (i=0; i<10; i++) {
	if (i > 5) {
		foo( i );
	}
}
// foo: 6
// foo: 7
// foo: 8
// foo: 9

// how many times was `foo` called?
console.log( foo.count ); // 0 -- WTF?
```

* When the code executes `foo.count = 0`, indeed it's adding a property `count` to the function object `foo`. But [...] `this` is not [...]pointing [...] to that function object.

* In fact [...] a `global variable` `count` (me: is accidentally created) (see Chapter 2 for *how* that happened!), and it currently has the value `NaN`.

* To reference a function object `from inside itself` [...] You [...] need a reference to the function object via a `lexical identifier` (variable) that points at it.

```js
function foo() {
	foo.count = 4; // `foo` refers to itself
}

setTimeout( function(){
	// anonymous function (no name), cannot
	// refer to itself
}, 10 );
```

* `now deprecated` [...] `arguments.callee` reference inside a function `also` points to the function object of the `currently executing function`. This [...] is [...] the only way to access an `anonymous function's object` `from inside itself`.

* another solution to our running example.

```js
function foo(num) {
	console.log( "foo: " + num );

	// keep track of how many times `foo` is called
	foo.count++;
}

foo.count = 0;

var i;

for (i=0; i<10; i++) {
	if (i > 5) {
		foo( i );
	}
}
// foo: 6
// foo: 7
// foo: 8
// foo: 9

// how many times was `foo` called?
console.log( foo.count ); // 4
```

### Its Scope
* next most common misconception about [...] `this` is that it [...] refers to the `function's scope`.

* internally, scope is kind of like an object with properties for each of the available `identifiers`. But the `scope "object"` is `not accessible` to `JavaScript code`.

## What's `this`?
* When a function is invoked, an `activation record`, otherwise known as an `execution context`, is created. This record contains information about [...] the `call-stack` [...] `how` the function was invoked, what parameters were passed, etc. One of the `properties` `of this record` is the `this` reference `which` will be used for the `duration of that function's execution`.

## Review (TL;DR)

# Chapter 2: `this` All Makes Sense Now!
## Call-site
* `call-site`: the `location` `in code` where a function is `called` (`not` where it's `declared`). We must `inspect` the `call-site` to answer the question: what's `this` a reference to?

* locate where a function is called from [...] not always that easy, as certain coding patterns can obscure the `true` `call-site`.

* the `call-stack` (the `stack of functions` that have been called to `get us to` the `current moment in execution`). The `call-site` [...] is in the `invocation` `before` `the currently executing function`.

```js
function baz() {
    // call-stack is: `baz`
    // so, our call-site is in the global scope

    console.log( "baz" );
    bar(); // <-- call-site for `bar`
}

function bar() {
    // call-stack is: `baz` -> `bar`
    // so, our call-site is in `baz`

    console.log( "bar" );
    foo(); // <-- call-site for `foo`
}

function foo() {
    // call-stack is: `baz` -> `bar` -> `foo`
    // so, our call-site is in `bar`

    console.log( "foo" );
}

baz(); // <-- call-site for `baz`
```

* Take care when [...] find the actual `call-site` (`from` the `call-stack`), because it's the only thing that matters for `this` binding.

* Another way of seeing the `call-stack` is using a `debugger` tool in your browser [...] you could have set a `breakpoint` in the `tools` for the first line of the `foo()` function, or [...] inserted the `debugger;` statement on that first line. When you run the page, the `debugger` will `pause at this location`, and will show you a `call stack`.

* So, if you're trying to diagnose `this` binding, use the `developer tools` to get the `call-stack`, then find the `second item from the top`, and that will show you the real `call-site`.

## Nothing But Rules
* You must inspect the `call-site` and determine which of `4 rules` applies. We will first explain each of these `4 rules` independently, and then we will illustrate their `order of precedence`, `if` `multiple rules` could apply to the `call-site`.

### Default Binding
The `first rule` [...] Think of this `this` rule as the default `catch-all rule` when `none of the other rules apply`.

```js
function foo() {
	console.log( this.a );
}

var a = 2;

foo(); // 2
```

* variables declared in the `global scope`, as `var a = 2` is, are synonymous with `global-object properties` of the same name.

* when `foo()` is called, `this.a` `resolves` to our global variable `a` [...] Because in this case, the `default binding` for `this` [...] points `this` at the `global object`.

* How do we know that the `default binding` rule applies here? We examine the `call-site` to see how `foo()` is called. In our snippet, `foo()` is called with a plain, `un-decorated function reference`. `None of the other rules` [...] will apply here, so the `default binding` applies instead.

* If `strict mode` is in effect, the `global object` is `not eligible` for the `default binding`, so the `this` is instead set to `undefined`.

```js
function foo() {
	"use strict";

	console.log( this.a );
}

var a = 2;

foo(); // TypeError: `this` is `undefined`
```

* (me: beware of mixing `use strict` and `non-strict mode`).

```js
function foo() {
	console.log( this.a );
}

var a = 2;

(function(){
	"use strict";

	foo(); // 2
})();
```

* mixing `strict mode` and `non-strict mode` [...] is [...] `frowned upon`. Your entire program should [...] either be **Strict** or `non-Strict`. `However`, sometimes you include a `third-party library` that has `different` `Strict`'ness than your own code, so care must be taken over these subtle compatibility details.

### Implicit Binding
* `Another rule` [...] is: does the `call-site` have a `context object`, [...] an `owning` or `containing` `object`, though `these` alternate terms could be slightly misleading.

```js
function foo() {
	console.log( this.a );
}

var obj = {
	a: 2,
	foo: foo
};

obj.foo(); // 2
```

* notice [...] `foo()` is declared and then later added as a `reference property` onto `obj`. `Regardless of` whether `foo()` is `initially declared` `on` `obj`, or is `added as a reference later` [...] in `neither` case is the `function` `really` "`owned`" or "`contained`" by the `obj` object.

* `However`, the `call-site` `uses` the `obj` context to `reference` the function, so you `could` say that the `obj` object "`owns`" or "`contains`" the `function reference` `at the time` `the function is called`.

* `at the point` that `foo()` `is called`, it's `preceded` by an `object reference` to `obj`. `When` there is a `context object` for a function reference, the `implicit binding` rule says that it's `that` `object` which should be used for the `function call's` `this` binding.

* `Only` the [...] `last` level of an `object property reference chain` matters to the `call-site`.

```js
function foo() {
	console.log( this.a );
}

var obj2 = {
	a: 42,
	foo: foo
};

var obj1 = {
	a: 2,
	obj2: obj2
};

obj1.obj2.foo(); // 42
```

#### Implicitly Lost
One of the most common frustrations [...] (me: with) `this` binding [...] is when an `implicitly bound function` `loses that binding`, which usually means it falls back to the `default binding`, of `either` the `global object` or `undefined`, depending on `strict mode`.

```js
function foo() {
	console.log( this.a );
}

var obj = {
	a: 2,
	foo: foo
};

var bar = obj.foo; // function reference/alias!

var a = "oops, global"; // `a` also property on global object

bar(); // "oops, global"
```

* `bar` appears to be a reference to `obj.foo`, in fact, it's really just another reference to `foo` itself. Moreover, the call-site is what matters, and the call-site is `bar()`, which is a plain, un-decorated call and thus the *default binding* applies.

* The [...] more unexpected way this occurs is when we consider passing a callback function:

```js
function foo() {
	console.log( this.a );
}

function doFoo(fn) {
	// `fn` is just another reference to `foo`

	fn(); // <-- call-site!
}

var obj = {
	a: 2,
	foo: foo
};

var a = "oops, global"; // `a` also property on global object

doFoo( obj.foo ); // "oops, global"
```

* passing a function (me: as an argument), [...] an `implicit reference assignment`, so the end result is the same as the previous snippet.

* `Event handlers` [...] forcing your callback to have a `this` which points to [...] the `DOM element` that triggered the event.
### Explicit Binding
* want to `force` a function call to use a particular object for the `this` binding, `without` putting a `property function reference on the object`?

* functions have `call(..)` and `apply(..)` methods [...] both take, as their `first parameter`, an `object` to use for the `this`, and then `invoke` the function with that `this` specified.

```js
function foo() {
	console.log( this.a );
}

var obj = {
	a: 2
};

foo.call( obj ); // 2
```

* If you pass a [...] `primitive value` (of type `string`, `boolean`, or `number`) as the `this` binding, the primitive value is `wrapped` in its `object-form` (`new String(..)`, `new Boolean(..)`, or `new Number(..)` [...]). This is often referred to as "`boxing`".

#### Hard Binding
```js
function foo() {
	console.log( this.a );
}

var obj = {
	a: 2
};

var bar = function() {
	foo.call( obj );
};

bar(); // 2
setTimeout( bar, 100 ); // 2

// `bar` hard binds `foo`'s `this` to `obj`
// so that it cannot be overriden
bar.call( window ); // 2
```

* (me: when `apply` is helpful).

```js
function foo(something) {
	console.log( this.a, something );
	return this.a + something;
}

var obj = {
	a: 2
};

var bar = function() {
	return foo.apply( obj, arguments );
};

var b = bar( 3 ); // 2 3
console.log( b ); // 5
```

* `ES5`: `Function.prototype.bind`.

```js
function foo(something) {
	console.log( this.a, something );
	return this.a + something;
}

var obj = {
	a: 2
};

var bar = foo.bind( obj );

var b = bar( 3 ); // 2 3
console.log( b ); // 5
```

* As of `ES6` [...] function produced by `bind(..)` has a `.name` property [...] For example: `bar = foo.bind(..)` should have a `bar.name` value of `"bound foo"`, which is the `function call name` [...] `show up in a stack trace`.

#### API Call "Contexts"
* many new `built-in functions` in the `JavaScript` [...] provide an `optional parameter`, usually called "`context`" [...] for you not having to use `bind(..)` to ensure your `callback function` uses a particular `this`.

```js
function foo(el) {
	console.log( el, this.id );
}

var obj = {
	id: "awesome"
};

// use `obj` as `this` for `foo(..)` calls
[1, 2, 3].forEach( foo, obj ); // 1 awesome  2 awesome  3 awesome
```

### `new` Binding
* `The fourth` and final rule for `this` binding.

* JavaScript has a `new` operator [...] `However`, there [...] is `no connection` to `class-oriented`.

* `any [...] function` [...] can be called with `new` in front of it [...] makes that `function call` a `constructor call` [...] there's really no such thing as "`constructor functions`", but rather `construction calls` `of functions`.

* When a function is invoked with `new` in front of it, otherwise known as a `constructor call`, the following things are done automatically:
  1. a brand new `object` is created (aka, constructed).
  2. `newly constructed object` is `[[Prototype]]`-`linked`.
  3. `newly constructed object` is set as the `this` binding for `that function call`.
  4. `unless` the function `returns` its own alternate `object`, the [...] `function call` will `automatically return` the `newly constructed object`.

* **So `new` is the `final way` that a `function call`'s `this` can be bound.** We'll call this `new binding`.

## Everything In Order
* the `default binding` is the `lowest priority` rule of the `4`. So we'll just set that one aside.

```js
function foo() {
	console.log( this.a );
}

var obj1 = {
	a: 2,
	foo: foo
};

var obj2 = {
	a: 3,
	foo: foo
};

obj1.foo(); // 2
obj2.foo(); // 3

obj1.foo.call( obj2 ); // 3
obj2.foo.call( obj1 ); // 2
```

* So, `explicit binding` takes precedence over `implicit binding`.

```js
function foo(something) {
	this.a = something;
}

var obj1 = {
	foo: foo
};

var obj2 = {};

obj1.foo( 2 );
console.log( obj1.a ); // 2

obj1.foo.call( obj2, 3 );
console.log( obj2.a ); // 3

var bar = new obj1.foo( 4 );
console.log( obj1.a ); // 2
console.log( bar.a ); // 4
```

* OK, `new binding` is `more precedent than` `implicit binding`.

* `Note:` `new` and `call`/`apply` `cannot` be used together, so `new foo.call(obj1)` is `not allowed` (me: `throw a Type error`).

```js
function foo(something) {
	this.a = something;
}

var obj1 = {};

var bar = foo.bind( obj1 );
bar( 2 );
console.log( obj1.a ); // 2

var baz = new bar( 3 );
console.log( obj1.a ); // 2
console.log( baz.a ); // 3
```

* (me: `new binding` is `more precedent than` `explicit binding`.)

* One of the capabilities of `bind(..)` is that `any arguments` `passed after` the first `this` binding argument are `defaulted` [...] arguments to the `underlying function`.

```js
function foo(p1,p2) {
	this.val = p1 + p2;
}

// using `null` here because we don't care about
// the `this` hard-binding in this scenario, and
// it will be overridden by the `new` call anyway!
var bar = foo.bind( null, "p1" );

var baz = new bar( "p2" );

baz.val; // p1p2
```

### Determining `this`
* Now, we can summarize the rules for determining `this` [...] in their `order of precedence`. Ask these questions in this order, and stop when the first rule applies.

  1. Is the function called with `new` (`new binding`)? If so, `this` is the newly constructed object.

      `var bar = new foo()`

  2. Is the function called with `call` or `apply` (`explicit binding`), [...] `bind`? If so, `this` is the explicitly specified object.

      `var bar = foo.call( obj2 )`

  3. Is the function called `with a context` (`implicit binding`), [...] an `owning` or `containing object`? If so, `this` is `that` context object.

      `var bar = obj1.foo()`

  4. Otherwise, default the `this` (`default binding`). If in `strict mode`, pick `undefined`, otherwise pick the `global` object.

      `var bar = foo()`

* That's `all it takes` to understand the rules of `this` binding for normal function calls. Well... `almost`.

## Binding Exceptions
* there are some `exceptions` to the "rules".

### Ignored `this`
* If you pass `null` or `undefined` as a `this` binding parameter to `call`, `apply`, or `bind`, those values are effectively ignored, and instead the `default binding` rule applies to the invocation.

```js
function foo() {
	console.log( this.a );
}

var a = 2;

foo.call( null ); // 2
```

* there's a [...] `danger` in always using `null` when you don't care about the `this` binding. If you ever use that against a `function call` (for instance, a `third-party library function` [...]), and that function `does` make a `this` reference [...] it might inadvertently reference (or worse, `mutate`!) the `global` object (`window` in the browser).

#### Safer `this`
* "`safer`" practice is to pass a specifically set up object for `this` which is guaranteed not to [...] create [...] `side effects` in your program. Borrowing terminology from `networking` (and the `military`), we can create a "`DMZ`" [...] object [...] a `completely empty` [...] insulates our program's `global` object from `side-effects`.

* the `easiest way` to set it up as `totally empty` is `Object.create(null)` (see Chapter 5). `Object.create(null)` is similar to `{ }`, but `without` the `delegation` to `Object.prototype`, so it's "`more empty`" than just `{ }`.

### Indirection
* `indirect references` occur is from an assignment:

```js
function foo() {
	console.log( this.a );
}

var a = 2;
var o = { a: 3, foo: foo };
var p = { a: 4 };

o.foo(); // 3
(p.foo = o.foo)(); // 2

// (me)
p.foo() // 4
```

### Softening Binding
## Lexical `this`
* `ES6` introduces a special kind of function [...] `arrow-function` [...] Instead of using the four standard `this` rules, `arrow-functions` adopt the `this` binding from the `enclosing (function or global) scope`.

```js
function foo() {
	// return an arrow function
	return (a) => {
		// `this` here is lexically adopted from `foo()`
		console.log( this.a );
	};
}

var obj1 = {
	a: 2
};

var obj2 = {
	a: 3
};

var bar = foo.call( obj1 );
bar.call( obj2 ); // 2, not 3!
```

* The arrow-function created in `foo()` `lexically captures` whatever `foo()`s `this` is `at its call-time`. Since `foo()` was `this`-bound to `obj1`, `bar` (a reference to the `returned arrow-function`) will also be `this`-bound to `obj1`.

* The `lexical binding` of an `arrow-function` `cannot be overridden` (`even with` `new`!).

* most common use-case will likely be in the use of `callbacks`.

```js
function foo() {
	setTimeout(() => {
		// `this` here is lexically adopted from `foo()`
		console.log( this.a );
	},100);
}

var obj = {
	a: 2
};

foo.call( obj ); // 2
```

## Review (TL;DR)
* order of precedence (me: of 4 `this` binding rules):
  1. Called with `new`? Use the newly constructed object.

  2. Called with `call` or `apply` (or `bind`)? Use the specified object.

  3. Called with a context object owning the call? Use that context object.

  4. Default: `undefined` in `strict mode`, global object otherwise.

# Chapter 3: Objects
## Syntax

* Objects come in two forms:

* The `literal syntax`.

```js
var myObj = {
	key: value
	// ...
};
```

* The `constructed form`.

```js
var myObj = new Object();
myObj.key = value;
```

## Type
* `6` `primary` types.

* `string`
* `number`
* `boolean`
* `null`
* `undefined`
* `object`

* `simple primitives` (`string`, `number`, `boolean`, `null`, and `undefined`) are `not` [...] `objects`.

* `null` is sometimes referred to as an `object type`, but this `misconception` stems from a `bug in the language` which causes `typeof null` to return the string `"object"` incorrectly [...] `null` is its own primitive type.

* there *are* a few `special object sub-types`, which we can refer to as `complex primitives`.

* `function` is a `sub-type of object` (technically, a "`callable object`").

* `Arrays` are also a form of objects, with `extra behavior`. The organization of contents in arrays is [...] more structured than for general objects.

### Built-in Objects
* There are `several other` `object` `sub-types`, usually referred to as `built-in objects`.
  * `String`
  * `Number`
  * `Boolean`
  * `Object`
  * `Function`
  * `Array`
  * `Date`
  * `RegExp`
  * `Error`

* `these` are [...] just `built-in functions` [...] can be used as `a constructor` ([...] a function call with the `new` operator [...]).

```js
var strPrimitive = "I am a string";
typeof strPrimitive;							// "string"
strPrimitive instanceof String;					// false

var strObject = new String( "I am a string" );
typeof strObject; 								// "object"
strObject instanceof String;					// true

// inspect the object sub-type
Object.prototype.toString.call( strObject );	// [object String]
```

* We'll see in detail in a later chapter exactly how the `Object.prototype.toString...` bit works, but briefly [...] you can see it reveals that `strObject` is an object [...] created by the `String` constructor.

* The primitive value `"I am a string"` is `not an object`, it's a `primitive literal` and `immutable` value. To perform operations on it, such as checking its length [...] a `String` object is required.

* the language `automatically` coerces a `"string"` primitive to a `String` object `when necessary`.

```js
var strPrimitive = "I am a string";

console.log( strPrimitive.length );			// 13

console.log( strPrimitive.charAt( 3 ) );	// "m"
```

* In both cases, we call a `property` or `method` `on a string primitive` [...] the engine `automatically` coerces it to a `String` object, so that the `property/method access` works.

* `null` and `undefined` have `no object wrapper` form [...] By contrast, `Date` values can `only` be created with their `constructed object form`.

`Object`s, `Array`s, `Function`s, and `RegExp`s [...] are all objects `regardless of` whether the `literal` or `constructed form` is used.

* `Error` [...] can be created with the `constructed form` `new Error(..)`, but it's often unnecessary.

## Contents
* It's important to note that while we say "`contents`" which implies that these values are `actually` stored inside the object, that's merely an appearance [...] What is stored [...] are [...] `property names`, which act as `pointers` (technically, `references`) to `where the values are stored`.

* The `.a` syntax is usually referred to as `"property" access`, whereas the ["a"] syntax is usually referred to as `"key" access` [...] they both access the same location, and will pull out the same value [...] so the terms can be used interchangeably. We will use the `most common term`, "`property access`" from here on.

```js
var myObject = {
	a: 2
};

myObject.a;		// 2

myObject["a"];	// 2
```

* property names are `always` `strings`. If you use any other value besides a `string` (primitive) as the property, it will first be `converted to a string` [...] includes numbers, which are commonly used as `array indexes`.

```js
var myObject = { };

myObject[true] = "foo";
myObject[3] = "bar";
myObject[myObject] = "baz";

myObject["true"];				// "foo"
myObject["3"];					// "bar"
myObject["[object Object]"];	// "baz"
```

### Computed Property Names
* `ES6` adds `computed property names`:

```js
var prefix = "foo";

var myObject = {
	[prefix + "bar"]: "hello",
	[prefix + "baz"]: "world"
};

myObject["foobar"]; // hello
myObject["foobaz"]; // world
```

* most common usage of `computed property names` will probably be for ES6 `Symbol`s [...] `new primitive data type` [...] You will be strongly discouraged from working with the `actual value` of a `Symbol` [...] `different` `between different JS engines` [...] [...] name of the `Symbol`, like `Symbol.Something` (just a `made up name`!), will be what you use:

```js
var myObject = {
	[Symbol.Something]: "hello world"
};
```

### Property vs. Method
* in other languages, `functions` which `belong to` `objects` (aka, "`classes`") are referred to as "`methods`", it's `not uncommon` to hear, "`method access`" as opposed to "`property access`".

* `The specification` makes this `same distinction`, interestingly.

* `Technically`, `functions` `never` "belong" to `objects`.

* conclusion [...] "`function`" and "`method`" are `interchangeable` in JavaScript.

* Even when you declare a `function expression` as part of the `object-literal`, that function `doesn't` [...] `belong` [...] to the object -- still just `multiple references` to the `same function object`:

```js
var myObject = {
	foo: function foo() {
		console.log( "foo" );
	}
};

var someFoo = myObject.foo;

someFoo;		// function foo(){..}

myObject.foo;	// function foo(){..}
```

### Arrays
* `are` `objects` [...] you can [...] add properties onto the array:

```js
var myArray = [ "foo", 42, "bar" ];

myArray.baz = "baz";

myArray.length;	// 3

myArray.baz;	// "baz"
```

* `Notice` that adding `named properties` [...] does `not` change the reported `length` of the array.

* You `could` use an array as a plain `key/value object`, and never add any `numeric indices` [...] a `bad idea` because arrays have [...] `optimizations` specific to their `intended use`, and `likewise` with `plain objects`. Use objects to store `key/value` pairs, and arrays to store values `at numeric indices`.

* **Be careful:** If you try to add a property to an array, but the `property name` `looks` like a `number`, it will end up [...] as a `numeric index` (thus `modifying the array contents`):

```js
var myArray = [ "foo", 42, "bar" ];

myArray["3"] = "baz";

myArray.length;	// 4

myArray[3];		// "baz"
```

### Duplicating Objects
* One of the most commonly requested features [...] is how to duplicate an object [...] like [...] a built-in `copy()` method [...] it's a little more complicated [...] because it's not fully clear `what`, `by default`, should be the `algorithm for the duplication`.

```js
function anotherFunction() { /*..*/ }

var anotherObject = {
	c: true
};

var anotherArray = [];

var myObject = {
	a: 2,
	b: anotherObject,	// reference, not a copy!
	c: anotherArray,	// another reference!
	d: anotherFunction
};

anotherArray.push( anotherObject, myObject );
```

* Firstly, we should answer if it should be a `shallow` or `deep` copy.

* (me: for) `a shallow copy` [...] `ES6` has now defined `Object.assign(..)` for this task. `Object.assign(..)` takes a `target object` as its `first parameter`, and `one or more` `source objects` as its `subsequent parameters`. It `iterates` over all the `enumerable` (see below), `owned keys` [...] on the `source object(s`) and `copies` them (via `=` assignment `only`) to `target`. It also [...] returns `target`.

```js
var newObj = Object.assign( {}, myObject );

newObj.a;						// 2
newObj.b === anotherObject;		// true
newObj.c === anotherArray;		// true
newObj.d === anotherFunction;	// true
```

### Property Descriptors
* as of `ES5`, `all` properties are described in terms of a **property descriptor**.

```js
var myObject = {
	a: 2
};

Object.getOwnPropertyDescriptor( myObject, "a" );
// {
//    value: 2,
//    writable: true,
//    enumerable: true,
//    configurable: true
// }
```

* use `Object.defineProperty(..)` to `add a new property`, `or` modify an `existing one` (`if` it's `configurable`!), `with` the `desired characteristics`.

```js
var myObject = {};

Object.defineProperty( myObject, "a", {
	value: 2,
	writable: true,
	configurable: true,
	enumerable: true
} );

myObject.a; // 2
```

* you generally wouldn't use this `manual approach` `unless` you wanted to modify [...] the `descriptor characteristics` from `its normal behavior`.

#### Writable
* The ability for you to `change the value` of a property.

```js
var myObject = {};

Object.defineProperty( myObject, "a", {
	value: 2,
	writable: false, // not writable!
	configurable: true,
	enumerable: true
} );

myObject.a = 3;

myObject.a; // 2
```

* `modification` of the `value` `silently failed`. If we try in `strict mode`, we get (me: a `TypeError error`):

#### Configurable
* `As long as` a property is currently `configurable`, we can modify its `descriptor definition`, using [...] `defineProperty(..)`.

```js
var myObject = {
	a: 2
};

myObject.a = 3;
myObject.a;					// 3

Object.defineProperty( myObject, "a", {
	value: 4,
	writable: true,
	configurable: false,	// not configurable!
	enumerable: true
} );

myObject.a;					// 4
myObject.a = 5;
myObject.a;					// 5

Object.defineProperty( myObject, "a", {
	value: 6,
	writable: true,
	configurable: true,
	enumerable: true
} ); // TypeError
```

* The final `defineProperty(..)` call results in a `TypeError`, `regardless of` `strict mode` [...] Be careful: as you can see, changing `configurable` to `false` is a `one-way action, and cannot be undone!`.

* There's a [...] `exception` [...]: `even if` the property is already `configurable:false`, `writable` `can always be changed` from `true` to `false` `without error`, but `not back` to `true` if already `false`.

* Another thing `configurable:false` `prevents` is the ability to use the `delete` operator to `remove an existing property`.

```js
var myObject = {
	a: 2
};

myObject.a;				// 2
delete myObject.a;
myObject.a;				// undefined

Object.defineProperty( myObject, "a", {
	value: 2,
	writable: true,
	configurable: false,
	enumerable: true
} );

myObject.a;				// 2
delete myObject.a;
myObject.a;				// 2
```

#### Enumerable
* controls if a property will `show up` in certain object-property `enumerations` [...] even though it's still [...] `accessible`.

* `All` normal `user-defined properties` are `defaulted` to `enumerable`.

### Immutability
* note that `all` of these approaches create `shallow immutability`. That is, they affect `only` the `object` and `its direct property characteristics`. If an object has (me: `properties` `referencing to other objects` (`array`, `object`, `function`, etc), the `contents` of these objects are `not affected`, and `remain mutable`).

#### Object Constant
* By combining `writable:false` and `configurable:false`, you can essentially create a `constant` (`cannot be` `changed`, `redefined` or `deleted`) as an `object property`:

```js
var myObject = {};

Object.defineProperty( myObject, "FAVORITE_NUMBER", {
	value: 42,
	writable: false,
	configurable: false
} );
```

#### Prevent Extensions
* If you want to prevent an object from `having new properties added` to it, `but` otherwise leave the `rest of the object's properties` alone, call `Object.preventExtensions(..)`:

```js
var myObject = {
	a: 2
};

Object.preventExtensions( myObject );

myObject.b = 3;
myObject.b; // undefined

// (me: the rest of the object's properties are let alone)
myObject.a = 3
myObject.a // 3
```

* In `strict mode`, it throws a `TypeError`.

#### Seal
* `Object.seal(..)` [...] takes an existing object and essentially calls `Object.preventExtensions(..)` on it, but also marks `all its existing properties` as `configurable:false`.

* So, `not only` can you `not` add any more properties [...] you also `cannot` `reconfigure` or `delete` any `existing properties` (`though` you `can still modify` their values).

#### Freeze
* `Object.freeze(..)` [...] takes an existing object and essentially calls `Object.seal(..)` on it, but it also marks `all "data accessor" properties` as `writable:false`, so that `their values` `cannot be changed` [...] `though`, as mentioned above, the contents of any `referenced other objects` are `unaffected`.

### `[[Get]]`
```js
var myObject = {
	a: 2
};

myObject.a; // 2
```

* The `myObject.a` is a `property access`, but it doesn't `just` look in `myObject` for a property of the name `a`, as it might seem.

* the code above [...] performs a `[[Get]]` operation (kinda like a function call: `[[Get]]()`) on the `myObject`. The `default built-in` `[[Get]]` operation [...] inspects the object for a property of the `requested name`, and [...] return the value accordingly [...] if it does `not` find a property [...] We will examine in `Chapter 5` what happens `next` (`traversal` of the `[[Prototype]]` chain, `if any`).

* if (me: `[[Get]]`)(it) `cannot` [...] come up with a value for the `requested property`, it instead returns the value `undefined`.

```js
var myObject = {
	a: 2
};

myObject.b; // undefined
```

* Inspecting `only the value` results, you `cannot` distinguish whether a property `exists` and holds the `explicit value` `undefined`, or whether the property `does not exist` [...] `However`, we will see shortly how you `can` distinguish these two scenarios.

### `[[Put]]`
* When invoking `[[Put]]`, how it behaves differs based on a number of factors:
  * If the property is `present`, the `[[Put]]` algorithm will roughly check:
    1. Is the property an (me: read below)(`accessor descriptor`) (see "Getters & Setters" section below)? **If so, call the setter, if any.**
    2. Is the property a `data descriptor` with `writable` of `false`? **If so, silently fail in `non-strict mode`, or throw `TypeError` in `strict mode`.**
    3. `Otherwise`, set the value to the existing property.

  * If the property is `not yet present` [...] the `[[Put]]` operation is even more [...] complex. We will revisit this [...] in `Chapter 5`.

### Getters & Setters
* The default `[[Put]]` and `[[Get]]` operations for objects `completely control` how values `are set` to `existing or new properties`, or `retrieved` `from existing properties`, respectively.

* `ES5` introduced a way to `override` `part of these default operations`, `not` on an `object level` but a `per-property level`
  * `Getters` `are properties` which actually call a `hidden function` to `retrieve a value`.
  * `Setters` `are properties` which actually call a `hidden function` to `set a value`.

* When [...] a property [...] have either a `getter` or a `setter` or `both`, its definition becomes an "`accessor descriptor`" (as `opposed to` a "`data descriptor`"). For `accessor-descriptors`, the `value` and `writable` `characteristics of the descriptor` are [...] `ignored`, and instead JS considers the `set` and `get` `characteristics of the property` (`as well as` `configurable` and `enumerable`).

```js
var myObject = {
	// define a getter for `a`
	get a() {
		return 2;
	}
};

Object.defineProperty(
	myObject,	// target
	"b",		// property name
	{			// descriptor
		// define a getter for `b`
		get: function(){ return this.a * 2 },

		// make sure `b` shows up as an object property
		enumerable: true
	}
);

myObject.a; // 2

myObject.b; // 4
```

* Either through `object-literal` syntax with `get a() { .. }` or through `explicit definition` with `defineProperty(..)` [...] we created a property on the object that [...] doesn't hold a value [...] whose `access` automatically results in a `hidden function call` to the `getter function`, with whatever value it `returns` being the result of the `property access`.

```js
var myObject = {
	// define a getter for `a`
	get a() {
		return 2;
	}
};

myObject.a = 3;

myObject.a; // 2
```

* Since we `only` defined a `getter` for `a`, if we try to `set the value` of `a` `later`, the set operation [...] will just `silently` `throw the assignment away`.

* properties should also be defined with `setters`, which `override` the default `[[Put]]` operation [...] You will almost certainly want to always declare both `getter` and `setter`.

```js
var myObject = {
	// define a getter for `a`
	get a() {
		return this._a_;
	},

	// define a setter for `a`
	set a(val) {
		this._a_ = val * 2;
	}
};

myObject.a = 2;

myObject.a; // 4
```

### Existence
```js
var myObject = {
	a: 2
};

("a" in myObject);				// true
("b" in myObject);				// false

myObject.hasOwnProperty( "a" );	// true
myObject.hasOwnProperty( "b" );	// false
```

* The `in` operator will check [...] (me: for the `existence of a property name` in the object), or if it `exists` at any `higher level` of the `[[Prototype]]` chain.
* `hasOwnProperty(..)` checks to see if `only` `myObject` `has the property or not`, and will `not` consult the `[[Prototype]]` chain.

* `hasOwnProperty(..)` is `accessible` for `all normal objects` via `delegation to` `Object.prototype` (see Chapter 5). But it's possible to create an object that `does not` link to `Object.prototype` (via `Object.create(null)` -- see Chapter 5). In this case, a method call like `myObject.hasOwnProperty(..)` would fail.

* In that scenario, a more robust [...] is `Object.prototype.hasOwnProperty.call(myObject,"a")`, which `borrows` the base `hasOwnProperty(..)` method and uses `explicit` `this` binding (see Chapter 2) to apply it against our `myObject`.

#### Enumeration
```js
var myObject = { };

Object.defineProperty(
	myObject,
	"a",
	// make `a` enumerable, as normal
	{ enumerable: true, value: 2 }
);

Object.defineProperty(
	myObject,
	"b",
	// make `b` NON-enumerable
	{ enumerable: false, value: 3 }
);

myObject.b; // 3
("b" in myObject); // true
myObject.hasOwnProperty( "b" ); // true

// .......

for (var k in myObject) {
	console.log( k, myObject[k] );
}
// "a" 2
```

* `myObject.b` [...] `exists` and has an accessible value, but it doesn't show up in a `for..in` loop.

* `for..in` loops applied to arrays [...] will include not only all the `numeric indices`, but also any `enumerable` properties. It's a good idea to use `for..in` loops `only` on `objects`, and traditional `for` loops with `numeric index iteration` for the values stored in arrays.

* Another way that enumerable and non-enumerable properties can be distinguished:

```js
var myObject = { };

Object.defineProperty(
	myObject,
	"a",
	// make `a` enumerable, as normal
	{ enumerable: true, value: 2 }
);

Object.defineProperty(
	myObject,
	"b",
	// make `b` non-enumerable
	{ enumerable: false, value: 3 }
);

myObject.propertyIsEnumerable( "a" ); // true
myObject.propertyIsEnumerable( "b" ); // false

Object.keys( myObject ); // ["a"]
Object.getOwnPropertyNames( myObject ); // ["a", "b"]
```

* `propertyIsEnumerable(..)` tests whether the given property name exists `directly` on the object and is also `enumerable:true`.
* `Object.keys(..)` returns an array of all `enumerable` `properties`.
* whereas `Object.getOwnPropertyNames(..)` returns an array of `all` properties.

* `Object.keys(..)` and `Object.getOwnPropertyNames(..)` `both` inspect `only` the `direct object specified`.

## Iteration
* The `for..in` loop iterates over the list of `enumerable properties` on an object (`including` its `[[Prototype]]` chain). But what if you instead want to iterate over the values?

* `As contrasted with` iterating over an array's `indices` in a `numerically ordered` way [...] the `order` of iteration over an `object's properties` [...] may vary between different `JS engines`.

* if you want to iterate over the values `directly` [...] `ES6` adds a `for..of` loop syntax for iterating over `arrays` (and `objects`, `if` the object defines its own custom `iterator`):

```js
var myArray = [ 1, 2, 3 ];

for (var v of myArray) {
	console.log( v );
}
// 1
// 2
// 3
```

* The `for..of` loop asks for an `iterator` object (from a default `internal function` known as `@@iterator` [...]) of the `thing` `to be iterated`, and the `loop` then iterates over the `successive return values` from calling that `iterator object`'s `next()` method, `once` for `each loop iteration`.

* let's `manually` iterate the array, using the built-in `@@iterator`, to see how it works:

```js
var myArray = [ 1, 2, 3 ];
var it = myArray[Symbol.iterator]();

it.next(); // { value:1, done:false }
it.next(); // { value:2, done:false }
it.next(); // { value:3, done:false }
it.next(); // { done:true }
```

* always [...] reference such special properties (me: like `@@iterator`) by `Symbol` name reference `instead` of by the [...] `value it [...] hold`. Also [...] `@@iterator` is `not the iterator object` itself, but a `function` that `returns the iterator object`.

* Notice the value `3` was returned with a `done:false` [...] You have to call the `next()` a `fourth time` ([...] the `for..of` loop [...] automatically does) to get `done:true`.

* `regular objects` `do not have` a `built-in @@iterator` [...] It is possible to define your own default `@@iterator` for `any object`.

```js
var myObject = {
	a: 2,
	b: 3
};

Object.defineProperty( myObject, Symbol.iterator, {
	enumerable: false,
	writable: false,
	configurable: true,
	value: function() {
		var o = this;
		var idx = 0;
		var ks = Object.keys( o );
		return {
			next: function() {
				return {
					value: o[ks[idx++]],
					done: (idx > ks.length)
				};
			}
		};
	}
} );

// iterate `myObject` manually
var it = myObject[Symbol.iterator]();
it.next(); // { value:2, done:false }
it.next(); // { value:3, done:false }
it.next(); // { value:undefined, done:true }

// iterate `myObject` with `for..of`
for (var v of myObject) {
	console.log( v );
}
// 2
// 3
```

* We used `Object.defineProperty(..)` to define our custom `@@iterator` [...] so we could make it `non-enumerable`.
* `but` [...] we could have declared it `directly`, like `var myObject = { a:2, b:3, [Symbol.iterator]: function(){ /* .. */ } }`.

## Review (TL;DR)
* Properties don't have to contain values -- they can be "`accessor properties`" (me: `getters` and `setters`).

# Chapter 4: Mixing (Up) "Class" Objects
## Class Theory
* `OO` or `class oriented programming` stresses that data [...] has associated `behavior` [...] so `proper design` is to [...] encapsulate [...] the data and the behavior together.

* `Classes` also imply a way of `classifying` [...] Let's [...] looking at a commonly cited example. A `car` can be described as a `specific implementation` of a `more general` "class" [...] called a `vehicle`.

* The definition of `Vehicle` might include things like [...] the ability to carry people, etc. [...] all be the `behaviors`. What we define in `Vehicle` is all the stuff that is `common` [...] different types of vehicles (the "`planes`, `trains`, and `automobiles`").

* we define that capability `once` in `Vehicle`, and then when we define `Car`, we simply indicate that it "`inherits`" (or "`extends`") the base definition from `Vehicle`.

* Another key concept with classes is "`polymorphism`" [...] idea (me: is) that a general behavior from a `parent class` `can be overridden` in a `child class` [...] In fact, `relative polymorphism` lets us `reference` the `base behavior` `from the overridden behavior`.

### "Class" Design Pattern
* it's most common to [...] an assumption that [...] `OO` is a given foundation for *all* (proper) code.

* "`procedural programming`" as a way of describing code which only consists of `procedures` (aka, `functions`) calling `other functions` [...] You may have been taught that `classes` were the *proper* way to transform `procedural-style` "`spaghetti code`" into well-formed, well-organized code.

* (me: in) "`functional programming`" [...] `classes` are just one of several common `design patterns`.

* languages (like Java) don't give you the choice [...] everything's a class. Other languages like `C/C++` or `PHP` give you both procedural and class-oriented syntaxes.

### JavaScript "Classes"
* But does [...] JavaScript actually *has* classes? [...] **No.**

* classes are an `optional pattern` in software design, and you have the choice to use them in JavaScript or not

## Class Mechanics
### Building
* A class is a `blue-print`. To actually `get` an object we can interact with, we must build (aka, "`instantiate`") something `from the class`. The end result of such "`construction`" is an `object`, [...] an "`instance`", which we can directly `call methods` on and `access` any public data properties from [...] This object is a `copy` of all the (me: contents)(characteristics) described by the class (me: to prevent modifications on `instances' contents` from being reflected to their base class. in the other words,) [...] A class is `instantiated into` `object form` by a `copy operation`.

### Constructor
* Instances of classes are `constructed` by a special method of the class [...] a `constructor`. This method's [...] job is to (me: `receive arguments`, if there are ones and) `initialize` any information (`state`) the instance will need.

## Class Inheritance
* In class-oriented languages [...] you can define another class that **inherits** from the first class.

* once a child class is defined [...] The child class contains an initial `copy` of the behavior from the parent, but can then `override` any `inherited behavior` and `even define new behavior`.

```js
class Vehicle {
	engines = 1

	ignition() {
		output( "Turning on my engine." )
	}

	drive() {
		ignition()
		output( "Steering and moving forward!" )
	}
}

class Car inherits Vehicle {
	wheels = 4

	drive() {
		inherited:drive()
		output( "Rolling on all ", wheels, " wheels!" )
	}
}

class SpeedBoat inherits Vehicle {
	engines = 2

	ignition() {
		output( "Turning on my ", engines, " engines." )
	}

	pilot() {
		inherited:drive()
		output( "Speeding through the water with ease!" )
	}
}
```

* For clarity [...] `constructors` for these classes have been `omitted`.

### Polymorphism
* `Polymorphism` is a much broader topic than we will exhaust here, but our current "`relative`" semantics refers to one particular aspect: the idea that `any method` can reference `another method` (of the `same` or `different name`) at a `higher level` of the `inheritance hierarchy`. We say "`relative`" because we don't [...] define which `inheritance level` (aka, `class`) we want to access, but rather `relatively` reference it by [...] "`look one level up`".

* In many languages, the keyword `super` is used, in place of this example's `inherited:`, which leans on the idea that a "`super class`" is the `parent/ancestor` of the current class.

* Another aspect of polymorphism is that a `method name` can have `multiple definitions` at `different levels of the inheritance chain` [...] We see two occurrences of that behavior in our example above: `drive()` is defined in both `Vehicle` and `Car`, and `ignition()` is defined in both `Vehicle` and `SpeedBoat`.

* Another thing that [...] `class-oriented` languages give you via `super` is a direct way for the constructor of a `child class` to reference the constructor of `its parent class`.

* Inside `pilot()`, a [...] reference is made to [...] `Vehicle`s version of `drive()`. `But` that `drive()` references an `ignition()` method `just by name` [...] Which version of `ignition()` will the language engine use, the one from `Vehicle` or the one from `SpeedBoat`? It uses the `SpeedBoat` version of `ignition()`. `If` you `were` to instantiate `Vehicle` class [...] and then call `its` `drive()`, the language engine would instead just use `Vehicle`s `ignition()` method.

* When classes `are inherited`, there is a way `for the classes themselves` (`not` the `object instances` created from them!) to [...] reference the class `inherited from`, and this [...] `reference` is usually called `super`.

* `Class inheritance implies copies` (me: read a more comprehensive explaination in the `Mixing Copies` section below).

### Multiple Inheritance
* Some `class-oriented` languages allow you to specify more than one "`parent`" class to "`inherit`" from. `Multiple-inheritance` means that each parent class definition is `copied` into the `child class` [...] However, there are certainly some complicating questions [...] arise.

* `JavaScript` [...] does not provide a `native` mechanism for "`multiple inheritance`".

## Mixins
* JavaScript's object mechanism does not *automatically* perform copy behavior when you "inherit" or "instantiate". Plainly, there are no "classes" in JavaScript to instantiate, only objects. And objects don't get copied to other objects, they get *linked together* (more on that in Chapter 5).

* let's examine how JS developers **fake** the *missing* `copy behavior` of classes in JavaScript: `mixins`. We'll look at two types of "mixin": **explicit** and **implicit**.

### Explicit Mixins
* we can [...] create a utility that `manually copies`. Such a utility is often called `extend(..)` by many `libraries/frameworks`, but we will call it `mixin(..)` here.

```js
// vastly simplified `mixin(..)` example:
function mixin( sourceObj, targetObj ) {
	for (var key in sourceObj) {
		// only copy if not already present
		if (!(key in targetObj)) {
			targetObj[key] = sourceObj[key];
		}
	}

	return targetObj;
}

var Vehicle = {
	engines: 1,

	ignition: function() {
		console.log( "Turning on my engine." );
	},

	drive: function() {
		this.ignition();
		console.log( "Steering and moving forward!" );
	}
};

var Car = mixin( Vehicle, {
	wheels: 4,

	drive: function() {
		Vehicle.drive.call( this );
		console.log( "Rolling on all " + this.wheels + " wheels!" );
	}
} );
```

* there are `no classes` in JavaScript. `Vehicle` and `Car` are just objects that we make copies from and to, respectively.

* `Car` now has a `copy` of the properties [...] from `Vehicle`. `Technically`, `functions` are `not actually duplicated`, but rather `references` to the functions are copied.

* `Car` *already* had a `drive` property (function), so `that property reference` was `not overridden`.

#### "Polymorphism" Revisited
* Let's examine this statement: `Vehicle.drive.call( this )`. This is what I call "`explicit pseudo-polymorphism`". Recall in our previous `pseudo-code` this line was `inherited:drive()`, which we called "`relative` polymorphism".

* JavaScript `does not` have (`prior` to `ES6` [...]) a facility for `relative` `polymorphism`. So, **because both `Car` and `Vehicle` had a function of the same name: `drive()`**, to distinguish a call [...] we must make an `absolute` (`not relative`) reference. We explicitly specify the `Vehicle` object by name, and call the `drive()` function on it.

* But if we said `Vehicle.drive()`, the `this` binding for that function call would be the `Vehicle` [...] So, instead we use `.call( this )` [...] to ensure that `drive()` is executed in the `context` of the `Car` object.

* In `class-oriented languages` [...] the `linkage` between `Car` and `Vehicle` is established once, at the top of the class definition, which makes for only `one place` to maintain such relationships.

* `shadowing` [...] creates brittle manual/explicit linkage **in `every single function` where you need such a (`pseudo`-)polymorphic reference** [...] [...] is [...] more complex [...] harder-to-maintain code. **`Explicit` `pseudo`-polymorphism should be avoided wherever possible**.

#### Mixing Copies
* Recall the `mixin(..)` utility from above [...] we [...] explicitly copied the `non-overlapping` contents of `Vehicle` into `Car`. The name "`mixin`" comes from an alternate way of explaining the task: `Car` has `Vehicle`s contents **mixed-in** [...] As a result of the `copy operation`, `Car` will operate somewhat `separately` from `Vehicle`. If you add a property onto `Car`, it `will not affect` `Vehicle`, and `vice versa`.

* There are still some subtle ways the `two objects` can "`affect`" each other even `after copying`, such as if they both share a reference to a [...] `object` (such as an `array`).

* Since the two objects also share `references` to their common `functions` (me: since) [...] JavaScript functions can't really be duplicated (in a [...] reliable way) [...] you end up with instead is a **duplicated reference** to the same `shared function object` [...] If you modified one of the shared **function objects** [...] by adding properties on top of it, for instance, both `Vehicle` and `Car` would be "`affected`" via the `shared reference`.

* `mix-in` two or more objects into your target object, you can **partially emulate** the behavior of "`multiple inheritance`".

#### Parasitic Inheritance
* A variation on this explicit mixin pattern, which is both in some ways explicit and in other ways implicit, is called "`parasitic inheritance`".

```js
// "Traditional JS Class" `Vehicle`
function Vehicle() {
	this.engines = 1;
}
Vehicle.prototype.ignition = function() {
	console.log( "Turning on my engine." );
};
Vehicle.prototype.drive = function() {
	this.ignition();
	console.log( "Steering and moving forward!" );
};

// "Parasitic Class" `Car`
function Car() {
	// first, `car` is a `Vehicle`
	var car = new Vehicle();

	// now, let's modify our `car` to specialize it
	car.wheels = 4;

	// save a privileged reference to `Vehicle::drive()`
	var vehDrive = car.drive;

	// override `Vehicle::drive()`
	car.drive = function() {
		vehDrive.call( this );
		console.log( "Rolling on all " + this.wheels + " wheels!" );
	};

	return car;
}

var myCar = new Car();

myCar.drive();
// Turning on my engine.
// Steering and moving forward!
// Rolling on all 4 wheels!
```

* when we call `new Car()`, a new object is created and referenced by `Car`s `this` reference [...] But since we don't use that object, and instead return our own `car` object, the initially created object is just `discarded`. So, `Car()` could be called without the `new` keyword, and the functionality above would be `identical`, but `without the wasted object creation/garbage-collection`.

### Implicit Mixins
Implicit mixins are closely related to *explicit pseudo-polymorphism* as explained previously. As such, they come with the same caveats and warnings.

Consider this code:

```js
var Something = {
	cool: function() {
		this.greeting = "Hello World";
		this.count = this.count ? this.count + 1 : 1;
	}
};

Something.cool();
Something.greeting; // "Hello World"
Something.count; // 1

var Another = {
	cool: function() {
		// implicit mixin of `Something` to `Another`
		Something.cool.call( this );
	}
};

Another.cool();
Another.greeting; // "Hello World"
Another.count; // 1 (not shared state with `Something`)
```

* With `Something.cool.call( this )` [...] we essentially "borrow" the function `Something.cool()` and call it `in the context` of `Another` (me: `Another.cool()`) [...] instead of `Something`. The end result is that the assignments that `Something.cool()` makes are applied against the `Another` object rather than the `Something` object [...] **avoid such constructs where possible** to keep cleaner and more maintainable code.

## Review (TL;DR)

# Chapter 5: Prototypes
## `[[Prototype]]`
* Objects in JavaScript have an internal property [...] `[[Prototype]]` [...] a reference to `another object`. `Almost` all objects are given a `non-null value` for this property, `at the time of their creation`.

* `everything` [...] about normal `[[Get]]` and `[[Put]]` behavior does `not apply` if a `Proxy` (me: `ES6`) is involved.

* The default `[[Get]]` operation proceeds to follow the `[[Prototype]]` **link** of the object if it cannot find the requested property (me: `on the object itself`, this operation continues until it either find the property or the `[[Prototype]]` chain `ends`. If no matching property is found the return result from the `[[Get]]` is `undefined`).

```js
var anotherObject = {
	a: 2
};

// create an object linked to `anotherObject`
var myObject = Object.create( anotherObject );

myObject.a; // 2
```

* We will explain what `Object.create(..)` [...] shortly. For now, just assume it creates an object with the `[[Prototype]]` linkage we're examining to the object `specified` [...] So [...] `myObject` that is now `[[Prototype]]` linked to `anotherObject`.

* `for..in` loop [...] iterate over [...] any property that can be reached via its chain  [...] and is also `enumerable` [...] `in` (me: operator also) check the entire chain of the object (`regardless of` *enumerability*).

```js
var anotherObject = {
	a: 2
};

// create an object linked to `anotherObject`
var myObject = Object.create( anotherObject );

for (var k in myObject) {
	console.log("found: " + k);
}
// found: a

("a" in myObject); // true
```

### `Object.prototype`
* But *where* exactly does the `[[Prototype]]` chain "end"?

* The (me: `higher-level implementation`)(`top-end`) of every *normal* `[[Prototype]]` chain is the built-in `Object.prototype` [...] includes [...] common utilities used all over JS, because all [...] built-in [...] objects in JavaScript "`descend from`" [...] the `Object.prototype` object.

* Some utilities found here [...] include `.toString()` and `.valueOf()` [...] `.hasOwnProperty(..)` [...] `.isPrototypeOf(..)`.

### Setting & Shadowing Properties
* Back in `Chapter 3`, we mentioned that `setting properties` [...] was more nuanced than just `adding a new property` [...] or changing an existing property's value.

```js
myObject.foo = "bar";
```

* If the `myObject` object already has a [...] property called `foo` `directly` present on it, the `assignment` is as simple as `changing the value of the existing property`.

* If `foo` is not already present directly on `myObject`, the `[[Prototype]]` chain is `traversed`, just like for the `[[Get]]` operation. If `foo` is `not found anywhere` [...] the property `foo` is added `directly` to `myObject` with the specified value.

* `However`, if foo is `already present` somewhere (me: `deeper`)(`higher`) in the chain [...] surprising [...] behavior can occur with the `myObject.foo = "bar"` assignment [...] We will now examine `three scenarios` (me: that could happen).
  1. If a normal `data accessor [...] property` (me: not an `accessor property` which are merely a `getter` or a `setter`, `data accessor property` is just a `normal property`, sometimes called a `data property`) named `foo` is found anywhere (me: `deeper`)(`higher`) on the `[[Prototype]]` chain, **and it's `not` marked as read-only (`writable:false`)** then a new property called `foo` is added directly to `myObject`, resulting in a **shadowed property**.

	```js
	// (me)
	var anotherObject = {
		foo: 2
	};

	var myObject = Object.create( anotherObject );

	myObject.foo = 3

	console.log(anotherObject); // {foo: 2}
	console.log(myObject); // {foo: 3}: `foo` is directly added to the object `myObject`
	```

	2. If a `foo` is found (me: `deeper`)(`higher`) on the `[[Prototype]]` chain, but it's marked as **read-only (`writable:false`)** [...] (me: and) If the code is running in `strict mode`, an error will be thrown. Otherwise, the (me: assignment)(setting) of the property value will `silently be ignored` [...] **no shadowing occurs**.

	```js
	// (me)
	var anotherObject = {};

	Object.defineProperty(anotherObject, 'foo', {
		value: 2,
		writable: false
	});

	var myObject = Object.create( anotherObject );

	myObject.foo = 3

	console.log(anotherObject); // {foo: 2}
	console.log(myObject); // {}: no shadowing occurs
	```

	```js
	// (me)
	"use strict";
	var anotherObject = {};

	Object.defineProperty(anotherObject, 'foo', {
		value: 2,
		writable: false
	});

	var myObject = Object.create( anotherObject );

	myObject.foo = 3

	console.log(anotherObject);
	console.log(myObject);

	// results:
	//
	// TypeError: Cannot assign to read only property 'foo'
	```
  
	3. If a `foo` is found (`deeper`)`higher` on the `[[Prototype]]` chain and it's a `setter` [...] then the `setter` will `always be called`. No `foo` will be added to [...] `myObject`, nor will the `foo` setter be `redefined`.

	```js
	// (me)
	var anotherObject = {
			_foo: 2,
			set foo(newVal) {
					this._foo = newVal;
			}
	};

	var myObject = Object.create( anotherObject );

	myObject.foo = 3

	console.log(anotherObject); // {_foo: 2}
	console.log(myObject); // {_foo: 3}: no `foo` is added to `myObject`. Instead, the `foo` setter is called in the context of the object `myObject`
	```

* Most developers assume that assignment of a property (`[[Put]]`) will `always` result in `shadowing` if the property already exists (me: `deeper`)(higher) on the `[[Prototype]]` chain, but as you can see, that's only true in [...] `#1`.

* If you want to shadow `foo` in cases `#2` and `#3`, you `cannot` use `=` assignment [...] `must instead` use `Object.defineProperty(..)` [...] to add `foo` to `myObject`.

* `shadowing` is `more complicated` [...] `than it's worth` [...] See `Chapter 6` for an `alternative design pattern`.

* `Shadowing` can even occur `implicitly`.

```js
var anotherObject = {
	a: 2
};

var myObject = Object.create( anotherObject );

anotherObject.a; // 2
myObject.a; // 2

anotherObject.hasOwnProperty( "a" ); // true
myObject.hasOwnProperty( "a" ); // false

myObject.a++; // oops, implicit shadowing!

anotherObject.a; // 2
myObject.a; // 3

myObject.hasOwnProperty( "a" ); // true
```

* the `++` operation corresponds to `myObject.a = myObject.a + 1`. The result is `[[Get]]` looking up `a` property via `[[Prototype]]` to get the current value `2` from `anotherObject.a` [...] then `[[Put]]` assigning the `3` value to a `new shadowed property` `a` on `myObject`. Oops!

## "Class"
### "Class" Functions
* `all functions` by default get a `public`, `non-enumerable` [...] property on them called `prototype`, which points at an otherwise arbitrary object.

```js
function Foo() {
	// ...
}

Foo.prototype; // { }
```

* each object `created from calling` `new Foo()` [...] end up [...] `[[Prototype]]-linked` to this "`Foo dot prototype`" `object`.

Let's illustrate:

```js
function Foo() {
	// ...
}

var a = new Foo();

Object.getPrototypeOf( a ) === Foo.prototype; // true
```

* When `a` is created by calling `new Foo()`, one of the things [...] happens is that `a` gets an internal `[[Prototype]]` link to the object that `Foo.prototype` is pointing at.

* That's it. We didn't instantiate a class. We [...] didn't do any `copying of behavior` from a "class" into a [...] object [...] `new Foo()` is an `indirect` [...] way to end up with what we want: **a new object linked to another object** [...] a `more direct` way? [...] is `Object.create(..)`.

#### What's in a name?
* I believe the label "`prototypal inheritance`" itself (and trying to `mis-apply` all its associated class-orientation terminology, like "class", "constructor", "instance", "polymorphism", etc) has done **more harm than good** in explaining how JavaScript's mechanism *really* works.

* "Inheritance" implies a *copy* [...] JavaScript doesn't copy object properties (`natively, by default`). Instead, JS creates a link between two objects, where one object can essentially *delegate* property/function access to another object. "`Delegation`" (see Chapter 6) is a `much more accurate` term for JavaScript's `object-linking mechanism`.
### "Constructors"
```js
function Foo() {
	// ...
}

Foo.prototype.constructor === Foo; // true

var a = new Foo();
a.constructor === Foo; // true
```

* The `Foo.prototype` object `by default` (at `declaration time` on line 1 of the snippet!) gets a `public`, `non-enumerable` [...] property called `.constructor`, and this property is a reference back to the `function` (`Foo` in this case) that the object is associated with.

* object `a` [...] `seems` to also have a property on it called `.constructor` which similarly points to "`the function which created it`" [...] This is `not actually true`. `a` has no `.constructor` property `on it`, and though `a.constructor` does in fact resolve to the `Foo` function, "`constructor`" **does not actually mean** "`was constructed by`" [...] We'll explain this strangeness shortly.

#### Constructor Or Call?
* a "`constructor`" is **any function called with the `new` keyword** in front of it [...] Functions aren't constructors, `but` `function calls` are (me: called) "`constructor calls`" if and only if `new` is used.

### Mechanics
* (me: revise) At the beginning of this chapter, we explained the `[[Prototype]]` link, and how it provides the `fall-back look-up` steps `if` a property reference `isn't found` `directly` on an object, as part of the `default` `[[Get]]` algorithm.

#### "Constructor" Redux
* There are actually several ways that the `ill-fated assumption` of `.constructor` meaning "`was constructed by`" can come back to bite you.

Consider:

```js
function Foo() { /* .. */ }

Foo.prototype = { /* .. */ }; // create a new prototype object

var a1 = new Foo();
a1.constructor === Foo; // false!
a1.constructor === Object; // true!
```

* It [...] `seems like` `Foo()` "constructed" (me: `a1`) [...] `but` where everything falls apart is when you think "`constructor`" means "`was constructed by`".

* What's happening? `a1` has no `.constructor` property, so it delegates up the `[[Prototype]]` chain to `Foo.prototype`. `But` that object `doesn't` have a `.constructor` either (like the `default` `Foo.prototype` object would have had!), so it `keeps delegating`, this time up to `Object.prototype`, the `top of the delegation chain`. *That* object indeed has a `.constructor` on it, which points to the built-in `Object(..)` function.

* The best thing to do is remind yourself, "`constructor does not mean constructed by`".

* `.constructor` is [...] *is* non-enumerable [...] but its value is writable (can be changed), and moreover, you can add or overwrite (intentionally or accidentally) a property of the name `constructor` on any object in any `[[Prototype]]` chain, with any value you see fit [...] `.constructor` is `extremely unreliable` [...] **such references should be avoided where possible.**

## "(Prototypal) Inheritance"
```js
function Foo(name) {
	this.name = name;
}

Foo.prototype.myName = function() {
	return this.name;
};

function Bar(name,label) {
	Foo.call( this, name );
	this.label = label;
}

// here, we make a new `Bar.prototype`
// linked to `Foo.prototype`
Bar.prototype = Object.create( Foo.prototype );

// Beware! Now `Bar.prototype.constructor` is gone,
// and might need to be manually "fixed" if you're
// in the habit of relying on such properties!

Bar.prototype.myLabel = function() {
	return this.label;
};

var a = new Bar( "a", "obj a" );

a.myName(); // "a"
a.myLabel(); // "obj a"
```

* The important part is `Bar.prototype = Object.create( Foo.prototype )`. `Object.create(..)` *creates* a "new" object [...] and links that new object's internal `[[Prototype]]` to the object you specify (`Foo.prototype` in this case).

* When `function Bar() { .. }` is declared [...] (me: it) has a `.prototype` link to `its default object`. But *that* object `is not` linked to `Foo.prototype` like we want. So, we create a *new* object that *is* linked as we want, effectively throwing away the original incorrectly-linked object.

* A common mis-conception/confusion here is that `either` of the following approaches would `also work`, `but` they do `not work as you'd expect`:

```js
// doesn't work like you want!
Bar.prototype = Foo.prototype;

// works kinda like you want, but with
// side-effects you probably don't want :(
Bar.prototype = new Foo();
```

* `Bar.prototype = Foo.prototype` [...] just makes `Bar.prototype` [...] (me: a `direct`) reference to `Foo.prototype` [...] This means when you start `assigning`, like `Bar.prototype.myLabel = ...`, you're modifying [...] `Foo.prototype` object itself, which would `affect` any `objects` `linked to` `Foo.prototype`.

* `Bar.prototype = new Foo()` **does in fact** create a new object [...] linked to `Foo.prototype` as we'd want [...] uses the `Foo(..)` "`constructor call`" [...] If that function has any `side-effects` (such as `logging`, `changing state` [...] etc.), those side-effects happen at the time of this linking [...] `rather than` `only when` the eventual `Bar()` (me: `instances`)("descendants") are created.

* It would be *nice* if there was a standard and reliable way to modify the linkage of an existing object. `Prior to ES6`, there's a `non-standard` and `not fully-cross-browser` way, via the `.__proto__` property, which is `settable`. ES6 adds a `Object.setPrototypeOf(..)` helper utility, which does the trick in a standard and predictable way.

* Compare the pre-ES6 and ES6-`standardized` techniques for linking `Bar.prototype` to `Foo.prototype`, side-by-side:

```js
// pre-ES6
// throws away default existing `Bar.prototype`
Bar.prototype = Object.create( Foo.prototype );

// ES6+
// modifies existing `Bar.prototype`
Object.setPrototypeOf( Bar.prototype, Foo.prototype );
```

### Inspecting "Class" Relationships
* `Inspecting` an instance [...] for its `inheritance ancestry` (`delegation linkage` in JS) is often called *introspection* (or *reflection*) in [...] class-oriented environments.
`inheritance ancestry` 
```js
function Foo() {
	// ...
}

Foo.prototype.blah = ...;

var a = new Foo();
```

* How do we then introspect `a` [...] The `first approach`.

```js
a instanceof Foo; // true
```

* The question `instanceof` answers is: **in the entire `[[Prototype]]` chain of `a`, `does` (me: `the` in the word `the object` `does not` refer to any of the `S`s (in English grammar) mentioned earlier in this argument, it is there before the word `object` ony because the word `object` is used inside a `passive clause`. Technically, the word `the object` here refers to an object in side the `[[prototype]]` chain which is being processed by the `instanceof` operator)(the object) arbitrarily pointed to by `Foo.prototype` `ever appear`?**

* Unfortunately, this means that you can `only` inquire about the "ancestry" of some object [...] if you have some **function** [...] with its attached `.prototype` reference (me: read the above argument to gain a better understanding of how the `instanceof` work and why the `.prototype` reference is emphasized here) [...] to test with. If you have two [...] objects, say `a` and `b`, and want to find out if *the objects* are related to each other through a `[[Prototype]]` chain, `instanceof` alone can't help.

* If you use the [...] `.bind(..)` [...] to make a hard-bound function [...] the function created `will not have` a `.prototype` property. Using `instanceof` with such a function `transparently` `substitutes` the `.prototype` of the *target function* that the hard-bound function `was created from`.

* This snippet illustrates the [...] trying to reason about relationships between **two objects** using [...] `instanceof`:

```js
// helper utility to see if `o1` is
// related to (delegates to) `o2`
function isRelatedTo(o1, o2) {
	function F(){}
	F.prototype = o2;
	return o1 instanceof F;
}

var a = {};
var b = Object.create( a );

isRelatedTo( b, a ); // true
```

* Inside `isRelatedTo(..)`, we borrow a `throw-away` function `F`, reassign its `.prototype` to arbitrarily point to some object `o2`, then ask if `o1` is an "instance of" `F` (me: revise the third argument of this section for a better understand of how `instanceof` works). Obviously `o1` isn't *actually* inherited or descended or even constructed from `F`, so it should be clear why this kind of exercise is [...] confusing. **The problem comes down to the `awkwardness` of class semantics `forced upon` JavaScript**

* The [...] much cleaner [...] approach.

```js
Foo.prototype.isPrototypeOf( a ); // true
```

* we don't really care about [...] `Foo`, we just need an **object** (in our case [...] `Foo.prototype`) to test against another **object**. The question `isPrototypeOf(..)` answers is: **in the entire `[[Prototype]]` chain of `a`, does `Foo.prototype` ever appear?**

* our `isRelatedTo(..)` utility above is built-in to the language, and it's called `isPrototypeOf(..)`.

* `directly` retrieve the `[[Prototype]]` of an object. As of ES5, the standard way to do this is:

```js
Object.getPrototypeOf( a );
```

* And you'll notice that object reference is what we'd expect:

```js
Object.getPrototypeOf( a ) === Foo.prototype; // true
```

* `Most` browsers (`not all`!) have also long supported a `non-standard alternate` way of accessing the internal `[[Prototype]]`:

```js
a.__proto__ === Foo.prototype; // true
```

* `.__proto__` (`not standardized` `until ES6`!) property [...] retrieves the internal `[[Prototype]]` of an object as a reference, which is quite helpful if you want to directly inspect (or even traverse: `.__proto__.__proto__...`) the chain.

* `.__proto__` doesn't actually exist on the object you're inspecting [...] In fact, it exists (`non-enumerable` [...]) on the built-in `Object.prototype`.

* `.__proto__` is also a `settable` property [...] However, generally you **should not change the `[[Prototype]]` of an existing object**.

## Object Links
* As we've now seen, the `[[Prototype]]` mechanism is an internal link that exists on one object which references some other object.

### `Create()`ing Links
```js
var foo = {
	something: function() {
		console.log( "Tell me something good..." );
	}
};

var bar = Object.create( foo );

bar.something(); // Tell me something good...
```

* `Object.create(..)` creates a new object (`bar`) linked to the object we specified (`foo`), which gives us all the power (`delegation`) of the `[[Prototype]]` mechanism, but `without` any of the unnecessary complication of `new` functions `acting as` classes and `constructor calls` [...]  or any of that extra stuff.

* `Object.create(null)` creates an object that has an `empty` (aka, `null`) `[[Prototype]]` `linkage` [...] thus the object `can't delegate anywhere`. Since such an object `has no prototype chain`, the `instanceof` operator (`explained earlier`) `has nothing to check`, so it will always return `false`. These special empty-`[[Prototype]]` objects are often called "`dictionaries`" as they are typically used `purely` for `storing data` in properties, mostly because they have `no possible surprise effects` from any `delegated properties/functions` on the `[[Prototype]]` chain.

#### `Object.create()` Polyfilled
### Links As Fallbacks?
* It may be tempting to think that these links between objects *primarily* provide a sort of `fallback` for `"missing" properties or methods`. While that may be an `observed outcome`, I `don't think` it represents the `right way` of thinking about `[[Prototype]]`.

```js
var anotherObject = {
	cool: function() {
		console.log( "cool!" );
	}
};

var myObject = Object.create( anotherObject );

myObject.cool(); // "cool!"
```

* That code will work `by virtue of` `[[Prototype]]`, but if you wrote it that way so that `anotherObject` was acting as a fallback **just in case** `myObject` couldn't handle some property/method [...] `odds are that` your software is going to be a bit more "`magical`" and harder to understand and maintain.

* In `ES6`, an `advanced functionality` called `Proxy` is introduced which can provide something of a "`method not found`" type of behavior.

* You can however design your API with `less "magic"` [...] but still take advantage of the power of `[[Prototype]]` linkage.

```js
var anotherObject = {
	cool: function() {
		console.log( "cool!" );
	}
};

var myObject = Object.create( anotherObject );

myObject.doCool = function() {
	this.cool(); // internal delegation!
};

myObject.doCool(); // "cool!"
```

* `delegation` will tend to be `less surprising/confusing` [...] We will expound on **delegation** in great detail in the next chapter.

## Review (TL;DR)

# Chapter 6: Behavior Delegation
## Towards Delegation-Oriented Design
### Class Theory
* Let's say we have several similar tasks ("XYZ", "ABC", etc) [...] in our software [...] With classes, the way you design the scenario is: define a general parent (base) class like `Task`, defining shared behavior for all the "alike" tasks. Then, you define child classes `XYZ` and `ABC`, both of which inherit from `Task`, and each of which adds specialized behavior to handle their respective tasks.

```js
class Task {
	id;

	// constructor `Task()`
	Task(ID) { id = ID; }
	outputTask() { output( id ); }
}

class XYZ inherits Task {
	label;

	// constructor `XYZ()`
	XYZ(ID,Label) { super( ID ); label = Label; }
	outputTask() { super(); output( label ); }
}

class ABC inherits Task {
	// ...
}
```

### Delegation Theory
* let's try to think about the `same problem` [...] using *behavior delegation* instead of *classes*.

* You will first define an **object** (`not a class`, `nor a function` [...]) called `Task`, and it will have concrete behavior on it that includes utility methods that various tasks can use (read: *delegate to*!). Then, for each task ("XYZ", "ABC"), you define an **object** to hold that task-specific data/behavior. You **link** your task-specific object(s) to the `Task` utility object, allowing them to `delegate to` it when they need to.

```js
var Task = {
	setID: function(ID) { this.id = ID; },
	outputID: function() { console.log( this.id ); }
};

// make `XYZ` delegate to `Task`
var XYZ = Object.create( Task );

XYZ.prepareTask = function(ID,Label) {
	this.setID( ID );
	this.label = Label;
};

XYZ.outputTaskDetails = function() {
	this.outputID();
	console.log( this.label );
};

// ABC = Object.create( Task );
// ABC ... = ...
```

* `Task` and `XYZ` are not classes (or functions), they're **just objects**. `XYZ` is set up via `Object.create(..)` to `[[Prototype]]` `delegate to` the `Task` object.

* As compared to class-orientation (aka, OO -- object-oriented), I call this style of code **"OLOO"** (objects-linked-to-other-objects). All we *really* care about is that the `XYZ` object delegates to the `Task` object (as does the `ABC` object).

* Some other differences to note with **OLOO style code**:
  1. Both `id` and `label` data members from the [...] example are data properties `directly` on `XYZ` (`neither` is on `Task`). In general, with `[[Prototype]]` delegation involved, **you want state to be on the `delegators`** (`XYZ`, `ABC`), not on the `delegate` (`Task`).
 
  2. With the `class design pattern`, we `intentionally` named `outputTask` the same on both parent (`Task`) and child (`XYZ`), so that we could take advantage of `overriding (polymorphism)`. In `behavior delegation` [...] **we avoid if at all possible naming things the same** at different levels of the `[[Prototype]]` chain (called `shadowing` [...]), because having those name `collisions` creates awkward/brittle syntax to `disambiguate` references (see `Chapter 4`).

  3. `this.setID(ID);` inside of a method on the `XYZ` object first looks on `XYZ` for `setID(..)`, but since it `doesn't find` a method of that name on `XYZ`, `[[Prototype]]` *delegation* means it can follow the link to `Task` to look for `setID(..)`, which it of course finds. `Moreover`, because of `implicit call-site` `this` `binding rules` (see `Chapter 2`), when `setID(..)` runs [...] the `this` binding for that function call is `XYZ`.

* **Behavior Delegation** means: let some object (`XYZ`) provide a delegation (to `Task`) for property or method references if not found on the object (`XYZ`).

* This [...] (me: `Delegation`)(design pattern) [...] Rather than organizing the objects in your mind `vertically`, with Parents flowing down to Children, think of objects `side-by-side` [...] with [...] direction of delegation links between the objects as necessary.

#### Mutual Delegation (Disallowed)
* You `cannot` create a *cycle* where `two or more` objects are `mutually delegated` (bi-directionally) to each other. If you make `B` linked to `A`, and then try to link `A` to `B`, you will get an error.

* If you made a reference to a property/method which didn't exist in `either` place, you'd have an `infinite recursion` on the `[[Prototype]]` loop.

#### Debugged
* Consider this traditional "class constructor" style JS code, as it would appear in the *console* of `Chrome Developer Tools`:

```js
function Foo() {}

var a1 = new Foo();

a1; // Foo {}
```

* the last line of that snippet: the output of evaluating the `a1` expression, which prints `Foo {}`. If you try this same code in Firefox, you will likely see `Object {}`. Why the difference? What do these outputs mean?

* Chrome is [...] saying "{} is an empty object [...] constructed by a function with name 'Foo'". Firefox is saying "{} is an empty object of general construction from Object".

* is [...] Chrome [...] outputting "Foo", by simply examining the object's `.constructor.name`? [...] the answer is both "yes" and "no".

```js
function Foo() {}

var a1 = new Foo();

Foo.prototype.constructor = function Gotcha(){};

a1.constructor; // Gotcha(){}
a1.constructor.name; // "Gotcha"

a1; // Foo {}
```

* Even though we change `a1.constructor.name` to [...] something else ("Gotcha"), Chrome's console still uses the "Foo" name [...] So, it `would appear` the answer to previous question (does it use `.constructor.name`?) is **no** [...] But:

```js
var Foo = {};

var a1 = Object.create( Foo );

a1; // Object {}

Object.defineProperty( Foo, "constructor", {
	enumerable: false,
	value: function Gotcha(){}
});

a1; // Gotcha {}
```

* Ah-ha! **Gotcha!** Here, Chrome's console **did** find and use the `.constructor.name`. Actually [...] this exact behavior `was` identified as a bug in Chrome, and `by the time` you're reading this, it `may` have already been fixed. So you `may` instead have seen `the corrected` `a1; // Object {}`.

* Aside from that bug, the `internal tracking` (apparently `only` `for debug output purposes`) of the "`constructor name`" that Chrome does (shown in the `earlier snippets`) is an `intentional` `Chrome-only` `extension of behavior` `beyond` what the `JS specification` calls for.

* If you don't use a "`constructor`" to `make your objects` [...] you'll get objects that `Chrome` `does not` track an `internal "constructor name"` for, and such objects will `correctly` only be outputted as "`Object {}`", meaning "`object generated from Object() construction`".

* this represents a `drawback` of OLOO-style coding. When you code with OLOO and `behavior delegation` as your `design pattern`, *who* "constructed" (that is, *which function* was called with `new`?) some object is an `irrelevant` detail. `Chrome's specific internal "constructor name" tracking` is really `only` useful if you're `fully embracing` "`class-style`" coding.

### Mental Models Compared
* We'll examine some more theoretical [...] code, and compare [...] OO vs. OLOO [...] The first snippet uses the classical ("prototypal") OO style:

```js
function Foo(who) {
	this.me = who;
}
Foo.prototype.identify = function() {
	return "I am " + this.me;
};

function Bar(who) {
	Foo.call( this, who );
}
Bar.prototype = Object.create( Foo.prototype );

Bar.prototype.speak = function() {
	alert( "Hello, " + this.identify() + "." );
};

var b1 = new Bar( "b1" );
var b2 = new Bar( "b2" );

b1.speak();
b2.speak();
```

* let's implement **the exact same functionality** using *OLOO* style code:

```js
var Foo = {
	init: function(who) {
		this.me = who;
	},
	identify: function() {
		return "I am " + this.me;
	}
};

var Bar = Object.create( Foo );

Bar.speak = function() {
	alert( "Hello, " + this.identify() + "." );
};

var b1 = Object.create( Bar );
b1.init( "b1" );
var b2 = Object.create( Bar );
b2.init( "b2" );

b1.speak();
b2.speak();
```

* now we just set up **objects** linked to each other, without needing [...] things that look (but don't behave!) like classes, with constructors and prototypes and `new` calls.

* (WCRL)(Ask yourself: if I can get the same functionality with OLOO style code as I do with "class" style code, but OLOO is simpler and has less things to think about, **isn't OLOO better**?)

## Classes vs. Objects
* We'll first examine a typical scenario in `front-end web dev`: creating UI widgets (buttons, drop-downs, etc).

### Widget "Classes"
* Because you're probably still `so used to` the OO design pattern, you'll likely immediately think of this problem domain in terms of a parent class (perhaps called `Widget`) with all the `common` base widget behavior, and then child derived classes for specific widget types (like `Button`).

* We're going to use `jQuery` here for DOM and CSS manipulation [...] None of this code cares which JS framework [...] you might solve such mundane tasks with.

```js
// Parent class
function Widget(width,height) {
	this.width = width || 50;
	this.height = height || 50;
	this.$elem = null;
}

Widget.prototype.render = function($where){
	if (this.$elem) {
		this.$elem.css( {
			width: this.width + "px",
			height: this.height + "px"
		} ).appendTo( $where );
	}
};

// Child class
function Button(width,height,label) {
	// "super" constructor call
	Widget.call( this, width, height );
	this.label = label || "Default";

	this.$elem = $( "<button>" ).text( this.label );
}

// make `Button` "inherit" from `Widget`
Button.prototype = Object.create( Widget.prototype );

// override base "inherited" `render(..)`
Button.prototype.render = function($where) {
	// "super" call
	Widget.prototype.render.call( this, $where );
	this.$elem.click( this.onClick.bind( this ) );
};

Button.prototype.onClick = function(evt) {
	console.log( "Button '" + this.label + "' clicked!" );
};

$( document ).ready( function(){
	var $body = $( document.body );
	var btn1 = new Button( 125, 30, "Hello" );
	var btn2 = new Button( 150, 40, "World" );

	btn1.render( $body );
	btn2.render( $body );
} );
```

* Notice the `ugliness` of *explicit pseudo-polymorphism* (see Chapter 4) with `Widget.call` and `Widget.prototype.render.call` references for `faking` "super" calls.

#### ES6 `class` sugar
* We cover ES6 `class` syntax sugar in detail in `Appendix A`, but let's briefly demonstrate how we'd implement the same code using `class`:

```js
class Widget {
	constructor(width,height) {
		this.width = width || 50;
		this.height = height || 50;
		this.$elem = null;
	}
	render($where){
		if (this.$elem) {
			this.$elem.css( {
				width: this.width + "px",
				height: this.height + "px"
			} ).appendTo( $where );
		}
	}
}

class Button extends Widget {
	constructor(width,height,label) {
		super( width, height );
		this.label = label || "Default";
		this.$elem = $( "<button>" ).text( this.label );
	}
	render($where) {
		super.render( $where );
		this.$elem.click( this.onClick.bind( this ) );
	}
	onClick(evt) {
		console.log( "Button '" + this.label + "' clicked!" );
	}
}

$( document ).ready( function(){
	var $body = $( document.body );
	var btn1 = new Button( 125, 30, "Hello" );
	var btn2 = new Button( 150, 40, "World" );

	btn1.render( $body );
	btn2.render( $body );
} );
```

* a number of the syntax uglies of the previous classical approach have been smoothed over with ES6's `class`.

* **these are not *real* classes**, as they still operate on top of the `[[Prototype]]` mechanism [...] Appendix A will expound on the ES6 `class` syntax and its implications in detail. We'll see why solving syntax hiccups doesn't substantially solve our class confusions in JS.

### Delegating Widget Objects
* Here's our simpler `Widget` / `Button` example, using **OLOO style delegation**:

```js
var Widget = {
	init: function(width,height){
		this.width = width || 50;
		this.height = height || 50;
		this.$elem = null;
	},
	insert: function($where){
		if (this.$elem) {
			this.$elem.css( {
				width: this.width + "px",
				height: this.height + "px"
			} ).appendTo( $where );
		}
	}
};

var Button = Object.create( Widget );

Button.setup = function(width,height,label){
	// delegated call
	this.init( width, height );
	this.label = label || "Default";

	this.$elem = $( "<button>" ).text( this.label );
};
Button.build = function($where) {
	// delegated call
	this.insert( $where );
	this.$elem.click( this.onClick.bind( this ) );
};
Button.onClick = function(evt) {
	console.log( "Button '" + this.label + "' clicked!" );
};

$( document ).ready( function(){
	var $body = $( document.body );

	var btn1 = Object.create( Button );
	btn1.setup( 125, 30, "Hello" );

	var btn2 = Object.create( Button );
	btn2.setup( 150, 40, "World" );

	btn1.build( $body );
	btn2.build( $body );
} );
```

* With this OLOO-style approach, we don't think of `Widget` as a parent and `Button` as a child. Rather, `Widget` **is just an object** [...] a utility collection that any specific type of widget might want to `delegate to`, and `Button` **is also just a stand-alone object** (with a delegation link to `Widget`).

* we chose different names (`insert(..)` and `build(..)`) that were more descriptive of what task each does specifically. The *initialization* methods are called `init(..)` and `setup(..)`, respectively, for the same reasons [...] (me: by) doing so [...] OLOO happens to avoid the ugliness of the explicit `pseudo-polymorphic calls` (`Widget.call` and `Widget.prototype.render.call`) [...] (me: instead uses) delegated calls to `this.init(..)` and `this.insert(..)` (me: duplicating `delegatable function`'s name is OK. However, since the `delegatable function`'s name is duplicated with `its caller function`'s name, we can not delegate to the `delegatable function` by simply calling `this.delegatableFunction`, but will have to instead use `pseudo-polymorphic calls`).

## Simpler Design
* The scenario we'll examine is two `controller objects`, one for handling the login form of a web page, and another for actually handling the authentication (communication) with the server [...] We'll need a `utility helper` for `making the Ajax communication`.

* We don't cover Promises here.

* Following the typical class design pattern.

```js
// Parent class
function Controller() {
	this.errors = [];
}
Controller.prototype.showDialog = function(title,msg) {
	// display title & message to user in dialog
};
Controller.prototype.success = function(msg) {
	this.showDialog( "Success", msg );
};
Controller.prototype.failure = function(err) {
	this.errors.push( err );
	this.showDialog( "Error", err );
};
```

```js
// Child class
function LoginController() {
	Controller.call( this );
}
// Link child class to parent
LoginController.prototype = Object.create( Controller.prototype );
LoginController.prototype.getUser = function() {
	return document.getElementById( "login_username" ).value;
};
LoginController.prototype.getPassword = function() {
	return document.getElementById( "login_password" ).value;
};
LoginController.prototype.validateEntry = function(user,pw) {
	user = user || this.getUser();
	pw = pw || this.getPassword();

	if (!(user && pw)) {
		return this.failure( "Please enter a username & password!" );
	}
	else if (pw.length < 5) {
		return this.failure( "Password must be 5+ characters!" );
	}

	// got here? validated!
	return true;
};
// Override to extend base `failure()`
LoginController.prototype.failure = function(err) {
	// "super" call
	Controller.prototype.failure.call( this, "Login invalid: " + err );
};
```

```js
// Child class
function AuthController(login) {
	Controller.call( this );
	// in addition to inheritance, we also need composition
	this.login = login;
}
// Link child class to parent
AuthController.prototype = Object.create( Controller.prototype );
AuthController.prototype.server = function(url,data) {
	return $.ajax( {
		url: url,
		data: data
	} );
};
AuthController.prototype.checkAuth = function() {
	var user = this.login.getUser();
	var pw = this.login.getPassword();

	if (this.login.validateEntry( user, pw )) {
		this.server( "/check-auth",{
			user: user,
			pw: pw
		} )
		.then( this.success.bind( this ) )
		.fail( this.failure.bind( this ) );
	}
};
// Override to extend base `success()`
AuthController.prototype.success = function() {
	// "super" call
	Controller.prototype.success.call( this, "Authenticated!" );
};
// Override to extend base `failure()`
AuthController.prototype.failure = function(err) {
	// "super" call
	Controller.prototype.failure.call( this, "Auth Failed: " + err );
};
```

```js
var auth = new AuthController(
	// in addition to inheritance, we also need composition
	new LoginController()
);
auth.checkAuth();
```

* note that `AuthController` needs an instance of `LoginController` to interact with the login form, so that becomes a member data property.

* There *might* have been a [...] temptation to make `AuthController` inherit from `LoginController`, or vice versa [...] But this is a strongly clear example of what's wrong with class inheritance as *the* model for the problem domain, because neither `AuthController` nor `LoginController` are specializing base behavior of the other, so inheritance between them makes little sense except if classes are your only design pattern.

### De-class-ified
* OLOO-style behavior delegation:

```js
var LoginController = {
	errors: [],
	getUser: function() {
		return document.getElementById( "login_username" ).value;
	},
	getPassword: function() {
		return document.getElementById( "login_password" ).value;
	},
	validateEntry: function(user,pw) {
		user = user || this.getUser();
		pw = pw || this.getPassword();

		if (!(user && pw)) {
			return this.failure( "Please enter a username & password!" );
		}
		else if (pw.length < 5) {
			return this.failure( "Password must be 5+ characters!" );
		}

		// got here? validated!
		return true;
	},
	showDialog: function(title,msg) {
		// display success message to user in dialog
	},
	failure: function(err) {
		this.errors.push( err );
		this.showDialog( "Error", "Login invalid: " + err );
	}
};
```

```js
// Link `AuthController` to delegate to `LoginController`
var AuthController = Object.create( LoginController );

AuthController.errors = [];
AuthController.checkAuth = function() {
	var user = this.getUser();
	var pw = this.getPassword();

	if (this.validateEntry( user, pw )) {
		this.server( "/check-auth",{
			user: user,
			pw: pw
		} )
		.then( this.accepted.bind( this ) )
		.fail( this.rejected.bind( this ) );
	}
};
AuthController.server = function(url,data) {
	return $.ajax( {
		url: url,
		data: data
	} );
};
AuthController.accepted = function() {
	this.showDialog( "Success", "Authenticated!" )
};
AuthController.rejected = function(err) {
	this.failure( "Auth Failed: " + err );
};
```

* to perform our task. All we need to do is:

```js
AuthController.checkAuth();
```

* We somewhat arbitrarily chose to have `AuthController` delegate to `LoginController` -- it [...] (me: could) have been [...] (me: in) the reverse direction.

* We didn't need a base `Controller` class to "share" behavior between the two, because delegation [...] give us the functionality we need. We also [...] don't need to instantiate our classes to work with them [...] there's no need for *composition* as delegation gives the two objects the ability to cooperate *differentially* as needed [...] we avoided the `polymorphism pitfalls` of class-oriented design by not having the names `success(..)` and `failure(..)` be the same on both objects.

## Nicer Syntax
One of the nicer things that makes ES6's `class` so deceptively attractive (see Appendix A on why to avoid it!) is the short-hand syntax for declaring class methods:

```js
class Foo {
	methodName() { /* .. */ }
}
```

We get to drop the word `function` from the declaration, which makes JS developers everywhere cheer!

And you may have noticed and been frustrated that the suggested OLOO syntax above has lots of `function` appearances, which seems like a bit of a detractor to the goal of OLOO simplification. **But it doesn't have to be that way!**

As of ES6, we can use *concise method declarations* in any object literal, so an object in OLOO style can be declared this way (same short-hand sugar as with `class` body syntax):

```js
var LoginController = {
	errors: [],
	getUser() { // Look ma, no `function`!
		// ...
	},
	getPassword() {
		// ...
	}
	// ...
};
```

About the only difference is that object literals will still require `,` comma separators between elements whereas `class` syntax doesn't. Pretty minor concession in the whole scheme of things.

Moreover, as of ES6, the clunkier syntax you use (like for the `AuthController` definition), where you're assigning properties individually and not using an object literal, can be re-written using an object literal (so that you can use concise methods), and you can just modify that object's `[[Prototype]]` with `Object.setPrototypeOf(..)`, like this:

```js
// use nicer object literal syntax w/ concise methods!
var AuthController = {
	errors: [],
	checkAuth() {
		// ...
	},
	server(url,data) {
		// ...
	}
	// ...
};

// NOW, link `AuthController` to delegate to `LoginController`
Object.setPrototypeOf( AuthController, LoginController );
```

OLOO-style as of ES6, with concise methods, **is a lot friendlier** than it was before (and even then, it was much simpler and nicer than classical prototype-style code). **You don't have to opt for class** (complexity) to get nice clean object syntax!

### Unlexical

There *is* one drawback to concise methods that's subtle but important to note. Consider this code:

```js
var Foo = {
	bar() { /*..*/ },
	baz: function baz() { /*..*/ }
};
```

Here's the syntactic de-sugaring that expresses how that code will operate:

```js
var Foo = {
	bar: function() { /*..*/ },
	baz: function baz() { /*..*/ }
};
```

See the difference? The `bar()` short-hand became an *anonymous function expression* (`function()..`) attached to the `bar` property, because the function object itself has no name identifier. Compare that to the manually specified *named function expression* (`function baz()..`) which has a lexical name identifier `baz` in addition to being attached to a `.baz` property.

So what? In the *"Scope & Closures"* title of this *"You Don't Know JS"* book series, we cover the three main downsides of *anonymous function expressions* in detail. We'll just briefly repeat them so we can compare to the concise method short-hand.

Lack of a `name` identifier on an anonymous function:

1. makes debugging stack traces harder
2. makes self-referencing (recursion, event (un)binding, etc) harder
3. makes code (a little bit) harder to understand

Items 1 and 3 don't apply to concise methods.

Even though the de-sugaring uses an *anonymous function expression* which normally would have no `name` in stack traces, concise methods are specified to set the internal `name` property of the function object accordingly, so stack traces should be able to use it (though that's implementation dependent so not guaranteed).

Item 2 is, unfortunately, **still a drawback to concise methods**. They will not have a lexical identifier to use as a self-reference. Consider:

```js
var Foo = {
	bar: function(x) {
		if (x < 10) {
			return Foo.bar( x * 2 );
		}
		return x;
	},
	baz: function baz(x) {
		if (x < 10) {
			return baz( x * 2 );
		}
		return x;
	}
};
```

The manual `Foo.bar(x*2)` reference above kind of suffices in this example, but there are many cases where a function wouldn't necessarily be able to do that, such as cases where the function is being shared in delegation across different objects, using `this` binding, etc. You would want to use a real self-reference, and the function object's `name` identifier is the best way to accomplish that.

Just be aware of this caveat for concise methods, and if you run into such issues with lack of self-reference, make sure to forgo the concise method syntax **just for that declaration** in favor of the manual *named function expression* declaration form: `baz: function baz(){..}`.

## Introspection

If you've spent much time with class oriented programming (either in JS or other languages), you're probably familiar with *type introspection*: inspecting an instance to find out what *kind* of object it is. The primary goal of *type introspection* with class instances is to reason about the structure/capabilities of the object based on *how it was created*.

Consider this code which uses `instanceof` (see Chapter 5) for introspecting on an object `a1` to infer its capability:

```js
function Foo() {
	// ...
}
Foo.prototype.something = function(){
	// ...
}

var a1 = new Foo();

// later

if (a1 instanceof Foo) {
	a1.something();
}
```

Because `Foo.prototype` (not `Foo`!) is in the `[[Prototype]]` chain (see Chapter 5) of `a1`, the `instanceof` operator (confusingly) pretends to tell us that `a1` is an instance of the `Foo` "class". With this knowledge, we then assume that `a1` has the capabilities described by the `Foo` "class".

Of course, there is no `Foo` class, only a plain old normal function `Foo`, which happens to have a reference to an arbitrary object (`Foo.prototype`) that `a1` happens to be delegation-linked to. By its syntax, `instanceof` pretends to be inspecting the relationship between `a1` and `Foo`, but it's actually telling us whether `a1` and (the arbitrary object referenced by) `Foo.prototype` are related.

The semantic confusion (and indirection) of `instanceof` syntax means that to use `instanceof`-based introspection to ask if object `a1` is related to the capabilities object in question, you *have to* have a function that holds a reference to that object -- you can't just directly ask if the two objects are related.

Recall the abstract `Foo` / `Bar` / `b1` example from earlier in this chapter, which we'll abbreviate here:

```js
function Foo() { /* .. */ }
Foo.prototype...

function Bar() { /* .. */ }
Bar.prototype = Object.create( Foo.prototype );

var b1 = new Bar( "b1" );
```

For *type introspection* purposes on the entities in that example, using `instanceof` and `.prototype` semantics, here are the various checks you might need to perform:

```js
// relating `Foo` and `Bar` to each other
Bar.prototype instanceof Foo; // true
Object.getPrototypeOf( Bar.prototype ) === Foo.prototype; // true
Foo.prototype.isPrototypeOf( Bar.prototype ); // true

// relating `b1` to both `Foo` and `Bar`
b1 instanceof Foo; // true
b1 instanceof Bar; // true
Object.getPrototypeOf( b1 ) === Bar.prototype; // true
Foo.prototype.isPrototypeOf( b1 ); // true
Bar.prototype.isPrototypeOf( b1 ); // true
```

It's fair to say that some of that kinda sucks. For instance, intuitively (with classes) you might want to be able to say something like `Bar instanceof Foo` (because it's easy to mix up what "instance" means to think it includes "inheritance"), but that's not a sensible comparison in JS. You have to do `Bar.prototype instanceof Foo` instead.

Another common, but perhaps less robust, pattern for *type introspection*, which many devs seem to prefer over `instanceof`, is called "duck typing". This term comes from the adage, "if it looks like a duck, and it quacks like a duck, it must be a duck".

Example:

```js
if (a1.something) {
	a1.something();
}
```

Rather than inspecting for a relationship between `a1` and an object that holds the delegatable `something()` function, we assume that the test for `a1.something` passing means `a1` has the capability to call `.something()` (regardless of if it found the method directly on `a1` or delegated to some other object). In and of itself, that assumption isn't so risky.

But "duck typing" is often extended to make **other assumptions about the object's capabilities** besides what's being tested, which of course introduces more risk (aka, brittle design) into the test.

One notable example of "duck typing" comes with ES6 Promises (which as an earlier note explained are not being covered in this book).

For various reasons, there's a need to determine if any arbitrary object reference *is a Promise*, but the way that test is done is to check if the object happens to have a `then()` function present on it. In other words, **if any object** happens to have a `then()` method, ES6 Promises will assume unconditionally that the object **is a "thenable"** and therefore will expect it to behave conformantly to all standard behaviors of Promises.

If you have any non-Promise object that happens for whatever reason to have a `then()` method on it, you are strongly advised to keep it far away from the ES6 Promise mechanism to avoid broken assumptions.

That example clearly illustrates the perils of "duck typing". You should only use such approaches sparingly and in controlled conditions.

Turning our attention once again back to OLOO-style code as presented here in this chapter, *type introspection* turns out to be much cleaner. Let's recall (and abbreviate) the `Foo` / `Bar` / `b1` OLOO example from earlier in the chapter:

```js
var Foo = { /* .. */ };

var Bar = Object.create( Foo );
Bar...

var b1 = Object.create( Bar );
```

Using this OLOO approach, where all we have are plain objects that are related via `[[Prototype]]` delegation, here's the quite simplified *type introspection* we might use:

```js
// relating `Foo` and `Bar` to each other
Foo.isPrototypeOf( Bar ); // true
Object.getPrototypeOf( Bar ) === Foo; // true

// relating `b1` to both `Foo` and `Bar`
Foo.isPrototypeOf( b1 ); // true
Bar.isPrototypeOf( b1 ); // true
Object.getPrototypeOf( b1 ) === Bar; // true
```

We're not using `instanceof` anymore, because it's confusingly pretending to have something to do with classes. Now, we just ask the (informally stated) question, "are you *a* prototype of me?" There's no more indirection necessary with stuff like `Foo.prototype` or the painfully verbose `Foo.prototype.isPrototypeOf(..)`.

I think it's fair to say these checks are significantly less complicated/confusing than the previous set of introspection checks. **Yet again, we see that OLOO is simpler than (but with all the same power of) class-style coding in JavaScript.**

## Review (TL;DR)
