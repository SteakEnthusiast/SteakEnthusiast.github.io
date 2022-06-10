---
title: >-
  Deobfuscating Javascript via AST: Replacing References to Constant Variables
  with Their Actual Value
date: 2022-05-31 11:10:37
tags:
---

# Preface

This article assumes a preliminary understanding of Abstract Syntex Tree structure and [BabelJS](https://babeljs.io/). [Click Here](http://SteakEnthusiast.github.io/none) to read my introductory article on the usage of Babel.

# Definition of a Constant Variable

For our purposes, a constant variable is any variable that meets _**all three**_ of the following conditions:

- The variable is declared **AND** initialized at the same time.
- The variable is initialized to a _literal value_, e.g. _StringLiteral_, _NumericLiteral_, _BooleanLiteral_ etc.
- The variable is never reassigned another value in the script

Therefore, a variable's declaration keyword (`let`,`var`,`const`) has no bearing on whether or not it is a constant.

Here is a quick example:

```javascript
const a = [1, 2, 3];
var d = 12;
let e = "String!";
let f = 13;
let g;

f += 2;

console.log(a, b, d, e, f);
g = 14;
```

In this example:

- `a` is not a constant, since it's initialized as an _ArrayExpression_, not a _Literal_
- `d` is a constant, as it is declared and initialized to a _NumericLiteral_. Declaration and initialization happen at the same time. It is also never reassigned.
- `e` is a constant, as it is declared and initialized to a _StringLiteral_. Declaration and initialization happen at the same time. It is also never reassigned.
- `f` is not a constant, since it is reassigned after initialization: `f+=2`
- `g` is not a constant, since it is not declared and initialized at the same time.

The reasoning for declared but uninitialized variables not counting as a constant is important concept to understand. Take the following script as an example:

```javascript
let foo; // Initialization

console.log(foo); // => undefined

foo = 2;

console.log(foo); // => 2
```

Console Output:

```
undefined
2
```

If, in this case, we tried to substitute `foo`'s initialization value (`2`) for each reference of`foo`:

```javascript
let foo; // Initialization

console.log(2); // => 2, NOT undefined!

foo = 2;

console.log(2); // => 2
```

Console Output:

```
2
2
```

Which clearly breaks the original functionality of the script due to not accounting for the state of the variable at certain points in the script. Therefore, we must follow the 3 conditions when determining a constant variable.

I'll now discuss an example where substituting in constant variables can be useful for deobfuscation purposes.

# Examples

Let's say we have a very simple, unobfuscated script that looks like this:

```javascript
/**
 * "Input.js"
 * Original, unobfuscated code.
 *
 */
var url = "https://api.n0tar3als1t3.dev:1423/getData";
const req = function () {
  let random = Math.random() * 1000;
  var xhr = new XMLHttpRequest();
  xhr.open("GET", url);
  xhr.setRequestHeader("RandomInt", random);
  xhr.setRequestHeader("Accept", "text/html");
  xhr.setRequestHeader("Accept-Encoding", "gzip, deflate, br");
  xhr.setRequestHeader("Accept-Language", "en-US,en;q=0.9");
  xhr.setRequestHeader("Cache-Control", "no-cache");
  xhr.setRequestHeader("Connection", "keep-alive");
  xhr.setRequestHeader("Host", "n0tar3als1t3.dev");
  xhr.setRequestHeader("Pragma", "no-cache");
  xhr.setRequestHeader("Referer", "https://n0tar3als1t3.dev");
  xhr.setRequestHeader(
    `sec-ch-ua", "" Not A;Brand";v="99", "Chromium";v="101", "Google Chrome";v="101"`
  );
  xhr.setRequestHeader("sec-ch-ua-mobile", "?0");
  xhr.setRequestHeader("sec-ch-ua-platform", `"Windows"`);
  xhr.setRequestHeader("Sec-Fetch-Dest", "empty");
  xhr.setRequestHeader("Sec-Fetch-Mode", "cors");
  xhr.setRequestHeader("Sec-Fetch-Site", "same-origin");
  xhr.setRequestHeader(
    "User-Agent",
    "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/101.0.0.0 Safari/537.36"
  );

  xhr.onreadystatechange = function () {
    if (xhr.readyState === 4) {
      console.log(xhr.status);
      console.log(xhr.responseText);
    }
  };

  xhr.send();
};
```

We can obfuscate it by replacing all references to the string literals with references to constant variables:

```javascript
/**
 * "constantReferencesObfuscated.js"
 * This is the resulting code after obfuscation.
 *
 */

const QY$e_yOs = "https://api.n0tar3als1t3.dev:1423/getData";
let apNykoxUn = "sec-ch-ua-mobile";
const zgDT = "Connection";
let A$E =
  "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/101.0.0.0 Safari/537.36";
const XVyy$qGVDc = "Sec-Fetch-Dest";
var EkoMLkb = "Cache-Control";
let $jAONLEC = "Host";
var PGOSDhGVlcd = "https://n0tar3als1t3.dev";
const m$ua = "Accept-Encoding";
var Hw$seiMEes = "Pragma";
const ZHCx = "Sec-Fetch-Site";
var PfxQUj = "Referer";
const e_WXHbgheSe = "Accept";
const _VTGows = "GET";
var kphzJIkbgb = "gzip, deflate, br";

const req = function () {
  const SNgfg = "no-cache";
  let vOqEy = "text/html";
  const uugBXYcdsHp = "same-origin";
  const AH$HwC = "Accept-Language";
  var PnAJsD =
    'sec-ch-ua", "" Not A;Brand";v="99", "Chromium";v="101", "Google Chrome";v="101"';
  const Svno = "n0tar3als1t3.dev";
  let OTCqIvdmed = '"Windows"';
  let mVu = "RandomInt";
  const UgLln = "empty";
  const HwjBe = "?0";
  var QnXFnewjh = "Sec-Fetch-Mode";
  var lGhlU$gqPoK = "cors";
  const GcictYiOQ = "User-Agent";
  const AfYNl = "no-cache";
  var cLAVjnFa = "keep-alive";
  var V$lt = "en-US,en;q=0.9";
  const TlMBXe = "sec-ch-ua-platform";
  let random = Math.random() * 1000;
  var xhr = new XMLHttpRequest();
  var url = QY$e_yOs;
  xhr.open(_VTGows, url);
  xhr.setRequestHeader(mVu, random);
  xhr.setRequestHeader(e_WXHbgheSe, vOqEy);
  xhr.setRequestHeader(m$ua, kphzJIkbgb);
  xhr.setRequestHeader(AH$HwC, V$lt);
  xhr.setRequestHeader(EkoMLkb, SNgfg);
  xhr.setRequestHeader(zgDT, cLAVjnFa);
  xhr.setRequestHeader($jAONLEC, Svno);
  xhr.setRequestHeader(Hw$seiMEes, AfYNl);
  xhr.setRequestHeader(PfxQUj, PGOSDhGVlcd);
  xhr.setRequestHeader(PnAJsD);
  xhr.setRequestHeader(apNykoxUn, HwjBe);
  xhr.setRequestHeader(TlMBXe, OTCqIvdmed);
  xhr.setRequestHeader(XVyy$qGVDc, UgLln);
  xhr.setRequestHeader(QnXFnewjh, lGhlU$gqPoK);
  xhr.setRequestHeader(ZHCx, uugBXYcdsHp);
  xhr.setRequestHeader(GcictYiOQ, A$E);

  xhr.onreadystatechange = function () {
    if (xhr.readyState === 4) {
      console.log(xhr.status);
      console.log(xhr.responseText);
    }
  };

  xhr.send();
};
```

## Analysis Methodology

Obviously, the obfuscated script is much more difficult to read. If you were to manually deobfuscate it, you'd have to search up each referenced variable and replace each occurence of it with the actual variable. That could get tedious for a large amount of variables, so we're going to do it the Babel way. As always, let's start by pasting the code into [AST Explorer](https://astexplorer.net/).

![View of the obfuscated code in AST Explorer](replacing1.PNG)

Our target of interest are the extra variable declarations. Let's take a closer look at one of them:

![A closer look at one of the nodes of interest](replacing2.PNG)

So, the target node type appears to be of type _VariableDeclaration_. However, each of these _VariableDeclarations_ contain an array of _VariableDeclarators_. It is the _VariableDeclarator_ that actually contains the information of the variables, including its `id` and `init` value. So, the actual node type we should focus on are _VariableDeclarators_.

Recall that we want to identify all **_constant_** variables, then replace all their **_references_** with their actual value. It's important to note that variables in different scopes (e.g. local vs. global), may share the same name but have different values. So, the solution isn't as simple as blindly replacing all matching identifiers with their initial value.

This would be a convoluted process if not for Babel's 'Scope' API. I won't dive too deep into the available scope API's, but you can refer to the [Babel Plugin Handbook](https://github.com/jamiebuilds/babel-handbook/blob/master/translations/en/plugin-handbook.md#toc-scopes) to learn more about them. In our case, the `scope.getBinding(${identifierName})` method will be incredibly useful for us, as it directly returns information regarding if a variable is constant and all of its references.

Putting all this knowledge together, the steps for creating the deobfuscator are as follows:

1. Traverse the ast in search of _VariableDeclarators_. If one is found:
   1. Check if the variable is initialized. If it is, check that the initial value is a _Literal_ type. If not, skip the node by returning.
   2. Use the `path.scope.getBinding(${identifierName})` method with the name of the current variable as the argument.
   3. Store the returned `constant` and `referencedPaths` properties in their own respective variables.
   4. Check if the `constant` property is `true`. If it isn't, skip the node by returning.
   5. Loop through all NodePaths in the `referencedPaths` array, and replace them with the current _VariableDeclarator_ 's initial value (`path.node.init`)
   6. After finishing the loop, remove the original _VariableDeclarator_ node since it has no further use.

The babel implementation is shown below:

# Babel Deobfuscation Script

```Javascript

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

  // Visitor for replacing constants

  const replaceRefsToConstants = {
    VariableDeclarator(path) {
      const { id, init } = path.node;
      // Ensure the the variable is initialized to a Literal type.
      if (!t.isLiteral(init)) return;
      let {constant, referencePaths} = path.scope.getBinding(id.name);
      // Make sure it's constant
      if (!constant) return;
      // Loop through all references and replace them with the actual value.
      for (let referencedPath of referencePaths) {
        referencedPath.replaceWith(init);
      }
      // Delete the now useless VariableDeclarator
      path.remove();
    },
  };

  // Execute the visitor
  traverse(ast, replaceRefsToConstants);

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

deobfuscate(readFileSync("./constantReferencesObfuscated.js", "utf8"));

```

After processing the obfuscated script with the babel plugin above, we get the following result:

# Post-Deobfuscation Result

```javascript
const req = function () {
  let random = Math.random() * 1000;
  var xhr = new XMLHttpRequest();
  xhr.open("GET", "https://api.n0tar3als1t3.dev:1423/getData");
  xhr.setRequestHeader("RandomInt", random);
  xhr.setRequestHeader("Accept", "text/html");
  xhr.setRequestHeader("Accept-Encoding", "gzip, deflate, br");
  xhr.setRequestHeader("Accept-Language", "en-US,en;q=0.9");
  xhr.setRequestHeader("Cache-Control", "no-cache");
  xhr.setRequestHeader("Connection", "keep-alive");
  xhr.setRequestHeader("Host", "n0tar3als1t3.dev");
  xhr.setRequestHeader("Pragma", "no-cache");
  xhr.setRequestHeader("Referer", "https://n0tar3als1t3.dev");
  xhr.setRequestHeader(
    'sec-ch-ua", "" Not A;Brand";v="99", "Chromium";v="101", "Google Chrome";v="101"'
  );
  xhr.setRequestHeader("sec-ch-ua-mobile", "?0");
  xhr.setRequestHeader("sec-ch-ua-platform", '"Windows"');
  xhr.setRequestHeader("Sec-Fetch-Dest", "empty");
  xhr.setRequestHeader("Sec-Fetch-Mode", "cors");
  xhr.setRequestHeader("Sec-Fetch-Site", "same-origin");
  xhr.setRequestHeader(
    "User-Agent",
    "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/101.0.0.0 Safari/537.36"
  );

  xhr.onreadystatechange = function () {
    if (xhr.readyState === 4) {
      console.log(xhr.status);
      console.log(xhr.responseText);
    }
  };

  xhr.send();
};
```

And the code is restored. Even better than the original actually, since we substituted in the `url` variable too!

# Conclusion

Substitution of constant variables is a must-know deobfuscation technique. It'll usually be one of your first steps in the deobfuscation, combined with _constant folding_. If you would like to learn about constant folding, you can read my article about it [here](http://SteakEnthusiast.github.io/2022/05/28/Deobfuscating-Javascript-via-AST-Manipulation-Constant-Folding/).

This article also gave a nice introduction to one of the useful Babel API methods. Unfortunately, there isn't much good documentation out there aside from the [Babel Plugin Handbook](https://github.com/jamiebuilds/babel-handbook/blob/master/translations/en/plugin-handbook.md). However, you can discover a lot more useful features Babel has to offer by reading its source code, or using the debugger of an IDE to list and test helper methods (the latter of which I personally prefer 😄).

If you're interested, you can find the source code for all the examples in [this repository](https://github.com/SteakEnthusiast/Supplementary-AST-Based-Deobfuscation-Materials).

Okay, that's all I have for you today. I hope that this article helped you learn something new. Thanks for reading, and happy reversing!