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
* A class is a `blue-print`. To actually `get` an object we can interact with, we must build (aka, "`instantiate`") something `from the class`. The end result of such "`construction`" is an `object`, [...] an "`instance`", which we can directly `call methods` on and `access` any public data properties from.

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
* `Polymorphism` is a much broader topic than we will exhaust here, but our current "`relative`" semantics refers to one particular aspect: the idea that `any method` can reference `another method` (of the `same` or `different name`) at a `higher level` of the `inheritance hierarchy`. We say "`relative`" because we don't [...] define which `inheritance level` (aka, `class`) we want to access, but rather relatively reference it by [...] "`look one level up`".

* In many languages, the keyword `super` is used, in place of this example's `inherited:`, which leans on the idea that a "`super class`" is the `parent/ancestor` of the current class.

* Another aspect of polymorphism is that a `method name` can have `multiple definitions` at `different levels of the inheritance chain` [...] We see two occurrences of that behavior in our example above: `drive()` is defined in both `Vehicle` and `Car`, and `ignition()` is defined in both `Vehicle` and `SpeedBoat`.

* Another thing that [...] `class-oriented` languages give you via `super` is a direct way for the constructor of a `child class` to reference the constructor of `its parent class`.

* Inside `pilot()`, a [...] reference is made to [...] `Vehicle`s version of `drive()`. `But` that `drive()` references an `ignition()` method `just by name` [...] Which version of `ignition()` will the language engine use, the one from `Vehicle` or the one from `SpeedBoat`? It uses the `SpeedBoat` version of `ignition()`. `If` you `were` to instantiate `Vehicle` class [...] and then call `its` `drive()`, the language engine would instead just use `Vehicle`s `ignition()` method.

* When classes `are inherited`, there is a way `for the classes themselves` (`not` the `object instances` created from them!) to [...] reference the class `inherited from`, and this [...] `reference` is usually called `super`.

* it would seem a `child class` [...] (me: `linked to`)(can `access`) behavior in `its parent class` [...] using a `relative polymorphic reference` (aka, `super`). `However` [...] the child class is merely given a `copy` of the `inherited behavior` `from its parent class`. If the child "`overrides`" (me: like `super().prop1 = 1` or `super().method1 = () => {}`) a method it inherits, `both` the `original` and `overridden versions` of the method are actually `maintained`, `so that` they are both accessible. [...] `Class inheritance implies copies`.

```js
// (me: a demonstration for the above argument by utilizing the "class" implementation of JS)
class A {
	a = 1
}

class B extends A {
	constructor() {
		/* (CL:
			note that "super()" is similar to "A.call(this)" with:
			"this" refers to the "instance of" "B" and
			"A" is actually: "function A { this.a = 2; }"
		) */
		super().a = 2;
	}
}

new B() // {a: 2} => the result of the override made via "super" is maintained
new A() // {a: 1} => the original property value is also maintained
```

### Multiple Inheritance
Recall our earlier discussion of parent(s) and children and DNA? We said that the metaphor was a bit weird because biologically most offspring come from two parents. If a class could inherit from two other classes, it would more closely fit the parent/child metaphor.

Some class-oriented languages allow you to specify more than one "parent" class to "inherit" from. Multiple-inheritance means that each parent class definition is copied into the child class.

On the surface, this seems like a powerful addition to class-orientation, giving us the ability to compose more functionality together. However, there are certainly some complicating questions that arise. If both parent classes provide a method called `drive()`, which version would a `drive()` reference in the child resolve to? Would you always have to manually specify which parent's `drive()` you meant, thus losing some of the gracefulness of polymorphic inheritance?

There's another variation, the so called "Diamond Problem", which refers to the scenario where a child class "D" inherits from two parent classes ("B" and "C"), and each of those in turn inherits from a common "A" parent. If "A" provides a method `drive()`, and both "B" and "C" override (polymorph) that method, when `D` references `drive()`, which version should it use (`B:drive()` or `C:drive()`)?

<img src="fig2.png">

These complications go even much deeper than this quick glance. We address them here only so we can contrast to how JavaScript's mechanisms work.

JavaScript is simpler: it does not provide a native mechanism for "multiple inheritance". Many see this as a good thing, because the complexity savings more than make up for the "reduced" functionality. But this doesn't stop developers from trying to fake it in various ways, as we'll see next.

## Mixins

JavaScript's object mechanism does not *automatically* perform copy behavior when you "inherit" or "instantiate". Plainly, there are no "classes" in JavaScript to instantiate, only objects. And objects don't get copied to other objects, they get *linked together* (more on that in Chapter 5).

