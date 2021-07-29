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
The `Object(..)`/`Function(..)`/`RegExp(..)` constructors are also generally optional (and thus should usually be avoided unless specifically called for):

```js
var c = new Object();
c.foo = "bar";
c; // { foo: "bar" }

var d = { foo: "bar" };
d; // { foo: "bar" }

var e = new Function( "a", "return a * 2;" );
var f = function(a) { return a * 2; };
function g(a) { return a * 2; }

var h = new RegExp( "^a*b+", "g" );
var i = /^a*b+/g;
```

There's practically no reason to ever use the `new Object()` constructor form, especially since it forces you to add properties one-by-one instead of many at once in the object literal form.

The `Function` constructor is helpful only in the rarest of cases, where you need to dynamically define a function's parameters and/or its function body. **Do not just treat `Function(..)` as an alternate form of `eval(..)`.** You will almost never need to dynamically define a function in this way.

Regular expressions defined in the literal form (`/^a*b+/g`) are strongly preferred, not just for ease of syntax but for performance reasons -- the JS engine precompiles and caches them before code execution. Unlike the other constructor forms we've seen so far, `RegExp(..)` has some reasonable utility: to dynamically define the pattern for a regular expression.

```js
var name = "Kyle";
var namePattern = new RegExp( "\\b(?:" + name + ")+\\b", "ig" );

var matches = someText.match( namePattern );
```

This kind of scenario legitimately occurs in JS programs from time to time, so you'd need to use the `new RegExp("pattern","flags")` form.

### `Date(..)` and `Error(..)`

The `Date(..)` and `Error(..)` native constructors are much more useful than the other natives, because there is no literal form for either.

To create a date object value, you must use `new Date()`. The `Date(..)` constructor accepts optional arguments to specify the date/time to use, but if omitted, the current date/time is assumed.

By far the most common reason you construct a date object is to get the current timestamp value (a signed integer number of milliseconds since Jan 1, 1970). You can do this by calling `getTime()` on a date object instance.

But an even easier way is to just call the static helper function defined as of ES5: `Date.now()`. And to polyfill that for pre-ES5 is pretty easy:

```js
if (!Date.now) {
	Date.now = function(){
		return (new Date()).getTime();
	};
}
```

**Note:** If you call `Date()` without `new`, you'll get back a string representation of the date/time at that moment. The exact form of this representation is not specified in the language spec, though browsers tend to agree on something close to: `"Fri Jul 18 2014 00:31:02 GMT-0500 (CDT)"`.

The `Error(..)` constructor (much like `Array()` above) behaves the same with the `new` keyword present or omitted.

The main reason you'd want to create an error object is that it captures the current execution stack context into the object (in most JS engines, revealed as a read-only `.stack` property once constructed). This stack context includes the function call-stack and the line-number where the error object was created, which makes debugging that error much easier.

You would typically use such an error object with the `throw` operator:

```js
function foo(x) {
	if (!x) {
		throw new Error( "x wasn't provided" );
	}
	// ..
}
```

Error object instances generally have at least a `message` property, and sometimes other properties (which you should treat as read-only), like `type`. However, other than inspecting the above-mentioned `stack` property, it's usually best to just call `toString()` on the error object (either explicitly, or implicitly through coercion -- see Chapter 4) to get a friendly-formatted error message.

**Tip:** Technically, in addition to the general `Error(..)` native, there are several other specific-error-type natives: `EvalError(..)`, `RangeError(..)`, `ReferenceError(..)`, `SyntaxError(..)`, `TypeError(..)`, and `URIError(..)`. But it's very rare to manually use these specific error natives. They are automatically used if your program actually suffers from a real exception (such as referencing an undeclared variable and getting a `ReferenceError` error).

### `Symbol(..)`

New as of ES6, an additional primitive value type has been added, called "Symbol". Symbols are special "unique" (not strictly guaranteed!) values that can be used as properties on objects with little fear of any collision. They're primarily designed for special built-in behaviors of ES6 constructs, but you can also define your own symbols.

Symbols can be used as property names, but you cannot see or access the actual value of a symbol from your program, nor from the developer console. If you evaluate a symbol in the developer console, what's shown looks like `Symbol(Symbol.create)`, for example.

There are several predefined symbols in ES6, accessed as static properties of the `Symbol` function object, like `Symbol.create`, `Symbol.iterator`, etc. To use them, do something like:

