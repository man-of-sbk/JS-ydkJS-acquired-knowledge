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

### `ToNumber`
* If any `non-number` value is used in a way that requires it to be a `number`, such as a `mathematical operation`, the ES5 spec defines the `ToNumber` abstract operation

* For example, `true` becomes `1` and `false` becomes `0`. `undefined` becomes `NaN`, but (curiously) `null` becomes `0`.

* `ToNumber` for a `string` value essentially works [...] like the `rules/syntax for numeric literals` (see `Chapter 3` in the section talking about the `Number` native function) If it fails, the result is `NaN` [...] One [...] difference is that `0`-`prefixed octal numbers` are not handled as octals (just as normal `base-10 decimals`) in this operation, though such octals are valid as `number` literals (see Chapter 2).

* Objects (and arrays) will first be converted to their `primitive value equivalent` (me: see how in the next argument), and the resulting value (if a primitive but `not already` a `number`) is coerced to a `number` according to the `ToNumber` rules just mentioned.

* To convert to this `primitive value equivalent`, the `ToPrimitive` abstract operation [...] will consult the value (using the internal `DefaultValue` operation -- ES5 spec, section 8.12.8) in question to see if it has a `valueOf()` method. If `valueOf()` is available and it returns a primitive value, *that* value is used for the coercion. If not, but `toString()` is available, (me: the `toString()` function)(it) will provide the value for the coercion [...] If neither operation can provide a primitive value, a `TypeError` is thrown.

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

Earlier in this chapter, we explored *explicitly* coercing between `string` and `number` values. Now, let's explore the same task but with *implicit* coercion approaches. But before we do, we have to examine some nuances of operations that will *implicitly* force coercion.

The `+` operator is overloaded to serve the purposes of both `number` addition and `string` concatenation. So how does JS know which type of operation you want to use? Consider:

```js
var a = "42";
var b = "0";

var c = 42;
var d = 0;

a + b; // "420"
c + d; // 42
```

What's different that causes `"420"` vs `42`? It's a common misconception that the difference is whether one or both of the operands is a `string`, as that means `+` will assume `string` concatenation. While that's partially true, it's more complicated than that.

Consider:

```js
var a = [1,2];
var b = [3,4];

a + b; // "1,23,4"
```

Neither of these operands is a `string`, but clearly they were both coerced to `string`s and then the `string` concatenation kicked in. So what's really going on?

(**Warning:** deeply nitty gritty spec-speak coming, so skip the next two paragraphs if that intimidates you!)

-----

According to ES5 spec section 11.6.1, the `+` algorithm (when an `object` value is an operand) will concatenate if either operand is either already a `string`, or if the following steps produce a `string` representation. So, when `+` receives an `object` (including `array`) for either operand, it first calls the `ToPrimitive` abstract operation (section 9.1) on the value, which then calls the `[[DefaultValue]]` algorithm (section 8.12.8) with a context hint of `number`.

If you're paying close attention, you'll notice that this operation is now identical to how the `ToNumber` abstract operation handles `object`s (see the "`ToNumber`"" section earlier). The `valueOf()` operation on the `array` will fail to produce a simple primitive, so it then falls to a `toString()` representation. The two `array`s thus become `"1,2"` and `"3,4"`, respectively. Now, `+` concatenates the two `string`s as you'd normally expect: `"1,23,4"`.

-----

Let's set aside those messy details and go back to an earlier, simplified explanation: if either operand to `+` is a `string` (or becomes one with the above steps!), the operation will be `string` concatenation. Otherwise, it's always numeric addition.

**Note:** A commonly cited coercion gotcha is `[] + {}` vs. `{} + []`, as those two expressions result, respectively, in `"[object Object]"` and `0`. There's more to it, though, and we cover those details in "Blocks" in Chapter 5.

What's that mean for *implicit* coercion?

You can coerce a `number` to a `string` simply by "adding" the `number` and the `""` empty `string`:

```js
var a = 42;
var b = a + "";

b; // "42"
```

**Tip:** Numeric addition with the `+` operator is commutative, which means `2 + 3` is the same as `3 + 2`. String concatenation with `+` is obviously not generally commutative, **but** with the specific case of `""`, it's effectively commutative, as `a + ""` and `"" + a` will produce the same result.

It's extremely common/idiomatic to (*implicitly*) coerce `number` to `string` with a `+ ""` operation. In fact, interestingly, even some of the most vocal critics of *implicit* coercion still use that approach in their own code, instead of one of its *explicit* alternatives.

**I think this is a great example** of a useful form in *implicit* coercion, despite how frequently the mechanism gets criticized!

Comparing this *implicit* coercion of `a + ""` to our earlier example of `String(a)` *explicit* coercion, there's one additional quirk to be aware of. Because of how the `ToPrimitive` abstract operation works, `a + ""` invokes `valueOf()` on the `a` value, whose return value is then finally converted to a `string` via the internal `ToString` abstract operation. But `String(a)` just invokes `toString()` directly.

Both approaches ultimately result in a `string`, but if you're using an `object` instead of a regular primitive `number` value, you may not necessarily get the *same* `string` value!

Consider:

```js
var a = {
	valueOf: function() { return 42; },
	toString: function() { return 4; }
};

a + "";			// "42"

String( a );	// "4"
```

Generally, this sort of gotcha won't bite you unless you're really trying to create confusing data structures and operations, but you should be careful if you're defining both your own `valueOf()` and `toString()` methods for some `object`, as how you coerce the value could affect the outcome.

What about the other direction? How can we *implicitly coerce* from `string` to `number`?

```js
var a = "3.14";
var b = a - 0;

b; // 3.14
```

The `-` operator is defined only for numeric subtraction, so `a - 0` forces `a`'s value to be coerced to a `number`. While far less common, `a * 1` or `a / 1` would accomplish the same result, as those operators are also only defined for numeric operations.

What about `object` values with the `-` operator? Similar story as for `+` above:

```js
var a = [3];
var b = [1];

a - b; // 2
```

Both `array` values have to become `number`s, but they end up first being coerced to `strings` (using the expected `toString()` serialization), and then are coerced to `number`s, for the `-` subtraction to perform on.

So, is *implicit* coercion of `string` and `number` values the ugly evil you've always heard horror stories about? I don't personally think so.

Compare `b = String(a)` (*explicit*) to `b = a + ""` (*implicit*). I think cases can be made for both approaches being useful in your code. Certainly `b = a + ""` is quite a bit more common in JS programs, proving its own utility regardless of *feelings* about the merits or hazards of *implicit* coercion in general.

### Implicitly: Booleans --> Numbers

I think a case where *implicit* coercion can really shine is in simplifying certain types of complicated `boolean` logic into simple numeric addition. Of course, this is not a general-purpose technique, but a specific solution for specific cases.

Consider:

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

This `onlyOne(..)` utility should only return `true` if exactly one of the arguments is `true` / truthy. It's using *implicit* coercion on the truthy checks and *explicit* coercion on the others, including the final return value.

But what if we needed that utility to be able to handle four, five, or twenty flags in the same way? It's pretty difficult to imagine implementing code that would handle all those permutations of comparisons.

But here's where coercing the `boolean` values to `number`s (`0` or `1`, obviously) can greatly help:

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

**Note:** Of course, instead of the `for` loop in `onlyOne(..)`, you could more tersely use the ES5 `reduce(..)` utility, but I didn't want to obscure the concepts.

What we're doing here is relying on the `1` for `true`/truthy coercions, and numerically adding them all up. `sum += arguments[i]` uses *implicit* coercion to make that happen. If one and only one value in the `arguments` list is `true`, then the numeric sum will be `1`, otherwise the sum will not be `1` and thus the desired condition is not met.

We could of course do this with *explicit* coercion instead:

```js
function onlyOne() {
	var sum = 0;
	for (var i=0; i < arguments.length; i++) {
		sum += Number( !!arguments[i] );
	}
	return sum === 1;
}
```

We first use `!!arguments[i]` to force the coercion of the value to `true` or `false`. That's so you could pass non-`boolean` values in, like `onlyOne( "42", 0 )`, and it would still work as expected (otherwise you'd end up with `string` concatenation and the logic would be incorrect).

