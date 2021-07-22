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

Consider:

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
Objects in JavaScript have an internal property, denoted in the specification as `[[Prototype]]`, which is simply a reference to another object. Almost all objects are given a non-`null` value for this property, at the time of their creation.

**Note:** We will see shortly that it *is* possible for an object to have an empty `[[Prototype]]` linkage, though this is somewhat less common.

Consider:

```js
var myObject = {
	a: 2
};

myObject.a; // 2
```

What is the `[[Prototype]]` reference used for? In Chapter 3, we examined the `[[Get]]` operation that is invoked when you reference a property on an object, such as `myObject.a`. For that default `[[Get]]` operation, the first step is to check if the object itself has a property `a` on it, and if so, it's used.

**Note:** ES6 Proxies are outside of our discussion scope in this book (will be covered in a later book in the series!), but everything we discuss here about normal `[[Get]]` and `[[Put]]` behavior does not apply if a `Proxy` is involved.

But it's what happens if `a` **isn't** present on `myObject` that brings our attention now to the `[[Prototype]]` link of the object.

The default `[[Get]]` operation proceeds to follow the `[[Prototype]]` **link** of the object if it cannot find the requested property on the object directly.

```js
var anotherObject = {
	a: 2
};

// create an object linked to `anotherObject`
var myObject = Object.create( anotherObject );

myObject.a; // 2
```

**Note:** We will explain what `Object.create(..)` does, and how it operates, shortly. For now, just assume it creates an object with the `[[Prototype]]` linkage we're examining to the object specified.

So, we have `myObject` that is now `[[Prototype]]` linked to `anotherObject`. Clearly `myObject.a` doesn't actually exist, but nevertheless, the property access succeeds (being found on `anotherObject` instead) and indeed finds the value `2`.

But, if `a` weren't found on `anotherObject` either, its `[[Prototype]]` chain, if non-empty, is again consulted and followed.

This process continues until either a matching property name is found, or the `[[Prototype]]` chain ends. If no matching property is *ever* found by the end of the chain, the return result from the `[[Get]]` operation is `undefined`.

Similar to this `[[Prototype]]` chain look-up process, if you use a `for..in` loop to iterate over an object, any property that can be reached via its chain (and is also `enumerable` -- see Chapter 3) will be enumerated. If you use the `in` operator to test for the existence of a property on an object, `in` will check the entire chain of the object (regardless of *enumerability*).

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

So, the `[[Prototype]]` chain is consulted, one link at a time, when you perform property look-ups in various fashions. The look-up stops once the property is found or the chain ends.

### `Object.prototype`

But *where* exactly does the `[[Prototype]]` chain "end"?

The top-end of every *normal* `[[Prototype]]` chain is the built-in `Object.prototype`. This object includes a variety of common utilities used all over JS, because all normal (built-in, not host-specific extension) objects in JavaScript "descend from" (aka, have at the top of their `[[Prototype]]` chain) the `Object.prototype` object.

Some utilities found here you may be familiar with include `.toString()` and `.valueOf()`. In Chapter 3, we introduced another: `.hasOwnProperty(..)`. And yet another function on `Object.prototype` you may not be familiar with, but which we'll address later in this chapter, is `.isPrototypeOf(..)`.

### Setting & Shadowing Properties

Back in Chapter 3, we mentioned that setting properties on an object was more nuanced than just adding a new property to the object or changing an existing property's value. We will now revisit this situation more completely.

```js
myObject.foo = "bar";
```

If the `myObject` object already has a normal data accessor property called `foo` directly present on it, the assignment is as simple as changing the value of the existing property.

If `foo` is not already present directly on `myObject`, the `[[Prototype]]` chain is traversed, just like for the `[[Get]]` operation. If `foo` is not found anywhere in the chain, the property `foo` is added directly to `myObject` with the specified value, as expected.

However, if `foo` is already present somewhere higher in the chain, nuanced (and perhaps surprising) behavior can occur with the `myObject.foo = "bar"` assignment. We'll examine that more in just a moment.

If the property name `foo` ends up both on `myObject` itself and at a higher level of the `[[Prototype]]` chain that starts at `myObject`, this is called *shadowing*. The `foo` property directly on `myObject` *shadows* any `foo` property which appears higher in the chain, because the `myObject.foo` look-up would always find the `foo` property that's lowest in the chain.

As we just hinted, shadowing `foo` on `myObject` is not as simple as it may seem. We will now examine three scenarios for the `myObject.foo = "bar"` assignment when `foo` is **not** already on `myObject` directly, but **is** at a higher level of `myObject`'s `[[Prototype]]` chain:

