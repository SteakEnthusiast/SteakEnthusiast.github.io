---
title: "Deobfuscating Javascript via AST: A Peculiar JSFuck-esque Case"
date: 2022-06-14 12:46:47
tags:
  - Javascript
  - Deobfuscation
  - Babel
  - Reverse Engineering
categories:
  - Deobfuscation
---

# Preface

This article assumes a preliminary understanding of Abstract Syntax Tree structure and [BabelJS](https://babeljs.io/). [Click Here](http://SteakEnthusiast.github.io/2022/05/21/Deobfuscating-Javascript-via-AST-An-Introduction-to-Babel/) to read my introductory article on the usage of Babel.

It also assumes that you've read my article about constant folding. If you haven't already read it, you can do so by clicking [here](https://steakenthusiast.github.io/2022/05/28/Deobfuscating-Javascript-via-AST-Manipulation-Constant-Folding/)

# Introduction

[JSFuck](http://www.jsfuck.com/) is an esoteric and educational programming style based on the atomic parts of JavaScript. It uses only six different characters to write and execute code. I won't be covering the intricacies of how JSFuck operates, so please refer to the official site if you'd like to learn more about it.

# Example 1: A Simple JSFuck Case

Code obfuscated in JSFuck style tends to look like this:

```js
// JSFUckObfuscated.js
+~!+!+!~+!!(
  !+(
    !+[] +
    !![] +
    !![] +
    !![] +
    !![] +
    !![] +
    !![] +
    !![] +
    [] +
    (!+[] + !![] + !![]) +
    (!+[] + !![] + !![] + !![] + !![] + !![] + !![] + !![]) +
    (!+-[] + +-!![] + -[]) +
    (!+[] + !![] + !![] + !![]) +
    -~~~[] +
    (!+[] + !![] + !![] + !![] + !![] + !![]) +
    (!+[] + !![] + !![] + !![]) +
    (!+[] + !![] + !![] + !![] + !![] + !![] + !![])
  ) /
  +(
    !+[] +
    !![] +
    !![] +
    !![] +
    !![] +
    !![] +
    !![] +
    !![] +
    [] +
    (!+[] + !![] + !![] + !![] + !![] + !![] + !![] + !![]) +
    (!+[] + !![] + !![] + !![] + !![] + !![] + !![] + !![]) +
    (!+[] + !![] - []) +
    (!+[] + !![] + !![]) +
    (!+-[] + +-!![] + -[]) +
    (!+[] + !![] + !![] + !![] + !![] + !![] + !![] + !![]) +
    (!+[] + !![] + !![] + !![] + !![]) +
    (!+[] + !![] + !![] + !![] + !![] + !![] + !![])
  )
);
```

Let's try evaluating this code in the console:

![Result of evaluating the code in the console](jsfuck1.PNG)

We can see that it leads to a constant: `-1`. Our goal is to simplify code looking like this down to a constant number.

Before anything, let's try reusing the [code from my constant folding article](https://steakenthusiast.github.io/2022/05/28/Deobfuscating-Javascript-via-AST-Manipulation-Constant-Folding/#Babel-Deobfuscation-Script).

## Trying the Original Deobfuscator

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
      let { confident, value } = path.evaluate(); // Evaluate the binary expression
      if (!confident) return; // Skip if not confident
      let actualVal = t.valueToNode(value); // Create a new node, infer the type
      if (!t.isLiteral(actualVal)) return; // Skip if not a Literal type (e.g. StringLiteral, NumericLiteral, Boolean Literal etc.)
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
      console.log("Error writing file", err);
    } else {
      console.log(`Wrote file to ${outputPath}`);
    }
  });
}

