# Foreword
# Preface
* While JavaScript [...] one of the easiest languages to get up and running [...] make solid mastery of the language a vastly `less common`.
* Where it takes a pretty in-depth knowledge of [...] C or C++ to write a `full-scale program`, `full-scale production JavaScript` can [...] barely scratch the surface of what the language can do.

# Chapter 1: Into Programming
## Code
* The `rules for` `valid format` and `combinations of instructions` is [...] `computer language`, sometimes [...] `syntax`.

### Statements
* `a group of` words, numbers, and operators that `performs a specific task` is a `statement`.

```js
a = b * 2;
```

* `a` and `b` are [...] variables [...] store [...] values.
* `2` is just a value [...] a `literal value`, because it `stands alone` without being stored in a variable.
* `=` and `*` characters are operators [...] perform actions with the values and variable.

### Expressions
* is any `reference to` a variable or value, or a `set of` variable(s) and value(s) combined with operators.

```js
a = b * 2;
```

* `2` is a `literal value expression`.
* `b` is a `variable expression`, which means to `retrieve its current value`.
* `b * 2` is an `arithmetic expression`, which means to do the multiplication (me: and return the result of the calculation).
* `a = b * 2` is an `assignment expression`, which means to assign the result of the `b * 2` expression to the variable `a` (me: the variable `a` return the result of the assignment).

* A general expression that `stands alone` is also called an `expression statement`.

```js
b * 2;
```

* `call expression statement` [...] is the `function call expression`.

```js
alert( a );
```

### Executing a Program
* Statements like `a = b * 2` are helpful for developers [...] not [...] a form the computer can directly understand. So [...] an `interpreter` or a `compiler` [...] is used to translate the code you write into commands a computer can understand.

* For some computer languages, this translation of commands is typically done from top to bottom, line by line, `every time the program is run` [...] called `interpreting the code`.
* For other languages, the translation is done `ahead of time`, called `compiling` the code, so when the program runs [...] what's running is actually the `already compiled` computer instructions.

* It's [...] asserted that JavaScript is `interpreted`, because your JavaScript source code is processed each time it's run. But [...] `not entirely accurate`. The JavaScript engine actually `compiles` the program `on the fly` and then `immediately runs` the compiled code.
* For more information on JavaScript compiling, see the `first two chapters` of the `Scope & Closures` title of this series.

## Try It Yourself
### Output
* how we print text (aka `output` to the user) in the developer console.
```js
alert( b );
```

```js
console.log( b );
```

### Input
* `input` (i.e., receiving information from the user).

```js
age = prompt( "Please tell me your age:" );

console.log( age );
```

## Operators
* `=` equals operator is used for `assignment` -- we first calculate the value on the `right-hand side` (`source value`) of the = and then put it into the variable that we specify on the `left-hand side` (`target variable`).
* `var` [...] the primary way you `declare` (aka `create`) variables.

* most common operators:
  * Assignment: `=`.
  * Math: `+` (addition), `-` (subtraction), `*` (multiplication), and `/` (division).
  * Compound Assignment: `+=`, `-=`, `*=`, and `/=`.
  * Increment/Decrement: `++` (increment), `--` (decrement).
  * Object Property Access: `.`.
    * Objects are `values that hold other values` `at specific named locations` called `properties`.
    * Properties can alternatively be accessed as `obj["a"]`.
  * Equality: `==` (loose-equals), `===` (strict-equals), `!=` (loose not-equals), `!==` (strict not-equals).
  * Comparison: `<` (less than), `>` (greater than), `<=` (less than or loose-equals), `>=` (greater than or loose-equals).
  * Logical: `&&` (and), `||` (or).

## Values & Types
* Values [...] included directly in the source code are called `literals`.

### Converting Between Types
* in JavaScript [...] (me: like converting `number` to `string`)(conversion) is called "`coercion`.".

```js
var a = "42";
var b = Number( a );

console.log( a );	// "42"
console.log( b );	// 42
```

* Using `Number(..)` [...] is an `explicit coercion` from `any other type` to the `number` type.

