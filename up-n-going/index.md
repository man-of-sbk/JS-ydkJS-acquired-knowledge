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