deobfuscate(readFileSync("./JSFuckObfuscated.js", "utf8"));
```

After processing the obfuscated script with the babel plugin above, we get the following result:

#### Post-Deobfuscation Result

```javascript
+~!+!+!~+!!0;
```

That's a lot of simplification! We could now just deduce from manual inspection that the result would be equal to `-1`. But, we'd prefer our debugger to do all the work for us. Time to analyze our code and make some changes!

## Analysis Methodology

As always, we start our analysis using [AST Explorer](https://astexplorer.net/).

But first, let's make a small simplification to the analysis process. Normally, we would paste the entire obfuscated script into AST explorer. However, we already know that our original constant folding visitor can do the majority of the cases for us. So, instead of analyzing the entire original script, we can shift our focus to what our deobfuscator _is not doing_. Therefore, we'll only analyze the resulting code of our deobfuscator to figure out what we need to add.

That means we only need to analyze this one-liner: `+~!+!+!~+!!0;`. Let's paste that into AST Explorer and see what we get.

![View of the obfuscated code in AST Explorer](jsfuck2.PNG)

Even though this code contains `+` operators, there are no `BinaryExpression`s present. In this case, the `+`'s are [Unary Operators](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Unary_plus). In fact, this code only contains nodes of type `UnaryExpression`, which then act on a single `NumericLiteral` node.

So, you may have realized by now why our deobfuscator doesn't fully work. Our deobfuscator is only accounting for `BinaryExpressions`, and we have yet to add functionality to handle `UnaryExpressions`! So, let's do that.

## Writing the Deobfuscator Logic

Thankfully for us, the `path.evaluate()` method can also be used for UnaryExpressions. So, we should also create a visitor for nodes of type `UnaryExpression`, and run the same transformation for them.

If you're still new to Babel, your first instinct might be to create two separate visitors: one for `UnaryExpression`s, and one for `BinaryExpression`s; then copy-paste the original plugin code inside of both. However, there is a much cleaner way of accomplishing the same thing. Babel allows you to run the same function for multiple visitor nodes by separating them with a `|` in the method name as a string. In our case, that would look like: `"BinaryExpression|UnaryExpression"(path)`.

In essence, all we need to do is change `BinaryExpression(path)` to `"BinaryExpression|UnaryExpression"(path)` in our deobfuscator. This will _mostly_ work, but I want to explain some interesting findings regarding evaluation of UnaryExpressions.

### The Problem

#### Problems with UnaryExpression Evaluation

`path.evaluate()`, `UnaryExpression`s, and `t.valueToNode()` don't work very well with each other due to their source code implementation. I'll explain with a short example:

Let's say we have the following code:

```js
~false;
```

and we want to simplify it to:

```js
-1;
```

If we use the original code from the constant folding article and only replace the method name, we'll have this visitor:

```js
// ...
const foldConstantsVisitor = {
  "BinaryExpression|UnaryExpression"(path) {
    let { confident, value } = path.evaluate(); // Evaluate the binary expression
    if (!confident) return; // Skip if not confident
    let actualVal = t.valueToNode(value); // Create a new node, infer the type
    if (!t.isLiteral(actualVal)) return; // Skip if not a Literal type (e.g. StringLiteral, NumericLiteral, Boolean Literal etc.)
    path.replaceWith(actualVal); // Replace the BinaryExpression with the simplified value
  },
};

// Execute the visitor
traverse(ast, foldConstantsVisitor);

// ...
```

But, if we run this, we'll see that it returns:

```js
~false;
```

which isn't simplified at all.

Here's why:

1. `t.valueToNode()`'s implementation. Running `path.evaluate()` correctly returns an integer value, `-1`. However, t.valueToNode(-1) doesn't create a `NumericLiteral` node with a value of `-1` as we would expect. Instead, it creates another `UnaryExpression` node, with properties `operator: -` and `argument: 1`. As such, `if (!t.isLiteral(actualVal)) return` results in an early return before replacement.

2. Even if we delete `if (!t.isLiteral(actualVal)) return` from our code, there's still an issue. Since `t.valueToNode(-1)` constructs a `UnaryExpression`, we are checking UnaryExpressions, and we have no additional checks, our program will result in an infinite recursive loop, crashing our program once the maximum call stack size is exceeded:

![Maximum Call Stack Size Exceeded Error](jsfuck3.png)

3.  Though not directly applicable to this code snippet, another problem is worth mentioning. Unary expressions can also have a `void` operator. Based on Babel's source code, calling `path.evaluate()` on any `UnaryExpression` with a `void` operator will simplify it to `undefined`, _regardless of what the argument is_.

![Babel Source Code Snippet: \node_modules@babel\traverse\lib\path\evaluation.js](jsfuck4.png)

This can be problematic in some cases, such as this example:

- Snippet 1:

```js
var a = 1;
function set() {
  a = 2;
}
void set();
console.log(a); // => 2
```

Calling `path.evaluate()` to simplify the `void set()` `UnaryExpression` yields this:

- Snippet 2:

```js
var a = 1;
function set() {
  a = 2;
}
undefined;