Since observed class behaviors in other languages imply copies, let's examine how JS developers **fake** the *missing* copy behavior of classes in JavaScript: mixins. We'll look at two types of "mixin": **explicit** and **implicit**.

### Explicit Mixins

Let's again revisit our `Vehicle` and `Car` example from before. Since JavaScript will not automatically copy behavior from `Vehicle` to `Car`, we can instead create a utility that manually copies. Such a utility is often called `extend(..)` by many libraries/frameworks, but we will call it `mixin(..)` here for illustrative purposes.

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

**Note:** Subtly but importantly, we're not dealing with classes anymore, because there are no classes in JavaScript. `Vehicle` and `Car` are just objects that we make copies from and to, respectively.

`Car` now has a copy of the properties and functions from `Vehicle`. Technically, functions are not actually duplicated, but rather *references* to the functions are copied. So, `Car` now has a property called `ignition`, which is a copied reference to the `ignition()` function, as well as a property called `engines` with the copied value of `1` from `Vehicle`.

`Car` *already* had a `drive` property (function), so that property reference was not overridden (see the `if` statement in `mixin(..)` above).

#### "Polymorphism" Revisited

Let's examine this statement: `Vehicle.drive.call( this )`. This is what I call "explicit pseudo-polymorphism". Recall in our previous pseudo-code this line was `inherited:drive()`, which we called "relative polymorphism".

JavaScript does not have (prior to ES6; see Appendix A) a facility for relative polymorphism. So, **because both `Car` and `Vehicle` had a function of the same name: `drive()`**, to distinguish a call to one or the other, we must make an absolute (not relative) reference. We explicitly specify the `Vehicle` object by name, and call the `drive()` function on it.

But if we said `Vehicle.drive()`, the `this` binding for that function call would be the `Vehicle` object instead of the `Car` object (see Chapter 2), which is not what we want. So, instead we use `.call( this )` (Chapter 2) to ensure that `drive()` is executed in the context of the `Car` object.

**Note:** If the function name identifier for `Car.drive()` hadn't overlapped with (aka, "shadowed"; see Chapter 5) `Vehicle.drive()`, we wouldn't have been exercising "method polymorphism". So, a reference to `Vehicle.drive()` would have been copied over by the `mixin(..)` call, and we could have accessed directly with `this.drive()`. The chosen identifier overlap **shadowing** is *why* we have to use the more complex *explicit pseudo-polymorphism* approach.

In class-oriented languages, which have relative polymorphism, the linkage between `Car` and `Vehicle` is established once, at the top of the class definition, which makes for only one place to maintain such relationships.

But because of JavaScript's peculiarities, explicit pseudo-polymorphism (because of shadowing!) creates brittle manual/explicit linkage **in every single function where you need such a (pseudo-)polymorphic reference**. This can significantly increase the maintenance cost. Moreover, while explicit pseudo-polymorphism can emulate the behavior of "multiple inheritance", it only increases the complexity and brittleness.

The result of such approaches is usually more complex, harder-to-read, *and* harder-to-maintain code. **Explicit pseudo-polymorphism should be avoided wherever possible**, because the cost outweighs the benefit in most respects.

#### Mixing Copies

Recall the `mixin(..)` utility from above:

```js
// vastly simplified `mixin()` example:
function mixin( sourceObj, targetObj ) {
	for (var key in sourceObj) {
		// only copy if not already present
		if (!(key in targetObj)) {
			targetObj[key] = sourceObj[key];
		}
	}

	return targetObj;
}
```

Now, let's examine how `mixin(..)` works. It iterates over the properties of `sourceObj` (`Vehicle` in our example) and if there's no matching property of that name in `targetObj` (`Car` in our example), it makes a copy. Since we're making the copy after the initial object exists, we are careful to not copy over a target property.

If we made the copies first, before specifying the `Car` specific contents, we could omit this check against `targetObj`, but that's a little more clunky and less efficient, so it's generally less preferred:

```js
// alternate mixin, less "safe" to overwrites
function mixin( sourceObj, targetObj ) {
	for (var key in sourceObj) {
		targetObj[key] = sourceObj[key];
	}

	return targetObj;
}

var Vehicle = {
	// ...
};

// first, create an empty object with
// Vehicle's stuff copied in
var Car = mixin( Vehicle, { } );

// now copy the intended contents into Car
mixin( {
	wheels: 4,

	drive: function() {
		// ...
	}
}, Car );
```

Either approach, we have explicitly copied the non-overlapping contents of `Vehicle` into `Car`. The name "mixin" comes from an alternate way of explaining the task: `Car` has `Vehicle`s contents **mixed-in**, just like you mix in chocolate chips into your favorite cookie dough.