* compare two values that are `not` already of the `same type` [...] require `implicit coercion`.
* if you use the `==` `loose equals` [...] `"99.99" == 99.99`, JavaScript will convert the `left-hand side` [...] number equivalent `99.99`. The comparison then becomes `99.99 == 99.99`, which is of course `true`.
* For more information on `coercion`, see `Chapter 2 of this title` and `Chapter 4` of the `Types & Grammar` title of this series.

## Code Comments
* The `interpreter/compiler` will always ignore [...] comments.
* two types of comments [...] a `single-line comment` and a `multiline comment`.

```js
// This is a single-line comment

/* But this is
       a multiline
             comment.
                      */
```

* The `only` thing that cannot appear inside a `multiline comment` is a `*/`.

## Variables
* `variable` -- so called because the value in this container can `vary over time`.
* `console.log(..)` command has to `implicitly coerce` [...] number value to a string to print it out.
* `amount = "$" + String(amount)` `explicitly coerces` the (me: the value held by `amount`)(199.98) value to a string and adds a "`$`" character to the beginning.

* By convention, JavaScript variables as `constants` are usually `capitalized`, with underscores `_` between `multiple words`.

```js
var TAX_RATE = 0.08;	// 8% sales tax
```

* The `TAX_RATE` variable is only constant `by convention` -- there's nothing [...] prevents it from being changed.

* The `newest version of JavaScript` [...] (commonly called "`ES6`") includes a new way to declare `constants`, by using `const`.
* For more information [...] see the `Types & Grammar` title of this series.

## Blocks
* a `block` is defined by [...] { .. }.

```js
var amount = 99.99;

// a general block
{
	amount = amount * 2;
	console.log( amount );	// 199.98
}
```

* `{ .. }` [...] is valid, `but` isn't as commonly seen in JS programs. Typically, blocks are attached to some other `control statement`.

```js
var amount = 99.99;

// is amount big enough?
if (amount > 10) {			// <-- block attached to `if`
	amount = amount * 2;
	console.log( amount );	// 199.98
}
```

* the `{ .. }` block with its two statements is attached to `if (amount > 10)`.
* a `block statement` does not need a `semicolon` (;).

## Conditionals
* The `if` statement expects a boolean, but if you pass it something that's `not already boolean`, `coercion` will occur.

* JavaScript defines a list of specific values that are considered "`falsy`" because when coerced to a `boolean`, they become `false` [...] Any other value not on the "`falsy`" list is automatically "`truthy`" -- when coerced to a `boolean` they become `true` [...] See "`Truthy & Falsy`" in Chapter 2.

## Loops
* A loop includes the `test condition` as well as a `block` [...] Each time the loop `block` executes, `that's called` an `iteration`.

```js
while (numOfCustomers > 0) {
	console.log( "How may I help you?" );

	// help the customer...

	numOfCustomers = numOfCustomers - 1;
}

// versus:

do {
	console.log( "How may I help you?" );

	// help the customer...

	numOfCustomers = numOfCustomers - 1;
} while (numOfCustomers > 0);
```

* The only [...] difference between these loops is whether the conditional is tested `before the first iteration` (`while`) `or` `after the first iteration` (`do..while`).
* if the condition is `initially false`, a `while loop` will never run, but a `do..while` loop will run `just the first time`.

* We can use [...] `break` statement to `stop a loop`.

```js
var i = 0;

// a `while..true` loop would run forever, right?
while (true) {
	// stop the loop?
	if ((i <= 9) === false) {
		break;
	}

	console.log( i );
	i = i + 1;
}
// 0 1 2 3 4 5 6 7 8 9
```

* there's another syntactic form called a `for` loop.

```js
for (var i = 0; i <= 9; i = i + 1) {
	console.log( i );
}
// 0 1 2 3 4 5 6 7 8 9
```

## Functions
* is generally a `named section` of code that can be "`called`" by name, and the code inside it will be run.
* `arguments` (aka `parameters`) -- values you pass in.
* are often used for code that you plan to `call multiple times` [...] also be useful just to `organize` related bits of code into named collections, `even if` [...] only plan to call them `once`.

