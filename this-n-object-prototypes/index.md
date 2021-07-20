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

The `for..in` loop iterates over the list of enumerable properties on an object (including its `[[Prototype]]` chain). But what if you instead want to iterate over the values?

With numerically-indexed arrays, iterating over the values is typically done with a standard `for` loop, like:

```js
var myArray = [1, 2, 3];

for (var i = 0; i < myArray.length; i++) {
	console.log( myArray[i] );
}
// 1 2 3
```

This isn't iterating over the values, though, but iterating over the indices, where you then use the index to reference the value, as `myArray[i]`.

ES5 also added several iteration helpers for arrays, including `forEach(..)`, `every(..)`, and `some(..)`. Each of these helpers accepts a function callback to apply to each element in the array, differing only in how they respectively respond to a return value from the callback.

`forEach(..)` will iterate over all values in the array, and ignores any callback return values. `every(..)` keeps going until the end *or* the callback returns a `false` (or "falsy") value, whereas `some(..)` keeps going until the end *or* the callback returns a `true` (or "truthy") value.

These special return values inside `every(..)` and `some(..)` act somewhat like a `break` statement inside a normal `for` loop, in that they stop the iteration early before it reaches the end.

If you iterate on an object with a `for..in` loop, you're also only getting at the values indirectly, because it's actually iterating only over the enumerable properties of the object, leaving you to access the properties manually to get the values.

**Note:** As contrasted with iterating over an array's indices in a numerically ordered way (`for` loop or other iterators), the order of iteration over an object's properties is **not guaranteed** and may vary between different JS engines. **Do not rely** on any observed ordering for anything that requires consistency among environments, as any observed agreement is unreliable.

But what if you want to iterate over the values directly instead of the array indices (or object properties)? Helpfully, ES6 adds a `for..of` loop syntax for iterating over arrays (and objects, if the object defines its own custom iterator):

```js
var myArray = [ 1, 2, 3 ];

for (var v of myArray) {
	console.log( v );
}
// 1
// 2
// 3
```

The `for..of` loop asks for an iterator object (from a default internal function known as `@@iterator` in spec-speak) of the *thing* to be iterated, and the loop then iterates over the successive return values from calling that iterator object's `next()` method, once for each loop iteration.

Arrays have a built-in `@@iterator`, so `for..of` works easily on them, as shown. But let's manually iterate the array, using the built-in `@@iterator`, to see how it works:

```js
var myArray = [ 1, 2, 3 ];
var it = myArray[Symbol.iterator]();

it.next(); // { value:1, done:false }
it.next(); // { value:2, done:false }
it.next(); // { value:3, done:false }
it.next(); // { done:true }
```

**Note:** We get at the `@@iterator` *internal property* of an object using an ES6 `Symbol`: `Symbol.iterator`. We briefly mentioned `Symbol` semantics earlier in the chapter (see "Computed Property Names"), so the same reasoning applies here. You'll always want to reference such special properties by `Symbol` name reference instead of by the special value it may hold. Also, despite the name's implications, `@@iterator` is **not the iterator object** itself, but a **function that returns** the iterator object -- a subtle but important detail!

As the above snippet reveals, the return value from an iterator's `next()` call is an object of the form `{ value: .. , done: .. }`, where `value` is the current iteration value, and `done` is a `boolean` that indicates if there's more to iterate.

Notice the value `3` was returned with a `done:false`, which seems strange at first glance. You have to call the `next()` a fourth time (which the `for..of` loop in the previous snippet automatically does) to get `done:true` and know you're truly done iterating. The reason for this quirk is beyond the scope of what we'll discuss here, but it comes from the semantics of ES6 generator functions.

While arrays do automatically iterate in `for..of` loops, regular objects **do not have a built-in `@@iterator`**. The reasons for this intentional omission are more complex than we will examine here, but in general it was better to not include some implementation that could prove troublesome for future types of objects.

It *is* possible to define your own default `@@iterator` for any object that you care to iterate over. For example:

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

**Note:** We used `Object.defineProperty(..)` to define our custom `@@iterator` (mostly so we could make it non-enumerable), but using the `Symbol` as a *computed property name* (covered earlier in this chapter), we could have declared it directly, like `var myObject = { a:2, b:3, [Symbol.iterator]: function(){ /* .. */ } }`.

Each time the `for..of` loop calls `next()` on `myObject`'s iterator object, the internal pointer will advance and return back the next value from the object's properties list (see a previous note about iteration ordering on object properties/values).

The iteration we just demonstrated is a simple value-by-value iteration, but you can of course define arbitrarily complex iterations for your custom data structures, as you see fit. Custom iterators combined with ES6's `for..of` loop are a powerful new syntactic tool for manipulating user-defined objects.

For example, a list of `Pixel` objects (with `x` and `y` coordinate values) could decide to order its iteration based on the linear distance from the `(0,0)` origin, or filter out points that are "too far away", etc. As long as your iterator returns the expected `{ value: .. }` return values from `next()` calls, and a `{ done: true }` after the iteration is complete, ES6's `for..of` can iterate over it.

In fact, you can even generate "infinite" iterators which never "finish" and always return a new value (such as a random number, an incremented value, a unique identifier, etc), though you probably will not use such iterators with an unbounded `for..of` loop, as it would never end and would hang your program.

```js
var randoms = {
	[Symbol.iterator]: function() {
		return {
			next: function() {
				return { value: Math.random() };
			}
		};
	}
};

var randoms_pool = [];
for (var n of randoms) {
	randoms_pool.push( n );

	// don't proceed unbounded!
	if (randoms_pool.length === 100) break;
}
```

This iterator will generate random numbers "forever", so we're careful to only pull out 100 values so our program doesn't hang.

## Review (TL;DR)