As a result of the copy operation, `Car` will operate somewhat separately from `Vehicle`. If you add a property onto `Car`, it will not affect `Vehicle`, and vice versa.

**Note:** A few minor details have been skimmed over here. There are still some subtle ways the two objects can "affect" each other even after copying, such as if they both share a reference to a common object (such as an array).

Since the two objects also share references to their common functions, that means that **even manual copying of functions (aka, mixins) from one object to another doesn't *actually emulate* the real duplication from class to instance that occurs in class-oriented languages**.

JavaScript functions can't really be duplicated (in a standard, reliable way), so what you end up with instead is a **duplicated reference** to the same shared function object (functions are objects; see Chapter 3). If you modified one of the shared **function objects** (like `ignition()`) by adding properties on top of it, for instance, both `Vehicle` and `Car` would be "affected" via the shared reference.

Explicit mixins are a fine mechanism in JavaScript. But they appear more powerful than they really are. Not much benefit is *actually* derived from copying a property from one object to another, **as opposed to just defining the properties twice**, once on each object. And that's especially true given the function-object reference nuance we just mentioned.

If you explicitly mix-in two or more objects into your target object, you can **partially emulate** the behavior of "multiple inheritance", but there's no direct way to handle collisions if the same method or property is being copied from more than one source. Some developers/libraries have come up with "late binding" techniques and other exotic work-arounds, but fundamentally these "tricks" are *usually* more effort (and lesser performance!) than the pay-off.

Take care only to use explicit mixins where it actually helps make more readable code, and avoid the pattern if you find it making code that's harder to trace, or if you find it creates unnecessary or unwieldy dependencies between objects.

**If it starts to get *harder* to properly use mixins than before you used them**, you should probably stop using mixins. In fact, if you have to use a complex library/utility to work out all these details, it might be a sign that you're going about it the harder way, perhaps unnecessarily. In Chapter 6, we'll try to distill a simpler way that accomplishes the desired outcomes without all the fuss.

#### Parasitic Inheritance

A variation on this explicit mixin pattern, which is both in some ways explicit and in other ways implicit, is called "parasitic inheritance", popularized mainly by Douglas Crockford.

Here's how it can work:

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

As you can see, we initially make a copy of the definition from the `Vehicle` "parent class" (object), then mixin our "child class" (object) definition (preserving privileged parent-class references as needed), and pass off this composed object `car` as our child instance.

**Note:** when we call `new Car()`, a new object is created and referenced by `Car`s `this` reference (see Chapter 2). But since we don't use that object, and instead return our own `car` object, the initially created object is just discarded. So, `Car()` could be called without the `new` keyword, and the functionality above would be identical, but without the wasted object creation/garbage-collection.

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

With `Something.cool.call( this )`, which can happen either in a "constructor" call (most common) or in a method call (shown here), we essentially "borrow" the function `Something.cool()` and call it in the context of `Another` (via its `this` binding; see Chapter 2) instead of `Something`. The end result is that the assignments that `Something.cool()` makes are applied against the `Another` object rather than the `Something` object.

So, it is said that we "mixed in" `Something`s behavior with (or into) `Another`.

While this sort of technique seems to take useful advantage of `this` rebinding functionality, it is the brittle `Something.cool.call( this )` call, which cannot be made into a relative (and thus more flexible) reference, that you should **heed with caution**. Generally, **avoid such constructs where possible** to keep cleaner and more maintainable code.

## Review (TL;DR)

Classes are a design pattern. Many languages provide syntax which enables natural class-oriented software design. JS also has a similar syntax, but it behaves **very differently** from what you're used to with classes in those other languages.

**Classes mean copies.**

When traditional classes are instantiated, a copy of behavior from class to instance occurs. When classes are inherited, a copy of behavior from parent to child also occurs.

Polymorphism (having different functions at multiple levels of an inheritance chain with the same name) may seem like it implies a referential relative link from child back to parent, but it's still just a result of copy behavior.

JavaScript **does not automatically** create copies (as classes imply) between objects.

The mixin pattern (both explicit and implicit) is often used to *sort of* emulate class copy behavior, but this usually leads to ugly and brittle syntax like explicit pseudo-polymorphism (`OtherObj.methodName.call(this, ...)`), which often results in harder to understand and maintain code.

Explicit mixins are also not exactly the same as class *copy*, since objects (and functions!) only have shared references duplicated, not the objects/functions duplicated themselves. Not paying attention to such nuance is the source of a variety of gotchas.

In general, faking classes in JS often sets more landmines for future coding than solving present *real* problems.