## Scope
* `scope` (technically called `lexical scope`) [...] each `function` gets its own scope. `Scope` is basically a `collection of variables` as well as the `rules for how` those variables are `accessed by name`.
* Only code inside that function can access that `function's scoped variables`.
* can't be two different `a` variables [...] next to each other. `But` the same variable name `a` could appear in `different scopes`.
* If one scope is `nested` inside another, code inside the `innermost` scope can access variables from `either` scope.

* more information about lexical scope, see the first three chapters of the `Scope & Closures` title of this series.

## Practice
## Review

# Chapter 2: Into JavaScript
* newest `version of JavaScript` [...] "ES6" [...] the 6th edition of ECMAScript -- the official name of the JS specification.

## Values & Types
* JavaScript has `typed values`, `not` `typed variables`.

* (me: `array` and `function` are `subtypes` -- specialized versions of the `object` type).
* built-in types.
	* string.
  * number.
  * boolean.
  * `null` and `undefined`.
  * object.
  * symbol (new to ES6).

* `typeof` operator that can examine a value and tell you what type it is:

```js
var a;
typeof a; // "undefined"

a = "hello world";
typeof a; // "string"

a = 42;
typeof a; // "number"

a = true;
typeof a; // "boolean"

a = null;
typeof a; // "object" -- weird, bug

a = undefined;
typeof a; // "undefined"

a = { b: "c" };
typeof a; // "object"

a = [1, 2, 3];
typeof a; // "object"

// me:
function foo() {
	return 42;
}

typeof foo;			// "function"
```

* The return value from the `typeof` operator is always one of `six` (`seven` as of `ES6`! - the "`symbol`" type).
* `typeof "abc"` returns `"string"`, `not` `string`.
* `Only` values have types in JavaScript; variables are just simple containers for those values.
* `typeof null` [...] `errantly` returns `"object"`, when you'd expect it to return `"null"` [...] This is a `long-standing bug` in JS [...] Too much code on the Web relies on `the bug` [...] fixing it would cause a lot more bugs!

* `a = undefined` [...] no different from `var a;`.
* A variable can get to [...] "`undefined`" value [...] including functions that return `no values` and usage of the `void` `operator`.

### Objects
* a compound value where you can set properties (named locations) that each hold their own values of any type.

* `Bracket notation` is useful if you have a property name that has `special characters` [...] `obj["hello world!"]`.
* `[ ] notation` requires either a `variable` [...] or a `string literal`.

* `array` and `function` [...] rather than being proper built-in types, these [...] more like `subtypes` -- specialized versions of the `object` type.

#### Arrays
* is an `object` that holds values (of `any type`) [...] in numerically indexed positions.
* they [...] have [...] the `automatically updated` `length` `property`.

* You theoretically could use an array as a `normal object` with your `own named properties`, or you could use an object but only give it `numeric properties` (0, 1, etc.) [...] improper usage.
* The best [...] approach is to use arrays for `numerically positioned values` and use objects for `named properties`.

#### Functions
```js
function foo() {
	return 42;
}

foo.bar = "hello world";

typeof foo;			// "function"
typeof foo();		// "number"
typeof foo.bar;	// "string"
```

* are a `subtype` of objects [...] -- and can thus have properties [...] like `foo.bar`.

### Built-In Type Methods
```js
var a = "hello world";
var b = 3.14159;

a.length;				 // 11
a.toUpperCase(); // "HELLO WORLD"
b.toFixed(4);		 // "3.1416"
```

* "how" behind being able to call `a.toUpperCase()` is.
  * there is a `String` (`capital` S) `object wrapper` [...] called a `"native,"`.
  * When you use a `primitive value` like `"hello world"` as an object by `referencing a property` or method (e.g., `a.toUpperCase()` [...]), JS `automatically` "boxes" the value to its `object wrapper` counterpart (hidden under the covers).
  * A `string` value can be wrapped by a `String` object, a `number` can be wrapped by a `Number` object, and a `boolean` can be wrapped by a `Boolean` object.

* more information on `JS natives` and `"boxing,"` see Chapter 3 of the Types & Grammar title of this series.

