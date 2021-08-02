# Foreword
# Preface
# Chapter 1: Types
* `ES5.1 specification` ([http://www.ecma-international.org/ecma-262/5.1/](http://www.ecma-international.org/ecma-262/5.1/)) [...] say.
> values [...] has an associated type [...] The `ECMAScript language types` are `Undefined`, `Null`, `Boolean`, `String`, `Number`, and `Object`.

* a *type* is an [...] built-in `set of characteristics` [...] (me: identifying) the `behavior` of a [...] value and `distinguishes` it from other values, both to the engine **and to the developer**.

## A Type By Any Other Name...
* `Coercion` confusion is perhaps one of the most [...] frustrations for JavaScript developers [...] often been criticized as [...] a flaw in the design of the language.

## Built-in Types
* `seven` built-in types:

* `null`
* `undefined`
* `boolean`
* `number`
* `string`
* `object`
* `symbol` -- added in ES6!

* All of these types `except` `object` are called "`primitives`".

* The `typeof` operator inspects the type of the given value [...] surprisingly, there's `not` an `exact 1-to-1 match` with the seven built-in types we just listed.

```js
typeof undefined     === "undefined"; // true
typeof true          === "boolean";   // true
typeof 42            === "number";    // true
typeof "42"          === "string";    // true
typeof { life: 42 }  === "object";    // true

// (me)
typeof [] === 'object' // true

// added in ES6!
typeof Symbol()      === "symbol";    // true
```

* As you may have noticed, I excluded `null` from the above listing. It's *special* [...] in the sense that it's buggy `when combined` with the `typeof` operator:

```js
typeof null === "object"; // true
```

* this `original bug` in JS has persisted for nearly two decades, and will likely never be fixed because there's too much existing web content that relies on its buggy behavior.

* If you want to test for a `null` value using its type, you need a compound condition:

```js
var a = null;

(!a && typeof a === "object"); // true
```

* `null` is the `only` primitive value that is "`falsy`" ([...] see Chapter 4) but [...] also returns `"object"` from the `typeof` check.

* the seventh string value that `typeof` can return.

```js
typeof function a(){ /* .. */ } === "function"; // true
```

* if you read the spec, you'll see (me: `function`)(it)'s actually a "`subtype`" of object [...] referred to as a "`callable object`" -- an object that has an internal `[[Call]]` property that allows it to be invoked.

* The function object has a `length` property set to the number of `formal parameters` it is `declared with`.

```js
a.length; // 2

// (me: my own content start here)
function a(...b) {}
a.length; // 0

function a(b, c) {}
a.length; // 2
```

* are (me: `arrays`)(they) a special type?

```js
typeof [1,2,3] === "object"; // true
```

## Values as Types
* In JavaScript, `variables` `don't have types` -- **values have types**. `Variables` can hold any value, at any time.

* Another way to think about JS types is that JS doesn't have "`type enforcement`," [...] A variable can, in one assignment statement, hold a `string`, and in the next hold a `number`.

* The *value* `42` has an [...] type of `number`, and its *type* cannot be changed [...] (me: value like) `"42"` with the `string` type, can be *created from* the `number` value `42` through a process called **coercion**.

* If you use `typeof` against a variable, it's not asking "what's the type of the `variable`?" [...] Instead, it's asking "what's the type of the `value` *in* the variable?"

### `undefined` vs "undeclared"
* Variables that have no value *currently*, actually have the `undefined` value. Calling `typeof` against such variables will return `"undefined"`:

* most developers to think of the word "`undefined`" and think of it as a `synonym` for "`undeclared`." However, in JS, these two concepts are quite different [...] An "`undefined`" variable is one that has been declared [...] but *at the moment* has no other value [...] `By contrast`, an "`undeclared`" variable is one that has `not` been `formally declared` `in the accessible scope`.

```js
var a;

a; // undefined
b; // ReferenceError: b is not defined
```

* An annoying confusion is the error message [...] "b is not defined," which is of course very easy [...] to confuse with "b is undefined." [...] It'd be nice if the browsers said something like "b is not found" or "b is not declared,".

* There's also a special behavior associated with `typeof` [...] that even further reinforces the confusion.

```js
var a;

typeof a; // "undefined"

typeof b; // "undefined"
```

* The `typeof` operator returns `"undefined"` even for "`undeclared`" (or "`not defined`") variables [...] there was `no error` thrown when we executed `typeof b` [...] This is a special `safety guard` in the behavior of `typeof`.

### `typeof` Undeclared
* Nevertheless, this `safety guard` is a useful feature when dealing with JavaScript in the browser, where `multiple script files` can load variables into the `shared global namespace`.

* Many developers believe there should `never` be any variables in the `global namespace` [...] everything should be contained in `modules` and `private/separate namespaces`. This is great [...] but nearly impossible in practicality [...] Fortunately, ES6 added first-class support for `modules`, which will eventually make that much more practical.

* imagine having a "`debug mode`" in your program [...] controlled by a `global variable` [...] called `DEBUG`. You'd want to check if that variable `was declared` before performing a debug task [...] A `top-level global` `var DEBUG = true` declaration would `only be included` in a "`debug.js`" [...] `only` load into the browser when you're in development/testing, but not in production [...] However, you have to [...] check for the global `DEBUG` variable [...] so that you don't throw a `ReferenceError`.

```js
// oops, this would throw an error!
if (DEBUG) {
	console.log( "Debugging is starting" );
}

// this is a safe existence check
if (typeof DEBUG !== "undefined") {
	console.log( "Debugging is starting" );
}
```

* Another way of doing these checks against `global variables` but without the `safety guard feature` of `typeof` is to observe that all `global variables` are also properties of the `global object` [...] the `window` object.

```js
if (window.DEBUG) {
	// ..
}

if (!window.atob) {
	// ..
}
```

* referencing the `global variable` with a `window` reference is something some developers prefer to avoid, especially if your code needs to run in multiple JS environments (not just browsers, but server-side node.js, for instance), where the global object may `not always` be called `window`.

* this safety guard on `typeof` is useful even if you're `not using` global variables, though these circumstances are less common.

* Other developers would prefer a design pattern called "`dependency injection`," where instead of `doSomethingCool()` inspecting [...] for `FeatureXYZ` to be defined outside/around it, it would need to have the dependency explicitly passed in, like:

```js
function doSomethingCool(FeatureXYZ) {
	var helper = FeatureXYZ ||
		function() { /*.. default feature ..*/ };

	var val = helper();
	// ..
}
```

## Review

# Chapter 2: Values
## Arrays
* JavaScript `array`s are just containers for `any` type of value.

* You don't need to `presize` your `array`s [...] you can just declare them and add values as you see fit:

* Using `delete` on an `array` value will remove that slot from the `array`, but [...] it does **not** update the `length` property.

* Be careful about [...] leaving or creating `empty/missing` slots:

```js
var a = [ ];

a[0] = 1;
// no `a[1]` slot set here
a[2] = [ 3 ];

a[1];		// undefined

a.length;	// 3
```

* While the slot appears to have the `undefined` value in it, it `will not` `behave` the same as if the slot is explicitly set (`a[1] = undefined`). See "Arrays" in `Chapter 3`.

* `array`s are numerically indexed [...] but [...] they also are objects that can have `string` keys/properties [...] but which don't count toward the `length` (me: property).

```js
var a = [ ];

a[0] = 1;
a["foobar"] = 2;

a.length;		// 1
a["foobar"];	// 2
a.foobar;		// 2
```

* However [...] if a `string` value intended as a key can be `coerced` to a standard `base-10` `number`, then it is assumed that you [...] use it as a `number index` rather than as a `string` key!

```js
var a = [ ];

a["13"] = 42;

a.length; // 14
```

### Array-Likes
* There will be occasions where you need to convert an `array-like value` (a numerically indexed collection of values) into a true `array` [...] so you can call array utilities (like `indexOf(..)`, `concat(..)`, `forEach(..)`, etc.) against the [...] values.

* For example, various `DOM query operations` return lists of DOM elements that are not true `array`s but are `array-like` [...] Another common example is when functions expose the `arguments` (`array`-like) object (as of `ES6`, deprecated) to access the arguments as a list.

```js
function foo() {
	var arr = Array.prototype.slice.call( arguments );
	arr.push( "bam" );
	console.log( arr );
}

foo( "bar", "baz" ); // ["bar","baz","bam"]
```

* If `slice()` is called without any other parameters [...] the default values for its parameters have the effect of `duplicating` the `array` (or, in this case, `array-like`).

* As of `ES6`, there's also a built-in utility called `Array.from(..)` that can do the same task:

```js
...
var arr = Array.from( arguments );
...
```

## Strings
```js
var a = "foo";
var b = ["f","o","o"];
```

* Strings do have a `shallow resemblance` to `array`s -- `array-likes` [...] for instance, both of them having a `length` property, an `indexOf(..)` method (`array` version only as of `ES5`), and a `concat(..)` method:

* (me: strings are) just "arrays of characters", right? `Not exactly` [...] JavaScript `string`s are `immutable`, while `array`s are [...] `mutable`. Moreover, the `a[1]` character position access form was `not always widely valid` [...] `Older versions` of IE did not allow that syntax [...] Instead, the *correct* approach has been `a.charAt(1)`.

* A further consequence of `immutable` `string`s is that none of the `string` methods that alter its contents can `modify` in-place, but rather `must` create and return `new string`s. By contrast, many of the methods that change `array` contents actually *do* modify in-place.

* Also, many of the `array` methods [...] could be helpful when dealing with `string`s are not actually available for them, but we can "`borrow`" `non-mutation` `array` methods against our `string`:

```js
a.join;			// undefined
a.map;			// undefined

var c = Array.prototype.join.call( a, "-" );
var d = Array.prototype.map.call( a, function(v){
	return v.toUpperCase() + ".";
} ).join( "" );

c;				// "f-o-o"
d;				// "F.O.O."
```

* (me: the `reverse` `arrays`' method does not work with `strings`) because `string`s are `immutable` and thus can't be modified in place [...] Another [...] hack [...] is to convert the `string` into an `array`, perform the desired operation, then convert it back to a `string`.

```js
var c = a
	// split `a` into an array of characters
	.split( "" )
	// reverse the array of characters
	.reverse()
	// join the array of characters back to a string
	.join( "" );

c; // "oof"
```

* Be careful! This approach **doesn't work** for `string`s with `complex (unicode) characters` in them [...] You need more sophisticated library utilities that are `unicode-aware` for such operations to be handled accurately.

## Numbers
* JavaScript has just one numeric type: `number`. This type includes both "integer" values and fractional decimal numbers.

### Numeric Syntax
* The leading portion of a decimal value, if `0`, is optional:

```js
var a = 0.42;
var b = .42;
```

* Similarly, the trailing portion (the fractional) of a decimal value after the `.`, if `0`, is optional:

```js
var a = 42.0;
var b = 42.;
```

* By default, most `number`s will be outputted as base-10 decimals, with trailing fractional `0`s removed. So:

```js
var a = 42.300;
var b = 42.0;

a; // 42.3
b; // 42
```

* `Very large` or `very small` `number`s will by default be outputted in `exponent form`, the same as the output of the `toExponential()` method, like:

```js
var a = 5E10;
a;					// 50000000000
a.toExponential();	// "5e+10"

var b = a * a;
b;					// 2.5e+21

var c = 1 / a;
c;					// 2e-11
```

* Because `number` values can be boxed with the `Number` object wrapper [...] `number` values can access methods that are built into the `Number.prototype`.

* don't have to use a variable with the value in it to access these methods; you can access these methods directly on `number` literals. But you have to be careful with the `.` operator. Since `.` is a `valid numeric character`.

```js
// invalid syntax:
42.toFixed( 3 );	// SyntaxError

// these are all valid:
(42).toFixed( 3 );	// "42.000"
0.42.toFixed( 3 );	// "0.420"
42..toFixed( 3 );	// "42.000"
```

* `42.toFixed(3)` is invalid syntax, because the `.` is swallowed up as part of the `42.` literal.

* `42..toFixed(3)` works because the first `.` is part of the `number` and the second `.` is the `property operator`.

* This is also technically valid (notice the space):

```js
42 .toFixed(3); // "42.000"
```

* `number`s can also be specified in exponent form, which is common when representing larger `number`s, such as:

```js
var onethousand = 1E3;						// means 1 * 10^3
var onemilliononehundredthousand = 1.1E6;	// means 1.1 * 10^6
```

* `number` literals can also be expressed in other bases, like binary, octal, and hexadecimal.

```js
0xf3; // hexadecimal for: 243
0Xf3; // ditto

0363; // octal for: 243
```

* Starting with ES6 + `strict` mode, the `0363` form of octal literals is `no longer allowed` [...] The `0363` form is still allowed in non-`strict` mode, but you should stop using it anyway, to be `future-friendly` (and because you should be using `strict` mode by now!).

* As of ES6, the following new forms are also valid:

```js
0o363;		// octal for: 243
0O363;		// ditto

0b11110011;	// binary for: 243
0B11110011; // ditto
```

* Please do your fellow developers a favor: never use the `0O363` form. `0` next to capital `O` is [...] confusion. Always use the lowercase predicates `0x`, `0b`, and `0o`.

### Small Decimal Values
* The most (in)famous side effect of using binary floating-point numbers (which [...] is true of **all** languages that use `IEEE 754` -- not *just* JavaScript [...]) is:

```js
0.1 + 0.2 === 0.3; // false
```

* Why is it `false`? [...] Simply put, the representations for `0.1` and `0.2` `in binary floating-point` are `not exact`, so when they are added, the result is not exactly `0.3`. It's **really** close: `0.30000000000000004`.

* The most commonly accepted practice is to use a `tiny "rounding error" value` as the *tolerance* for comparison. This `tiny value` is often called "`machine epsilon`," which is commonly `2^-52` (`2.220446049250313e-16`) for the kind of `number`s in JavaScript.

* As of `ES6`, `Number.EPSILON` is predefined with this tolerance value, so you'd want to use it, `but` you can `safely polyfill` the definition for pre-ES6:

```js
if (!Number.EPSILON) {
	Number.EPSILON = Math.pow(2,-52);
}
```

* We can use this `Number.EPSILON` to compare two `number`s for "equality" (within the rounding error tolerance):

```js
function numbersCloseEnoughToEqual(n1,n2) {
	return Math.abs( n1 - n2 ) < Number.EPSILON;
}

var a = 0.1 + 0.2;
var b = 0.3;

numbersCloseEnoughToEqual( a, b );					// true
numbersCloseEnoughToEqual( 0.0000001, 0.0000002 );	// false
```

* The maximum floating-point value that can be represented is roughly `1.798e+308` [...] predefined for you as `Number.MAX_VALUE`. On the small end, `Number.MIN_VALUE` is roughly `5e-324`.

### Safe Integer Ranges
* The `maximum integer` that can "safely" be represented (that is, there's a guarantee that the requested value is actually representable unambiguously) is `2^53 - 1`, which is `9007199254740991` [...] This value is actually automatically `predefined` in `ES6`, as `Number.MAX_SAFE_INTEGER`. Unsurprisingly, there's a minimum value, `-9007199254740991`, and it's defined in `ES6` as `Number.MIN_SAFE_INTEGER`.

* The main way that JS programs are confronted with dealing with such `large numbers` is when dealing with 64-bit `IDs` from `databases`, etc. `64-bit numbers` cannot be represented `accurately` with the `number` type.

* if you *do* need to perform math on these very large values [...] you'll need to use a *big number* utility. `Big numbers` may get official support in a future version of JavaScript.

### Testing for Integers
* To test if a value is an integer, you can use the ES6-specified `Number.isInteger(..)`:

```js
Number.isInteger( 42 );		// true
Number.isInteger( 42.000 );	// true
Number.isInteger( 42.3 );	// false
```

To polyfill `Number.isInteger(..)` for pre-ES6:

```js
if (!Number.isInteger) {
	Number.isInteger = function(num) {
		return typeof num == "number" && num % 1 == 0;
	};
}
```

* To test if a value is a *safe integer*, use the ES6-specified `Number.isSafeInteger(..)`:

```js
Number.isSafeInteger( Number.MAX_SAFE_INTEGER );	// true
Number.isSafeInteger( Math.pow( 2, 53 ) );			// false
Number.isSafeInteger( Math.pow( 2, 53 ) - 1 );		// true
```

### 32-bit (Signed) Integers
* While integers can range up to roughly 9 quadrillion `safely` (53 bits), there are some numeric operations (like the `bitwise` operators) that are `only defined for` `32-bit numbers`, so the "`safe range`" for `number`s used in `that way` [... is `Math.pow(-2,31)` (`-2147483648`, about -2.1 billion) up to `Math.pow(2,31)-1` (`2147483647`, about +2.1 billion).

* To `force` a `number` value in `a` to a `32-bit signed integer value`, use `a | 0`. This works because the `|` bitwise operator `only works` for `32-bit integer values` (meaning it can only pay attention to 32 bits and any `other bits will be lost`). Then, "or'ing" with zero is essentially a no-op bitwise speaking.

* Certain `special values` [...] such as `NaN` and `Infinity` are `not "32-bit safe,"` in that those values when passed to a bitwise operator will pass through the abstract operation `ToInt32` [...] and become simply the `+0` value for the purpose of that `bitwise operation`.

## Special Values
### The Non-value Values
* For the `undefined` type, there is one and only one value: `undefined`. For the `null` type, there is one and only one value: `null`. So for both of them, the label is both its type and its value.

* Both `undefined` and `null` are often taken to be `interchangeable` as either "empty" values or "non" values. Other developers prefer to `distinguish` between them with nuance. For example:

* `null` is an empty value
* `undefined` is a missing value

Or:

* `undefined` hasn't had a value yet
* `null` had a value and doesn't anymore

* `null` is a special keyword, `not an identifier`, and thus you cannot treat it `as a variable` to assign to [...] However, `undefined` *is* [...] an `identifier`.

### Undefined
* In `non-strict` mode, it's actually possible [...] to assign a value to the globally provided `undefined` identifier:

```js
function foo() {
	undefined = 2; // really bad idea!
}

foo();
```

```js
function foo() {
	"use strict";
	undefined = 2; // TypeError!
}

foo();
```

* In both non-`strict` mode and `strict` mode, however, you can `create` a local variable of the name `undefined`. But again, this is a terrible idea!

```js
function foo() {
	"use strict";
	var undefined = 2;
	console.log( undefined ); // 2
}

foo();
```

#### `void` Operator
* While `undefined` is a `built-in identifier` (me: see the above argument) that holds [...] the built-in `undefined` value, `another way` to get this value is the `void` operator.

* The expression `void ___` `"voids" out` `any value`, so that the result of the expression is `always` the `undefined` value. It `doesn't modify` the existing value; it just ensures that no value comes back from the operator expression.

```js
var a = 42;

console.log( void a, a ); // undefined 42
```

* By convention [...] to represent the `undefined` value stand-alone by using `void`, you'd use `void 0` (though clearly even `void true` or any other `void` expression does the same thing). There's no practical difference between `void 0`, `void 1`, and `undefined`.

* the `void` operator can be useful in a few other circumstances, if you need to ensure that an expression `has no result value` (`even if` it has `side effects`).

```js
function doSomething() {
	// note: `APP.ready` is provided by our application
	if (!APP.ready) {
		// try again later
		return void setTimeout( doSomething, 100 );
	}

	var result;

	// do some other stuff
	return result;
}

// were we able to do it right away?
if (doSomething()) {
	// handle next tasks right away
}
```

* the `setTimeout(..)` function returns a `numeric value` (the `unique identifier` of the `timer interval`, if you wanted to cancel it), but we want to `void` that out so that the return value of our function doesn't give a (CKL)(`false-positive`) with the `if` statement.

* (me: this) works the same but doesn't use the `void` operator:

```js
if (!APP.ready) {
	// try again later
	setTimeout( doSomething, 100 );
	return;
}
```

### Special Numbers
#### The Not Number, Number
* Any mathematic operation you perform without both operands being `number`s (or values that `can be interpreted` as regular `number`s in `base 10` or `base 16`) will result in the operation `failing to produce a valid` `number`, in which case you will get the `NaN` value [...] stands for "not a `number`", though [...] It would be much more accurate to think of `NaN` as being "invalid number," "failed number,".

```js
var a = 2 / "foo";		// NaN

typeof a === "number";	// true
```

* `NaN` is a kind of "sentinel value" [...] represents a special kind of `error condition` within the `number` set. The `error condition` is [...] "I tried to perform a mathematic operation but failed, so here's the failed `number` result instead."

* `NaN` is a very special value in that it's [...] never equal to itself.

* So how *do* we test for it, if we can't compare to `NaN` (since that comparison would always fail)?

```js
var a = 2 / "foo";

isNaN( a ); // true
```

* Not so fast [...] The `isNaN(..)` utility has a `fatal flaw` [...] (me: it) "test if the thing passed in is either not a `number` or is a `number`."

```js
var a = 2 / "foo";
var b = "foo";

a; // NaN
b; // "foo"

window.isNaN( a ); // true
window.isNaN( b ); // true -- ouch!
```

* `"foo"` is literally *not a `number`*, but it's `definitely not` the `NaN` value either! This `bug` has been in JS since the very beginning.

* As of `ES6`, finally a replacement utility [...] `Number.isNaN(..)`. A simple `polyfill` for it so that you can safely check `NaN` values [...] in `pre-ES6 browsers`.

```js
if (!Number.isNaN) {
	Number.isNaN = function(n) {
		return (
			typeof n === "number" &&
			window.isNaN( n )
		);
	};
}

var a = 2 / "foo";
var b = "foo";

Number.isNaN( a ); // true
Number.isNaN( b ); // false -- phew!
```

* we can implement a `Number.isNaN(..)` polyfill even easier, by taking advantage of that peculiar fact that `NaN` (me: is the `only` value) isn't equal to itself.

```js
if (!Number.isNaN) {
	Number.isNaN = function(n) {
		return n !== n;
	};
}
```

#### Infinities
* (WCRL).

#### Zeros
* JavaScript has both a normal zero `0` (otherwise known as a positive zero `+0`) *and* a negative zero `-0`.

* Besides being specified literally as `-0`, negative zero also results from certain mathematic operations. For example:

```js
var a = 0 / -3; // -0
var b = 0 * -3; // -0
```

* `Addition` and `subtraction` cannot result in a negative zero.

* A `negative zero` when examined in the `developer console` will usually reveal `-0`, though [...] some older browsers you encounter may still report it as `0`.

* However, if you try to `stringify` a `negative zero value`, it will `always` be reported as `"0"`.

```js
var a = 0 / -3;

// (some browser) consoles at least get it right
a;							// -0

// but the spec insists on lying to you!
a.toString();				// "0"
a + "";						// "0"
String( a );				// "0"

// strangely, even JSON gets in on the deception
JSON.stringify( a );		// "0"
```

* Interestingly, the `reverse operations` (going from `string` to `number`) don't lie:

```js
+"-0";				// -0
Number( "-0" );		// -0
JSON.parse( "-0" );	// -0
```

* (CKL)(The `JSON.stringify( -0 )` behavior of `"0"` is particularly strange when you observe that it's inconsistent with the reverse: `JSON.parse( "-0" )` reports `-0` as you'd correctly expect).

* In addition to stringification of negative zero being deceptive to hide its true value, the comparison operators are also (`intentionally`) configured to *lie*.

```js
var a = 0;
var b = 0 / -3;

a == b;		// true
-0 == 0;	// true

a === b;	// true
-0 === 0;	// true

0 > -0;		// false
a > b;		// false
```

* if you want to distinguish a `-0` from a `0` in your code, you `can't` just rely on what the `developer console outputs`, so you're going to have to be a bit more clever:

```js
function isNegZero(n) {
	n = Number( n );
	return (n === 0) && (1 / n === -Infinity);
}

isNegZero( -0 );		// true
isNegZero( 0 / -3 );	// true
isNegZero( 0 );			// false
```

* `why` do we need a negative zero [...] There are `certain applications` where developers use the `magnitude of a value` to represent one piece of information (like speed of movement per animation frame) and the (me: the `sign` before a number. For instance: `+3`)(`sign`) of that `number` to `represent another piece of information` (like the `direction of that movement`).

### Special Equality
* the `NaN` value and the `-0` value have `special behavior` when it comes to equality comparison. `NaN` is never equal to itself, so you have to use ES6's `Number.isNaN(..)` (or a polyfill). Similarly, `-0` lies and pretends that it's equal (even `===` strict equal [...]) to regular positive `0`, so you have to use the somewhat hackish `isNegZero(..)`.

* As of ES6, there's a new utility that can be used to test two values for `absolute equality`, without any of these exceptions. It's called `Object.is(..)`:

```js
var a = 2 / "foo";
var b = -3 * 0;

Object.is( a, NaN );	// true
Object.is( b, -0 );		// true

Object.is( b, 0 );		// false
```

* (me: a) simple polyfill for `Object.is(..)` for pre-ES6 environments:

```js
if (!Object.is) {
	Object.is = function(v1, v2) {
		// test for `-0`
		if (v1 === 0 && v2 === 0) {
			return 1 / v1 === 1 / v2;
		}
		// test for `NaN`
		if (v1 !== v1) {
			return v2 !== v2;
		}
		// everything else
		return v1 === v2;
	};
}
```

* `Object.is(..)` probably shouldn't be used in cases where `==` or `===` are known to be *safe* [...] `Object.is(..)` is mostly for these special cases of equality.

## Value vs. Reference
* A reference in JS points at a (`shared`) **value**, so if you have 10 [...] references, they are all always `distinct` references to a `single shared value`.

* in JavaScript [...] the *type* of the value *solely* controls whether that value will be assigned by `value-copy` or by `reference-copy`.

```js
var a = 2;
var b = a; // `b` is always a copy of the value in `a`
b++;
a; // 2
b; // 3

var c = [1,2,3];
var d = c; // `d` is a reference to the shared `[1,2,3]` value
d.push( 4 );
c; // [1,2,3,4]
d; // [1,2,3,4]
```

* Simple values (aka [...] primitives) are *always* assigned/passed by `value-copy`: `null`, `undefined`, `string`, `number`, `boolean`, and ES6's `symbol`.

* Compound values -- `object`s [...] *always* create a `copy of` the `reference` on assignment or passing.

* In the above snippet, because `2` is a [...] primitive [...] `b` is assigned another *copy* of the value. When changing `b`, you are in no way changing the value in `a`.

* But **both `c` and `d`** are separate `references` to the same `shared value` `[1,2,3]` [...] a compound value [...] `neither` `c` nor `d` [...] "owns" the `[1,2,3]` value [...] So, when using either reference to modify (`.push(4)`) the actual shared `array` value [...] it's affecting just the one shared value, and both references will reference the `newly modified value` `[1,2,3,4]`.

* you cannot use one reference to change where another reference is pointed:

```js
var a = [1,2,3];
var b = a;
a; // [1,2,3]
b; // [1,2,3]

// later
b = [4,5,6];
a; // [1,2,3]
b; // [4,5,6]
```

* When we make the assignment `b = [4,5,6]`, we are doing absolutely nothing to affect [...] `a` (me: which) is still referencing (`[1,2,3]`). To do that, `b` would have to be a pointer to `a` `rather than` a reference to the `array` -- but no such capability exists in JS!

* Remember: you cannot directly control/override value-copy vs. reference -- those semantics are controlled `entirely` by the type of the underlying value.

* To effectively pass a compound value (like an `array`) by `value-copy`, you need to `manually` make a copy of it, so that the reference passed doesn't still point to the original.

```js
a = [1, 2, 3] // (me)
foo( a.slice() );
```

* `slice(..)` with `no parameters` by default makes an `entirely new (shallow) copy` of the `array`. So, we pass in a reference only to the copied `array`, and thus `foo(..)` cannot affect the contents of `a`.

* To do the `reverse` -- pass a [...] primitive value in a way where its value updates can be seen (me: by the function receiving the value), kinda like a reference -- you have to (me: leverage the reference-copy feature of a compound value by wrapping) the value in another compound value.

```js
function foo(wrapper) {
	wrapper.a = 42;
}

var obj = {
	a: 2
};

foo( obj );

obj.a; // 42
```

* It may occur to you that if you wanted to pass in a reference to a [...] primitive value like `2`, you could just box the value in its `Number` object wrapper [...] It *is* true a copy of the reference to this `Number` object *will* be passed to the function, but `unfortunately`, having a reference to the [...] object is not going to give you the ability to modify the [...] primitive value, like you may expect:

```js
function foo(x) {
	x = x + 1;
	x; // 3
}

var a = 2;
var b = new Number( a ); // or equivalently `Object(a)`

foo( b );
console.log( b ); // 2, not 3
```

* The problem is that the underlying [...] primitive value is *not mutable* [...] If a `Number` object holds the [...] primitive value `2`, that exact `Number` object can `never be changed` to hold another value.

* When `x` is used in the expression `x + 1`, the underlying [...] primitive value `2` is unboxed (extracted) from the `Number` object automatically, so the line `x = x + 1` very subtly changes `x` `from` being a **shared reference to the `Number` object**, `to` just holding the [...] primitive value `3` as a result of the addition operation `2 + 1`. Therefore, `b` on the outside still references the original unmodified/immutable `Number` object holding the value `2`.

## Review

# Chapter 3: Natives
* a list of the most commonly used natives:

* `String()`
* `Number()`
* `Boolean()`
* `Array()`
* `Object()`
* `Function()`
* `RegExp()`
* `Date()`
* `Error()`
* `Symbol()` -- added in ES6!

* As you can see, these natives are actually built-in functions.

```js
var a = new String( "abc" );

typeof a; // "object" ... not "String"

a instanceof String; // true

Object.prototype.toString.call( a ); // "[object String]"
```

* The result of the constructor form of value creation (`new String("abc")`) is an object wrapper around the primitive (`"abc"`) value [...] Importantly, `typeof` shows that these objects [...] are subtypes of the `object` type.

* The point is, `new String("abc")` creates a string wrapper object around `"abc"`, not just the primitive `"abc"` value itself.

## Internal `[[Class]]`
* Values that are `typeof` `"object"` (such as an array) are additionally tagged with an internal `[[Class]]` property (think of this more as an internal *classification* `rather than` related to classes from `traditional class-oriented coding`). This property cannot be accessed `directly`, but can generally be revealed indirectly by borrowing the default `Object.prototype.toString(..)` method called against the value. For example:

```js
Object.prototype.toString.call( [1,2,3] );			// "[object Array]"

Object.prototype.toString.call( /regex-literal/i );	// "[object RegExp]"
```

* So, for the array in this example, the internal `[[Class]]` value is `"Array"`, and for the regular expression, it's `"RegExp"`. In most cases, this internal `[[Class]]` value corresponds to the `built-in native constructor` [...] related to the value, but that's not always the case.

What about primitive values? First, `null` and `undefined`:

```js
Object.prototype.toString.call( null );			// "[object Null]"
Object.prototype.toString.call( undefined );	// "[object Undefined]"
```

* there are no `Null()` or `Undefined()` native constructors, `but nevertheless` the `"Null"` and `"Undefined"` are the internal `[[Class]]` values exposed.

* But for the other simple primitives like `string`, `number`, and `boolean`, another behavior actually kicks in, which is usually called "`boxing`" (see "Boxing Wrappers" section next):

```js
Object.prototype.toString.call( "abc" );	// "[object String]"
Object.prototype.toString.call( 42 );		// "[object Number]"
Object.prototype.toString.call( true );		// "[object Boolean]"
```

* In this snippet, each of the simple primitives are `automatically boxed` by their respective `object wrappers`, which is why `"String"`, `"Number"`, and `"Boolean"` are revealed as the respective internal `[[Class]]` values.

* The behavior of `toString()` and `[[Class]]` as illustrated here has changed a bit from ES5 to ES6, but we cover those details in the *ES6 & Beyond* title of this series.

## Boxing Wrappers
* Primitive values don't have properties or methods, so to access `.length` or `.toString()` you need an object wrapper around the value. Thankfully, JS will automatically *box* (aka wrap) the primitive value to fulfill such accesses.

```js
var a = "abc";

a.length; // 3
a.toUpperCase(); // "ABC"
```

* if you're going to be accessing these properties/methods on your string values regularly, like a `i < a.length` condition in a `for` loop for instance, `it might seem` [...] to just have the object form of the value `from the start`, so the JS engine doesn't need to `implicitly` create it for you.

* But [...] that's a bad idea. Browsers `long ago` `performance-optimized` the common cases like `.length`, which means your program will *actually go slower* if you try to "preoptimize" by directly using the object form (which isn't on the optimized path).

### Object Wrapper Gotchas
* There are some `gotchas` with using the object wrappers directly.

```js
var a = new Boolean( false );

if (!a) {
	console.log( "Oops" ); // never runs
}
```

* The problem is that you've created an `object wrapper` around the `false` value, but objects themselves are "`truthy`" [... ]which is [...] contrary to normal expectation.

* you can (me: also) use the `Object(..)` function (no `new` keyword):

```js
var a = "abc";
var b = new String( a );
var c = Object( a );

typeof a; // "string"
typeof b; // "object"
typeof c; // "object"

b instanceof String; // true
c instanceof String; // true

Object.prototype.toString.call( b ); // "[object String]"
Object.prototype.toString.call( c ); // "[object String]"
```

## Unboxing
* use the `valueOf()` method:

```js
var a = new String( "abc" );
var b = new Number( 42 );
var c = new Boolean( true );

a.valueOf(); // "abc"
b.valueOf(); // 42
c.valueOf(); // true
```

* Unboxing can also happen `implicitly` [...] This process (coercion) will be covered in more detail in Chapter 4.

```js
var a = new String( "abc" );
var b = a + ""; // `b` has the unboxed primitive value "abc"

typeof a; // "object"
typeof b; // "string"
```

## Natives as Constructors
* For `array`, `object`, `function`, and `regular-expression` values, it's [...] preferred that you use the literal form for creating the values, `but` the literal form creates the `same sort of object` `as the constructor form does`.

### `Array(..)`
```js
var a = new Array( 1, 2, 3 );
a; // [1, 2, 3]

var b = [1, 2, 3];
b; // [1, 2, 3]
```

* The `Array(..)` constructor `does not` require the `new` keyword.

* The `Array` constructor has a special form where if `only one` `number` argument is passed [...] it's taken as a length to "presize the array" (well, sorta).

* This is a terrible idea [...] what you're creating is an otherwise **empty array**, `but` setting the `length` property of the array to the numeric value specified.

* An array with at least one "empty slot" in it is often called a "`sparse array`."

* this is yet another example where browser developer consoles vary on how they represent such an object.

```js
var a = new Array( 3 );

a.length; // 3
a;
```

* The serialization of `a` in Chrome is (at the time of writing): `[ undefined x 3 ]`. **This is really unfortunate.** It implies that there are three `undefined` values in the slots of this array, when `in fact` the slots **do not exist** [...] To visualize the difference, try this:

```js
var a = new Array( 3 );
var b = [ undefined, undefined, undefined ];
var c = [];
c.length = 3;

a;
b;
c;
```

* **Note:** Changing the `length` of an array to go beyond its number of actually-defined slot values, you implicitly introduce empty slots. In fact, you could even call `delete b[1]` in the above snippet, and it would introduce an empty slot into the middle of `b`.

* For `b` (in Chrome, currently), you'll find `[ undefined, undefined, undefined ]` as the serialization, as opposed to `[ undefined x 3 ]` for `a` and `c`.

* Worse than that, at the time of writing, Firefox reports `[ , , , ]` for `a` and `c` [...] `Three commas` implies `four slots`, not `three` slots like we'd expect [...] Firefox puts an extra `,` on the end of their serialization here because as of `ES5`, trailing commas in lists (array values, property lists, etc.) are `allowed` [...] a `[ , , , ]` value [...] actually [...] (me: is) `[ , , ]`.

* Unfortunately, it gets worse [...] `a` and `b` from the above code snippet actually behave the same in some cases **but differently in others**:

```js
a.join( "-" ); // "--"
b.join( "-" ); // "--"

a.map(function(v,i){ return i; }); // [ undefined x 3 ]
b.map(function(v,i){ return i; }); // [ 0, 1, 2 ]
```

* The `a.map(..)` call *fails* because the slots `don't actually exist` [...] `join(..)` works differently [...] we can think of it implemented sort of like this:

```js
function fakeJoin(arr,connector) {
	var str = "";
	for (var i = 0; i < arr.length; i++) {
		if (i > 0) {
			str += connector;
		}
		if (arr[i] !== undefined) {
			str += arr[i];
		}
	}
	return str;
}

var a = new Array( 3 );
fakeJoin( a, "-" ); // "--"

// (me)
var a = [1, 2];
fakeJoin( a, "-" ); // "1-2"
```

* to *actually* create an array of actual `undefined` values (not just "empty slots")

```js
var a = Array.apply( null, { length: 3 } );
a; // [ undefined, undefined, undefined ]
```

* The first argument is a `this` object binding [...] which we don't care about here, so we set it to `null`. The second argument is supposed to be an array (or [...] an "array-like object"). The `contents of` this "array" are "spread" out `as arguments` to the function in question.

* Inside of `apply(..)`, we can envision there's another `for` loop [...] that goes from `0` `up to`, but not including, `length` (`3` in our case) [...] For each index, it retrieves that key from the object. So if the `array-object parameter` was named `arr` internally [...] the property access would [...] be `arr[0]`, `arr[1]`, and `arr[2]`. Of course, none of those properties exist on the [...] object value, so all three of those property accesses would return the value `undefined`.

* In other words, it ends up calling `Array(..)` basically like this: `Array(undefined,undefined,undefined)`.

### `Object(..)`, `Function(..)`, and `RegExp(..)`
* There's practically no reason to ever use the `new Object()` constructor form.

* The `Function` constructor is helpful only in the rarest of cases, where you need to `dynamically` define a function's parameters and/or its function body.

* Regular expressions defined in the literal form (`/^a*b+/g`) are [...] preferred, not just for ease of syntax but for performance reasons -- the JS engine precompiles and caches them before code execution. Unlike the other constructor forms we've seen so far, `RegExp(..)` has some reasonable utility: to `dynamically define the pattern` for a regular expression.

```js
var name = "Kyle";
var namePattern = new RegExp( "\\b(?:" + name + ")+\\b", "ig" );

var matches = someText.match( namePattern );
```

### `Date(..)` and `Error(..)`
* The `Date(..)` and `Error(..)` native constructors are much more useful than the other natives, because there is no literal form for either.

* To create a date object value, you must use `new Date()`. The `Date(..)` constructor accepts optional arguments to specify the date/time to use, but if omitted, the current date/time is assumed.

* By far the most common reason you construct a date object is to get the current `timestamp value` (a signed integer number of `milliseconds` since Jan 1, 1970). You can do this by calling `getTime()` on a date object instance [...] But an even easier way is to just call the static helper function defined as of ES5: `Date.now()`.

* If you call `Date()` without `new`, you'll get back a `string` representation of the date/time at that moment. The exact form of this representation is not specified in the language spec, though browsers tend to agree on something close to: `"Fri Jul 18 2014 00:31:02 GMT-0500 (CDT)"`.

* The `Error(..)` constructor [...] behaves the same with the `new` keyword present or omitted.

* The main reason you'd want to create an error object is that it captures the `current execution stack context` into the object (in most JS engines, revealed as a read-only `.stack` property once (me: the object) constructed). This `stack context` `includes` the function call-stack and the line-number where the error object was created, which makes debugging that error much easier.

* Error object instances generally have at least a `message` property, and sometimes other properties (which you should treat as read-only), like `type`. However, other than inspecting the above-mentioned `stack` property, it's usually best to just call `toString()` on the error object [...] to get a friendly-formatted error message.

* in addition to the general `Error(..)` native, there are several other specific-error-type natives: `EvalError(..)`, `RangeError(..)`, `ReferenceError(..)`, `SyntaxError(..)`, `TypeError(..)`, and `URIError(..)`. But it's very rare to manually use these specific error natives.

### `Symbol(..)`
* New as of ES6, an additional primitive value type has been added [...] "Symbol" [...] special "`unique`" (`not strictly guaranteed`!) values that can be used as properties on objects with `little fear of any collision`. They're primarily designed for `special built-in behaviors` of ES6 constructs, but you can also define your own symbols.

```js
// (me)
a = Symbol( "my own symbol" );
b = Symbol( "my own symbol" );

b === a // false
a       // Symbol(my own symbol)
b       // Symbol(my own symbol)
```

* you cannot see or access the `actual` value of a symbol from your program, nor from the developer console.

* There are several `predefined symbols` in ES6, accessed as static properties of the `Symbol` function object, like `Symbol.create`, `Symbol.iterator`, etc.

```js
obj[Symbol.iterator] = function(){ /*..*/ };
```

* To define your own custom symbols, use the `Symbol(..)` native. The `Symbol(..)` native "constructor" is unique in that you're `not allowed` to use `new` with it, as doing so will throw an error.

```js
var mysym = Symbol( "my own symbol" );
mysym;				// Symbol(my own symbol)
mysym.toString();	// "Symbol(my own symbol)"
typeof mysym; 		// "symbol"

var a = { };
a[mysym] = "foobar";

Object.getOwnPropertySymbols( a );
// [ Symbol(my own symbol) ]
```

* While symbols are not actually private (`Object.getOwnPropertySymbols(..)` reflects on the object and reveals the symbols quite publicly), using them for private or special properties is likely their primary use-case. For most developers, they may take the place of property names with `_` underscore prefixes, which are almost always by convention signals to say, "hey, this is a private/special/internal property, so leave it alone!"

* `Symbol`s are *not* `object`s, they are simple [...] primitives.

### Native Prototypes
* By documentation convention, `String.prototype.XYZ` is shortened to `String#XYZ`, and likewise for all the other `.prototype`s.
  * `String#indexOf(..)`: find the position in the string of another substring
  * `String#charAt(..)`: access the character at a position in the string
  * `String#substr(..)`, `String#substring(..)`, and `String#slice(..)`: extract a portion of the string as a new string
  * `String#toUpperCase()` and `String#toLowerCase()`: create a new string that's converted to either uppercase or lowercase
  * `String#trim()`: create a new string that's stripped of any trailing or leading whitespace

* some of the (me: `prototype` properties of native functions)(native prototypes) aren't *just* plain objects:

```js
typeof Function.prototype;			// "function"
Function.prototype();				// it's an empty function!

RegExp.prototype.toString();		// "/(?:)/" -- empty regex
"abc".match( RegExp.prototype );	// [""]
```

* you can even modify these native prototypes.

```js
Array.isArray( Array.prototype );	// true
Array.prototype.push( 1, 2, 3 );	// 3
Array.prototype;					// [1,2,3]

// don't leave it that way, though, or expect weirdness!
// reset the `Array.prototype` to empty
Array.prototype.length = 0;
```

* As you can see, `Function.prototype` is a function, `RegExp.prototype` is a regular expression, and `Array.prototype` is an array. Interesting and cool, huh?

#### Prototypes As Defaults
* `Function.prototype` being an `empty function`, `RegExp.prototype` being an "empty" [...] regex, and `Array.prototype` being an empty array, make them all nice "default" values to assign to variables.

```js
function isThisCool(vals,fn,rx) {
	vals = vals || Array.prototype;
	fn = fn || Function.prototype;
	rx = rx || RegExp.prototype;

	return rx.test(
		vals.map( fn ).join( "" )
	);
}

isThisCool();		// true

isThisCool(
	["a","b","c"],
	function(v){ return v.toUpperCase(); },
	/D/
);					// false
```

* One minor side-benefit of this approach is that the `.prototype`s are `already created and built-in`, thus created *only once*. By contrast, using `[]`, `function(){}`, and `/(?:)/` values themselves for those defaults would (likely, `depending on engine implementations`) be `recreating` those values [...] That could be memory/CPU wasteful.

* Also, be `very careful` not to use `Array.prototype` as a default value **that will subsequently be modified**. In this example, `vals` is used `read-only`, but if you were to instead make in-place changes to `vals`, you would actually be modifying `Array.prototype` itself.

* While we're pointing out these native prototypes and some usefulness, be `cautious` of relying on [...] (me: and) modifying them [...] See `Appendix A`.

## Review

# Chapter 4: Coercion
## Converting Values
* JavaScript coercions `always` result in one of the [...] primitive [...] values [...] There is no coercion that results in a complex value like `object` or `function`. Chapter 3 covers "boxing," which wraps [...] primitive values in their `object` counterparts, but this is `not really coercion` in an accurate sense.

## Abstract Value Operations
* The `ES5` spec in section 9 defines several "`abstract operations`" (fancy spec-speak for "`internal-only operation`") with the rules of value conversion.

### `ToString`
* When any `non-string` value is coerced to a `string` representation, the conversion is handled by the `ToString` abstract operation.

* Built-in primitive values have natural stringification: `null` becomes `"null"`, `undefined` becomes `"undefined"` and `true` becomes `"true"`. `number`s are [...] (me: as what) you'd expect, but as we discussed in Chapter 2, very small or very large `numbers` are represented in exponent form:

* For regular objects [...] the default `toString()` (located in `Object.prototype.toString()`) will return the *internal `[[Class]]`* (see `Chapter 3`) [...] for instance `"[object Object]"` [...] if an object has its own `toString()` method on it, and you use that object in a `string-like` way, its `toString()` will automatically be called, and the `string` result of that call will be used instead.

* Arrays have an overridden default `toString()` that stringifies `as` the (string) concatenation of all its values [...] with `","` in between each value:

```js
var a = [1,2,3];

a.toString(); // "1,2,3"
```

* `toString()` can either be called explicitly, or it will automatically be called if a `non-string` is used in a `string context`.

#### JSON Stringification
* Another task that seems awfully related to `ToString` is when you use the `JSON.stringify(..)` [...] this stringification is not exactly the same thing as coercion.

* For most simple values, JSON stringification behaves basically the same as `toString()` conversions.

```js
JSON.stringify( 42 );	// "42"
JSON.stringify( "42" );	// ""42"" (a string with a quoted string value in it)
JSON.stringify( null );	// "null"
JSON.stringify( true );	// "true"
```

* Any *JSON-safe* value (me: Any value that can be represented `validly` in a JSON representation) can be stringified by `JSON.stringify(..)`.

* It may be easier to consider values that are **not** JSON-safe. Some examples: `undefined`s, `function`s, (ES6+) `symbol`s, and `object`s with `circular references` (where property references in an object structure create a never-ending cycle through each other). These are all illegal values for a standard JSON structure, mostly because they **aren't portable to other languages** that consume JSON values.

* The `JSON.stringify(..)` utility will **automatically omit** `undefined`, `function`, and `symbol` values when it comes across them. If such a value is found in an `array`, that value is replaced by `null` [...] If found as a property of an `object`, that property will simply be `excluded`.

Consider:

```js
JSON.stringify( undefined );					// undefined
JSON.stringify( function(){} );					// undefined

JSON.stringify( [1,undefined,function(){},4] );	// "[1,null,null,4]"
JSON.stringify( { a:2, b:function(){} } );		// "{"a":2}"
```

* if you try to `JSON.stringify(..)` an `object` with circular reference(s) in it, an error will be thrown.

* JSON stringification has the `special behavior` that if an `object` value has a `toJSON()` method defined, `this method` will be called `first` to get a value to use for serialization.

* `If you [...] JSON stringify an object` that may contain `illegal JSON value(s)`, or if you just have values in the `object` that aren't appropriate for the serialization, you should define a `toJSON()` method for it that returns a *JSON-safe* version of the `object` [...] For example:

```js
var o = { };

var a = {
	b: 42,
	c: o,
	d: function(){}
};

// create a circular reference inside `a`
o.e = a;

// would throw an error on the circular reference
// JSON.stringify( a );

// define a custom JSON value serialization
a.toJSON = function() {
	// only include the `b` property for serialization
	return { b: this.b };
};

JSON.stringify( a ); // "{"b":42}"
```

* a very common misconception that `toJSON()` should return a JSON stringification representation. That's probably incorrect [...] `toJSON()` should return the actual regular value (of whatever type) that's appropriate, and `JSON.stringify(..)` itself will handle the stringification.

* An optional second argument can be passed to `JSON.stringify(..)` that is called *replacer*. This argument can either be an `array` or a `function`. It's used to customize the recursive serialization of an `object` by providing a filtering mechanism for which properties should and should not be included, in a similar way to how `toJSON()` can prepare a value for serialization.

* If *replacer* is an `array`, it should be an `array` of `string`s, each of which will specify a `property name` that is allowed to be included in the serialization of the `object`. If a property exists that isn't in this list, it will be skipped.

* If *replacer* is a `function`, it will be called once for the `object` itself, and then once for each property in the `object`, and each time is passed two arguments, *key* and *value*. To skip a *key* in the serialization, return `undefined`. Otherwise, return the *value* provided.

```js
var a = {
	b: 42,
	c: "42",
	d: [1,2,3]
};

JSON.stringify( a, ["b","c"] ); // "{"b":42,"c":"42"}"

JSON.stringify( a, function(k,v){
	if (k !== "c") return v;
} );
// "{"b":42,"d":[1,2,3]}"
```

* In the `function` *replacer* case, the key argument `k` is `undefined` for the `first call` (where the `a` object `itself` is being passed in). The `if` statement **filters out** the property named `"c"`. Stringification is `recursive`, so the `[1,2,3]` array has each of its values (`1`, `2`, and `3`) passed as `v` to *replacer*, with indexes (`0`, `1`, and `2`) as `k`.

* A third optional argument can also be passed to `JSON.stringify(..)`, called *space*, which is used as `indentation` for prettier human-friendly output. *space* can be a positive integer to indicate how many space characters should be used at each indentation level. Or, *space* can be a `string`, in which case up to the first ten characters of its value will be used for each indentation level.

```js
var a = {
	b: 42,
	c: "42",
	d: [1,2,3]
};

JSON.stringify( a, null, 3 );
// "{
//    "b": 42,
//    "c": "42",
//    "d": [
//       1,
//       2,
//       3
//    ]
// }"

JSON.stringify( a, null, "-----" );
// "{
// -----"b": 42,
// -----"c": "42",
// -----"d": [
// ----------1,
// ----------2,
// ----------3
// -----]
// }"
```

1. `string`, `number`, `boolean`, and `null` values all stringify for JSON basically the same as how they coerce to `string` values via the rules of the `ToString` abstract operation.
2. If you pass an `object` value to `JSON.stringify(..)`, and that `object` has a `toJSON()` method on it, `toJSON()` is automatically called to (sort of) "coerce" the value to be *JSON-safe* before stringification.

### ToNumber
* If any `non-number` value is used in a way that requires it to be a `number`, such as a `mathematical operation`, the ES5 spec defines the `ToNumber` abstract operation

* For example, `true` becomes `1` and `false` becomes `0`. `undefined` becomes `NaN`, but (curiously) `null` becomes `0`.

* `ToNumber` for a `string` value essentially works [...] like the `rules/syntax for numeric literals` (see `Chapter 3` in the section talking about the `Number` native function) If it fails, the result is `NaN` [...] One [...] difference is that `0`-`prefixed octal numbers` are not handled as octals (just as normal `base-10 decimals`) in this operation, though such octals are valid as `number` literals (see Chapter 2).

* **Note:** The `differences` between `number literal` `grammar` and `ToNumber` on a string value are `subtle` [...] and thus will not be covered further here. Consult section 9.3.1 of the ES5 spec for more information.

* Objects (and arrays) (me: which are passed to the `ToNumber function`) will first be converted to their `primitive value equivalent` (me: see how in the next argument), and the resulting value (if a primitive but `not already` a `number`) is coerced to a `number` according to the `ToNumber` rules just mentioned.

* To convert to this `primitive value equivalent`, (me: the rest of this argument indicate how the `ToPrimitive` works) the `ToPrimitive` abstract operation [...] will consult the value (using the internal `DefaultValue` operation -- ES5 spec, section 8.12.8) in question to see if it has a `valueOf()` method. If `valueOf()` is available and it returns a primitive value, *that* value is used for the coercion. If not, but `toString()` is available, (me: the `toString()` function)(it) will provide the value for the coercion [...] If neither operation can provide a primitive value, a `TypeError` is thrown.

* As of ES5, you can create such a `noncoercible object` (me: an object without `valueOf()` and `toString()`) if it has a `null` value for its `[[Prototype]]`, typically created with `Object.create(null)`.

Consider:

```js
var a = {
	valueOf: function(){
		return "42";
	}
};

var b = {
	toString: function(){
		return "42";
	}
};

var c = [4,2];
c.toString = function(){
	return this.join( "" );	// "42"
};

Number( a );			// 42
Number( b );			// 42
Number( c );			// 42
Number( "" );			// 0
Number( [] );			// 0
Number( [ "abc" ] );	// NaN
```

### `ToBoolean`
* First and foremost, JS has actual keywords `true` and `false`, and they behave exactly as you'd expect of `boolean` values. It's a common misconception that the values `1` and `0` are identical to `true`/`false`.

#### Falsy Values
* All of JavaScript's values can be divided into two categories:
  1. values that will become `false` if coerced to `boolean`.
  2. everything else [...] `true`.

* "falsy" values list:
   * `undefined`
   * `null`
   * `false`
   * `+0`, `-0`, and `NaN`
   * `""`

* if a value is *not* on that list (me: then they are `truthy`) [...] But JS doesn't really define a "truthy" list [...] mostly the spec just implies: **anything not explicitly on the falsy list is therefore truthy.**

#### Falsy Objects
* **all `objects` (me: are) truthy** [...] There should be no such thing as a "`falsy object`." [...] You might be tempted to think it means an object wrapper [...] around a falsy value [...] But don't fall into that *trap*.

```js
var a = new Boolean( false );
var b = new Number( 0 );
var c = new String( "" );
```

* all three values here are objects [...] wrapped around [...] falsy values. But do these objects behave as `true` or as `false`?

```js
var d = Boolean( a && b && c );

d; // true
```

* So, all three behave as `true`, as that's the only way `d` could end up as `true`.

* **Tip:** Notice the `Boolean( .. )` wrapped around the `a && b && c` expression -- you might wonder why that's there. We'll come back to that later in this chapter [...] (me: but) try for yourself what `d` will be if you just do `d = a && b && c` without the `Boolean( .. )` call!

* So, if **"falsy objects" are not just `objects wrapped around falsy values`**, what the heck are they? [...] (me: they are) **not actually part of JavaScript itself** [...] There are [...] cases where browsers have created their own [...] values behavior [...] (me: such as) "`falsy objects`," `on top of` regular JS [...] A "falsy object" is a value that looks and acts like a `normal` object (properties, etc.), but when you coerce it to a `boolean`, it coerces to a `false` value.

* **Why!?**

* The most well-known case is `document.all`: `an array-like (object)` provided [...] *by the DOM* (not the JS engine itself) [...] exposes elements in your page [...] It *used* to [...] act truthy. But not anymore [...] `document.all` [...] has [...] been deprecated [...] (me: But) `if (document.all) { /* it's IE */ }` code is still out there [...] So, we can't remove `document.all` completely [...] so [...] new, standards-compliant code logic [...] `document.all` is falsy!"

#### Truthy Values
* the truthy list is `infinitely long`. It's impossible to make such a list. You can only make a `finite falsy list` and consult *it*.

## Explicit Coercion
### Explicitly: Strings <--> Numbers
* To coerce between `string`s and `number`s, we use the built-in `String(..)` and `Number(..)` functions [...] but **very importantly**, we do not use the `new` keyword in front of them. As such, we're `not creating object wrappers` [...] Instead, we're actually *explicitly coercing* between the two types:

```js
var a = 42;
var b = String( a );

var c = "3.14";
var d = Number( c );

b; // "42"
d; // 3.14
```

* `String(..)` coerces from `any other value` to a primitive `string` value, using the rules of the `ToString` operation [...] `Number(..)` coerces from any other value to a primitive `number` value, using the rules of the `ToNumber` operation.

* Besides `String(..)` and `Number(..)`, there are other ways to "explicitly" convert these values between `string` and `number`:

```js
var a = 42;
var b = a.toString();

var c = "3.14";
var d = +c;

b; // "42"
d; // 3.14
```

* Calling `a.toString()` is [...] explicit [...] but [...] `toString()` cannot be called on a *primitive* value like `42`. So JS `automatically "boxes"` [...] `42` in an object wrapper, so that `toString()` can be called against the object.

* `+c` here is showing the *unary operator* form (`operator with only one operand`) of the `+` operator. Instead of performing mathematic addition [...] the unary `+` explicitly coerces its operand (`c`) to a `number` value.

* The unary `-` operator also coerces like `+` does, but it also `flips` the sign of the number. However, you `cannot` put two `--` next to each other to unflip the sign, as that's parsed as the `decrement operator`. Instead, you would need to do: `- -"3.14"` [...] result in coercion to `3.14`.

#### `Date` To `number`
* Another common usage of the unary `+` operator is to coerce a `Date` object into a `number`, because the result is the `unix timestamp` (milliseconds elapsed since 1 January 1970 00:00:00 UTC) representation of the date/time value:

```js
var d = new Date( "Mon, 18 Aug 2014 08:53:06 CDT" );

+d; // 1408369986000
```

* A noncoercion approach is perhaps [...] preferable, as it's even more explicit:

```js
var timestamp = new Date().getTime();
// var timestamp = (new Date()).getTime();
// var timestamp = (new Date).getTime();
```

```js
var timestamp = Date.now();
```

#### The Curious Case of the `~`
* `~` operator (aka "`bitwise NOT`"). Many of those who even understand what it does will often times still want to avoid it.

* (WCRL).

##### Truncating Bits
* (WCRL).

### Explicitly: Parsing Numeric Strings
* A similar outcome to coercing a `string` to a `number` can be achieved by parsing a `number` out of a `string`'s character contents. There are, however, distinct differences between this parsing and the type conversion we examined above.

```js
var a = "42";
var b = "42px";

Number( a );	// 42
parseInt( a );	// 42

Number( b );	// NaN
parseInt( b );	// 42
```

* Parsing a numeric value out of a string is *tolerant* of non-numeric characters (me: it just stops parsing **left-to-right** when encountered) whereas coercion is *not tolerant* and fails resulting in the `NaN` value.

* `parseInt(..)` has a twin, `parseFloat(..)`, which [...] pulls out a floating-point number from a string.

* If you pass a non-`string`, the value you pass will automatically be coerced to a `string` first [...] It's a really bad idea to rely upon such a behavior in your program.

* Prior to `ES5`, another gotcha existed with `parseInt(..)`, which was the source of many JS programs' bugs. If you didn't pass a `second argument` to indicate which `numeric base` (aka `radix`) to use for interpreting the numeric `string` contents, `parseInt(..)` would look at `the beginning character(s)` to make a guess.

* If the first two characters were `"0x"` or `"0X"`, the guess [...] was that you wanted to interpret the `string` as a hexadecimal (base-16) `number`. Otherwise, if the first character was `"0"`, the guess [...] was that you wanted to interpret the `string` as an octal (base-8) `number`.

* Hexadecimal `string`s (with the leading `0x` or `0X`) aren't [...] easy to get mixed up. But the `octal number guessing` [...] (me: is) common.

```js
var hour = parseInt( selectedHour.value );
var minute = parseInt( selectedMinute.value );

console.log( "The time you selected was: " + hour + ":" + minute);
```

* Seems harmless, right? Try selecting `08` for the hour and `09` for the minute. You'll get `0:0`. Why? because `neither` `8` nor `9` are valid characters in `octal base-8`.

* **always pass `10` as the second argument**

* `As of ES5`, `parseInt(..)` no longer `guesses octal`. `Unless you say otherwise`, it assumes `base-10` (or base-16 for `"0x"` `prefixes`).

#### Parsing Non-Strings
```js
parseInt( 1/0, 19 ); // 18
```

* `parseInt( 1/0, 19 )` [...] essentially `parseInt( "Infinity", 19 )` [...] The first character is `"I"`, which is value `18` in the silly `base-19`. The second character `"n"` is not in the valid set of numeric characters, and as such the parsing simply `politely stops`, just like when it ran across `"p"` in `"42px"` [...] The result? (me: of the `parseInt` function is) `18`.

* Other examples of this behavior with `parseInt(..)`.

```js
parseInt( 0.000008 );		// 0   ("0" from "0.000008")
parseInt( 0.0000008 );		// 8   ("8" from "8e-7")
parseInt( false, 16 );		// 250 ("fa" from "false")
parseInt( parseInt, 16 );	// 15  ("f" from "function..")

parseInt( "0x10" );			// 16
parseInt( "103", 2 );		// 2
```

### Explicitly: * --> Boolean
* Just like with `String(..)` and `Number(..)` above, `Boolean(..)` (without the `new`, of course!) is an explicit way of forcing the `ToBoolean` coercion:

```js
var a = "0";
var b = [];
var c = {};

var d = "";
var e = 0;
var f = null;
var g;

Boolean( a ); // true
Boolean( b ); // true
Boolean( c ); // true

Boolean( d ); // false
Boolean( e ); // false
Boolean( f ); // false
Boolean( g ); // false
```

* Just like the unary `+` operator coerces a value to a `number` [...] the unary `!` negate operator `explicitly coerces` a value to a `boolean`. The *problem* is that it also `flips` the value from truthy to falsy or vice versa. So, the most common way JS developers explicitly coerce to `boolean` is to use the `!!` double-negate operator.

```js
var a = "0";
var b = [];
var c = {};

var d = "";
var e = 0;
var f = null;
var g;

!!a;	// true
!!b;	// true
!!c;	// true

!!d;	// false
!!e;	// false
!!f;	// false
!!g;	// false
```

* Any of these `ToBoolean` coercions would happen *implicitly* without the `Boolean(..)` or `!!`, if used in a `boolean` context such as an `if (..) ..` statement.

* If you come to JavaScript from Java, you may recognize this idiom:

```js
var a = 42;

var b = a ? true : false;
```

## Implicit Coercion
### Simplifying Implicitly
### Implicitly: Strings <--> Numbers
* The `+` operator is overloaded to serve the purposes of both `number` addition and `string` concatenation. So how does JS know which type of operation you want to use?

```js
var a = "42";
var b = "0";

var c = 42;
var d = 0;

a + b; // "420"
c + d; // 42
```

* It's a common misconception that the difference is whether one or both of the operands is a `string`, as that means `+` will assume `string` concatenation. While that's partially true, it's more complicated than that.

```js
var a = [1,2];
var b = [3,4];

a + b; // "1,23,4"
```

* Neither of these operands is a `string`, but clearly they were both coerced to `string`s and then the `string` concatenation kicked in. So what's really going on?
	* (me: According to ES5 spec section 11.6.1, the `+` algorithm [...] first calls the `ToPrimitive` abstract operation (section 9.1) on (me: `both of the operands`), if an operand is a primitive value (`undefined`, `null`, `boolean`, `number`, `string`) the `toPrimitive` operation returns the operand itself, but if an operand is an `object`, then the operation calls the (me: read the `section 8.12.8` for a more detail look at the implemtation of the `[[DefaultValue]]` method as well as its hint)(`[[DefaultValue]]` algorithm with a context `hint` of `number`) then if either the value returned by the `DefaultValue` operation is a string, the `string` concatenation of the 2 `DefaultValue operation-returned values`, now converted to `strings` by the `ToString` method kicks in. `Otherwise`, a `numberic addition`, the addition of the 2 `DefaultValue operation-returned values`, now converted to number by the `ToNumber` method is performed)

	* this operation (me: with the 2 `objects`, indeed 2 `arrays`) is now identical to how the `ToNumber` abstract operation handles `object`s (see the "`ToNumber`"" section earlier). The `valueOf()` operation on the `array` will fail to produce a simple primitive, so it then falls to a `toString()` representation. The two `array`s thus become `"1,2"` and `"3,4"`, respectively. Now, `+` concatenates the two `string`s as you'd normally expect: `"1,23,4"`.

* simplified explanation: if either operand to `+` is a `string` (or becomes one with the `above steps`!), the operation will be `string` concatenation. `Otherwise`, it's `always numeric addition`.

* **Note:** A commonly cited coercion gotcha is `[] + {}` vs. `{} + []`, as those two expressions result, respectively, in `"[object Object]"` and `0`. There's more to it, though, and we cover those details in "Blocks" in `Chapter 5`.

* What's that mean for *implicit* coercion? [...] You can coerce a `number` to a `string` simply by "adding" the `number` and the `""` empty `string`

```js
var a = 42;
var b = a + "";

b; // "42"
```

* Comparing this *implicit* coercion of `a + ""` to our earlier example of `String(a)` *explicit* coercion (me: with `a` returning an `object` only, check the `(me)` section in the example below), there's one additional quirk to be aware of. Because of how the `ToPrimitive` abstract operation works, `a + ""` invokes `valueOf()` on the `a` value, whose return value is then finally converted to a `string` via the internal `ToString` abstract operation. But `String(a)` just invokes `toString()` directly.

* Both approaches ultimately result in a `string`, but if you're using an `object` instead of a regular primitive `number` value, you may not necessarily get the *same* `string` value!

```js
var a = {
	valueOf: function() { return 42; },
	toString: function() { return 4; }
};

a + "";			// "42"

String( a );	// "4"
```

* How can we *implicitly coerce* from `string` to `number`?

```js
var a = "3.14";
var b = a - 0;

b; // 3.14
```

* The `-` operator is defined `only for numeric` subtraction, so `a - 0` forces `a`'s value to be coerced to a `number`. While far less common, `a * 1` or `a / 1` would accomplish the same result, as those operators are also `only defined for numeric operations`.

* What about `object` values with the `-` operator? Similar (me: to how the `toNumber` method works -- see the `ToNumber` section earlier)

```js
var a = [3];
var b = [1];

a - b; // 2
```

* Both `array` values have to become `number`s, but they end up first being coerced to `strings` (using the expected `toString()` serialization), and then are coerced to `number`s.

### Implicitly: Booleans --> Numbers
```js
function onlyOne(a,b,c) {
	return !!((a && !b && !c) ||
		(!a && b && !c) || (!a && !b && c));
}

var a = true;
var b = false;

onlyOne( a, b, b );	// true
onlyOne( b, a, b );	// true

onlyOne( a, b, a );	// false
```

* This `onlyOne(..)` utility should only return `true` if exactly one of the arguments is `true` / truthy [...] But what if we needed that utility to be able to handle [...] twenty flags [...] It's pretty difficult to [...] implementing [...] But here's where coercing the `boolean` values to `number`s (`0` or `1`, obviously) can greatly help:

```js
function onlyOne() {
	var sum = 0;
	for (var i=0; i < arguments.length; i++) {
		// skip falsy values. same as treating
		// them as 0's, but avoids NaN's.
		if (arguments[i]) {
			sum += arguments[i];
		}
	}
	return sum == 1;
}

var a = true;
var b = false;

onlyOne( b, a );		// true
onlyOne( b, a, b, b, b );	// true

onlyOne( b, b );		// false
onlyOne( b, a, b, b, b, a );	// false
```

* If one and only one value in the `arguments` list is `true`, then the numeric sum will be `1`, otherwise the sum will not be `1` and thus the desired condition is not met [...] We could of course do this with *explicit* coercion.

```js
function onlyOne() {
	var sum = 0;
	for (var i=0; i < arguments.length; i++) {
		sum += Number( !!arguments[i] );
	}
	return sum === 1;
}
```

* `!!arguments[i]` [...] so you could pass non-`boolean` values in [...] and it would still work as expected (otherwise you'd end up with `string` concatenation and the logic would be incorrect).

* you could easily make `onlyTwo(..)` or `onlyFive(..)` variations by simply changing the final comparison from `1`, to `2` or `5`, respectively.

### Implicitly: * --> Boolean
1. The test expression in an `if (..)` statement.
2. The test expression (second clause) in a `for ( .. ; .. ; .. )` header.
3. The test expression in `while (..)` and `do..while(..)` loops.
4. The test expression (first clause) in `? :` ternary expressions.
5. The left-hand operand (which serves as a test expression -- see below!) to the `||` ("logical or") and `&&` ("logical and") operators.

* Any value used in these contexts that is not already a `boolean` will be *implicitly* coerced to a `boolean` using the rules of the `ToBoolean` abstract operation.

### Operators `||` and `&&`
* these operators shouldn't even be called "logical ___ operators" [...] a more accurate (if more clumsy) name, I'd call them "selector operators," or more completely, "operand selector operators." [...] Because they don't actually result in a *logic* value [...] in JavaScript, as [...] in some other languages [...] (me: instead) **they select one of the two operand's values**.

* Both `||` and `&&` operators perform a `boolean` test on the **first operand** [...] If the operand is not already `boolean` [...] a normal `ToBoolean` coercion occurs.

* For the `||` operator, if the test is `true`, the `||` expression results in the value of the *first operand* [...] If the test is `false`, the `||` expression results in the value of the *second operand* [...] `Inversely`, for the `&&` operator.

* `a || b` "roughly equivalent" to `a ? a : b` because the outcome is identical, but there's a nuanced difference. In `a ? a : b`, if `a` was a more complex expression (like for instance one that might have side effects like calling a `function`, etc.), then the `a` expression would possibly be `evaluated twice` (if the first evaluation was truthy). By contrast, for `a || b`, the `a` expression is evaluated only once [...] that value is used both for the coercive test as well as the result value.

* An extremely common and helpful usage of (me: `||`).

```js
function foo(a,b) {
	a = a || "hello";
	b = b || "world";

	console.log( a + " " + b );
}

foo();					// "hello world"
foo( "yeah", "yeah!" );	// "yeah yeah!"
```

* **Be careful**, though!

```js
foo( "That's it!", "" ); // "That's it! world" <-- Oops!
```

* There's another idiom that is quite a bit less commonly [...] this usage is sometimes called the "`guard operator`".

```js
function foo() {
	console.log( a );
}

var a = 42;

a && foo(); // 42
```

* **Note:** Both the `a = b || "something"` and `a && b()` idioms rely on short circuiting behavior, which we cover in more detail in Chapter 5.

* (me: no need an example for this argument) `a && (b || c)` expression *actually* results in `"foo"`, not `true`. So, the `if` statement *then* forces the `"foo"` value to coerce to a `boolean`.

### Symbol Coercion
* `ES6` Symbols introduce a gotcha into the coercion system.

```js
var s1 = Symbol( "cool" );
String( s1 );					// "Symbol(cool)"

var s2 = Symbol( "not cool" );
s2 + "";						// TypeError

// (me)
Number(s2) // TypeError
```

* `symbol` values cannot coerce to `number` at all (throws an error either way), `but strangely` they can both *explicitly* and *implicitly* coerce to `boolean` (always `true`).

## Loose Equals vs. Strict Equals
* `common misconception` [...] "`==` checks values for equality and `===` checks both values and types for equality." [...] The correct description is: "`==` allows coercion in the equality comparison and `===` disallows coercion."

### Equality Performance
* `==` is the one *doing more work* because it has to follow through the steps of `coercion` if the types are different [...] this has [...] to do with performance [...] While it's [...] take *a little bit* of processing time, it's mere microseconds (yes, that's millionths of a second!) [...] If you're comparing two values of the `same types`, `==` and `===` use the identical algorithm.

* **Note:** The implication here then is that both `==` and `===` check the `types` of their operands. The difference is in how they respond if the types don't match.

### Abstract Equality
* The `==` operator's behavior is defined as "The Abstract Equality Comparison Algorithm" in section 11.9.3 of the ES5 spec.

* the first clause (11.9.3.1) says, if the two values being compared are of the `same type`, they are simply and naturally compared via `Identity` as you'd expect. For example, `42` is only equal to `42`, and `"abc"` is only equal to `"abc"` [...] Some minor exceptions to normal expectation to be aware of:
  * `NaN` is never equal to itself (see Chapter 2)
  * `+0` and `-0` are equal to each other (see Chapter 2)

* The final provision in clause 11.9.3.1 is for `==` loose equality comparison with `object`s (including `function`s and `array`s). Two such values are only *equal* if they are *both references to the exact same value*. `No coercion occurs` here.

* It's a very little known fact that **`==` and `===` behave identically** in the case where two `object`s are being compared!

* The rest of the algorithm in 11.9.3 specifies that if you use `==` loose equality to compare two values of `different types`, `one or both` of the values will need to be *implicitly* coerced. This coercion happens so that both values eventually end up as the same type, which can then directly be compared for equality using simple `value Identity`.

* The `!=` loose not-equality operation is [...] the `==` operation comparison performed in its entirety, then the `negation` of the result. The same goes for the `!==` strict not-equality operation.

#### Comparing: `string`s to `number`s
```js
var a = 42;
var b = "42";

a === b;	// false
a == b;		// true
```

* `a === b` fails, because `no coercion` is allowed, and [...] the `42` and `"42"` values are different.

* the second comparison `a == b` uses loose equality, which means that `if the types happen to be different`, the comparison algorithm will perform *implicit* coercion on one or both values [...] But [...] Does the `a` value of `42` become a `string`, or does the `b` value of `"42"` become a `number`? [...] In the ES5 spec, clauses 11.9.3.4-5 say:
	> 4. If Type(x) is Number and Type(y) is String,
	>    return the result of the comparison x == ToNumber(y).
	> 5. If Type(x) is String and Type(y) is Number,
	>    return the result of the comparison ToNumber(x) == y.

* **Warning:** The spec uses `Number` and `String` as the formal names for the types, while this book prefers `number` and `string` for the primitive types. Do not let the capitalization of `Number` in the spec confuse you for the `Number()` native function.

#### Comparing: anything to `boolean`
* Let's again quote the spec, clauses 11.9.3.6-7:
	> 6. If Type(x) is Boolean,
	>    return the result of the comparison ToNumber(x) == y.
	> 7. If Type(y) is Boolean,
	>    return the result of the comparison x == ToNumber(y).

```js
var x = true;
var y = "42";

x == y; // false
```

* The `Type(x)` is indeed `Boolean`, so it performs `ToNumber(x)`, which coerces `true` to `1`. Now, `1 == "42"` is evaluated. The types are still different, so (essentially recursively) we reconsult the algorithm, which just as above will coerce `"42"` to `42`, and `1 == 42` is clearly `false`.

* `"42"` is indeed `truthy`, but `"42" == true` **is not performing a boolean test/coercion** [...] `"42"` *is not* being coerced to a `boolean` (`true`), but instead `true` is being coerced to a `1`, and then `"42"` is being coerced to `42`.

* `=== true` and `=== false` wouldn't allow the coercion, so they're safe from this hidden `ToNumber` coercion.

```js
var a = "42";

// bad (will fail!):
if (a == true) {
	// ..
}

// also bad (will fail!):
if (a === true) {
	// ..
}

// good enough (works implicitly):
if (a) {
	// ..
}

// better (works explicitly):
if (!!a) {
	// ..
}

// also great (works explicitly):
if (Boolean( a )) {
	// ..
}
```

* avoid ever using `== true` or `== false` [...] (me: and) you'll never have to worry about this truthiness/falsiness mental gotcha.

#### Comparing: `null`s to `undefined`s
* again quoting the ES5 spec, clauses 11.9.3.2-3:
	> 2. If x is null and y is undefined, return true.
	> 3. If x is undefined and y is null, return true.

```js
var a = doSomething();

if (a == null) {
	// ..
}
```

* The `a == null` check will pass only if `doSomething()` returns `either` `null` or `undefined`.

#### Comparing: `object`s to non-`object`s
* If an `object`/`function`/`array` is compared to a simple [...] primitive (`string`, `number`, or `boolean`), the ES5 spec says in clauses 11.9.3.8-9:
	> 8. If Type(x) is either String or Number and Type(y) is Object,
	>    return the result of the comparison x == ToPrimitive(y).
	> 9. If Type(x) is Object and Type(y) is either String or Number,
	>    return the result of the comparison ToPrimitive(x) == y.

* **Note:** You may notice that these clauses only mention `String` and `Number`, but not `Boolean`. That's because, as quoted earlier, clauses 11.9.3.6-7 take care of coercing any `Boolean` operand presented to a `Number` first.

* **Tip:** All the quirks of the `ToPrimitive` abstract operation that we discussed earlier in this chapter (`toString()`, `valueOf()`) apply here as you'd expect. This can be quite useful if you have a complex data structure that you want to define a custom `valueOf()` method on, to provide a simple value for equality comparison purposes.

```js
var a = "abc";
var b = Object( a );	// same as `new String( a )` (me: read more in the Chapter 3 - unboxing section)

a === b;				// false
a == b;					// true
```

* `a == b` is `true` because `b` is coerced (aka "unboxed," unwrapped) via `ToPrimitive` to its underlying `"abc"` simple [...] primitive value, which is the same as the value in `a` [...] There are some values where this is `not the case`.

```js
var a = null;
var b = Object( a );	// same as `Object()`
a == b;					// false

var c = undefined;
var d = Object( c );	// same as `Object()`
c == d;					// false

var e = NaN;
var f = Object( e );	// same as `new Number( e )`
e == f;					// false
```

* The `null` and `undefined` values `cannot be boxed` -- they have no object wrapper equivalent -- so `Object(null)` is just like `Object()` in that both just produce a normal object.

* `NaN` can be boxed to its `Number` object wrapper equivalent, but [...] the `NaN == NaN` comparison fails because `NaN` is never equal to itself (see Chapter 2).

### Edge Cases
#### A Number By Any Other Value Would...
```js
Number.prototype.valueOf = function() {
	return 3;
};

new Number( 2 ) == 3;	// true
```

* **Warning:** `2 == 3` would not have fallen into this trap, because neither `2` nor `3` would have invoked the built-in `Number.prototype.valueOf()` method because both are already primitive `number` values and can be compared `directly`. However, `new Number(2)` must go through the `ToPrimitive` coercion, and thus invoke `valueOf()`.

* Next, let's consider another tricky example.

```js
if (a == 2 && a == 3) {
	// ..
}
```

* You might think this would be impossible [...] (me: but you are wrong) since the first expression `a == 2` happens strictly *before* `a == 3` [...] So, what if we make `a.valueOf()` have `side effects` each time it's called, such that the first time it returns `2` and the second time it's called it returns `3`? Pretty easy:

```js
var i = 2;

Number.prototype.valueOf = function() {
	return i++;
};

var a = new Number( 42 );

if (a == 2 && a == 3) {
	console.log( "Yep, this happened." );
}
```

#### False-y Comparisons
* The most common complaint against *implicit* coercion in `==` comparisons comes from how `falsy values` behave surprisingly when compared to each other.

```js
"0" == null;			// false
"0" == undefined;		// false
"0" == false;			// true -- UH OH! (me: false is converted to 0 by the `toNumber` method. "0" is then also converted to 0 by the `toNumber` method)
"0" == NaN;				// false
"0" == 0;				// true (me: "0" is converted to 0 by the `toNumber` method)
"0" == "";				// false

false == null;			// false
false == undefined;		// false
false == NaN;			// false
false == 0;				// true -- UH OH! (me: false is converted to 0 by the `toNumber` method)
false == "";			// true -- UH OH! (me: false is converted to 0 by the `toNumber` method, "" is then also converted 0 by the method)
false == [];			// true -- UH OH! (me: false is converted to 0 by the `toNumber method`, [] is then converted to "" by the `toPrimitive` method & then converted to 0 by the `toNumber` method)
false == {};			// false

"" == null;				// false
"" == undefined;		// false
"" == NaN;				// false
"" == 0;				// true -- UH OH! (me: "" is converted to 0 by the `toNumber` method)
"" == [];				// true -- UH OH! (me: [] is converted to "" by the `toPrimive` method)
"" == {};				// false

0 == null;				// false
0 == undefined;			// false
0 == NaN;				// false
0 == [];				// true -- UH OH! (me: [] is converted to "" by the `toPrimive` method & then converted to 0 by the `toNumber` method)
0 == {};				// false
```

#### The Crazy Ones
```js
[] == ![];		// true
```

* What do we know about the `!` unary operator? It `explicitly coerces` to a `boolean` using the `ToBoolean` rules (and it also `flips` the parity). So **before** `[] == ![]` is even processed, it's actually already translated to `[] == false`. We already saw that form in our above list (`false == []`), so its surprise result is *not new* to us.

* Another famously cited gotcha:

```js
0 == "\n";		// true
```

* empty `""`, `"\n"` (or `" "` or any other whitespace combination) is coerced via `ToNumber`, and the result is `0`. What other `number` value would you expect whitespace to coerce to? Does it bother you that *explicit* `Number(" ")` yields `0`?

#### Sanity Check
#### Safely Using Implicit Coercion
## Abstract Relational Comparison
* The "Abstract Relational Comparison" algorithm in ES5 section 11.8.5 essentially divides itself into two parts: what to do if the comparison involves both `string` values (second half), or anything else (first half).

* **Note:** The algorithm is only defined for `a < b`. So, `a > b` is handled as `b < a`.

* The algorithm first calls `ToPrimitive` coercion on `both values`, and if the return result of `either` call is `not a string`, then `both values` are coerced to `number` values using the `ToNumber` operation rules, and compared `numerically`.

```js
var a = [ 42 ];
var b = [ "43" ];

a < b;	// true
b < a;	// false
```

* Similar caveats for `-0` and `NaN` apply here as they did in the `==` algorithm discussed earlier.

* `However`, (me: after the `toPrimitive` operation is done) if `both values` are `string`s for the `<` comparison, simple lexicographic (`natural alphabetic`) comparison on the characters is performed:

```js
var a = [ "42" ];
var b = [ "043" ];

a < b;	// false
```

* `a` and `b` are *not* coerced to `number`s, because `both` of them `end up` as `string`s `after` the `ToPrimitive` coercion on the two `array`s. So, `"42"` is compared character by character to `"043"`, starting with the first characters `"4"` and `"0"`, respectively. Since `"0"` is lexicographically *less than* than `"4"`, the comparison returns `false`.

* What about:

```js
var a = { b: 42 };
var b = { b: 43 };

a < b;	// ??
```

* `a < b` is also `false`, because `a` becomes `[object Object]` and `b` becomes `[object Object]`, and so clearly `a` is not lexicographically less than `b` [...] But strangely:

```js
var a = { b: 42 };
var b = { b: 43 };

a < b;	// false
a == b;	// false
a > b;	// false

a <= b;	// true
a >= b;	// true
```

* Why is `a == b` not `true`? They're the same `string` value (`"[object Object]"`) [...] Recall the previous discussion about how `==` works with `object references`.

* But then how are `a <= b` and `a >= b` resulting in `true`, if `a < b` **and** `a == b` **and** `a > b` are all `false`? [...] Because the spec says for `a <= b`, it will actually evaluate `b < a` first, and then `negate` that result. Since `b < a` is *also* `false`, the result of `a <= b` is `true`.

* That's probably awfully contrary to how you might have explained what `<=` does [...] which would likely have been [...]: "less than *or* equal to." `JS` more accurately considers `<=` as "`not greater than`" (`!(a > b)`, which JS treats as `!(b < a)` (me: recall the previous argument: **`a > b` is handled as `b < a`**)). Moreover, `a >= b` is explained by `first considering` it as `b <= a`, and then applying the same (me: the explaination above in this argument)(reasoning).

* `Unfortunately`, there is `no "strict relational comparison"` [...] no way to prevent *implicit* coercion from occurring with relational comparisons like `a < b`.

## Review

# Chapter 5: Grammar
* **Note:** The term "grammar" may be a little less familiar to readers than the term "syntax." In many ways, they are similar terms, describing the *rules* for how the language works.

## Statements & Expressions
* It's fairly common [...] to assume that [...] "statement" and "expression" are roughly equivalent. But here we need to distinguish between the two, because there are some very important differences in our JS programs [...] To draw the distinction, let's borrow from terminology you may be more familiar with: the English language.

* A "sentence" is one complete formation of words that expresses a thought. It's comprised of one or more "phrases," each of which can be connected with punctuation marks or conjunction words ("and," "or," etc) [...] These rules are collectively called the *grammar* of the English language.

* And so it goes with JavaScript grammar. Statements are sentences, expressions are phrases, and operators are conjunctions/punctuation.

```js
var a = 3 * 6;
var b = a;
b;
```

* `3 * 6` is an expression (evaluates to the value `18`). But `a` on the second line is also an expression, as is `b` on the third line.

* each of the three lines is a statement containing expressions. `var a = 3 * 6` and `var b = a` are called "`declaration statements`" because they each declare a variable (and optionally assign a value to it). The `a = 3 * 6` and `b = a` assignments (minus the `var`s) are called `assignment expressions`.

* The third line contains just the expression `b`, but it's also a statement all by itself [...] This is generally referred to as an "`expression statement`."

### Statement Completion Values
* It's a fairly little known fact that statements all have completion values (even if that value is just `undefined`).

* The `b = a` assignment expression results in the value that was assigned (`18` above), but the `var` statement itself results in `undefined`. Why? Because `var` statements are defined that way in the spec [...] Technically [...] In the ES5 spec, section 12.2 "Variable Statement," the `VariableDeclaration` algorithm actually *does* return a value (a `string` containing the name of the variable declared [...]), but that value is [...] swallowed up (except for use by the `for..in` loop) by the `VariableStatement` algorithm, which forces an empty (aka `undefined`) completion value.

* So how can we capture the completion value? [...] Before we explain *how*, let's explore *why* you would want to do that.

* any regular `{ .. }` block has a completion value of the completion value of its `last contained statement/expression`.

```js
var b;

if (true) {
	b = 4 + 38;
}
```

* If you typed that into your console [...] you'd probably see `42`[...] since `42` is the completion value of the `if` block [...] took on the completion value of its last assignment expression statement `b = 4 + 38` [...] But there's an obvious problem. This kind of code doesn't work:

```js
var a, b;

a = if (true) {
	b = 4 + 38;
};
```

* We can't capture the completion value of a statement and assign it into another variable [...] (at least not yet!) [...] So, what can we do?

* **Warning**: For demo purposes only -- don't actually do the following in your real code! [...] We could use [...] `eval(..)` (sometimes pronounced "evil") function to capture this completion value.

```js
var a, b;

a = eval( "if (true) { b = 4 + 38; }" );

a;	// 42
```

* There's a proposal for ES7 called "do expression." Here's how it might work:

```js
var a, b;

a = do {
	if (true) {
		b = 4 + 38;
	}
};

a;	// 42
```

* The `do { .. }` expression executes a block [...] and the final statement completion value inside the block becomes the completion value *of* the `do` expression.

* The general idea is to be able to treat statements as expressions [...] without needing to wrap them in an inline function expression and perform an explicit `return ...`.

* **Warning:** [...] avoid `eval(..)`. Seriously. See the *Scope & Closures* title of this series for more explanation.

### Expression Side Effects
* most common example of an expression with (possible) side effects is a function call expression:

```js
function foo() {
	a = a + 1;
}

var a = 1;
foo();		// result: `undefined`, side effect: changed `a`
```

* (WCRL).

* When `++` is used in the `prefix` position as `++a`, its side effect (incrementing `a`) happens **before the value is returned from the expression**, rather than *after* as with `a++`.

* (WCRL).

### Short Circuited
* In Chapter 4, we mentioned in a side note the "short circuiting" nature of operators like `&&` and `||`. Let's revisit that in more detail now.

* For both `&&` and `||` operators, the right-hand operand will **not be evaluated** if the left-hand operand is sufficient to determine the outcome of the operation. Hence, the name "short circuited" (in that if possible, it will take an early shortcut out).

* For example, with `a && b`, `b` is not evaluated if `a` is falsy, because the result of the `&&` operand is already certain, so there's no point in bothering to check `b`. Likewise, with `a || b`, if `a` is truthy, the result of the operand is already certain, so there's no reason to check `b`.

```js
function doSomething(opts) {
	if (opts && opts.cool) {
		// ..
	}
}
```

* The `opts` part of the `opts && opts.cool` test acts as sort of a guard, because if `opts` is unset (or is not an `object`), the expression `opts.cool` would throw an error. The `opts` test failing plus the short circuiting means that `opts.cool` won't even be evaluated, thus no error!

* (WCRL).

## Function Arguments
* When using ES6's default parameter values, the default value is applied to the parameter if you either omit an argument, `or` you pass an `undefined` value in its place:

```js
function foo( a = 42, b = a + 1 ) {
	console.log( a, b );
}

foo();					// 42 43
foo( undefined );		// 42 43
foo( 5 );				// 5 6
foo( void 0, 7 );		// 42 7
foo( null );			// null 1
```

* **Note:** `null` is coerced to a `0` value in the `a + 1` expression. See Chapter 4 for more info.

* From the ES6 default parameter values perspective, there's no difference between omitting an argument and passing an `undefined` value. However, there is a way to detect the difference in some cases:

```js
function foo( a = 42, b = a + 1 ) {
	console.log(
		arguments.length, a, b,
		arguments[0], arguments[1]
	);
}

foo();					// 0 42 43 undefined undefined
foo( 10 );				// 1 10 11 10 undefined
foo( 10, undefined );	// 2 10 11 10 undefined
foo( 10, null );		// 2 10 null 10 null
```

* Even though the default parameter values are applied to the `a` and `b` parameters, if no arguments were passed in those slots, the `arguments` array will not have entries.

* if you pass an `undefined` argument `explicitly`, an entry will exist in the `arguments` array for that argument, but it will be `undefined` and not [...] the same as the `default value` that was applied to the named parameter.

```js
function foo(a) {
	a = 42;
	console.log( arguments[0] );
}

foo( 2 );	// 42 (linked)
foo();		// undefined (not linked)
```

* If you pass an argument, the `arguments` slot and the named parameter are linked to always have the same value. If you omit the argument, no such linkage occurs.

* `But` in `strict` mode, the linkage doesn't exist.

```js
function foo(a) {
	"use strict";
	a = 42;
	console.log( arguments[0] );
}

foo( 2 );	// 2 (not linked)
foo();		// undefined (not linked)
```

* (WCRL).

## Review

# Appendix A: Mixed Environment JavaScript
* Beyond the core language mechanics [...] there are several ways that your JS code can behave differently [...] If JS was executing purely inside an engine, it'd be entirely predictable based on [...] the spec. But JS pretty much always runs in the context of a `hosting environment` [...] For example [...] when your code runs in different types of JS engines (not just browsers).

## Annex B (ECMAScript)
* It's a little known fact that the **official name of the (me: `general-purpose programming language`)(language)** is `ECMAScript` [...] `JavaScript` [...] is basically the `browser implementation` of the spec.

* (WCRL).

## `<script>`s
* it's common to have a few or several `<script src=..></script>` elements in the page that load [...] files separately [...] they act more like `independent JS programs` in most, `but not all` [...] The one thing they *share* is the single `global` object (`window` in the browser), which means multiple files can `append their code` to that `shared namespace` and they can all interact.

* `global variable` scope *hoisting* [...] does not occur across these boundaries, so the following code would not work (because `foo()`'s declaration isn't yet declared).

```html
<script>foo();</script>

<script>
  function foo() { .. }
</script>
```

* But either of these *would* work instead:

```html
<script>
  foo();
  function foo() { .. }
</script>
```

Or:

```html
<script>
  function foo() { .. }
</script>

<script>foo();</script>
```

* if an `error` occurs in a `script` element [...] subsequent `script`s will run (still with the shared global).

* in the inline code block, the sequence of characters `</script>` cannot appear together, as (regardless of where it appears) it would be interpreted as the end of the code block. So, beware.

```html
<script>
  var code = "<script>alert( 'Hello World' )</script>";
</script>
```

* It looks harmless, but the `</script>` appearing inside the `string` literal will terminate the script block abnormally, causing an error. The most common workaround is:

```js
"</sc" + "ript>";
```

* Also, beware that code inside an `external file` will be interpreted in the `character set` (UTF-8, ISO-8859-8, etc.) the file is served with (`or the default`), `but` that same code in an inline `script` element in your HTML page will be interpreted by the `character set of the page` (or its default).

* **Warning:** The `charset` attribute will not work on inline script elements.

## Review

# Appendix B: Thank You's!
