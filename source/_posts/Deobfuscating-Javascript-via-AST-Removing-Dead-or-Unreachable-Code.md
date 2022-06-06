---
title: "Deobfuscating Javascript via AST: Removing Dead or Unreachable Code"
date: 2022-06-04 16:20:34
tags:
  - Javascript
  - Deobfuscation
  - Babel
---

# Preface

This article assumes a preliminary understanding of Abstract Syntex Tree structure and [BabelJS](https://babeljs.io/). [Click Here](http://SteakEnthusiast.github.io/none) to read my introductory article on the usage of Babel.

# Definitions

Both _dead code_ and _unreachable code_ are obfuscation techniques relying on the injection of junk code that does not alter the main functionality of a program. Their only purpose is to bloat the appearence of the appearance of the source code to make it more confusing for a human to analyze. Though being similar, there's a slight difference between the two.

## What is Dead Code?

**_Dead code_** is a section of code that is executed, but its results are never used in the rest of the program. In addition to increasing the file size, dead code also increases the program runtime and CPU usage since it is being executed.

## What is unreachable code?

**_Unreachable code_** is a section of code that is never executed, since there is no existing control flow path that leads to it's execution. This results in an increase in file size, but shouldn't affect the runtime of the program since it's contents are never executed.

# Examples

## Example 1: [Dead Code] Unused Variables and Functions

An unused variable/function is something that is declared (and often, but not always, initialized), but is never used in the program. Therefore, for a variable to be unused, it must:

1. Be constant, and
2. Have no references

The following obfuscated script contains many of them:

```javascript
/**
 * unreferencedVariablesObfuscated.js
 * Lots of useless variables!
 */

var a = 3;
var b;
function useless1(yeet) {
  console.log(yeet);
  console.log("I'm useless :(");
}
const c = 7;
let d = 293;
const u = "My favourite";
var useless2 = function (bruh) {
  console.log(bruh.useless, ":(");
};

function notHello() {
  console.log("No!");
  console.log(i + lmaod, u, el, dgajd, fg + lmaod);
}
let dbgad = 23172;
let dgajd = "is";
var i = "Hello";
let vnajkfdhg;
var dnakd;
let bvs = new Date();
var h = "yeet";
let bv = 23;
var fg = "steak";
var lmaod = "!";
const el = "food";
let n = 1363;
let vch = "dghda" + 2;
let lol = performance.now();
const sayHello = function () {
  console.log(i + lmaod, u, el, dgajd, fg + lmaod);
};
let dga = 3653817;
let sfa = "362813";
sayHello();
let lmao;
var fldfioan;
```

### Analysis Methodology

Let's begin our analysis by pasting the obfuscated script into [AST Explorer](https://astexplorer.net)

![A view of the obfuscated code in AST Explorer](unusedVars1.PNG)

We can observe from the AST structure that each new variable creations results in the creation of one of two types of nodes:

1. A _VariableDeclaration_, for variables assigned with `let`, `var`, and `const`. 2. Each of these _VariableDeclarations_ contain an array of _VariableDeclarators_. It is the _VariableDeclarator_ that actually contains the information of the variables, including its `id` and `init` value. So, we can make _VariableDeclarator_ nodes our focus of interest to avoid unnecessary extra traversals.
2. A _FunctionDeclaration_, for functions declared with a [function statement](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function#specifications).

Based on this, we can deem our target node types of interest to be _VariableDeclarator_ and _FunctionDeclaration_.

Recall that we want to identify all **_constant_** variables and **non-referenced** variables, then remove them. It's important to note that variables in different scopes (ex. local vs. global), may share the same name but have different values. So, we cannot simply base our solution off of how many times a variable name occurs in a program.

This would be a convoluted process if not for Babel's 'Scope' API. I won't dive too deep into the available scope API's, but you can refer to the [Babel Plugin Handbook](https://github.com/jamiebuilds/babel-handbook/blob/master/translations/en/plugin-handbook.md#toc-scopes) to learn more about them. In our case, the `scope.getBinding(${identifierName})` method will be incredibly useful for us, as it directly returns information regarding if a variable is constant and all of its references (or lack thereof).

We can use our observations to construct the following deobfuscation logic:

1. Traverse the AST for _VariableDeclarators_ and _FunctionDeclarations_. Since both node types have both have the `id` property, we can write a single plugin for both.
   - **Tip**: _To write a function that works for multiple visitor nodes, we can add an_ `|` _seperating them in the method name as a string like this:_ `"VariablDeclarator|FunctionDeclaration"`
2. Use the `path.scope.getBinding(${identifierName})` method with the name of the current variable as the argument.
3. If the method returns `constant` as `true` and `referenced` as `false`, the declaration is considered to be dead code and can be safely removed with `path.removed()`

The babel implementation is shown below:

### Babel Implementation

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
const { Referenced } = require("@babel/traverse/lib/path/lib/virtual-types");
const { constants } = require("buffer");

/**
 * Main function to deobfuscate the code.
 * @param source The source code of the file to be deobfuscated
 *
 */
function deobfuscate(source) {
  //Parse AST of Source Code
  const ast = parser.parse(source);

  // Visitor for constant folding
  const removedUnusedVariablesVisitor = {
    "VariableDeclarator|FunctionDeclaration"(path) {
      const { node, scope } = path;
      const { constant, referenced } = scope.getBinding(node.id.name);
      // If the variable is constant and never referenced, remove it.
      if (constant && !referenced) {
        path.remove();
      }
    },
  };

  // Execute the visitor
  traverse(ast, removedUnusedVariablesVisitor);

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

deobfuscate(readFileSync("./unreferencedVariablesObfuscated.js", "utf8"));
```

After processing the obfuscated script with the babel plugin above, we get the following result:

### Post-Deobfuscation Result

```javascript
const u = "My favourite";
let dgajd = "is";
var i = "Hello";
var fg = "steak";
var lmaod = "!";
const el = "food";

const sayHello = function () {
  console.log(i + lmaod, u, el, dgajd, fg + lmaod);
};

sayHello();
```

And all non-referenced variables are restored.

**Extra**: By manual analysis, you can probably realize that all the variables declared above the `sayHello` function can have their values substituted in place of their identifiers inside of the `console.log` statement. How to accomplish that is outside the scope of this article. But, if you're interested in learning how to do it, you can read my article about it [here](http://SteakEnthusiast.github.io/2022/05/31/Deobfuscating-Javascript-via-AST-Replacing-References-to-Constant-Variables-with-Their-Actual-Value/).

## Example 2: [Dead Code] Empty Statements

An _empty statement_ is simply a semi-colon (`;`) with no same-line code before it. This script is littered with them:

```javascript
/**
 * emptyStatementSrc.js
 * Ugly 'obfuscated' code.
 */
const a = 2;
const b = 3;
console.log("a is", a, "b is", b);
```

The presence of empty statements doesn't really add much to obfuscation, but removing them will still remove unecessary noise and optimize the appearence.

### Analysis Methodology

We begin by pasting the example obfuscated script into [AST Explorer](https://astexplorer.net)

![A view of the obfuscated code in AST Explorer](empty1.PNG)

We can observe that the top-level view is polluted by _EmptyStatement_ nodes, causing a slight inconvenience when trying to navigate through the AST structure.

The deobfuscator logic is very simple:

1. Traverse the AST for _EmptyStatement_ nodes.
2. When one is encountered, delete it with `path.remove()`

The babel implementation is shown below:

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
  const deleteEmptyStatementsVisitor = {
    EmptyStatement(path) {
      path.remove();
    },
  };

  // Execute the visitor
  traverse(ast, deleteEmptyStatementsVisitor);

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

deobfuscate(readFileSync("./emptyStatementSrc.js", "utf8"));
```

After processing the obfuscated script with the babel plugin above, we get the following result:

### Post-Deobfuscation Result

```javascript
const a = 2;
const b = 3;
console.log("a is", a, "b is", b);
```

And all of the useless _EmptyStatements_ are now removed, enabling easier reading and navigation through the AST.

## Example 3: [Unreachable Code] If Statements and Logical Expressions

Let's take the following 'obfuscated' code snippet as an example:

```javascript
/**
 * unreachableLogicalCodeObfuscated.js
 */

if (!![]) {
  console.log("This always runs! 1");
} else {
  console.log("This never runs.");
}

if (40 > 80) {
  console.log("This never runs.");
} else if (1 < 2) {
  console.log("This always runs! 2");
} else {
  console.log("This never runs.");
}

![] ? console.log("This never runs.") : console.log("This always runs! 3");

// Chained example

if (!![]) {
  console.log("This always runs! 4");
  ![]
    ? console.log("This never runs.")
    : false
    ? console.log("This never runs")
    : 40 < 20
    ? console.log("This never runs.")
    : 80 > 1
    ? console.log("This always runs! 5")
    : 40 > 2
    ? console.log("This never runs.")
    : console.log("This never runs.");
} else {
  console.log("This never runs.");
}
```

This is an extremely simple example, especially since the `console.log` calls tell you exactly what runs and what doesn't. Keep in mind that code you find in the wild will probably be layered with other obfuscation techniques too. However, this is a good example for understanding the core concept. Rest assured though: the method I'll discuss should still be universal to all types of unreachable code obfuscation.

### Analysis Methodology

As always, we start our analysis by pasting the code into [AST Explorer](https://astexplorer.net)

![A view of the obfuscated code in AST Explorer](unreachable1.PNG)

We can see that at the top-level, the file consists of _IfStatements_ and an _ExpressionStatement_. If we expand the _ExpressionStatement_, we can see that ternary operator logical expressions are actually of type _ConditionalExpression_. Expanding an _IfStatement_ and a _ConditionalExpression_ for comparison, we can notice some similarities:

![Comparison of IfStatement vs. ConditionalExpression nodes](unreachable2.PNG)

We can see that aside from their type, both an _IfStatement_ and a _ConditionalExpression_ both have the exact same structure. That is, both contain:

- A `test` property, which is the test condition. It will either evaluate to true or false.

- A `consequent` property, which contains the code to be executed if `test` evalutes to a truthy value.

- A `alternate` property, which contains the code to be executed if `test` evaluates to a falsy value.
  - Note: This property is optional, since an If Statement need not be accompanied by an else.

For our deobfuscator, we'll make use of one of the babel API methods: `NodePath.evaluateTruthy()`.

This method takes in a path, then evaluates its node. If the node evaluates to a _truthy_ value, the method returns `true`. If the node evaluates to a _falsy_ value, the method returns `false`. _See: **[Truthy Values](https://developer.mozilla.org/en-US/docs/Glossary/Truthy)** vs. **[Falsy Values](https://developer.mozilla.org/en-US/docs/Glossary/Falsy)**_

The steps for creating the deobfuscator are as follows:

1. Traverse the AST for _IfStatements_ and _ConditionalExpressions_. Since both node types have the same structure, we can write a single plugin for both.
   - **Tip**: _To write a function that works for multiple visitor nodes, we can add an _ `|` _seperating them in the method name as a string like this:_ `"IfStatement|ConditionalExpression"`
2. When one is encountered, use the `NodePath.evaluateTruthy()` method on the `test` property's NodePath.
3. if the `NodePath.evaluateTruthy()` method returns true:
   1. Replace the path with the contents of `consequent`.
   2. _(Optional)_ If the consequent`is contained within a _BlockStatement_ (curly braces), we can replace it with the`consequent.body` to get rid of the curly braces.
4. if the `NodePath.evaluateTruthy()` method returns false:
   1. If the `alternate` property exists, replace the path with its contents.
   2. _(Optional)_ If the alternate is contained within a _BlockStatement_ (curly braces), we can replace it with the `alternate.body` to get rid of the curly braces.

The babel implementation is shown below:

### Babel Implementation

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
  const simplifyIfAndLogicalVisitor = {
    "ConditionalExpression|IfStatement"(path) {
      let { consequent, alternate } = path.node;
      let testPath = path.get("test");
      const value = testPath.evaluateTruthy();
      if (value === true) {
        if (t.isBlockStatement(consequent)) {
          consequent = consequent.body;
        }
        path.replaceWithMultiple(consequent);
      } else if (value === false) {
        if (alternate != null) {
          if (t.isBlockStatement(alternate)) {
            alternate = alternate.body;
          }
          path.replaceWithMultiple(alternate);
        } else {
          path.remove();
        }
      }
    },
  };

  // Execute the visitor
  traverse(ast, simplifyIfAndLogicalVisitor);

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

deobfuscate(readFileSync("./unreachableLogicalCodeObfuscated.js", "utf8"));
```

After processing the obfuscated script with the babel plugin above, we get the following result:

### Post-Deobfuscation Result

```javascript
console.log("This always runs! 1");
console.log("This always runs! 2");
console.log("This always runs! 3");
console.log("This always runs! 4");
```

And only the code which is designed to execute persists, a full restoration of the code!

# Conclusion

Cleaning up dead/unreachable code is an essential component of the deobfuscation process. I would recommend doing at least twice per program:

1. Firstly, at the start of the deobfuscation process. This will make the obfuscated script much more managable to navigate, as you can focus on the important parts of the program instead of junk code
2. At the end of the deobfuscation process, as part of the clean-up stage. Simplfying obfuscated code tends to reveal more dead code that can be removed, and removing it at the end results in a cleaner final product.

This article also gave a nice introduction to one of the useful Babel API methods. Unfortunately, there isn't much good documentation out there aside from the [Babel Plugin Handbook](https://github.com/jamiebuilds/babel-handbook/blob/master/translations/en/plugin-handbook.md). However, you can discover a lot more useful features Babel has to offer by reading its source code, or using the debugger of an IDE to list and test helper methods (the latter of which I personally prefer 😄).

If you're interested, you can find the source code for all the examples in [this repository](https://github.com/SteakEnthusiast/Supplementary-AST-Based-Deobfuscation-Materials).

Okay, that's all I have for you today. I hope that this article helped you learn something new. Thanks for reading, and happy reversing!