### Comparing Values
* `two main` types of value comparison [...] `equality` and `inequality`.

#### Coercion
* `two forms` [...] `explicit` and `implicit`.
* `Chapter 4` of the `Types & Grammar title` [...] covers all sides.

* `explicit coercion`:

```js
var a = "42";

var b = Number( a );

a;				// "42"
b;				// 42 -- the number!
```

* `implicit coercion`:

```js
var a = "42";

var b = a * 1;	// "42" implicitly coerced to 42 here

a;				// "42"
b;				// 42 -- the number!
```

#### Truthy & Falsy
* list of "`falsy`" values:
  * `""` (empty string).
  * `0`, `-0`, `NaN` (invalid number).
  * `null`, `undefined`.
  * `false`.

* `Any value` that's `not` on this "falsy" list is "`truthy.`".

#### Equality
* four `equality operators`: `==`, `===`, `!=`, and `!==`. The `!` forms are [...] (me: `non-equality`)(`"not equal"`) versions of their counterparts;
* `non-equality` should `not` be confused with (me: more on this later)(`inequality`).

* `==` checks for `value equality` and `===` checks for both `value` and `type` equality. `However`, this is `inaccurate` [...] `==` checks for `value equality` `with coercion allowed`, and `===` checks for `value equality` `without allowing coercion`.

```js
var a = "42";
var b = 42;

a == b;			// true
a === b;		// false
```

* In the `a == b` [...] the types do not match, so it goes through an ordered series of steps to coerce `one or both` values to a different type until the types match [...] then a simple value equality can be checked.

* two possible ways `a == b` could give true via coercion. Either [...] `42 == 42` or [...] `"42" == "42"`. So which is it?
* The answer: `"42"` becomes `42` [...] it doesn't really seem to matter which way that process goes [...] There are more complex cases where it matters.

* The `a === b` produces `false`, because the `coercion is not allowed`.