Once we're sure it's a `boolean`, we do another *explicit* coercion with `Number(..)` to make sure the value is `0` or `1`.

Is the *explicit* coercion form of this utility "better"? It does avoid the `NaN` trap as explained in the code comments. But, ultimately, it depends on your needs. I personally think the former version, relying on *implicit* coercion is more elegant (if you won't be passing `undefined` or `NaN`), and the *explicit* version is needlessly more verbose.

But as with almost everything we're discussing here, it's a judgment call.

**Note:** Regardless of *implicit* or *explicit* approaches, you could easily make `onlyTwo(..)` or `onlyFive(..)` variations by simply changing the final comparison from `1`, to `2` or `5`, respectively. That's drastically easier than adding a bunch of `&&` and `||` expressions. So, generally, coercion is very helpful in this case.

### Implicitly: * --> Boolean

Now, let's turn our attention to *implicit* coercion to `boolean` values, as it's by far the most common and also by far the most potentially troublesome.

Remember, *implicit* coercion is what kicks in when you use a value in such a way that it forces the value to be converted. For numeric and `string` operations, it's fairly easy to see how the coercions can occur.

But, what sort of expression operations require/force (*implicitly*) a `boolean` coercion?

1. The test expression in an `if (..)` statement.
2. The test expression (second clause) in a `for ( .. ; .. ; .. )` header.
3. The test expression in `while (..)` and `do..while(..)` loops.
4. The test expression (first clause) in `? :` ternary expressions.
5. The left-hand operand (which serves as a test expression -- see below!) to the `||` ("logical or") and `&&` ("logical and") operators.

Any value used in these contexts that is not already a `boolean` will be *implicitly* coerced to a `boolean` using the rules of the `ToBoolean` abstract operation covered earlier in this chapter.

Let's look at some examples:

```js
var a = 42;
var b = "abc";
var c;
var d = null;

if (a) {
	console.log( "yep" );		// yep
}

while (c) {
	console.log( "nope, never runs" );
}

c = d ? a : b;
c;					// "abc"

if ((a && d) || c) {
	console.log( "yep" );		// yep
}
```

In all these contexts, the non-`boolean` values are *implicitly coerced* to their `boolean` equivalents to make the test decisions.

### Operators `||` and `&&`

It's quite likely that you have seen the `||` ("logical or") and `&&` ("logical and") operators in most or all other languages you've used. So it'd be natural to assume that they work basically the same in JavaScript as in other similar languages.

There's some very little known, but very important, nuance here.

In fact, I would argue these operators shouldn't even be called "logical ___ operators", as that name is incomplete in describing what they do. If I were to give them a more accurate (if more clumsy) name, I'd call them "selector operators," or more completely, "operand selector operators."

Why? Because they don't actually result in a *logic* value (aka `boolean`) in JavaScript, as they do in some other languages.

So what *do* they result in? They result in the value of one (and only one) of their two operands. In other words, **they select one of the two operand's values**.

Quoting the ES5 spec from section 11.11:

> The value produced by a && or || operator is not necessarily of type Boolean. The value produced will always be the value of one of the two operand expressions.

Let's illustrate:

```js
var a = 42;
var b = "abc";
var c = null;

a || b;		// 42
a && b;		// "abc"

c || b;		// "abc"
c && b;		// null
```

**Wait, what!?** Think about that. In languages like C and PHP, those expressions result in `true` or `false`, but in JS (and Python and Ruby, for that matter!), the result comes from the values themselves.

Both `||` and `&&` operators perform a `boolean` test on the **first operand** (`a` or `c`). If the operand is not already `boolean` (as it's not, here), a normal `ToBoolean` coercion occurs, so that the test can be performed.

For the `||` operator, if the test is `true`, the `||` expression results in the value of the *first operand* (`a` or `c`). If the test is `false`, the `||` expression results in the value of the *second operand* (`b`).

Inversely, for the `&&` operator, if the test is `true`, the `&&` expression results in the value of the *second operand* (`b`). If the test is `false`, the `&&` expression results in the value of the *first operand* (`a` or `c`).

The result of a `||` or `&&` expression is always the underlying value of one of the operands, **not** the (possibly coerced) result of the test. In `c && b`, `c` is `null`, and thus falsy. But the `&&` expression itself results in `null` (the value in `c`), not in the coerced `false` used in the test.

Do you see how these operators act as "operand selectors", now?

Another way of thinking about these operators:

```js
a || b;
// roughly equivalent to:
a ? a : b;

a && b;
// roughly equivalent to:
a ? b : a;
```

**Note:** I call `a || b` "roughly equivalent" to `a ? a : b` because the outcome is identical, but there's a nuanced difference. In `a ? a : b`, if `a` was a more complex expression (like for instance one that might have side effects like calling a `function`, etc.), then the `a` expression would possibly be evaluated twice (if the first evaluation was truthy). By contrast, for `a || b`, the `a` expression is evaluated only once, and that value is used both for the coercive test as well as the result value (if appropriate). The same nuance applies to the `a && b` and `a ? b : a` expressions.

An extremely common and helpful usage of this behavior, which there's a good chance you may have used before and not fully understood, is:

```js
function foo(a,b) {
	a = a || "hello";
	b = b || "world";

	console.log( a + " " + b );
}

foo();					// "hello world"
foo( "yeah", "yeah!" );	// "yeah yeah!"
```

The `a = a || "hello"` idiom (sometimes said to be JavaScript's version of the C# "null coalescing operator") acts to test `a` and if it has no value (or only an undesired falsy value), provides a backup default value (`"hello"`).

**Be careful**, though!

```js
foo( "That's it!", "" ); // "That's it! world" <-- Oops!
```

See the problem? `""` as the second argument is a falsy value (see `ToBoolean` earlier in this chapter), so the `b = b || "world"` test fails, and the `"world"` default value is substituted, even though the intent probably was to have the explicitly passed `""` be the value assigned to `b`.

This `||` idiom is extremely common, and quite helpful, but you have to use it only in cases where *all falsy values* should be skipped. Otherwise, you'll need to be more explicit in your test, and probably use a `? :` ternary instead.

This *default value assignment* idiom is so common (and useful!) that even those who publicly and vehemently decry JavaScript coercion often use it in their own code!

What about `&&`?

There's another idiom that is quite a bit less commonly authored manually, but which is used by JS minifiers frequently. The `&&` operator "selects" the second operand if and only if the first operand tests as truthy, and this usage is sometimes called the "guard operator" (also see "Short Circuited" in Chapter 5) -- the first expression test "guards" the second expression:

```js
function foo() {
	console.log( a );
}

var a = 42;

a && foo(); // 42
```

`foo()` gets called only because `a` tests as truthy. If that test failed, this `a && foo()` expression statement would just silently stop -- this is known as "short circuiting" -- and never call `foo()`.

Again, it's not nearly as common for people to author such things. Usually, they'd do `if (a) { foo(); }` instead. But JS minifiers choose `a && foo()` because it's much shorter. So, now, if you ever have to decipher such code, you'll know what it's doing and why.

OK, so `||` and `&&` have some neat tricks up their sleeve, as long as you're willing to allow the *implicit* coercion into the mix.

**Note:** Both the `a = b || "something"` and `a && b()` idioms rely on short circuiting behavior, which we cover in more detail in Chapter 5.

The fact that these operators don't actually result in `true` and `false` is possibly messing with your head a little bit by now. You're probably wondering how all your `if` statements and `for` loops have been working, if they've included compound logical expressions like `a && (b || c)`.

Don't worry! The sky is not falling. Your code is (probably) just fine. It's just that you probably never realized before that there was an *implicit* coercion to `boolean` going on **after** the compound expression was evaluated.

Consider:

```js
var a = 42;
var b = null;
var c = "foo";

if (a && (b || c)) {
	console.log( "yep" );
}
```

This code still works the way you always thought it did, except for one subtle extra detail. The `a && (b || c)` expression *actually* results in `"foo"`, not `true`. So, the `if` statement *then* forces the `"foo"` value to coerce to a `boolean`, which of course will be `true`.

See? No reason to panic. Your code is probably still safe. But now you know more about how it does what it does.

And now you also realize that such code is using *implicit* coercion. If you're in the "avoid (implicit) coercion camp" still, you're going to need to go back and make all of those tests *explicit*:

```js
if (!!a && (!!b || !!c)) {
	console.log( "yep" );
}
```

Good luck with that! ... Sorry, just teasing.

### Symbol Coercion

Up to this point, there's been almost no observable outcome difference between *explicit* and *implicit* coercion -- only the readability of code has been at stake.

But ES6 Symbols introduce a gotcha into the coercion system that we need to discuss briefly. For reasons that go well beyond the scope of what we'll discuss in this book, *explicit* coercion of a `symbol` to a `string` is allowed, but *implicit* coercion of the same is disallowed and throws an error.

Consider:

```js
var s1 = Symbol( "cool" );
String( s1 );					// "Symbol(cool)"

var s2 = Symbol( "not cool" );
s2 + "";						// TypeError
```

`symbol` values cannot coerce to `number` at all (throws an error either way), but strangely they can both *explicitly* and *implicitly* coerce to `boolean` (always `true`).

Consistency is always easier to learn, and exceptions are never fun to deal with, but we just need to be careful around the new ES6 `symbol` values and how we coerce them.

The good news: it's probably going to be exceedingly rare for you to need to coerce a `symbol` value. The way they're typically used (see Chapter 3) will probably not call for coercion on a normal basis.

## Loose Equals vs. Strict Equals

Loose equals is the `==` operator, and strict equals is the `===` operator. Both operators are used for comparing two values for "equality," but the "loose" vs. "strict" indicates a **very important** difference in behavior between the two, specifically in how they decide "equality."

A very common misconception about these two operators is: "`==` checks values for equality and `===` checks both values and types for equality." While that sounds nice and reasonable, it's inaccurate. Countless well-respected JavaScript books and blogs have said exactly that, but unfortunately they're all *wrong*.

The correct description is: "`==` allows coercion in the equality comparison and `===` disallows coercion."

### Equality Performance

Stop and think about the difference between the first (inaccurate) explanation and this second (accurate) one.

In the first explanation, it seems obvious that `===` is *doing more work* than `==`, because it has to *also* check the type. In the second explanation, `==` is the one *doing more work* because it has to follow through the steps of coercion if the types are different.

Don't fall into the trap, as many have, of thinking this has anything to do with performance, though, as if `==` is going to be slower than `===` in any relevant way. While it's measurable that coercion does take *a little bit* of processing time, it's mere microseconds (yes, that's millionths of a second!).

If you're comparing two values of the same types, `==` and `===` use the identical algorithm, and so other than minor differences in engine implementation, they should do the same work.

If you're comparing two values of different types, the performance isn't the important factor. What you should be asking yourself is: when comparing these two values, do I want coercion or not?

If you want coercion, use `==` loose equality, but if you don't want coercion, use `===` strict equality.

**Note:** The implication here then is that both `==` and `===` check the types of their operands. The difference is in how they respond if the types don't match.

### Abstract Equality

The `==` operator's behavior is defined as "The Abstract Equality Comparison Algorithm" in section 11.9.3 of the ES5 spec. What's listed there is a comprehensive but simple algorithm that explicitly states every possible combination of types, and how the coercions (if necessary) should happen for each combination.

**Warning:** When (*implicit*) coercion is maligned as being too complicated and too flawed to be a *useful good part*, it is these rules of "abstract equality" that are being condemned. Generally, they are said to be too complex and too unintuitive for developers to practically learn and use, and that they are prone more to causing bugs in JS programs than to enabling greater code readability. I believe this is a flawed premise -- that you readers are competent developers who write (and read and understand!) algorithms (aka code) all day long. So, what follows is a plain exposition of the "abstract equality" in simple terms. But I implore you to also read the ES5 spec section 11.9.3. I think you'll be surprised at just how reasonable it is.

Basically, the first clause (11.9.3.1) says, if the two values being compared are of the same type, they are simply and naturally compared via Identity as you'd expect. For example, `42` is only equal to `42`, and `"abc"` is only equal to `"abc"`.

Some minor exceptions to normal expectation to be aware of:

* `NaN` is never equal to itself (see Chapter 2)
* `+0` and `-0` are equal to each other (see Chapter 2)

The final provision in clause 11.9.3.1 is for `==` loose equality comparison with `object`s (including `function`s and `array`s). Two such values are only *equal* if they are both references to *the exact same value*. No coercion occurs here.

**Note:** The `===` strict equality comparison is defined identically to 11.9.3.1, including the provision about two `object` values. It's a very little known fact that **`==` and `===` behave identically** in the case where two `object`s are being compared!

The rest of the algorithm in 11.9.3 specifies that if you use `==` loose equality to compare two values of different types, one or both of the values will need to be *implicitly* coerced. This coercion happens so that both values eventually end up as the same type, which can then directly be compared for equality using simple value Identity.

**Note:** The `!=` loose not-equality operation is defined exactly as you'd expect, in that it's literally the `==` operation comparison performed in its entirety, then the negation of the result. The same goes for the `!==` strict not-equality operation.

#### Comparing: `string`s to `number`s

To illustrate `==` coercion, let's first build off the `string` and `number` examples earlier in this chapter:

```js
var a = 42;
var b = "42";

a === b;	// false
a == b;		// true
```

As we'd expect, `a === b` fails, because no coercion is allowed, and indeed the `42` and `"42"` values are different.

However, the second comparison `a == b` uses loose equality, which means that if the types happen to be different, the comparison algorithm will perform *implicit* coercion on one or both values.

But exactly what kind of coercion happens here? Does the `a` value of `42` become a `string`, or does the `b` value of `"42"` become a `number`?

In the ES5 spec, clauses 11.9.3.4-5 say:

> 4. If Type(x) is Number and Type(y) is String,
>    return the result of the comparison x == ToNumber(y).
> 5. If Type(x) is String and Type(y) is Number,
>    return the result of the comparison ToNumber(x) == y.

**Warning:** The spec uses `Number` and `String` as the formal names for the types, while this book prefers `number` and `string` for the primitive types. Do not let the capitalization of `Number` in the spec confuse you for the `Number()` native function. For our purposes, the capitalization of the type name is irrelevant -- they have basically the same meaning.

Clearly, the spec says the `"42"` value is coerced to a `number` for the comparison. The *how* of that coercion has already been covered earlier, specifically with the `ToNumber` abstract operation. In this case, it's quite obvious then that the resulting two `42` values are equal.

#### Comparing: anything to `boolean`

One of the biggest gotchas with the *implicit* coercion of `==` loose equality pops up when you try to compare a value directly to `true` or `false`.

Consider:

```js
var a = "42";
var b = true;

a == b;	// false
```

Wait, what happened here!? We know that `"42"` is a truthy value (see earlier in this chapter). So, how come it's not `==` loose equal to `true`?

The reason is both simple and deceptively tricky. It's so easy to misunderstand, many JS developers never pay close enough attention to fully grasp it.

Let's again quote the spec, clauses 11.9.3.6-7:

> 6. If Type(x) is Boolean,
>    return the result of the comparison ToNumber(x) == y.
> 7. If Type(y) is Boolean,
>    return the result of the comparison x == ToNumber(y).

Let's break that down. First:

```js
var x = true;
var y = "42";

x == y; // false
```

The `Type(x)` is indeed `Boolean`, so it performs `ToNumber(x)`, which coerces `true` to `1`. Now, `1 == "42"` is evaluated. The types are still different, so (essentially recursively) we reconsult the algorithm, which just as above will coerce `"42"` to `42`, and `1 == 42` is clearly `false`.

Reverse it, and we still get the same outcome:

```js
var x = "42";
var y = false;

x == y; // false
```

The `Type(y)` is `Boolean` this time, so `ToNumber(y)` yields `0`. `"42" == 0` recursively becomes `42 == 0`, which is of course `false`.

In other words, **the value `"42"` is neither `== true` nor `== false`.** At first, that statement might seem crazy. How can a value be neither truthy nor falsy?

But that's the problem! You're asking the wrong question, entirely. It's not your fault, really. Your brain is tricking you.

`"42"` is indeed truthy, but `"42" == true` **is not performing a boolean test/coercion** at all, no matter what your brain says. `"42"` *is not* being coerced to a `boolean` (`true`), but instead `true` is being coerced to a `1`, and then `"42"` is being coerced to `42`.

Whether we like it or not, `ToBoolean` is not even involved here, so the truthiness or falsiness of `"42"` is irrelevant to the `==` operation!

What *is* relevant is to understand how the `==` comparison algorithm behaves with all the different type combinations. As it regards a `boolean` value on either side of the `==`, a `boolean` always coerces to a `number` *first*.

If that seems strange to you, you're not alone. I personally would recommend to never, ever, under any circumstances, use `== true` or `== false`. Ever.

But remember, I'm only talking about `==` here. `=== true` and `=== false` wouldn't allow the coercion, so they're safe from this hidden `ToNumber` coercion.

Consider:

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

If you avoid ever using `== true` or `== false` (aka loose equality with `boolean`s) in your code, you'll never have to worry about this truthiness/falsiness mental gotcha.

#### Comparing: `null`s to `undefined`s

Another example of *implicit* coercion can be seen with `==` loose equality between `null` and `undefined` values. Yet again quoting the ES5 spec, clauses 11.9.3.2-3:

> 2. If x is null and y is undefined, return true.
> 3. If x is undefined and y is null, return true.

`null` and `undefined`, when compared with `==` loose equality, equate to (aka coerce to) each other (as well as themselves, obviously), and no other values in the entire language.

What this means is that `null` and `undefined` can be treated as indistinguishable for comparison purposes, if you use the `==` loose equality operator to allow their mutual *implicit* coercion.

```js
var a = null;
var b;

a == b;		// true
a == null;	// true
b == null;	// true

a == false;	// false
b == false;	// false
a == "";	// false
b == "";	// false
a == 0;		// false
b == 0;		// false
```

The coercion between `null` and `undefined` is safe and predictable, and no other values can give false positives in such a check. I recommend using this coercion to allow `null` and `undefined` to be indistinguishable and thus treated as the same value.

For example:

```js
var a = doSomething();

if (a == null) {
	// ..
}
```

The `a == null` check will pass only if `doSomething()` returns either `null` or `undefined`, and will fail with any other value, even other falsy values like `0`, `false`, and `""`.

The *explicit* form of the check, which disallows any such coercion, is (I think) unnecessarily much uglier (and perhaps a tiny bit less performant!):

```js
var a = doSomething();

if (a === undefined || a === null) {
	// ..
}
```

In my opinion, the form `a == null` is yet another example where *implicit* coercion improves code readability, but does so in a reliably safe way.

#### Comparing: `object`s to non-`object`s

If an `object`/`function`/`array` is compared to a simple scalar primitive (`string`, `number`, or `boolean`), the ES5 spec says in clauses 11.9.3.8-9:

> 8. If Type(x) is either String or Number and Type(y) is Object,
>    return the result of the comparison x == ToPrimitive(y).
> 9. If Type(x) is Object and Type(y) is either String or Number,
>    return the result of the comparison ToPrimitive(x) == y.

**Note:** You may notice that these clauses only mention `String` and `Number`, but not `Boolean`. That's because, as quoted earlier, clauses 11.9.3.6-7 take care of coercing any `Boolean` operand presented to a `Number` first.

Consider:

```js
var a = 42;
var b = [ 42 ];

a == b;	// true
```

The `[ 42 ]` value has its `ToPrimitive` abstract operation called (see the "Abstract Value Operations" section earlier), which results in the `"42"` value. From there, it's just `42 == "42"`, which as we've already covered becomes `42 == 42`, so `a` and `b` are found to be coercively equal.

**Tip:** All the quirks of the `ToPrimitive` abstract operation that we discussed earlier in this chapter (`toString()`, `valueOf()`) apply here as you'd expect. This can be quite useful if you have a complex data structure that you want to define a custom `valueOf()` method on, to provide a simple value for equality comparison purposes.

In Chapter 3, we covered "unboxing," where an `object` wrapper around a primitive value (like from `new String("abc")`, for instance) is unwrapped, and the underlying primitive value (`"abc"`) is returned. This behavior is related to the `ToPrimitive` coercion in the `==` algorithm:

```js
var a = "abc";
var b = Object( a );	// same as `new String( a )`

a === b;				// false
a == b;					// true
```

`a == b` is `true` because `b` is coerced (aka "unboxed," unwrapped) via `ToPrimitive` to its underlying `"abc"` simple scalar primitive value, which is the same as the value in `a`.

There are some values where this is not the case, though, because of other overriding rules in the `==` algorithm. Consider:

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

The `null` and `undefined` values cannot be boxed -- they have no object wrapper equivalent -- so `Object(null)` is just like `Object()` in that both just produce a normal object.

`NaN` can be boxed to its `Number` object wrapper equivalent, but when `==` causes an unboxing, the `NaN == NaN` comparison fails because `NaN` is never equal to itself (see Chapter 2).

### Edge Cases

Now that we've thoroughly examined how the *implicit* coercion of `==` loose equality works (in both sensible and surprising ways), let's try to call out the worst, craziest corner cases so we can see what we need to avoid to not get bitten with coercion bugs.

First, let's examine how modifying the built-in native prototypes can produce crazy results:

#### A Number By Any Other Value Would...

```js
Number.prototype.valueOf = function() {
	return 3;
};

new Number( 2 ) == 3;	// true
```

**Warning:** `2 == 3` would not have fallen into this trap, because neither `2` nor `3` would have invoked the built-in `Number.prototype.valueOf()` method because both are already primitive `number` values and can be compared directly. However, `new Number(2)` must go through the `ToPrimitive` coercion, and thus invoke `valueOf()`.

Evil, huh? Of course it is. No one should ever do such a thing. The fact that you *can* do this is sometimes used as a criticism of coercion and `==`. But that's misdirected frustration. JavaScript is not *bad* because you can do such things, a developer is *bad* **if they do such things**. Don't fall into the "my programming language should protect me from myself" fallacy.

Next, let's consider another tricky example, which takes the evil from the previous example to another level:

```js
if (a == 2 && a == 3) {
	// ..
}
```

You might think this would be impossible, because `a` could never be equal to both `2` and `3` *at the same time*. But "at the same time" is inaccurate, since the first expression `a == 2` happens strictly *before* `a == 3`.

So, what if we make `a.valueOf()` have side effects each time it's called, such that the first time it returns `2` and the second time it's called it returns `3`? Pretty easy:

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

Again, these are evil tricks. Don't do them. But also don't use them as complaints against coercion. Potential abuses of a mechanism are not sufficient evidence to condemn the mechanism. Just avoid these crazy tricks, and stick only with valid and proper usage of coercion.

#### False-y Comparisons

The most common complaint against *implicit* coercion in `==` comparisons comes from how falsy values behave surprisingly when compared to each other.

To illustrate, let's look at a list of the corner-cases around falsy value comparisons, to see which ones are reasonable and which are troublesome:

```js
"0" == null;			// false
"0" == undefined;		// false
"0" == false;			// true -- UH OH!
"0" == NaN;				// false
"0" == 0;				// true
"0" == "";				// false

false == null;			// false
false == undefined;		// false
false == NaN;			// false
false == 0;				// true -- UH OH!
false == "";			// true -- UH OH!
false == [];			// true -- UH OH!
false == {};			// false

"" == null;				// false
"" == undefined;		// false
"" == NaN;				// false
"" == 0;				// true -- UH OH!
"" == [];				// true -- UH OH!
"" == {};				// false

0 == null;				// false
0 == undefined;			// false
0 == NaN;				// false
0 == [];				// true -- UH OH!
0 == {};				// false
```

In this list of 24 comparisons, 17 of them are quite reasonable and predictable. For example, we know that `""` and `NaN` are not at all equatable values, and indeed they don't coerce to be loose equals, whereas `"0"` and `0` are reasonably equatable and *do* coerce as loose equals.

However, seven of the comparisons are marked with "UH OH!" because as false positives, they are much more likely gotchas that could trip you up. `""` and `0` are definitely distinctly different values, and it's rare you'd want to treat them as equatable, so their mutual coercion is troublesome. Note that there aren't any false negatives here.

#### The Crazy Ones

We don't have to stop there, though. We can keep looking for even more troublesome coercions:

```js
[] == ![];		// true
```

Oooo, that seems at a higher level of crazy, right!? Your brain may likely trick you that you're comparing a truthy to a falsy value, so the `true` result is surprising, as we *know* a value can never be truthy and falsy at the same time!

But that's not what's actually happening. Let's break it down. What do we know about the `!` unary operator? It explicitly coerces to a `boolean` using the `ToBoolean` rules (and it also flips the parity). So before `[] == ![]` is even processed, it's actually already translated to `[] == false`. We already saw that form in our above list (`false == []`), so its surprise result is *not new* to us.

How about other corner cases?

```js
2 == [2];		// true
"" == [null];	// true
```

As we said earlier in our `ToNumber` discussion, the right-hand side `[2]` and `[null]` values will go through a `ToPrimitive` coercion so they can be more readily compared to the simple primitives (`2` and `""`, respectively) on the left-hand side. Since the `valueOf()` for `array` values just returns the `array` itself, coercion falls to stringifying the `array`.

`[2]` will become `"2"`, which then is `ToNumber` coerced to `2` for the right-hand side value in the first comparison. `[null]` just straight becomes `""`.

So, `2 == 2` and `"" == ""` are completely understandable.

If your instinct is to still dislike these results, your frustration is not actually with coercion like you probably think it is. It's actually a complaint against the default `array` values' `ToPrimitive` behavior of coercing to a `string` value. More likely, you'd just wish that `[2].toString()` didn't return `"2"`, or that `[null].toString()` didn't return `""`.

But what exactly *should* these `string` coercions result in? I can't really think of any other appropriate `string` coercion of `[2]` than `"2"`, except perhaps `"[2]"` -- but that could be very strange in other contexts!

You could rightly make the case that since `String(null)` becomes `"null"`, then `String([null])` should also become `"null"`. That's a reasonable assertion. So, that's the real culprit.

*Implicit* coercion itself isn't the evil here. Even an *explicit* coercion of `[null]` to a `string` results in `""`. What's at odds is whether it's sensible at all for `array` values to stringify to the equivalent of their contents, and exactly how that happens. So, direct your frustration at the rules for `String( [..] )`, because that's where the craziness stems from. Perhaps there should be no stringification coercion of `array`s at all? But that would have lots of other downsides in other parts of the language.

Another famously cited gotcha:

```js
0 == "\n";		// true
```

As we discussed earlier with empty `""`, `"\n"` (or `" "` or any other whitespace combination) is coerced via `ToNumber`, and the result is `0`. What other `number` value would you expect whitespace to coerce to? Does it bother you that *explicit* `Number(" ")` yields `0`?

Really the only other reasonable `number` value that empty strings or whitespace strings could coerce to is the `NaN`. But would that *really* be better? The comparison `" " == NaN` would of course fail, but it's unclear that we'd have really *fixed* any of the underlying concerns.

The chances that a real-world JS program fails because `0 == "\n"` are awfully rare, and such corner cases are easy to avoid.

Type conversions **always** have corner cases, in any language -- nothing specific to coercion. The issues here are about second-guessing a certain set of corner cases (and perhaps rightly so!?), but that's not a salient argument against the overall coercion mechanism.

Bottom line: almost any crazy coercion between *normal values* that you're likely to run into (aside from intentionally tricky `valueOf()` or `toString()` hacks as earlier) will boil down to the short seven-item list of gotcha coercions we've identified above.

To contrast against these 24 likely suspects for coercion gotchas, consider another list like this:

```js
42 == "43";							// false
"foo" == 42;						// false
"true" == true;						// false

42 == "42";							// true
"foo" == [ "foo" ];					// true
```

In these nonfalsy, noncorner cases (and there are literally an infinite number of comparisons we could put on this list), the coercion results are totally safe, reasonable, and explainable.

#### Sanity Check

OK, we've definitely found some crazy stuff when we've looked deeply into *implicit* coercion. No wonder that most developers claim coercion is evil and should be avoided, right!?

But let's take a step back and do a sanity check.

By way of magnitude comparison, we have *a list* of seven troublesome gotcha coercions, but we have *another list* of (at least 17, but actually infinite) coercions that are totally sane and explainable.

If you're looking for a textbook example of "throwing the baby out with the bathwater," this is it: discarding the entirety of coercion (the infinitely large list of safe and useful behaviors) because of a list of literally just seven gotchas.

The more prudent reaction would be to ask, "how can I use the countless *good parts* of coercion, but avoid the few *bad parts*?"

Let's look again at the *bad* list:

```js
"0" == false;			// true -- UH OH!
false == 0;				// true -- UH OH!
false == "";			// true -- UH OH!
false == [];			// true -- UH OH!
"" == 0;				// true -- UH OH!
"" == [];				// true -- UH OH!
0 == [];				// true -- UH OH!
```

Four of the seven items on this list involve `== false` comparison, which we said earlier you should **always, always** avoid. That's a pretty easy rule to remember.

Now the list is down to three.

```js
"" == 0;				// true -- UH OH!
"" == [];				// true -- UH OH!
0 == [];				// true -- UH OH!
```

Are these reasonable coercions you'd do in a normal JavaScript program? Under what conditions would they really happen?

I don't think it's terribly likely that you'd literally use `== []` in a `boolean` test in your program, at least not if you know what you're doing. You'd probably instead be doing `== ""` or `== 0`, like:

```js
function doSomething(a) {
	if (a == "") {
		// ..
	}
}
```

You'd have an oops if you accidentally called `doSomething(0)` or `doSomething([])`. Another scenario:

```js
function doSomething(a,b) {
	if (a == b) {
		// ..
	}
}
```

Again, this could break if you did something like `doSomething("",0)` or `doSomething([],"")`.

So, while the situations *can* exist where these coercions will bite you, and you'll want to be careful around them, they're probably not super common on the whole of your code base.

#### Safely Using Implicit Coercion

The most important advice I can give you: examine your program and reason about what values can show up on either side of an `==` comparison. To effectively avoid issues with such comparisons, here's some heuristic rules to follow:

1. If either side of the comparison can have `true` or `false` values, don't ever, EVER use `==`.
2. If either side of the comparison can have `[]`, `""`, or `0` values, seriously consider not using `==`.

In these scenarios, it's almost certainly better to use `===` instead of `==`, to avoid unwanted coercion. Follow those two simple rules and pretty much all the coercion gotchas that could reasonably hurt you will effectively be avoided.

**Being more explicit/verbose in these cases will save you from a lot of headaches.**

The question of `==` vs. `===` is really appropriately framed as: should you allow coercion for a comparison or not?

There's lots of cases where such coercion can be helpful, allowing you to more tersely express some comparison logic (like with `null` and `undefined`, for example).

In the overall scheme of things, there's relatively few cases where *implicit* coercion is truly dangerous. But in those places, for safety sake, definitely use `===`.

**Tip:** Another place where coercion is guaranteed *not* to bite you is with the `typeof` operator. `typeof` is always going to return you one of seven strings (see Chapter 1), and none of them are the empty `""` string. As such, there's no case where checking the type of some value is going to run afoul of *implicit* coercion. `typeof x == "function"` is 100% as safe and reliable as `typeof x === "function"`. Literally, the spec says the algorithm will be identical in this situation. So, don't just blindly use `===` everywhere simply because that's what your code tools tell you to do, or (worst of all) because you've been told in some book to **not think about it**. You own the quality of your code.

Is *implicit* coercion evil and dangerous? In a few cases, yes, but overwhelmingly, no.

Be a responsible and mature developer. Learn how to use the power of coercion (both *explicit* and *implicit*) effectively and safely. And teach those around you to do the same.

Here's a handy table made by Alex Dorey (@dorey on GitHub) to visualize a variety of comparisons:

<img src="fig1.png" width="600">

Source: https://github.com/dorey/JavaScript-Equality-Table

## Abstract Relational Comparison

While this part of *implicit* coercion often gets a lot less attention, it's important nonetheless to think about what happens with `a < b` comparisons (similar to how we just examined `a == b` in depth).

The "Abstract Relational Comparison" algorithm in ES5 section 11.8.5 essentially divides itself into two parts: what to do if the comparison involves both `string` values (second half), or anything else (first half).

**Note:** The algorithm is only defined for `a < b`. So, `a > b` is handled as `b < a`.

The algorithm first calls `ToPrimitive` coercion on both values, and if the return result of either call is not a `string`, then both values are coerced to `number` values using the `ToNumber` operation rules, and compared numerically.

For example:

```js
var a = [ 42 ];
var b = [ "43" ];

a < b;	// true
b < a;	// false
```

**Note:** Similar caveats for `-0` and `NaN` apply here as they did in the `==` algorithm discussed earlier.

However, if both values are `string`s for the `<` comparison, simple lexicographic (natural alphabetic) comparison on the characters is performed:

```js
var a = [ "42" ];
var b = [ "043" ];

a < b;	// false
```

`a` and `b` are *not* coerced to `number`s, because both of them end up as `string`s after the `ToPrimitive` coercion on the two `array`s. So, `"42"` is compared character by character to `"043"`, starting with the first characters `"4"` and `"0"`, respectively. Since `"0"` is lexicographically *less than* than `"4"`, the comparison returns `false`.

The exact same behavior and reasoning goes for:

```js
var a = [ 4, 2 ];
var b = [ 0, 4, 3 ];

a < b;	// false
```

Here, `a` becomes `"4,2"` and `b` becomes `"0,4,3"`, and those lexicographically compare identically to the previous snippet.

What about:

```js
var a = { b: 42 };
var b = { b: 43 };

a < b;	// ??
```

`a < b` is also `false`, because `a` becomes `[object Object]` and `b` becomes `[object Object]`, and so clearly `a` is not lexicographically less than `b`.

But strangely:

```js
var a = { b: 42 };
var b = { b: 43 };

a < b;	// false
a == b;	// false
a > b;	// false

a <= b;	// true
a >= b;	// true
```

Why is `a == b` not `true`? They're the same `string` value (`"[object Object]"`), so it seems they should be equal, right? Nope. Recall the previous discussion about how `==` works with `object` references.

But then how are `a <= b` and `a >= b` resulting in `true`, if `a < b` **and** `a == b` **and** `a > b` are all `false`?

Because the spec says for `a <= b`, it will actually evaluate `b < a` first, and then negate that result. Since `b < a` is *also* `false`, the result of `a <= b` is `true`.

That's probably awfully contrary to how you might have explained what `<=` does up to now, which would likely have been the literal: "less than *or* equal to." JS more accurately considers `<=` as "not greater than" (`!(a > b)`, which JS treats as `!(b < a)`). Moreover, `a >= b` is explained by first considering it as `b <= a`, and then applying the same reasoning.

Unfortunately, there is no "strict relational comparison" as there is for equality. In other words, there's no way to prevent *implicit* coercion from occurring with relational comparisons like `a < b`, other than to ensure that `a` and `b` are of the same type explicitly before making the comparison.

Use the same reasoning from our earlier `==` vs. `===` sanity check discussion. If coercion is helpful and reasonably safe, like in a `42 < "43"` comparison, **use it**. On the other hand, if you need to be safe about a relational comparison, *explicitly coerce* the values first, before using `<` (or its counterparts).

```js
var a = [ 42 ];
var b = "043";

a < b;						// false -- string comparison!
Number( a ) < Number( b );	// true -- number comparison!
```

## Review