1. If a normal data accessor (see Chapter 3) property named `foo` is found anywhere higher on the `[[Prototype]]` chain, **and it's not marked as read-only (`writable:false`)** then a new property called `foo` is added directly to `myObject`, resulting in a **shadowed property**.
2. If a `foo` is found higher on the `[[Prototype]]` chain, but it's marked as **read-only (`writable:false`)**, then both the setting of that existing property as well as the creation of the shadowed property on `myObject` **are disallowed**. If the code is running in `strict mode`, an error will be thrown. Otherwise, the setting of the property value will silently be ignored. Either way, **no shadowing occurs**.
3. If a `foo` is found higher on the `[[Prototype]]` chain and it's a setter (see Chapter 3), then the setter will always be called. No `foo` will be added to (aka, shadowed on) `myObject`, nor will the `foo` setter be redefined.

Most developers assume that assignment of a property (`[[Put]]`) will always result in shadowing if the property already exists higher on the `[[Prototype]]` chain, but as you can see, that's only true in one (#1) of the three situations just described.

If you want to shadow `foo` in cases #2 and #3, you cannot use `=` assignment, but must instead use `Object.defineProperty(..)` (see Chapter 3) to add `foo` to `myObject`.

**Note:** Case #2 may be the most surprising of the three. The presence of a *read-only* property prevents a property of the same name being implicitly created (shadowed) at a lower level of a `[[Prototype]]` chain. The reason for this restriction is primarily to reinforce the illusion of class-inherited properties. If you think of the `foo` at a higher level of the chain as having been inherited (copied down) to `myObject`, then it makes sense to enforce the non-writable nature of that `foo` property on `myObject`. If you however separate the illusion from the fact, and recognize that no such inheritance copying *actually* occurred (see Chapters 4 and 5), it's a little unnatural that `myObject` would be prevented from having a `foo` property just because some other object had a non-writable `foo` on it. It's even stranger that this restriction only applies to `=` assignment, but is not enforced when using `Object.defineProperty(..)`.

Shadowing with **methods** leads to ugly *explicit pseudo-polymorphism* (see Chapter 4) if you need to delegate between them. Usually, shadowing is more complicated and nuanced than it's worth, **so you should try to avoid it if possible**. See Chapter 6 for an alternative design pattern, which among other things discourages shadowing in favor of cleaner alternatives.

Shadowing can even occur implicitly in subtle ways, so care must be taken if trying to avoid it. Consider:

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

Though it may appear that `myObject.a++` should (via delegation) look-up and just increment the `anotherObject.a` property itself *in place*, instead the `++` operation corresponds to `myObject.a = myObject.a + 1`. The result is `[[Get]]` looking up `a` property via `[[Prototype]]` to get the current value `2` from `anotherObject.a`, incrementing the value by one, then `[[Put]]` assigning the `3` value to a new shadowed property `a` on `myObject`. Oops!

Be very careful when dealing with delegated properties that you modify. If you wanted to increment `anotherObject.a`, the only proper way is `anotherObject.a++`.

## "Class"

At this point, you might be wondering: "*Why* does one object need to link to another object?" What's the real benefit? That is a very appropriate question to ask, but we must first understand what `[[Prototype]]` is **not** before we can fully understand and appreciate what it *is* and how it's useful.

As we explained in Chapter 4, in JavaScript, there are no abstract patterns/blueprints for objects called "classes" as there are in class-oriented languages. JavaScript **just** has objects.

In fact, JavaScript is **almost unique** among languages as perhaps the only language with the right to use the label "object oriented", because it's one of a very short list of languages where an object can be created directly, without a class at all.

In JavaScript, classes can't (being that they don't exist!) describe what an object can do. The object defines its own behavior directly. **There's *just* the object.**

### "Class" Functions

There's a peculiar kind of behavior in JavaScript that has been shamelessly abused for years to *hack* something that *looks* like "classes". We'll examine this approach in detail.

The peculiar "sort-of class" behavior hinges on a strange characteristic of functions: all functions by default get a public, non-enumerable (see Chapter 3) property on them called `prototype`, which points at an otherwise arbitrary object.

```js
function Foo() {
	// ...
}

Foo.prototype; // { }
```

This object is often called "Foo's prototype", because we access it via an unfortunately-named `Foo.prototype` property reference. However, that terminology is hopelessly destined to lead us into confusion, as we'll see shortly. Instead, I will call it "the object formerly known as Foo's prototype". Just kidding. How about: "object arbitrarily labeled 'Foo dot prototype'"?

Whatever we call it, what exactly is this object?

The most direct way to explain it is that each object created from calling `new Foo()` (see Chapter 2) will end up (somewhat arbitrarily) `[[Prototype]]`-linked to this "Foo dot prototype" object.

Let's illustrate:

```js
function Foo() {
	// ...
}

var a = new Foo();

Object.getPrototypeOf( a ) === Foo.prototype; // true
```

When `a` is created by calling `new Foo()`, one of the things (see Chapter 2 for all *four* steps) that happens is that `a` gets an internal `[[Prototype]]` link to the object that `Foo.prototype` is pointing at.

Stop for a moment and ponder the implications of that statement.