console.log(a); // => 1
```

The two pieces of code above are clearly not the same, as you can verify with their output.

### The Fix

Thankfully, these three conditions are simple to account for. We can solve each of them as follows:

1. Delete the `if (!t.isLiteral(actualVal)) return` check.
2. Add a check at the beginning of the visitor method to skip the node if it is a `UnaryExpression` with a `-` operator.
3. Add a check at the beginning of the visitor method to skip the node if it is a `UnaryExpression` with a `void` operator.

I've also neglected to mention this before, but when using `path.evaluate()`, it's best practice to also skip the replacement of nodes when it evaluates `Infinity` or `-Infinity` by returning early. This is because `t.valueToNode(Infinity)` creates a node of type BinaryExpression, which looks like `1 / 0`. Similarly, `t.valueToNode(-Infinity)` creates a node of type UnaryExpression, which looks like `-(1/0)`. In both of these cases, it can cause an infinite loop since our visitor will also visit the created nodes, which will crash our deobfuscator.

### Summarizing the Logic

So putting that all together, we have the following logic for our deobfuscator:

1. Traverse the ast for nodes of type `BinaryExpression` and `UnaryExpression`.

2. Upon encountering one:
3. Check if it is of type `UnaryExpression` and uses a `void` or `-` operator. If the condition is true, skip the node by returning.
4. Evaluate the node using `path.evaluate()`.
5. If `path.evaluate()` returns `{confident:false}`, or `{value:Infinity}` or `{value:-Infinity}`, skip the node by returning.
6. Construct a new node from the returned `value`, and replace the original node with it.

The Babel implementation looks like this:

## Babel Deobfuscation Script

```js
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
  const constantFold = {
    "BinaryExpression|UnaryExpression"(path) {
      const { node } = path;
      if (
        t.isUnaryExpression(node) &&
        (node.operator == "-" || node.operator == "void")
      )
        return;
      let { confident, value } = path.evaluate(); // Evaluate the binary expression
      if (!confident || value == Infinity || value == -Infinity) return; // Skip if not confident

      let actualVal = t.valueToNode(value); // Create a new node, infer the type
      path.replaceWith(actualVal); // Replace the BinaryExpression with the simplified value
    },
  };

  //Execute the visitor
  traverse(ast, constantFold);
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