```js
obj[Symbol.iterator] = function(){ /*..*/ };
```

To define your own custom symbols, use the `Symbol(..)` native. The `Symbol(..)` native "constructor" is unique in that you're not allowed to use `new` with it, as doing so will throw an error.

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

While symbols are not actually private (`Object.getOwnPropertySymbols(..)` reflects on the object and reveals the symbols quite publicly), using them for private or special properties is likely their primary use-case. For most developers, they may take the place of property names with `_` underscore prefixes, which are almost always by convention signals to say, "hey, this is a private/special/internal property, so leave it alone!"

**Note:** `Symbol`s are *not* `object`s, they are simple scalar primitives.

### Native Prototypes

Each of the built-in native constructors has its own `.prototype` object -- `Array.prototype`, `String.prototype`, etc.

These objects contain behavior unique to their particular object subtype.

For example, all string objects, and by extension (via boxing) `string` primitives, have access to default behavior as methods defined on the `String.prototype` object.

**Note:** By documentation convention, `String.prototype.XYZ` is shortened to `String#XYZ`, and likewise for all the other `.prototype`s.

* `String#indexOf(..)`: find the position in the string of another substring
* `String#charAt(..)`: access the character at a position in the string
* `String#substr(..)`, `String#substring(..)`, and `String#slice(..)`: extract a portion of the string as a new string
* `String#toUpperCase()` and `String#toLowerCase()`: create a new string that's converted to either uppercase or lowercase
* `String#trim()`: create a new string that's stripped of any trailing or leading whitespace

None of the methods modify the string *in place*. Modifications (like case conversion or trimming) create a new value from the existing value.

By virtue of prototype delegation (see the *this & Object Prototypes* title in this series), any string value can access these methods:

```js
var a = " abc ";

a.indexOf( "c" ); // 3
a.toUpperCase(); // " ABC "
a.trim(); // "abc"
```

The other constructor prototypes contain behaviors appropriate to their types, such as `Number#toFixed(..)` (stringifying a number with a fixed number of decimal digits) and `Array#concat(..)` (merging arrays). All functions have access to `apply(..)`, `call(..)`, and `bind(..)` because `Function.prototype` defines them.

But, some of the native prototypes aren't *just* plain objects:

```js
typeof Function.prototype;			// "function"
Function.prototype();				// it's an empty function!

RegExp.prototype.toString();		// "/(?:)/" -- empty regex
"abc".match( RegExp.prototype );	// [""]
```

A particularly bad idea, you can even modify these native prototypes (not just adding properties as you're probably familiar with):

```js
Array.isArray( Array.prototype );	// true
Array.prototype.push( 1, 2, 3 );	// 3
Array.prototype;					// [1,2,3]

// don't leave it that way, though, or expect weirdness!
// reset the `Array.prototype` to empty
Array.prototype.length = 0;
```

As you can see, `Function.prototype` is a function, `RegExp.prototype` is a regular expression, and `Array.prototype` is an array. Interesting and cool, huh?

#### Prototypes As Defaults

`Function.prototype` being an empty function, `RegExp.prototype` being an "empty" (e.g., non-matching) regex, and `Array.prototype` being an empty array, make them all nice "default" values to assign to variables if those variables wouldn't already have had a value of the proper type.

For example:

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

**Note:** As of ES6, we don't need to use the `vals = vals || ..` default value syntax trick (see Chapter 4) anymore, because default values can be set for parameters via native syntax in the function declaration (see Chapter 5).

One minor side-benefit of this approach is that the `.prototype`s are already created and built-in, thus created *only once*. By contrast, using `[]`, `function(){}`, and `/(?:)/` values themselves for those defaults would (likely, depending on engine implementations) be recreating those values (and probably garbage-collecting them later) for *each call* of `isThisCool(..)`. That could be memory/CPU wasteful.

Also, be very careful not to use `Array.prototype` as a default value **that will subsequently be modified**. In this example, `vals` is used read-only, but if you were to instead make in-place changes to `vals`, you would actually be modifying `Array.prototype` itself, which would lead to the gotchas mentioned earlier!

**Note:** While we're pointing out these native prototypes and some usefulness, be cautious of relying on them and even more wary of modifying them in any way. See Appendix A "Native Prototypes" for more discussion.

## Review