In class-oriented languages, multiple **copies** (aka, "instances") of a class can be made, like stamping something out from a mold. As we saw in Chapter 4, this happens because the process of instantiating (or inheriting from) a class means, "copy the behavior plan from that class into a physical object", and this is done again for each new instance.

But in JavaScript, there are no such copy-actions performed. You don't create multiple instances of a class. You can create multiple objects that `[[Prototype]]` *link* to a common object. But by default, no copying occurs, and thus these objects don't end up totally separate and disconnected from each other, but rather, quite ***linked***.

`new Foo()` results in a new object (we called it `a`), and **that** new object `a` is internally `[[Prototype]]` linked to the `Foo.prototype` object.

**We end up with two objects, linked to each other.** That's *it*. We didn't instantiate a class. We certainly didn't do any copying of behavior from a "class" into a concrete object. We just caused two objects to be linked to each other.

In fact, the secret, which eludes most JS developers, is that the `new Foo()` function calling had really almost nothing *direct* to do with the process of creating the link. **It was sort of an accidental side-effect.** `new Foo()` is an indirect, round-about way to end up with what we want: **a new object linked to another object**.

Can we get what we want in a more *direct* way? **Yes!** The hero is `Object.create(..)`. But we'll get to that in a little bit.

#### What's in a name?

In JavaScript, we don't make *copies* from one object ("class") to another ("instance"). We make *links* between objects. For the `[[Prototype]]` mechanism, visually, the arrows move from right to left, and from bottom to top.

<img src="fig3.png">