deobfuscate(readFileSync("./jsFuckObfuscated.js", "utf8"));
```

After processing the obfuscated script with the babel plugin above, we get the following result:

## Post-Deobfuscation Result

```js
-1;
```

And we finally arrive at the correct constant value!

# Example 2: A More Peculiar Case

That first example was just a warm-up, and not what I really wanted to focus on (hence the title of the article). This next example isn't too much more difficult, but it _will_ require you to think a bit outside of the box.

Here's our obfuscated sample:

```js
let obbedVar = +([[[[[[]], , ,]]]] != 0);
```

Let's try running this in a javascript console to see what it simplifies to:

![Result of evaluating the code in the console](jsfuck5.png)

So, it simplifies to a numeric literal, `1`!

The structure of the obfuscated sample looks similar to that of the first example. We know that it leads to a constant, so let's first try running this sample through the improved deobfuscator we created above.

If you do that, you'll see that it yields:

```js
let obbedVar = +([[[[[[]], , ,]]]] != 0);
```

Which is no different! So, why is our code breaking?

## Analysis Methodology

### The Problem

Intuitively, you can probably guess what's causing the issue. The only real difference is that there seems to be an array containing blank elements: `[, , ,]`.

But why would that even matter? Let's paste our code into [AST Explorer](https://astexplorer.net/) to try and figure out what's going on.

![View of the obfuscated code in AST Explorer](jsfuck6.PNG)

We know that everything else seems normal except for the array containing empty elements, so let's focus on that. We can highlight the empty elements in the code using our cursor to automatically show their respective nodes on the right-hand side.

![A closer look at the nodes of interest](jsfuck7.png)

You'll notice something strange! There are elements of the array that are `null`. Keep in mind, in Babel, the node types `Literal` and `Identifier` are used to represent `null` and `undefined` respectively (as shown below):

![Handling of null, undefined, and empty elements](jsfuck8.png)

But, in this case, we don't even have a node! It's just simply `null`.

Let's look inside of Babel's source code implementation for `path.evaluate()` to see why the script breaks when encountering this. You can view the original script from the [official GitHub repository](https://github.com/babel/babel/blob/main/packages/babel-traverse/src/path/evaluation.ts), or by navigating to `\node_modules\@babel\traverse\lib\path\evaluation.js`.

![snippet from \node_modules@babel\traverse\lib\path\evaluation.js](jsfuck9.PNG)

The above code snippet can be found in the `_evaluate()` function, which runs as a helper for the main `evaluate()` function.

We can see that when `path.evaluate()` is called on an array expression, it tries to recursively evaluate all of its inner elements.However, if the evaluation fails/returns `confident: false` for any element in the array, the entire evaluation short circuits.

But, Babel actually has an implementation to handle occurrences of `undefined` and `null` in source code:

![Handling of undefined](jsfuck10.png)

![Handling of null](jsfuck11.PNG)

However, that's only after they're converted to either a node of type `NullLiteral` or `Identifier`. In evaluation.js, there isn't any handling for when a `null` value is encountered, so the method will return `confident: false` whenever an empty array element is encountered.

### The Fix

We shouldn't give up though, since we _KNOW_ that it's possible to evaluate the code to a constant because we tested it in a console before. Let's use a console again, this time to see what an empty element in an array is actually equal to:

![Inspecting the value of an empty element](jsfuck12.PNG)

We can see that trying to access an empty element in an array returns `undefined`! Okay, but how does that help us?

Recall that Babel has an implementation for handling `undefined` in evaluation.js. However, the reason it didn't work was because Babel failed to convert the empty array elements to a node. To fix our problem, all we have to do is replace any empty elements in arrays with `undefined` beforehand, so Babel can recognize them as the `undefined` keyword and evaluate them properly!

## Writing the Deobfuscator Logic

The deobfuscator logic is as follows:

1. Traverse the ast for `ArrayExpression`s. When one is encountered:
2. Check if the element is falsy. A node representation of `undefined` or `null` still will not be falsy, since a node is an object.
3. If it's falsy, replace it with the node representation of `undefined`.
4. Run our constant folding plugin from Example 1.
   The babel deobfuscation code is shown below:

## Babel Deobfuscation Script

```js
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

  const fixArrays = {
    ArrayExpression(path) {
      for (elem of path.get("elements")) {
        if (!elem.node) {
          elem.replaceWith(t.valueToNode(undefined));
        }
      }
    },
  };

  traverse(ast, fixArrays);

  // Visitor for constant folding
  const constantFold = {
    "BinaryExpression|UnaryExpression"(path) {
      const { node } = path;
      if (
        t.isUnaryExpression(node) &&
        (node.operator == "-" || node.operator == "void")
      )
        return;
      let { confident, value } = path.evaluate(); // Evaluate the binary expression
      if (!confident || value == Infinity || value == -Infinity) return; // Skip if not confident

      path.replaceWith(t.valueToNode(value)); // Replace the BinaryExpression with a new node of inferred type
    },
  };

  //Execute the visitor
  traverse(ast, constantFold);
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

deobfuscate(readFileSync("./jsFuckObfuscated.js", "utf8"));
```

After processing the obfuscated script with the babel plugin above, we get the following result:

## Post-Deobfuscation Result

```js
let obbedVar = 1;
```

And we've successfully simplified it down to a constant!

# Conclusion
I will admit that this article may have been a bit longer than it needed to be. However, I felt that for my beginner-level readers, it would be more helpful to explain the entire reverse-engineering thought process; including where and why some things go wrong, and the logical process of constructing a solution.

Okay, that's all I have to cover for today. If you're interested, you can find the source code for all the examples in [this repository](https://github.com/SteakEnthusiast/Supplementary-AST-Based-Deobfuscation-Materials).

I hope this article helped you learn something new. Thanks for reading, and happy reversing! 😄

