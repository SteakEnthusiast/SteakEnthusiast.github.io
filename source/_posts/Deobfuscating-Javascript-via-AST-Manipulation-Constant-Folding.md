---
title: "Deobfuscating Javascript via AST: Constant Folding/Binary Expression Simplification"
date: 2022-05-28 22:56:01
tags:
  - Javascript
  - Deobfuscation
  - Babel
---

# Preface

This article assumes a preliminary understanding of Abstract Syntex Tree structure and [BabelJS](https://babeljs.io/). [Click Here](http://SteakEnthusiast.github.io/none) to read my introductory article on the usage of Babel.

# What is Constant Folding?

**Constant Folding**: "An optimization technique that eliminates expressions that calculate a value that can already be determined before code execution." ([Source](https://cran.r-project.org/web/packages/rco/vignettes/opt-constant-folding.html))

To better explain constant folding, it's perhaps more useful to first introduce the obfuscation technique that constant folding fights against. Take the following code for example:

# Examples

## Example #1: The Basic Case

```javascript
/**
 * "Input.js"
 * Original, unobfuscated code.
 *
 */
let foo = 27;
let bar = 4;
let baz = "I am a string literal, totally whole!";
```

An obfuscator may split each constant value into multiple _binary expressions_, which could look something like this after obfuscation:

```javascript
/**
 * splitConstantsObfuscated.js"
 * This is the resulting code after obfuscation.
 *
 */
let foo = 12373561 ^ (12373561 * 13 + 3 * 7) ^ 153794264;
let bar =
  (535 + false) ^
  (2318 + true * -1399 + true) ^
  (1321 - 1234 / 2340 + true + true * 50 + false) ^
  1232;
let baz =
  "I " +
  "a" +
  "m" +
  " a" +
  " st" +
  "ri" +
  "ng" +
  " lite" +
  "ra" +
  "l," +
  " tot" +
  "al" +
  "l" +
  "y" +
  " wh" +
  "ol" +
  "e" +
  "!";
```

As you can see, the obfuscation has transformed what used to be easy to read constants; `27`, `4`, `"I am a string literal, totally whole!"` ; into multiple expressions with arithmetic and bitwise operators. The code is even using mathematical operators on booleans! Someone reading the code would likely need evaluate each expression in a debugger to figure out the value of each variable. Let's paste the second snippet in the dev tools console to check:

![Checking the evaluated values in the DevTools console](./constantFolding1.PNG)

We can observe that each variable in the second snippet has an equivalent ability to that of its first snippet counter part. The way the javascript engine simplified the expressions down to a constant is the essence of **_Constant Folding_**. Now, hypothetically, you _could_ just evaluate each expression in a javascript intepreter and replace it by hand manually. And sure, you could do that in just a few seconds for the snippet I provided. But that isn't a feasible a feasible solution if there were hundreds, or even thousands of lines of code in a program similar to this. Thankfully for us, Babel has an inbuilt feature that can help us automate the simplification.

### Analysis Methodology

Let's start by pasting the obfuscated sample into [AST Explorer](https://astexplorer.net/)

![View of the obfuscated code in AST Explorer](./constantFolding2.PNG)

If we click on one of the expression chunks on the right hand side of the assignment expressions, we can take a closer look at the AST structure:

![A closer look at one of the nodes of interest](./constantFolding3.PNG)

We can see that in this case, the small chunk of the string is of type _StringLiteral_, and it's contained inside a bunch of nested _BinaryExpression_ nodes. If we look at any other fraction of the other expressions, we can observe two important commonalities

1. A constant value, or _Literal_ (ex. _StringLiteral_,_NumericLiteral_, or _BooleanLiteral_)

2. The _Literal_ is contained inside a single or nested BinaryExpression(s).

Our final goal is to evaluate all the binary expressions to reduce each right-hand side expression to a constant _Literal_ value. Based on the nested nature of the _BinaryExpressions_, you might be thinking to manually write a recursive algorithm. However, there's a much simpler way to accomplish the same effect using Babel's inbuilt `path.evaluate()` function. Here's how we're going to use it:

1. Traverse through the AST to search for BinaryExpressions
2. If a BinaryExpression is encountered, try to evaluate it using `path.evaluate()`.
3. Check if it returns `confident:true`. If `confident` is `false`, skip the node by returning.
4. Create a node from the value using `t.valueToNode(value)` to infer the type, and assign it to a new variable, `valueNode`
5. Check that the resulting `valueNode` is a _Literal_ type. If the check returns `false` skip the node by returning.
   - This will cover _StringLiteral_, _NumericLiteral_, _BooleanLiteral_ etc. types and skip over others that would result from invalid operations (ex. `t.valueToNode(Infinity)` is of type _BinaryExpression_, `t.valueToNode(undefined)` is of type identifier)
6. Replace the BinaryExpression node with our newly created `valueNode'.
   The babel implementation is shown below:

### Babel Deobfuscation Script

```javascript
/**
 * Deobfuscator.js
 * The babel script used to deobfuscate the target file
 *
 */
const parser = require("@babel/parser");
const traverse = require("@babel/traverse").default;
const t = require("@babel/types");
const generate = require("@babel/generator").default;
const beautify = require("js-beautify");
const { readFileSync, writeFile } = require("fs");

/**
 * Main function to deobfuscate the code.
 * @param source The source code of the file to be deobfuscated
 *
 */
function deobfuscate(source) {
  //Parse AST of Source Code
  const ast = parser.parse(source);

  // Visitor for constant folding
  const foldConstantsVisitor = {
    BinaryExpression(path) {
      debugger;
      let { confident, value } = path.evaluate(); // Evaluate the binary expression
      if (!confident) return; // Skip if not confident
      let actualVal = t.valueToNode(value); // Create a new node, infer the type
      if (!t.isLiteral(actualVal)) return; // Skip if not a Literal type (ex. StringLiteral, NumericLiteral, Boolean Literal etc.)
      path.replaceWith(actualVal); // Replace the BinaryExpression with the simplified value
    },
  };

  // Execute the visitor
  traverse(ast, foldConstantsVisitor);

  // Code Beautification
  let deobfCode = generate(ast, { comments: false }).code;
  deobfCode = beautify(deobfCode, {
    indent_size: 2,
    space_in_empty_paren: true,
  });
  // Output the deobfuscated result
  writeCodeToFile(deobfCode);
}
/**
 * Writes the deobfuscated code to output.js
 * @param code The deobfuscated code
 */
function writeCodeToFile(code) {
  let outputPath = "output.js";
  writeFile(outputPath, code, (err) => {
    if (err) {
      console.log("Error writing f ile", err);
    } else {
      console.log(`Wrote file to ${outputPath}`);
    }
  });
}

deobfuscate(readFileSync("./splitConstantsObfuscated.js", "utf8"));
```

After processing the obfuscated script with the babel plugin above, we get the following result:

### Post-Deobfuscation Result

```javascript
let foo = 27;
let bar = 4;
let baz = "I am a string literal, totally whole!";
```

And the original code is completely restored!

## Example #2: A Confident Complication

If you've read my article on [_String Concealing_](http://SteakEnthusiast.github.io/2022/05/22/Deobfuscating-Javascript-via-AST-Manipulation-Various-String-Concealing-Techniques/), specifically the section on [_String Concatenation_](http://SteakEnthusiast.github.io/2022/05/22/Deobfuscating-Javascript-via-AST-Manipulation-Various-String-Concealing-Techniques/#Example-3-String-Concatenation), you may know that you can encounter a problem using the babel script above.

Let's say you have a code snippet like this:

```javascript
/**
 * Snippet 1
 */
class Person {
  constructor(name, school, emoji) {
    this.name = name;
    this.school = school;
    this.favAnimal = emoji;
  }

  sayHello() {
    let helloStatement =
      "Hello, my name is " +
      this.name +
      ". I g" +
      "o t" +
      "o " +
      this.school +
      " an" +
      "d " +
      "m" +
      "y" +
      " fa" +
      "vo" +
      "ur" +
      "ite" +
      " ani" +
      "ma" +
      "l" +
      " is" +
      " a " +
      this.favAnimal;
    console.log(helloStatement);
  }
}
```

By manual inspection, you can probably deduce that it can be reduced to this:

```javascript
/**
 * Snippet 1, manually deobfuscated
 */
class Person {
  constructor(name, school, animal) {
    this.name = name;
    this.school = school;
    this.favAnimal = animal;
  }
  sayHello() {
    let helloStatement =
      "Hello, my name is " +
      this.name +
      ". I go to " +
      this.school +
      " and my favourite animal is a " +
      this.favAnimal;
    console.log(helloStatement);
  }
}
```

However, if we try running running the deobfuscator we made above against the obfuscated snippet, it yields this result:

```javascript
/**
 * Snippet 1, processed through the deobfuscator
 */
class Person {
  constructor(name, school, emoji) {
    this.name = name;
    this.school = school;
    this.favAnimal = emoji;
  }

  sayHello() {
    let helloStatement =
      "Hello, my name is " +
      this.name +
      ". I g" +
      "o t" +
      "o " +
      this.school +
      " an" +
      "d " +
      "m" +
      "y" +
      " fa" +
      "vo" +
      "ur" +
      "ite" +
      " ani" +
      "ma" +
      "l" +
      " is" +
      " a " +
      this.favAnimal;
    console.log(helloStatement);
  }
}
```

It hasn't been simplified at all! But why?

### Where The Issue Lies

To figure out what the problem is, let's use a debugger and set breakpoints to try and understand what our deobfuscator is actually doing.

![Placing a debugger statement](constantFolding4.png)

We know that our visitor is acting on nodes of type _BinaryExpression_. A binary expression always has 3 main components: a left side, a right side, and an operator. For our example, the operator is always addition, `+`. On each iteration, let's run these commands in the debug console to check what our left and right side are.

- `generate(path.node).code`
- `generate(path.node.left).code`
- `generate(path.node.right).code`

Below is a screencap of what the first two iterations would look like:

![The first and second pause](constantFolding5.png)

When the visitor is first called, path.evaluate() will not return a value and the `confident` return value will be `false`. A `false` value for `confident` arises when the expression to be evaluated contains a variable whose value is currently unknown, and therefore Babel cannot be "confident" when attempting to compute an expression containing it. In the case of the first expression, the unknown variable (`this.favAnimal`) on the right side of the expression, and two unknown variables: (`this.name` & `this.school`) on the left side of the expression prevent path.evaluate() for returning a literal value. When the debugger statement is reached for a second time, the right hand side of the expression is a _StringLiteral_ (`"a"`). However, the left-hand side still contains variables with an unknown value. If we were to continue this for each time the breakpoint is encountered, the structure would look like this:

| Iteration | Left Side                                                                                                                                                              | Operator | Right Side     |
| --------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | -------------- |
| 1         | "Hello, my name is " + this.name + ". I g" + "o t" + "o " + this.school + " an" + "d " + "m" + "y" + " fa" + "vo" + "ur" + "ite" + " ani" + "ma" + "l" + " is" + " a " | +        | this.favAnimal |
| 2         | "Hello, my name is " + this.name + ". I g" + "o t" + "o " + this.school + " an" + "d " + "m" + "y" + " fa" + "vo" + "ur" + "ite" + " ani" + "ma" + "l" + " is"         | +        | " a"           |
| 3         | "Hello, my name is " + this.name + ". I g" + "o t" + "o " + this.school + " an" + "d " + "m" + "y" + " fa" + "vo" + "ur" + "ite" + " ani" + "ma" + "l"                 | +        | " is"          |
| 4         | "Hello, my name is " + this.name + ". I g" + "o t" + "o " + this.school + " an" + "d " + "m" + "y" + " fa" + "vo" + "ur" + "ite" + " ani" + "ma"                       | +        | "l"            |
| 5         | "Hello, my name is " + this.name + ". I g" + "o t" + "o " + this.school + " an" + "d " + "m" + "y" + " fa" + "vo" + "ur" + "ite" + " ani"                              | +        | "ma"           |
| 6         | "Hello, my name is " + this.name + ". I g" + "o t" + "o " + this.school + " an" + "d " + "m" + "y" + " fa" + "vo" + "ur" + "ite"                                       | +        | " ani"         |
| 7         | "Hello, my name is " + this.name + ". I g" + "o t" + "o " + this.school + " an" + "d " + "m" + "y" + " fa" + "vo" + "ur"                                               | +        | "ite"          |
| 8         | "Hello, my name is " + this.name + ". I g" + "o t" + "o " + this.school + " an" + "d " + "m" + "y" + " fa" + "vo"                                                      | +        | "ur"           |
| 9         | "Hello, my name is " + this.name + ". I g" + "o t" + "o " + this.school + " an" + "d " + "m" + "y" + " fa"                                                             | +        | "vo"           |
| 10        | "Hello, my name is " + this.name + ". I g" + "o t" + "o " + this.school + " an" + "d " + "m" + "y"                                                                     | +        | " fa"          |
| 11        | "Hello, my name is " + this.name + ". I g" + "o t" + "o " + this.school + " an" + "d " + "m"                                                                           | +        | "y"            |
| 12        | "Hello, my name is " + this.name + ". I g" + "o t" + "o " + this.school + " an" + "d "                                                                                 | +        | "m"            |
| 13        | "Hello, my name is " + this.name + ". I g" + "o t" + "o " + this.school + " an"                                                                                        | +        | "d "           |
| 14        | "Hello, my name is " + this.name + ". I g" + "o t" + "o " + this.school                                                                                                | +        | " an"          |
| 14        | "Hello, my name is " + this.name + ". I g" + "o t" + "o "                                                                                                              | +        | this.school    |
| 14        | "Hello, my name is " + this.name + ". I g" + "o t"                                                                                                                     | +        | "o "           |
| 14        | "Hello, my name is " + this.name + ". I g"                                                                                                                             | +        | "o t"          |
| 14        | "Hello, my name is " + this.name                                                                                                                                       | +        | ". I g"        |
| 14        | "Hello, my name is "                                                                                                                                                   | +        | this.name      |

It's evident that at every encounter, one of the sides will _always_ contain a variable of unknown value. Therefore, path.evaluate() will return `confident: false` be useless in simplfying the expression. So, we'll need to try something else.

### Constructing the Solution

#### Idea #1: Prioritizing Chunks of Consecutive Literals

We know that the issue lies with the one of the sides containing a variable. However, we can see that there are chunks of the code that contain _**consecutive string literals only**_:

```javascript
//Chunk 1
"Hello, my name is ";
//Chunk 2
". I g" + "o t" + "o ";

// Chunk 3
" an" +
  "d " +
  "m" +
  "y" +
  " fa" +
  "vo" +
  "ur" +
  "ite" +
  " ani" +
  "ma" +
  "l" +
  " is" +
  " a ";
```

If there were some way to prioritize these smaller chunks, then surely path.evaluate() would be able to simplify them. This is indeed the case, as we can prove this by manually wrapping each of these chunks in parentheses to force them to be evaluated first:

```javascript
/**
 * Snippet 2: wrapping consecutive strings in parentheses
 */
class Person {
  constructor(name, school, emoji) {
    this.name = name;
    this.school = school;
    this.favAnimal = emoji;
  }

  sayHello() {
    let helloStatement =
      "Hello, my name is " +
      this.name +
      (". I g" + "o t" + "o ") +
      this.school +
      (" an" +
        "d " +
        "m" +
        "y" +
        " fa" +
        "vo" +
        "ur" +
        "ite" +
        " ani" +
        "ma" +
        "l" +
        " is" +
        " a ") +
      this.favAnimal;
    console.log(helloStatement);
  }
}
```

Running this through the deobfuscator, we get our desired result:

```javascript
/**
 * Snippet 2, processed through the deobfuscator.
 */
class Person {
  constructor(name, school, emoji) {
    this.name = name;
    this.school = school;
    this.favAnimal = emoji;
  }

  sayHello() {
    let helloStatement =
      "Hello, my name is " +
      this.name +
      ". I go to " +
      this.school +
      " and my favourite animal is a " +
      this.favAnimal;
    console.log(helloStatement);
  }
}
```

Alright, so that did the job. But for very long binary expressions which you might encounter in wild obfuscated scripts, you certainly _do not_ want to have to spend time manually wrapping chunks of consecutive strings in parentheses. Sure, you could probably automate it with Regex, or write an AST-based algorithm to add brackets to the source string, but there has to be a less complicated way, right?

The answer: Yes, there is. And we are on the right track.

#### Idea #2: Even _Smaller_ Chunks

Okay, so we know that trying to pinpoint and prioritize chunks of consecutive strings with varying lengths can be troublesome. But what if we just split the binary expression into the smallest possible pieces and prioritized those?

I'll admit, that probably sounds confusing. So I'll do my best to explain what I mean with an example.

We know that a binary expression, in it's simplest form, consists of a left side, an operator, and a right side. Let's refer back to the first three rows of the table we made:

| Iteration | Left Side                                                                                                                                                              | Operator | Right Side     |
| --------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | -------------- |
| 1         | "Hello, my name is " + this.name + ". I g" + "o t" + "o " + this.school + " an" + "d " + "m" + "y" + " fa" + "vo" + "ur" + "ite" + " ani" + "ma" + "l" + " is" + " a " | +        | this.favAnimal |
| 2         | "Hello, my name is " + this.name + ". I g" + "o t" + "o " + this.school + " an" + "d " + "m" + "y" + " fa" + "vo" + "ur" + "ite" + " ani" + "ma" + "l" + " is"         | +        | " a"           |
| 3         | "Hello, my name is " + this.name + ". I g" + "o t" + "o " + this.school + " an" + "d " + "m" + "y" + " fa" + "vo" + "ur" + "ite" + " ani" + "ma" + "l"                 | +        | " is"          |

The right side of our binary expression is always only one element long, and is either a literal value or an identifier. However, the left side isn't a single element long. Rather, it's also a binary expression, containing both literal values and identifiers. What I propose is developing an algorithm to ensure that both the left side and right side are only a single element long. Then, if both are string literals, we can concat them. If not, we can simply move on.

To do this, lets look at the above table again, but only the first two rows. However, for the left side, let's only take into consideration the **_right-most edge_** of the expression. That would look something like this:

| Iteration | Right Edge of Left Side                                                                                                                                                        | Operator | Right Side         |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------- | ------------------ |
| 1         | ~~"Hello, my name is " + this.name + ". I g" + "o t" + "o " + this.school + " an" + "d " + "m" + "y" + " fa" + "vo" + "ur" + "ite" + " ani" + "ma" + "l" + " is" +~~ **" a "** | +        | **this.favAnimal** |
| 2         | ~~"Hello, my name is " + this.name + ". I g" + "o t" + "o " + this.school + " an" + "d " + "m" + "y" + " fa" + "vo" + "ur" + "ite" + " ani" + "ma" + "l" +~~ **" is"**         | +        | **" a"**           |

If we were to evaluate these now, we would observe the following:

- Row 1 still cannot be evaluated, since it is a `StringLiteral` + `a variable of unknown value`.
- Row 2 can be simplified into a single string literal: `" is" + " a"` => `" is a"`

We can then replace the _right-most edge of the left side_ with the simplified result, so it will become the right side in the next iteration. This way, we can simplify consecutive concatenation of strings step by step. Keep in mind, each simplification will affect the next iteration's right side. The reason I included the first few rows and not the third, is because the simplification in the second iteration would change the right side of the third iteration, so it would no longer look like the original table.

Following the new algorithm, each iteration of our visitor would look like this:

| Iteration | Right Edge of Left Side | Operator | Right Side                      |
| --------- | ----------------------- | -------- | ------------------------------- |
| 1         | " a "                   | +        | this.favAnimal                  |
| 2         | " is"                   | +        | " a"                            |
| 3         | "l"                     | +        | " is a"                         |
| 4         | "ma"                    | +        | "l is a"                        |
| 5         | " ani"                  | +        | "mal is a"                      |
| 6         | "ite"                   | +        | " animal is a"                  |
| 7         | "ur"                    | +        | "ite animal is a"               |
| 8         | "vo"                    | +        | "urite animal is a"             |
| 9         | " fa"                   | +        | "vourite animal is a"           |
| 10        | "y"                     | +        | " favourite animal is a"        |
| 11        | "m"                     | +        | "y favourite animal is a"       |
| 12        | "d "                    | +        | "my favourite animal is a"      |
| 13        | " an"                   | +        | "d my favourite animal is a"    |
| 14        | this.school             | +        | " and my favourite animal is a" |
| 14        | "o "                    | +        | this.school                     |
| 14        | "o t"                   | +        | "o "                            |
| 14        | ". I g"                 | +        | "o to "                         |
| 14        | this.name               | +        | ". I go to "                    |
| 14        | "Hello, my name is "    | +        | this.name                       |

So, by following this algorithm, we should be able to combine all consecutive string literals into one.

<span style="font-size:35px;bold"><b>WAIT! There's something wrong here...</b></span>

The line of code we start with is `let helloStatement = "Hello, my name is " + this.name + ". I g" + "o t" + "o " + this.school + " an" + "d " + "m" + "y" + " fa" + "vo" + "ur" + "ite" + " ani" + "ma" + "l" + " is" + " a " + this.favAnimal;`

We know that on the second iteration, `" is"` and `" a"` will be concatenated into a single string literal, then the right most edge of the left side will be replaced with the resulting value. That is,

`" is"` => `" is a"`

The problem here is that we are adding the right side of the expression to the right edge of the left side of the expression. However, the original right side remains unchanged despite already being accounted for. The code after one iteration would then look like this:

`let helloStatement = "Hello, my name is " + this.name + ". I g" + "o t" + "o " + this.school + " an" + "d " + "m" + "y" + " fa" + "vo" + "ur" + "ite" + " ani" + "ma" + "l" + " is a " + " a " + this.favAnimal;`

Notice the extra duplicate near the end, `" is a " + " a "`. To fix this, we need to ensure that we **delete the original right side of the expression after doing the concatenation and replacement**.

So, based on this logic, the correct steps for creating the deobfuscator are as follows:

### Writing The Deobfuscator Logic

1. Traverse the ast in search of BinaryExpressions. When one is encountered:

   1. If both the right side (`path.node.right`) and the left side (`path.node.left`) are of type _StringLiteral_, we can use the algorithm for the basic case.
   2. If not:
      1. Check if the right side, (`path.node.right`) is a _StringLiteral_. If it isn't, skip this node by returning.
      2. Check if the _right-most edge of the left-side_ (`path.node.left.right`) is a _StringLiteral_. If it isn't, skip this node by returning.
      3. Check if the operator is addition (`+`). If it isn't, skip this node by returning.
      4. Evaluate the _right-most edge of the left-side_ + the right side; `path.node.left.right.value + path.node.right.value` and assign it's StringLiteral representation to a variable, `concatResult`.
      5. Replace the _right-most edge of the left-side_ with `concatResult`.
      6. Remove the original right side of the expression as it is now a duplicate.

The Babel implementation is as follows:

```javascript
/**
 * Deobfuscator.js
 * The babel script used to deobfuscate the target file
 *
 */
const parser = require("@babel/parser");
const traverse = require("@babel/traverse").default;
const t = require("@babel/types");
const generate = require("@babel/generator").default;
const beautify = require("js-beautify");
const { readFileSync, writeFile } = require("fs");

/**
 * Main function to deobfuscate the code.
 * @param source The source code of the file to be deobfuscated
 *
 */
function deobfuscate(source) {
  //Parse AST of Source Code
  const ast = parser.parse(source);

  // Visitor for constant folding
  const foldConstantsVisitor = {
    BinaryExpression(path) {
      const left = path.get("left");
      const right = path.get("right");
      const operator = path.get("operator").node;

      if (t.isStringLiteral(left.node) && t.isStringLiteral(right.node)) {
        // In this case, we can use the old algorithm
        // Evaluate the binary expression
        let { confident, value } = path.evaluate();
        // Skip if not confident
        if (!confident) return;
        // Create a new node, infer the type
        let actualVal = t.valueToNode(value);
        // Skip if not a Literal type (ex. StringLiteral, NumericLiteral, Boolean Literal etc.)
        if (!t.isStringLiteral(actualVal)) return;
        // Replace the BinaryExpression with the simplified value
        path.replaceWith(actualVal);
      } else {
        // Check if the right side is a StringLiteral. If it isn't, skip this node by returning.
        if (!t.isStringLiteral(right.node)) return;
        //Check if the right sideis a StringLiteral. If it isn't, skip this node by returning.
        if (!t.isStringLiteral(left.node.right)) return;
        // Check if the operator is addition (+). If it isn't, skip this node by returning.
        if (operator !== "+") return;

        // If all conditions are fine:

        // Evaluate the _right-most edge of the left-side_ + the right side;
        let concatResult = t.StringLiteral(
          left.node.right.value + right.node.value
        );
        // Replace the _right-most edge of the left-side_ with `concatResult`.
        left.get("right").replaceWith(concatResult);
        //Remove the original right side of the expression as it is now a duplicate.
        right.remove();
      }
    },
  };

  // Execute the visitor
  traverse(ast, foldConstantsVisitor);

  // Code Beautification
  let deobfCode = generate(ast, { comments: false }).code;
  deobfCode = beautify(deobfCode, {
    indent_size: 2,
    space_in_empty_paren: true,
  });
  // Output the deobfuscated result
  writeCodeToFile(deobfCode);
}
/**
 * Writes the deobfuscated code to output.js
 * @param code The deobfuscated code
 */
function writeCodeToFile(code) {
  let outputPath = "output.js";
  writeFile(outputPath, code, (err) => {
    if (err) {
      console.log("Error writing file", err);
    } else {
      console.log(`Wrote file to ${outputPath}`);
    }
  });
}

deobfuscate(readFileSync("./splitStringsObfuscated.js", "utf8"));
```

After processing the obfuscated script with the babel plugin above, we get the following result:

### Post-Deobfuscation Result

```javascript
class Person {
  constructor(name, school, emoji) {
    this.name = name;
    this.school = school;
    this.favAnimal = emoji;
  }

  sayHello() {
    let helloStatement =
      "Hello, my name is " +
      this.name +
      ". I go to " +
      this.school +
      " and my favourite animal is a " +
      this.favAnimal;
    console.log(helloStatement);
  }
}
```

And all consecutive StringLiterals have been concatenated! Huzzah!

# Conclusion

Okay, I hope that second example wasn't too confusing. Sometimes, you'll be able to avoid the problem with unknown variable values by replacing references to a constant variable with their actual value. If you're interested in that, you can read my article about it [here](http://SteakEnthusiast.github.io/2022/05/31/Deobfuscating-Javascript-via-AST-Replacing-References-to-Constant-Variables-with-Their-Actual-Value/). In this case, however, it was unavoidable due to being within a class definition where variables have yet to be initialized.

Keep in mind that the second example will only work for _String Literals_ and _addition_. But, it can easily be adapted to other node types and operators. I'll leave that as a challenge for you, dear reader, if you wish to pursue it further 😉

If you're interested, you can find the source code for all the examples in [this repository](https://github.com/SteakEnthusiast/Supplementary-AST-Based-Deobfuscation-Materials).

Anyways, that's all I have for you today. I hope that this article helped you learn something new. Thanks for reading, and happy reversing!