This mechanism is often called "prototypal inheritance" (we'll explore the code in detail shortly), which is commonly said to be the dynamic-language version of "classical inheritance". It's an attempt to piggy-back on the common understanding of what "inheritance" means in the class-oriented world, but *tweak* (**read: pave over**) the understood semantics, to fit dynamic scripting.

The word "inheritance" has a very strong meaning (see Chapter 4), with plenty of mental precedent. Merely adding "prototypal" in front to distinguish the *actually nearly opposite* behavior in JavaScript has left in its wake nearly two decades of miry confusion.

I like to say that sticking "prototypal" in front of "inheritance" to drastically reverse its actual meaning is like holding an orange in one hand, an apple in the other, and insisting on calling the apple a "red orange". No matter what confusing label I put in front of it, that doesn't change the *fact* that one fruit is an apple and the other is an orange.

The better approach is to plainly call an apple an apple -- to use the most accurate and direct terminology. That makes it easier to understand both their similarities and their **many differences**, because we all have a simple, shared understanding of what "apple" means.

Because of the confusion and conflation of terms, I believe the label "prototypal inheritance" itself (and trying to mis-apply all its associated class-orientation terminology, like "class", "constructor", "instance", "polymorphism", etc) has done **more harm than good** in explaining how JavaScript's mechanism *really* works.

"Inheritance" implies a *copy* operation, and JavaScript doesn't copy object properties (natively, by default). Instead, JS creates a link between two objects, where one object can essentially *delegate* property/function access to another object. "Delegation" (see Chapter 6) is a much more accurate term for JavaScript's object-linking mechanism.

Another term which is sometimes thrown around in JavaScript is "differential inheritance". The idea here is that we describe an object's behavior in terms of what is *different* from a more general descriptor. For example, you explain that a car is a kind of vehicle, but one that has exactly 4 wheels, rather than re-describing all the specifics of what makes up a general vehicle (engine, etc).

If you try to think of any given object in JS as the sum total of all behavior that is *available* via delegation, and **in your mind you flatten** all that behavior into one tangible *thing*, then you can (sorta) see how "differential inheritance" might fit.

But just like with "prototypal inheritance", "differential inheritance" pretends that your mental model is more important than what is physically happening in the language. It overlooks the fact that object `B` is not actually differentially constructed, but is instead built with specific characteristics defined, alongside "holes" where nothing is defined. It is in these "holes" (gaps in, or lack of, definition) that delegation *can* take over and, on the fly, "fill them in" with delegated behavior.

The object is not, by native default, flattened into the single differential object, **through copying**, that the mental model of "differential inheritance" implies. As such, "differential inheritance" is just not as natural a fit for describing how JavaScript's `[[Prototype]]` mechanism actually works.

You *can choose* to prefer the "differential inheritance" terminology and mental model, as a matter of taste, but there's no denying the fact that it *only* fits the mental acrobatics in your mind, not the physical behavior in the engine.

### "Constructors"

Let's go back to some earlier code:

```js
function Foo() {
	// ...
}

var a = new Foo();
```

What exactly leads us to think `Foo` is a "class"?

For one, we see the use of the `new` keyword, just like class-oriented languages do when they construct class instances. For another, it appears that we are in fact executing a *constructor* method of a class, because `Foo()` is actually a method that gets called, just like how a real class's constructor gets called when you instantiate that class.

To further the confusion of "constructor" semantics, the arbitrarily labeled `Foo.prototype` object has another trick up its sleeve. Consider this code:

```js
function Foo() {
	// ...
}

Foo.prototype.constructor === Foo; // true

var a = new Foo();
a.constructor === Foo; // true
```

The `Foo.prototype` object by default (at declaration time on line 1 of the snippet!) gets a public, non-enumerable (see Chapter 3) property called `.constructor`, and this property is a reference back to the function (`Foo` in this case) that the object is associated with. Moreover, we see that object `a` created by the "constructor" call `new Foo()` *seems* to also have a property on it called `.constructor` which similarly points to "the function which created it".

**Note:** This is not actually true. `a` has no `.constructor` property on it, and though `a.constructor` does in fact resolve to the `Foo` function, "constructor" **does not actually mean** "was constructed by", as it appears. We'll explain this strangeness shortly.

Oh, yeah, also... by convention in the JavaScript world, "class"es are named with a capital letter, so the fact that it's `Foo` instead of `foo` is a strong clue that we intend it to be a "class". That's totally obvious to you, right!?

**Note:** This convention is so strong that many JS linters actually *complain* if you call `new` on a method with a lowercase name, or if we don't call `new` on a function that happens to start with a capital letter. That sort of boggles the mind that we struggle so much to get (fake) "class-orientation" *right* in JavaScript that we create linter rules to ensure we use capital letters, even though the capital letter doesn't mean ***anything* at all** to the JS engine.

#### Constructor Or Call?

In the above snippet, it's tempting to think that `Foo` is a "constructor", because we call it with `new` and we observe that it "constructs" an object.

In reality, `Foo` is no more a "constructor" than any other function in your program. Functions themselves are **not** constructors. However, when you put the `new` keyword in front of a normal function call, that makes that function call a "constructor call". In fact, `new` sort of hijacks any normal function and calls it in a fashion that constructs an object, **in addition to whatever else it was going to do**.

For example:

```js
function NothingSpecial() {
	console.log( "Don't mind me!" );
}

var a = new NothingSpecial();
// "Don't mind me!"

a; // {}
```

`NothingSpecial` is just a plain old normal function, but when called with `new`, it *constructs* an object, almost as a side-effect, which we happen to assign to `a`. The **call** was a *constructor call*, but `NothingSpecial` is not, in and of itself, a *constructor*.

In other words, in JavaScript, it's most appropriate to say that a "constructor" is **any function called with the `new` keyword** in front of it.

Functions aren't constructors, but function calls are "constructor calls" if and only if `new` is used.

### Mechanics

Are *those* the only common triggers for ill-fated "class" discussions in JavaScript?

**Not quite.** JS developers have strived to simulate as much as they can of class-orientation:

```js
function Foo(name) {
	this.name = name;
}

Foo.prototype.myName = function() {
	return this.name;
};

var a = new Foo( "a" );
var b = new Foo( "b" );

a.myName(); // "a"
b.myName(); // "b"
```

This snippet shows two additional "class-orientation" tricks in play:

1. `this.name = name`: adds the `.name` property onto each object (`a` and `b`, respectively; see Chapter 2 about `this` binding), similar to how class instances encapsulate data values.

2. `Foo.prototype.myName = ...`: perhaps the more interesting technique, this adds a property (function) to the `Foo.prototype` object. Now, `a.myName()` works, but perhaps surprisingly. How?

In the above snippet, it's strongly tempting to think that when `a` and `b` are created, the properties/functions on the `Foo.prototype` object are *copied* over to each of `a` and `b` objects. **However, that's not what happens.**

At the beginning of this chapter, we explained the `[[Prototype]]` link, and how it provides the fall-back look-up steps if a property reference isn't found directly on an object, as part of the default `[[Get]]` algorithm.

So, by virtue of how they are created, `a` and `b` each end up with an internal `[[Prototype]]` linkage to `Foo.prototype`. When `myName` is not found on `a` or `b`, respectively, it's instead found (through delegation, see Chapter 6) on `Foo.prototype`.

#### "Constructor" Redux

Recall the discussion from earlier about the `.constructor` property, and how it *seems* like `a.constructor === Foo` being true means that `a` has an actual `.constructor` property on it, pointing at `Foo`? **Not correct.**

This is just unfortunate confusion. In actuality, the `.constructor` reference is also *delegated* up to `Foo.prototype`, which **happens to**, by default, have a `.constructor` that points at `Foo`.

It *seems* awfully convenient that an object `a` "constructed by" `Foo` would have access to a `.constructor` property that points to `Foo`. But that's nothing more than a false sense of security. It's a happy accident, almost tangentially, that `a.constructor` *happens* to point at `Foo` via this default `[[Prototype]]` delegation. There are actually several ways that the ill-fated assumption of `.constructor` meaning "was constructed by" can come back to bite you.

For one, the `.constructor` property on `Foo.prototype` is only there by default on the object created when `Foo` the function is declared. If you create a new object, and replace a function's default `.prototype` object reference, the new object will not by default magically get a `.constructor` on it.

Consider:

```js
function Foo() { /* .. */ }

Foo.prototype = { /* .. */ }; // create a new prototype object

var a1 = new Foo();
a1.constructor === Foo; // false!
a1.constructor === Object; // true!
```

`Object(..)` didn't "construct" `a1` did it? It sure seems like `Foo()` "constructed" it. Many developers think of `Foo()` as doing the construction, but where everything falls apart is when you think "constructor" means "was constructed by", because by that reasoning, `a1.constructor` should be `Foo`, but it isn't!

What's happening? `a1` has no `.constructor` property, so it delegates up the `[[Prototype]]` chain to `Foo.prototype`. But that object doesn't have a `.constructor` either (like the default `Foo.prototype` object would have had!), so it keeps delegating, this time up to `Object.prototype`, the top of the delegation chain. *That* object indeed has a `.constructor` on it, which points to the built-in `Object(..)` function.

**Misconception, busted.**

Of course, you can add `.constructor` back to the `Foo.prototype` object, but this takes manual work, especially if you want to match native behavior and have it be non-enumerable (see Chapter 3).

For example:

```js
function Foo() { /* .. */ }

Foo.prototype = { /* .. */ }; // create a new prototype object

// Need to properly "fix" the missing `.constructor`
// property on the new object serving as `Foo.prototype`.
// See Chapter 3 for `defineProperty(..)`.
Object.defineProperty( Foo.prototype, "constructor" , {
	enumerable: false,
	writable: true,
	configurable: true,
	value: Foo    // point `.constructor` at `Foo`
} );
```

That's a lot of manual work to fix `.constructor`. Moreover, all we're really doing is perpetuating the misconception that "constructor" means "was constructed by". That's an *expensive* illusion.

The fact is, `.constructor` on an object arbitrarily points, by default, at a function who, reciprocally, has a reference back to the object -- a reference which it calls `.prototype`. The words "constructor" and "prototype" only have a loose default meaning that might or might not hold true later. The best thing to do is remind yourself, "constructor does not mean constructed by".

`.constructor` is not a magic immutable property. It *is* non-enumerable (see snippet above), but its value is writable (can be changed), and moreover, you can add or overwrite (intentionally or accidentally) a property of the name `constructor` on any object in any `[[Prototype]]` chain, with any value you see fit.

By virtue of how the `[[Get]]` algorithm traverses the `[[Prototype]]` chain, a `.constructor` property reference found anywhere may resolve quite differently than you'd expect.

See how arbitrary its meaning actually is?

The result? Some arbitrary object-property reference like `a1.constructor` cannot actually be *trusted* to be the assumed default function reference. Moreover, as we'll see shortly, just by simple omission, `a1.constructor` can even end up pointing somewhere quite surprising and insensible.

`.constructor` is extremely unreliable, and an unsafe reference to rely upon in your code. **Generally, such references should be avoided where possible.**

## "(Prototypal) Inheritance"

We've seen some approximations of "class" mechanics as typically hacked into JavaScript programs. But JavaScript "class"es would be rather hollow if we didn't have an approximation of "inheritance".

Actually, we've already seen the mechanism which is commonly called "prototypal inheritance" at work when `a` was able to "inherit from" `Foo.prototype`, and thus get access to the `myName()` function. But we traditionally think of "inheritance" as being a relationship between two "classes", rather than between "class" and "instance".

<img src="fig3.png">

Recall this figure from earlier, which shows not only delegation from an object (aka, "instance") `a1` to object `Foo.prototype`, but from `Bar.prototype` to `Foo.prototype`, which somewhat resembles the concept of Parent-Child class inheritance. *Resembles*, except of course for the direction of the arrows, which show these are delegation links rather than copy operations.

And, here's the typical "prototype style" code that creates such links:

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

**Note:** To understand why `this` points to `a` in the above code snippet, see Chapter 2.

The important part is `Bar.prototype = Object.create( Foo.prototype )`. `Object.create(..)` *creates* a "new" object out of thin air, and links that new object's internal `[[Prototype]]` to the object you specify (`Foo.prototype` in this case).

In other words, that line says: "make a *new* 'Bar dot prototype' object that's linked to 'Foo dot prototype'."

When `function Bar() { .. }` is declared, `Bar`, like any other function, has a `.prototype` link to its default object. But *that* object is not linked to `Foo.prototype` like we want. So, we create a *new* object that *is* linked as we want, effectively throwing away the original incorrectly-linked object.

**Note:** A common mis-conception/confusion here is that either of the following approaches would *also* work, but they do not work as you'd expect:

```js
// doesn't work like you want!
Bar.prototype = Foo.prototype;

// works kinda like you want, but with
// side-effects you probably don't want :(
Bar.prototype = new Foo();
```

`Bar.prototype = Foo.prototype` doesn't create a new object for `Bar.prototype` to be linked to. It just makes `Bar.prototype` be another reference to `Foo.prototype`, which effectively links `Bar` directly to **the same object as** `Foo` links to: `Foo.prototype`. This means when you start assigning, like `Bar.prototype.myLabel = ...`, you're modifying **not a separate object** but *the* shared `Foo.prototype` object itself, which would affect any objects linked to `Foo.prototype`. This is almost certainly not what you want. If it *is* what you want, then you likely don't need `Bar` at all, and should just use only `Foo` and make your code simpler.

`Bar.prototype = new Foo()` **does in fact** create a new object which is duly linked to `Foo.prototype` as we'd want. But, it uses the `Foo(..)` "constructor call" to do it. If that function has any side-effects (such as logging, changing state, registering against other objects, **adding data properties to `this`**, etc.), those side-effects happen at the time of this linking (and likely against the wrong object!), rather than only when the eventual `Bar()` "descendants" are created, as would likely be expected.

So, we're left with using `Object.create(..)` to make a new object that's properly linked, but without having the side-effects of calling `Foo(..)`. The slight downside is that we have to create a new object, throwing the old one away, instead of modifying the existing default object we're provided.

It would be *nice* if there was a standard and reliable way to modify the linkage of an existing object. Prior to ES6, there's a non-standard and not fully-cross-browser way, via the `.__proto__` property, which is settable. ES6 adds a `Object.setPrototypeOf(..)` helper utility, which does the trick in a standard and predictable way.

Compare the pre-ES6 and ES6-standardized techniques for linking `Bar.prototype` to `Foo.prototype`, side-by-side:

```js
// pre-ES6
// throws away default existing `Bar.prototype`
Bar.prototype = Object.create( Foo.prototype );

// ES6+
// modifies existing `Bar.prototype`
Object.setPrototypeOf( Bar.prototype, Foo.prototype );
```

Ignoring the slight performance disadvantage (throwing away an object that's later garbage collected) of the `Object.create(..)` approach, it's a little bit shorter and may be perhaps a little easier to read than the ES6+ approach. But it's probably a syntactic wash either way.

### Inspecting "Class" Relationships

What if you have an object like `a` and want to find out what object (if any) it delegates to? Inspecting an instance (just an object in JS) for its inheritance ancestry (delegation linkage in JS) is often called *introspection* (or *reflection*) in traditional class-oriented environments.

Consider:

```js
function Foo() {
	// ...
}

Foo.prototype.blah = ...;

var a = new Foo();
```

How do we then introspect `a` to find out its "ancestry" (delegation linkage)? The first approach embraces the "class" confusion:

```js
a instanceof Foo; // true
```

The `instanceof` operator takes a plain object as its left-hand operand and a **function** as its right-hand operand. The question `instanceof` answers is: **in the entire `[[Prototype]]` chain of `a`, does the object arbitrarily pointed to by `Foo.prototype` ever appear?**

Unfortunately, this means that you can only inquire about the "ancestry" of some object (`a`) if you have some **function** (`Foo`, with its attached `.prototype` reference) to test with. If you have two arbitrary objects, say `a` and `b`, and want to find out if *the objects* are related to each other through a `[[Prototype]]` chain, `instanceof` alone can't help.

**Note:** If you use the built-in `.bind(..)` utility to make a hard-bound function (see Chapter 2), the function created will not have a `.prototype` property. Using `instanceof` with such a function transparently substitutes the `.prototype` of the *target function* that the hard-bound function was created from.

It's fairly uncommon to use hard-bound functions as "constructor calls", but if you do, it will behave as if the original *target function* was invoked instead, which means that using `instanceof` with a hard-bound function also behaves according to the original function.

This snippet illustrates the ridiculousness of trying to reason about relationships between **two objects** using "class" semantics and `instanceof`:

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

Inside `isRelatedTo(..)`, we borrow a throw-away function `F`, reassign its `.prototype` to arbitrarily point to some object `o2`, then ask if `o1` is an "instance of" `F`. Obviously `o1` isn't *actually* inherited or descended or even constructed from `F`, so it should be clear why this kind of exercise is silly and confusing. **The problem comes down to the awkwardness of class semantics forced upon JavaScript**, in this case as revealed by the indirect semantics of `instanceof`.

The second, and much cleaner, approach to `[[Prototype]]` reflection is:

```js
Foo.prototype.isPrototypeOf( a ); // true
```

Notice that in this case, we don't really care about (or even *need*) `Foo`, we just need an **object** (in our case, arbitrarily labeled `Foo.prototype`) to test against another **object**. The question `isPrototypeOf(..)` answers is: **in the entire `[[Prototype]]` chain of `a`, does `Foo.prototype` ever appear?**

Same question, and exact same answer. But in this second approach, we don't actually need the indirection of referencing a **function** (`Foo`) whose `.prototype` property will automatically be consulted.

We *just need* two **objects** to inspect a relationship between them. For example:

```js
// Simply: does `b` appear anywhere in
// `c`s [[Prototype]] chain?
b.isPrototypeOf( c );
```

Notice, this approach doesn't require a function ("class") at all. It just uses object references directly to `b` and `c`, and inquires about their relationship. In other words, our `isRelatedTo(..)` utility above is built-in to the language, and it's called `isPrototypeOf(..)`.

We can also directly retrieve the `[[Prototype]]` of an object. As of ES5, the standard way to do this is:

```js
Object.getPrototypeOf( a );
```

And you'll notice that object reference is what we'd expect:

```js
Object.getPrototypeOf( a ) === Foo.prototype; // true
```

Most browsers (not all!) have also long supported a non-standard alternate way of accessing the internal `[[Prototype]]`:

```js
a.__proto__ === Foo.prototype; // true
```

The strange `.__proto__` (not standardized until ES6!) property "magically" retrieves the internal `[[Prototype]]` of an object as a reference, which is quite helpful if you want to directly inspect (or even traverse: `.__proto__.__proto__...`) the chain.

Just as we saw earlier with `.constructor`, `.__proto__` doesn't actually exist on the object you're inspecting (`a` in our running example). In fact, it exists (non-enumerable; see Chapter 2) on the built-in `Object.prototype`, along with the other common utilities (`.toString()`, `.isPrototypeOf(..)`, etc).

Moreover, `.__proto__` looks like a property, but it's actually more appropriate to think of it as a getter/setter (see Chapter 3).

Roughly, we could envision `.__proto__` implemented (see Chapter 3 for object property definitions) like this:

```js
Object.defineProperty( Object.prototype, "__proto__", {
	get: function() {
		return Object.getPrototypeOf( this );
	},
	set: function(o) {
		// setPrototypeOf(..) as of ES6
		Object.setPrototypeOf( this, o );
		return o;
	}
} );
```

So, when we access (retrieve the value of) `a.__proto__`, it's like calling `a.__proto__()` (calling the getter function). *That* function call has `a` as its `this` even though the getter function exists on the `Object.prototype` object (see Chapter 2 for `this` binding rules), so it's just like saying `Object.getPrototypeOf( a )`.

`.__proto__` is also a settable property, just like using ES6's `Object.setPrototypeOf(..)` shown earlier. However, generally you **should not change the `[[Prototype]]` of an existing object**.

There are some very complex, advanced techniques used deep in some frameworks that allow tricks like "subclassing" an `Array`, but this is commonly frowned on in general programming practice, as it usually leads to *much* harder to understand/maintain code.

**Note:** As of ES6, the `class` keyword will allow something that approximates "subclassing" of built-in's like `Array`. See Appendix A for discussion of the `class` syntax added in ES6.

The only other narrow exception (as mentioned earlier) would be setting the `[[Prototype]]` of a default function's `.prototype` object to reference some other object (besides `Object.prototype`). That would avoid replacing that default object entirely with a new linked object. Otherwise, **it's best to treat object `[[Prototype]]` linkage as a read-only characteristic** for ease of reading your code later.

**Note:** The JavaScript community unofficially coined a term for the double-underscore, specifically the leading one in properties like `__proto__`: "dunder". So, the "cool kids" in JavaScript would generally pronounce `__proto__` as "dunder proto".

## Object Links

As we've now seen, the `[[Prototype]]` mechanism is an internal link that exists on one object which references some other object.

This linkage is (primarily) exercised when a property/method reference is made against the first object, and no such property/method exists. In that case, the `[[Prototype]]` linkage tells the engine to look for the property/method on the linked-to object. In turn, if that object cannot fulfill the look-up, its `[[Prototype]]` is followed, and so on. This series of links between objects forms what is called the "prototype chain".

### `Create()`ing Links

We've thoroughly debunked why JavaScript's `[[Prototype]]` mechanism is **not** like *classes*, and we've seen how it instead creates **links** between proper objects.

What's the point of the `[[Prototype]]` mechanism? Why is it so common for JS developers to go to so much effort (emulating classes) in their code to wire up these linkages?

Remember we said much earlier in this chapter that `Object.create(..)` would be a hero? Now, we're ready to see how.

```js
var foo = {
	something: function() {
		console.log( "Tell me something good..." );
	}
};

var bar = Object.create( foo );

bar.something(); // Tell me something good...
```

`Object.create(..)` creates a new object (`bar`) linked to the object we specified (`foo`), which gives us all the power (delegation) of the `[[Prototype]]` mechanism, but without any of the unnecessary complication of `new` functions acting as classes and constructor calls, confusing `.prototype` and `.constructor` references, or any of that extra stuff.

**Note:** `Object.create(null)` creates an object that has an empty (aka, `null`) `[[Prototype]]` linkage, and thus the object can't delegate anywhere. Since such an object has no prototype chain, the `instanceof` operator (explained earlier) has nothing to check, so it will always return `false`. These special empty-`[[Prototype]]` objects are often called "dictionaries" as they are typically used purely for storing data in properties, mostly because they have no possible surprise effects from any delegated properties/functions on the `[[Prototype]]` chain, and are thus purely flat data storage.

We don't *need* classes to create meaningful relationships between two objects. The only thing we should **really care about** is objects linked together for delegation, and `Object.create(..)` gives us that linkage without all the class cruft.

#### `Object.create()` Polyfilled

`Object.create(..)` was added in ES5. You may need to support pre-ES5 environments (like older IE's), so let's take a look at a simple **partial** polyfill for `Object.create(..)` that gives us the capability that we need even in those older JS environments:

```js
if (!Object.create) {
	Object.create = function(o) {
		function F(){}
		F.prototype = o;
		return new F();
	};
}
```

This polyfill works by using a throw-away `F` function and overriding its `.prototype` property to point to the object we want to link to. Then we use `new F()` construction to make a new object that will be linked as we specified.

This usage of `Object.create(..)` is by far the most common usage, because it's the part that *can be* polyfilled. There's an additional set of functionality that the standard ES5 built-in `Object.create(..)` provides, which is **not polyfillable** for pre-ES5. As such, this capability is far-less commonly used. For completeness sake, let's look at that additional functionality:

```js
var anotherObject = {
	a: 2
};

var myObject = Object.create( anotherObject, {
	b: {
		enumerable: false,
		writable: true,
		configurable: false,
		value: 3
	},
	c: {
		enumerable: true,
		writable: false,
		configurable: false,
		value: 4
	}
} );

myObject.hasOwnProperty( "a" ); // false
myObject.hasOwnProperty( "b" ); // true
myObject.hasOwnProperty( "c" ); // true

myObject.a; // 2
myObject.b; // 3
myObject.c; // 4
```

The second argument to `Object.create(..)` specifies property names to add to the newly created object, via declaring each new property's *property descriptor* (see Chapter 3). Because polyfilling property descriptors into pre-ES5 is not possible, this additional functionality on `Object.create(..)` also cannot be polyfilled.

The vast majority of usage of `Object.create(..)` uses the polyfill-safe subset of functionality, so most developers are fine with using the **partial polyfill** in pre-ES5 environments.

Some developers take a much stricter view, which is that no function should be polyfilled unless it can be *fully* polyfilled. Since `Object.create(..)` is one of those partial-polyfill'able utilities, this narrower perspective says that if you need to use any of the functionality of `Object.create(..)` in a pre-ES5 environment, instead of polyfilling, you should use a custom utility, and stay away from using the name `Object.create` entirely. You could instead define your own utility, like:

```js
function createAndLinkObject(o) {
	function F(){}
	F.prototype = o;
	return new F();
}

var anotherObject = {
	a: 2
};

var myObject = createAndLinkObject( anotherObject );

myObject.a; // 2
```

I do not share this strict opinion. I fully endorse the common partial-polyfill of `Object.create(..)` as shown above, and using it in your code even in pre-ES5. I'll leave it to you to make your own decision.

### Links As Fallbacks?

It may be tempting to think that these links between objects *primarily* provide a sort of fallback for "missing" properties or methods. While that may be an observed outcome, I don't think it represents the right way of thinking about `[[Prototype]]`.

Consider:

```js
var anotherObject = {
	cool: function() {
		console.log( "cool!" );
	}
};

var myObject = Object.create( anotherObject );

myObject.cool(); // "cool!"
```

That code will work by virtue of `[[Prototype]]`, but if you wrote it that way so that `anotherObject` was acting as a fallback **just in case** `myObject` couldn't handle some property/method that some developer may try to call, odds are that your software is going to be a bit more "magical" and harder to understand and maintain.

That's not to say there aren't cases where fallbacks are an appropriate design pattern, but it's not very common or idiomatic in JS, so if you find yourself doing so, you might want to take a step back and reconsider if that's really appropriate and sensible design.

**Note:** In ES6, an advanced functionality called `Proxy` is introduced which can provide something of a "method not found" type of behavior. `Proxy` is beyond the scope of this book, but will be covered in detail in a later book in the *"You Don't Know JS"* series.

**Don't miss an important but nuanced point here.**

Designing software where you intend for a developer to, for instance, call `myObject.cool()` and have that work even though there is no `cool()` method on `myObject` introduces some "magic" into your API design that can be surprising for future developers who maintain your software.

You can however design your API with less "magic" to it, but still take advantage of the power of `[[Prototype]]` linkage.

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

Here, we call `myObject.doCool()`, which is a method that *actually exists* on `myObject`, making our API design more explicit (less "magical"). *Internally*, our implementation follows the **delegation design pattern** (see Chapter 6), taking advantage of `[[Prototype]]` delegation to `anotherObject.cool()`.

In other words, delegation will tend to be less surprising/confusing if it's an internal implementation detail rather than plainly exposed in your API design. We will expound on **delegation** in great detail in the next chapter.

## Review (TL;DR)