* You can read `section 11.9.3` of the `ES5 specification` ([http://www.ecma-international.org/ecma-262/5.1/](http://www.ecma-international.org/ecma-262/5.1/)) to see the exact (me: rules of how `==` works)(rules).

* The `!=` `non-equality` form pairs with `==`, and the `!==` form pairs with `===`. `All` the rules and observations we just discussed hold symmetrically for `these` non-equality comparisons.

* if you're comparing `two non-primitive values`, like `objects` (including `function` and `array`). Because those values are actually `held by reference`, both `==` and `===` comparisons will simply check whether the `references` match, `not` [...] `values`.

* For example, `arrays` are by default coerced to strings by simply `joining` all the values with `commas` (,) in between. You might think that two arrays with the same contents would be == equal, but they're not:

```js
var a = [1,2,3];
var b = [1,2,3];
var c = "1,2,3";

a == c;		// true
b == c;		// true
a == b;		// (me: different references)(false)
```

#### Inequality
* `<`, `>`, `<=`, and `>=`.
* Typically [...] used with ordinally `comparable values` like `numbers` [...] `3 < 4`.

* But [...] `string` values can also be compared for inequality [...] `"bar" < "foo"`.
* Similar rules as `==` comparison (though `not exactly` identical!) apply to the `inequality operators`.
* there are `no` `"strict inequality"` operators that would `disallow` coercion the same way `===` `"strict equality"` does.

```js
var a = 41;
var b = "42";
var c = "43";

a < b;		// true
b < c;		// true
```

* if `both` values in the < comparison are `strings`, as it is with `b < c`, the comparison is made `lexicographically` (aka alphabetically like a dictionary).
* if `one or both` is not a string, as it is with `a < b`, then `both` values are coerced to be `numbers`.

* The biggest gotcha you may run into here with comparisons between potentially `different value types` -- remember, there are `no` "strict inequality" forms to use -- is when one of the values `cannot be made` into a `valid number`, such as:

```js
var a = 42;
var b = "foo";

a < b;		// false
a > b;		// false
a == b;		// false
```

* how can all three of those comparisons be `false`? Because the `b` value is being coerced to the `"invalid number value"` `NaN` in the `<` and `>` comparisons, and the specification says that `NaN` is `neither` `greater-than` nor `less-than` `any other value`.
* The `==` comparison fails for a different reason. `a == b` could fail if it's interpreted either as `42 == NaN` or `"42" == "foo"`.

* more information about the `inequality` comparison rules, see section [`11.8.5`](https://262.ecma-international.org/5.1/#sec-11.8.5) of the `ES5 specification` and also consult `Chapter 4` of the `Types & Grammar` title of this series.

## Variables
* The strict and complete rules for valid characters in `identifiers` are a little complex when you consider `nontraditional characters` such as `Unicode`. If you only consider typical `ASCII alphanumeric characters` [...] the rules are [...] An `identifier` must start with `a-z`, `A-Z`, `$`, or `_`. It can then contain any of `those` characters plus the numerals `0-9`.

* same rules apply to a `property name` [...] `However`, certain words `cannot` be used as variables, `but` are OK as property names. These words are called `"reserved words,"` `and` include the JS `keywords` (`for`, `in`, `if`, etc.) as well as `null`, `true`, and `false`.
* more information about `reserved words`, see `Appendix A` of the `Types & Grammar` title of this series.

### Function Scopes
#### Hoisting
* `hoisting`, when a var declaration is [...] `"moved"` to the top of `its enclosing scope`.

```js
var a = 2;

foo();					// works because `foo()`
						// declaration is "hoisted"

function foo() {
	a = 3;

	console.log( a );	// 3

	var a;				// declaration is "hoisted"
						// to the top of `foo()`
}

console.log( a );	// 2
```

* `not` common [...] to rely on `variable hoisting`.
* common and accepted to use `hoisted function declarations`, as we do with the `foo()` call.

#### Nested Scopes
* When you declare a variable, it is available `anywhere` in that scope, as well as any `lower/inner scopes`.

* If you try to set a variable that `hasn't been declared` [...] end up `creating a variable` in the `top-level global scope` (`bad!`) or getting an error, depending on "`strict mode`" (see "`Strict Mode`").

```js
function foo() {
	a = 1;	// `a` not formally declared
}

foo();
a;			// 1 -- oops, auto global variable :( (me: no error here)
```

* `ES6` lets you declare variables to belong to `individual blocks` (pairs of `{ .. }`).

* more information about `scope`, see the `Scope & Closures` title of this series. See the `ES6 & Beyond` title of this series for more information about `let` `block scoping`.

## Conditionals
* `break` is important if you want only the statement(s) in `one` `case` to run. If you omit `break` from a `case`, and that `case` matches or runs, execution will continue with the next `case`'s statements `regardless of` that case matching. This so called "`fall through`".

```js
switch (a) {
	case 2:
	case 10:
		// some cool stuff
		break;
	case 42:
		// other stuff
		break;
	default:
		// fallback
}
```

* if `a` is either `2` or `10`, it will execute the "`some cool stuff`" code statements.

* "`conditional operator,`" often called the "`ternary operator.`" [...] a more concise form of a single `if..else`.
* more information [...] see the `Types & Grammar` title of this series.

## Strict Mode
* `ES5` added a `"strict mode"` [...] tightens the rules for certain behaviors [...] more `optimizable` `by the engine`.

* `strict mode` for an `individual function`, or an `entire file`, depending on where you put the `strict mode`.

```js
function foo() {
	"use strict";

	// this code is strict mode

	function bar() {
		// this code is strict mode
	}
}

// this code is not strict mode
```

```js
"use strict";

function foo() {
	// this code is strict mode

	function bar() {
		// this code is strict mode
	}
}

// this code is strict mode
```

* One key difference [...] with `strict mode` is disallowing [...] `omitting` the `var`.

```js
function foo() {
	"use strict";	// turn on strict mode
	a = 1;			// `var` missing, ReferenceError
}

foo();
```

* more information [...] see the `Chapter 5` of the `Types & Grammar` title of this series.

## Functions As Values
```js
function foo() {
	// ..
}
```

* `foo` is basically just a variable [...] that's given a `reference` to the `function` [...] itself is a `value`, just like `42` or `[1,2,3]` would be.

```js
var foo = function() {
	// ..
};

var x = function bar(){
	// ..
};

// (me: bar is not accessible)
x(); // ok
bar(); // error: ReferenceError: bar is not defined
```

* The first function [...] is called `anonymous` because it has `no name`.
* more information, see the `Scope & Closures` title of this series.

## Immediately Invoked Function Expressions (IIFEs)
* immediately invoked function expression (IIFE).


```js
function foo() { .. }

// `foo` function reference expression,
// then `()` executes it
foo();

// `IIFE` function expression,
// then `()` executes it
(function IIFE(){ .. })();

// (me: IIFE is not accessible)
IIFE; // error: ReferenceError: IIFE is not defined
```

* IIFEs can [...] return values:

```js
var x = (function IIFE(){
	return 42;
})();

x;	// 42
```

## Closure
* a way to "`remember`" and `continue to access` a function's `scope` (its `variables`) `even` once the function has `finished running`.

```js
function makeAdder(x) {
	// parameter `x` is an inner variable

	// inner function `add()` uses `x`, so
	// it has a "closure" over it
	function add(y) {
		return y + x;
	};

	return add;
}
```

* The reference to the inner `add(..)` function that gets returned with each call to the outer `makeAdder(..)` is able to remember whatever `x` value was passed in to `makeAdder(..)` [...] let's use `makeAdder(..)`:

```js
// `plusOne` gets a reference to the inner `add(..)`
// function with closure over the `x` parameter of
// the outer `makeAdder(..)`
var plusOne = makeAdder( 1 );

// `plusTen` gets a reference to the inner `add(..)`
// function with closure over the `x` parameter of
// the outer `makeAdder(..)`
var plusTen = makeAdder( 10 );

plusOne( 3 );		// 4  <-- 1 + 3
plusOne( 41 );		// 42 <-- 1 + 41

plusTen( 13 );		// 23 <-- 10 + 13
```

* `plusOne(3)`, it adds `3` (its inner `y`) to the `1` (`remembered by` `x`), and we get 4 as the result.

### Modules
* `most common usage` of closure in JavaScript is the `module pattern` [...] let you define `private` implementation details (variables, functions) [...] public API that is accessible from the outside.

```js
function User(){
	var username, password;

	function doLogin(user,pw) {
		username = user;
		password = pw;

		// do the rest of the login work
	}

	var publicAPI = {
		login: doLogin
	};

	return publicAPI;
}

// create a `User` module instance
var fred = User();

fred.login( "fred", "12Battery34!" );
```

* The inner `doLogin()` function `has a closure` over `username` and `password`, meaning it will retain its access to them `even after` the `User()` function `finishes running`.

* the outer `User()` function has `finished executing` [...] you'd think the inner variables like `username` and `password` have gone away [...] they have not, because there's `a closure` in the `login()` function keeping them alive.

* read the `Scope & Closures` title of this series for a much more in-depth exploration.

## `this` Identifier
* If a function has a `this` reference inside it, that `this` reference usually points to an `object`. But which `object` it points to depends on `how the function was called`.

* It's `important` to realize that `this` does `not refer to the function itself`.

```js
function foo() {
	console.log( this.bar );
}

var bar = "global";

var obj1 = {
	bar: "obj1",
	foo: foo
};

var obj2 = {
	bar: "obj2"
};

// --------

foo();				// "global"
obj1.foo();			// "obj1"
foo.call( obj2 );		// "obj2"
new foo();			// undefined
```

* `four rules` for how `this` gets set, and they're shown in those last four lines of that snippet.
  * `foo()` [...] setting `this` to the `global object` in `non-strict mode` -- in `strict mode`, this would be `undefined` and you'd get an `error` in accessing the `bar` property.
  * `obj1.foo()` sets `this` to the `obj1 object`.
  * `foo.call(obj2)` sets `this` to the `obj2` object.
  * `new foo()` sets `this` to a brand new `empty object`.

* more information about this, see `Chapters 1 and 2` of the `this & Object Prototypes` title of this series.

## Prototypes
