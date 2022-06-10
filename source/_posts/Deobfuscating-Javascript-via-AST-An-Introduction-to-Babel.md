---
title: "An Introduction to Javascript Obfuscation & Babel"
date: 2022-05-21 23:44:58
tags:
---

# Introduction

Welcome to the first article in my series about Javascript deobfuscation. I won't be going indepth regarding practical deobfuscation techniques; that'll be reserved for later articles. Rather, this post serve as a brief overview of the state of Javascript obfuscation, different methods of analysis, and provide resources to learn more about reverse engineering Javascript.

# What is Obfuscation?

## Definition

Obfuscation is a series of code transformations that turn human-readable code into something that is deliberately difficult for a human to understand, while (for the most part) still maintaining its original functionality. Code authors may choose to obfuscate their code for many reasons, including but not limited to:

- To make it harder to modify, debug, or reproduce (e.g. some javascript-based games or programs)
- To hide it a malicious intent intent (e.g. malware)
- To enchance security, i.e obscuring the logic behind javascript-based challenges or fingerprinting (e.g. ReCAPTCHA, Shape Security, PerimeterX, Akamai, DataDome)

## Example

For example, the obfuscation process can convert this human readable script:

```js
console.log("Hello");
```

Into something incomprehensible to humans:

```js
var _0x3b8ba1 = _0x57e2;
function _0x57e2(_0x23db1e, _0x36111b) {
  var _0x3bbee9 = _0x5936();
  return (
    (_0x57e2 = function (_0x194e56, _0x27d4e2) {
      _0x194e56 = _0x194e56 - 0x17e;
      var _0x2b5447 = _0x3bbee9[_0x194e56];
      return _0x2b5447;
    }),
    _0x57e2(_0x23db1e, _0x36111b)
  );
}
(function (_0x3d5379, _0x27d8c9) {
  var _0x26235b = _0x57e2,
    _0x556a19 = _0x3d5379();
  while (!![]) {
    try {
      var _0x15999f =
        parseInt(_0x26235b(0x183)) / 0x1 +
        parseInt(_0x26235b(0x185)) / 0x2 +
        (parseInt(_0x26235b(0x194)) / 0x3) *
          (-parseInt(_0x26235b(0x18d)) / 0x4) +
        parseInt(_0x26235b(0x188)) / 0x5 +
        (-parseInt(_0x26235b(0x18b)) / 0x6) *
          (-parseInt(_0x26235b(0x187)) / 0x7) +
        -parseInt(_0x26235b(0x182)) / 0x8 +
        -parseInt(_0x26235b(0x195)) / 0x9;
      if (_0x15999f === _0x27d8c9) break;
      else _0x556a19["push"](_0x556a19["shift"]());
    } catch (_0x3cc29a) {
      _0x556a19["push"](_0x556a19["shift"]());
    }
  }
})(_0x5936, 0x4bc84);
var _0x5cff45 = (function () {
    var _0x5a2bb8 = !![];
    return function (_0x2e90c1, _0x495f04) {
      var _0x1ac9d1 = _0x5a2bb8
        ? function () {
            var _0x261249 = _0x57e2;
            if (_0x495f04) {
              var _0x3c800c = _0x495f04[_0x261249(0x198)](_0x2e90c1, arguments);
              return (_0x495f04 = null), _0x3c800c;
            }
          }
        : function () {};
      return (_0x5a2bb8 = ![]), _0x1ac9d1;
    };
  })(),
  _0x1ea628 = _0x5cff45(this, function () {
    var _0x4f765e = _0x57e2;
    return _0x1ea628[_0x4f765e(0x17f)]()
      ["search"](_0x4f765e(0x18e))
      ["toString"]()
      ["constructor"](_0x1ea628)
      [_0x4f765e(0x192)](_0x4f765e(0x18e));
  });
_0x1ea628();
function _0x5936() {
  var _0x7289e8 = [
    "Hello\x20World!",
    "toString",
    "log",
    "__proto__",
    "2888432EGELDh",
    "516645rknrWL",
    "trace",
    "928870xUjHrE",
    "error",
    "27965akgdka",
    "2813765Wufwlg",
    "return\x20(function()\x20",
    "warn",
    "48zUcTLM",
    "bind",
    "2668xZhNIu",
    "(((.+)+)+)+$",
    "prototype",
    "console",
    "table",
    "search",
    "length",
    "615NtfKnc",
    "6908400qvcpUL",
    "exception",
    "constructor",
    "apply",
  ];
  _0x5936 = function () {
    return _0x7289e8;
  };
  return _0x5936();
}
var _0x27d4e2 = (function () {
    var _0x494152 = !![];
    return function (_0x2d8431, _0x2bbb6a) {
      var _0x1528ad = _0x494152
        ? function () {
            if (_0x2bbb6a) {
              var _0x4f8607 = _0x2bbb6a["apply"](_0x2d8431, arguments);
              return (_0x2bbb6a = null), _0x4f8607;
            }
          }
        : function () {};
      return (_0x494152 = ![]), _0x1528ad;
    };
  })(),
  _0x194e56 = _0x27d4e2(this, function () {
    var _0x2df84e = _0x57e2,
      _0x50a5eb;
    try {
      var _0x458538 = Function(
        _0x2df84e(0x189) + "{}.constructor(\x22return\x20this\x22)(\x20)" + ");"
      );
      _0x50a5eb = _0x458538();
    } catch (_0x55824d) {
      _0x50a5eb = window;
    }
    var _0x22e34f = (_0x50a5eb[_0x2df84e(0x190)] =
        _0x50a5eb[_0x2df84e(0x190)] || {}),
      _0x4b7f35 = [
        _0x2df84e(0x180),
        _0x2df84e(0x18a),
        "info",
        _0x2df84e(0x186),
        _0x2df84e(0x196),
        _0x2df84e(0x191),
        _0x2df84e(0x184),
      ];
    for (
      var _0x24f5c9 = 0x0;
      _0x24f5c9 < _0x4b7f35[_0x2df84e(0x193)];
      _0x24f5c9++
    ) {
      var _0x126b34 =
          _0x27d4e2[_0x2df84e(0x197)][_0x2df84e(0x18f)][_0x2df84e(0x18c)](
            _0x27d4e2
          ),
        _0x427a50 = _0x4b7f35[_0x24f5c9],
        _0xdec475 = _0x22e34f[_0x427a50] || _0x126b34;
      (_0x126b34[_0x2df84e(0x181)] = _0x27d4e2[_0x2df84e(0x18c)](_0x27d4e2)),
        (_0x126b34[_0x2df84e(0x17f)] =
          _0xdec475["toString"][_0x2df84e(0x18c)](_0xdec475)),
        (_0x22e34f[_0x427a50] = _0x126b34);
    }
  });
_0x194e56(), console[_0x3b8ba1(0x180)](_0x3b8ba1(0x17e));
```

Yet, believe it or not, both of these scripts have the exact same functionality! You can test it yourself: both scripts output

```
Hello World
```

to the console.

## The State of Javascript Obfuscation

There are many available javascript obfuscators, both closed and open-source. Here's a small list:

**Open-Source**

- [Obfuscator.io](https://obfuscator.io)
- [JSFuck](http://www.jsfuck.com/)
- [DefendJS](https://github.com/alexhorn/defendjs)

**Closed-Source**

- [Jscrambler](https://jscrambler.com/)
- [Javascript Obfuscator](https://javascriptobfuscator.com)
- [JSDefender](https://www.preemptive.com/products/jsdefender/)

For further reading on on the why and how's of Javascript Obfuscation, I recommend checking out the [Jscrambler blog posts](https://blog.jscrambler.com/javascript-obfuscation-the-definitive-guide/). For now though, I'll shift the topic towards reverse engineering.

# How is Obfuscated Code Analyzed?

In general, most reverse engineering/deobfuscation techniques fall under two categories: _static analysis_ and _dynamic analysis_

## Static Analysis

Static analysis refers to the inspection of source code **without actually executing the program**. An example of static analysis is simplifying source code with Regex.

## Dynamic Analysis

Dynamic analysis refers to the testing and analysis of an application **during run time/evaluation**. An example of dynamic analysis is using a debugger.

## Static vs. Dynamic Analysis Use-Cases

Since static analysis does not execute code, it makes it ideal for analyzing untrusted scripts. For example, when analyzing malware, you may want to use static analysis to avoid infection of your computer.

Dynamic analysis is used when a script is known to be safe to run. Debuggers can be powerful tools for reverse engineering, as they allow you to view the state of the program at different points in the runtime. Additionally, dyanmic analysis can be (and often is) used for malware analysis too, but only after taking proper security precautions (i.e sandboxing).

Static and dynamic analysis are powerful when used together. For example, debugging a script containing a lot of junk code can be difficult. Or, the code may contain anti-debugging protection (e.g. [infinite debugger loops](https://x-c3ll.github.io/posts/javascript-antidebugging/)). In this case, someone may first use static inspection of source code to simplify the source code, then proceed with dynamic analysis using the modified source.

# Introducing Babel

[Babel](https://babeljs.io) is a Javascript to Javascript compiler. The functionalities included with the Babel framework make it exceptionally useful for any javascript deobfuscation use case, since you can use it for static analysis and dynamic analysis!

Let me give a short explanation of how it works:

Javascript is an intepreted programming language. For Javascript to be interpreted by an engine (e.g. Chrome's V8 engine or Firefox's Spidermonkey) into machine code, it is first parsed into an **_Abstract Syntax Tree_** (AST). After that, the AST is used to generate machine-readable byte-code, which is then executed.

Babel works in a similar fashion. It takes in Javascript code, parses it into an AST, then outputs javascript based on that AST.

Okay, sounds interesting. But what even is an AST?

## Definition: Abstract Syntax Tree

An **_Abstract Syntax Tree_** (AST) is a tree-like structure that hierarchaly represents the syntax of a piece of source code. Each node of the tree represents the occurence of a predefined structure in the source code. Any piece of source code, from any programming language, can be represented as an AST.

Note: _Even though the concepts behind an AST are universal, different programming languages may have a different AST specification based on their capabilities._

Some practical uses of ASTs include:

- Validating Code
- Formatting Code
- Syntax Highlighting

And, of course, due to more verbose nature of ASTs relative to plaintext source code, it makes them a great tool for reverse engineering 😁

Unfortunately, I won't be giving a more indepth definition of ASTs. This is for the sake of time, and since that'd be more akin to the subject of compiler theory than deobfuscation. I'd prefer to get right into explaining the usage of Babel as quickly as possible. However, I'll leave you with some resources to read up more about ASTs (which probably offer a better explanation than I could muster anyways):

[Wikipedia - Abstract Syntax Trees](https://en.wikipedia.org/wiki/Abstract_syntax_tree)
[How JavaScript works: Parsing, Abstract Syntax Trees (ASTs) + 5 tips on how to minimize parse time](https://blog.sessionstack.com/how-javascript-works-parsing-abstract-syntax-trees-asts-5-tips-on-how-to-minimize-parse-time-abfcf7e8a0c8)

## How Babel Works

Babel can be installed the same way as any other NodeJS package. For our purposes, the following packages are relevant:

`@babel/core` This encapsulates the entire Babel compiler API.
`@babel/parser` The module Babel uses to parse Javascript source code and generate an AST
`@babel/traverse` The module that allows for traversing and modifying the generated AST
`@babel/generator` The module Babel uses to generate Javascript code from the AST.
`@babel/types` A module for verifying and generating node types as defined by the Babel AST implementation.

When compiling code, Babel goes through three main phases:

1. **Parsing** => Uses `@babel/parser` API
2. **Transforming** => Uses `@babel/traverse` API
3. **Code Generation** => Uses `@babel/generator` API

I'll give you a (very) short summary of each of these phases:

### Stages of Babel

#### Phase #1: Parsing

During this phase, Babel takes source code as an input and outputs an AST. Two stages of parsing are [Lexical Analysis](https://en.wikipedia.org/wiki/Lexical_analysis) and [Syntactic Analysis](https://en.wikipedia.org/wiki/Parsing).

To parse code into an AST, we make use of `@babel/parser`. The following is an example of parsing code from a file, `sourcecode.js`:

```javascript
const parser = require("@babel/parser");
const code = fs.readFileSync("sourcecode.js", "utf-8");
let ast = parser.parse(code);
```

You can read more about the parsing phase here:
[Babel Plugin Handbook - Parsing](https://github.com/jamiebuilds/babel-handbook/blob/master/translations/en/plugin-handbook.md#parse)
[Babel Docs - @babel/parser](https://babeljs.io/docs/en/babel-parser)

#### Phase 2: Transforming

The transformation phase is the most important phase. During this phase, Babel takes the generated AST and traverses it to add, update, or remove nodes. All the deobfuscation transformations we write are executed in this stage. This stage will be the main focus of future tutorials.

#### Phase 3: Code Generation

The code generation phase takes in the final AST and converts it back to executable Javascript.

# The Babel Workflow

This section will not discuss any practical deobfuscation techniques. It will only detail the general process of analyzing source code. I'll be using an unobfuscated piece of code as an example.

When deobfuscating Javascript, I typically follow this workflow:

1. Visualization
2. Analysis
3. Writing the Deobfuscator

## Phase 1: Visualization with AST Explorer

Before we can write any plugins for a deobfuscator, we should always first visualize the code's AST. To help us with that, we will leverage an online tool: [AstExplorer.net](https://astexplorer.net).

AST Explorer serves as an interactive AST playground. It allows you to choose a programming language and parser. In this case, we would select Javascript as the programming language and `@babel/parser` as the parser. After that, we can paste some source code into the window and inspect the generated AST on the right-hand side.

As an example, I'll use this snippet:

```javascript
function operation(arg1, arg2) {
  let step1 = arg1 + arg2;
}

let foo = operation(6, 8);
```

![Result from pasting the code snippet in AST Explorer](intro1.PNG)

The generated AST looks like this:

<details>
  <summary> Click to Expand</summary>

```javascript
{
  "type": "File",
  "start": 0,
  "end": 78,
  "loc": {
    "start": {
      "line": 1,
      "column": 0
    },
    "end": {
      "line": 5,
      "column": 24
    }
  },
  "errors": [],
  "program": {
    "type": "Program",
    "start": 0,
    "end": 78,
    "loc": {
      "start": {
        "line": 1,
        "column": 0
      },
      "end": {
        "line": 5,
        "column": 24
      }
    },
    "sourceType": "module",
    "interpreter": null,
    "body": [
      {
        "type": "FunctionDeclaration",
        "start": 0,
        "end": 52,
        "loc": {
          "start": {
            "line": 1,
            "column": 0
          },
          "end": {
            "line": 3,
            "column": 1
          }
        },
        "id": {
          "type": "Identifier",
          "start": 9,
          "end": 18,
          "loc": {
            "start": {
              "line": 1,
              "column": 9
            },
            "end": {
              "line": 1,
              "column": 18
            },
            "identifierName": "operation"
          },
          "name": "operation"
        },
        "generator": false,
        "async": false,
        "params": [
          {
            "type": "Identifier",
            "start": 19,
            "end": 23,
            "loc": {
              "start": {
                "line": 1,
                "column": 19
              },
              "end": {
                "line": 1,
                "column": 23
              },
              "identifierName": "arg1"
            },
            "name": "arg1"
          },
          {
            "type": "Identifier",
            "start": 24,
            "end": 28,
            "loc": {
              "start": {
                "line": 1,
                "column": 24
              },
              "end": {
                "line": 1,
                "column": 28
              },
              "identifierName": "arg2"
            },
            "name": "arg2"
          }
        ],
        "body": {
          "type": "BlockStatement",
          "start": 29,
          "end": 52,
          "loc": {
            "start": {
              "line": 1,
              "column": 29
            },
            "end": {
              "line": 3,
              "column": 1
            }
          },
          "body": [
            {
              "type": "ReturnStatement",
              "start": 32,
              "end": 50,
              "loc": {
                "start": {
                  "line": 2,
                  "column": 1
                },
                "end": {
                  "line": 2,
                  "column": 19
                }
              },
              "argument": {
                "type": "BinaryExpression",
                "start": 39,
                "end": 50,
                "loc": {
                  "start": {
                    "line": 2,
                    "column": 8
                  },
                  "end": {
                    "line": 2,
                    "column": 19
                  }
                },
                "left": {
                  "type": "Identifier",
                  "start": 39,
                  "end": 43,
                  "loc": {
                    "start": {
                      "line": 2,
                      "column": 8
                    },
                    "end": {
                      "line": 2,
                      "column": 12
                    },
                    "identifierName": "arg1"
                  },
                  "name": "arg1"
                },
                "operator": "+",
                "right": {
                  "type": "Identifier",
                  "start": 46,
                  "end": 50,
                  "loc": {
                    "start": {
                      "line": 2,
                      "column": 15
                    },
                    "end": {
                      "line": 2,
                      "column": 19
                    },
                    "identifierName": "arg2"
                  },
                  "name": "arg2"
                }
              }
            }
          ],
          "directives": []
        }
      },
      {
        "type": "VariableDeclaration",
        "start": 54,
        "end": 78,
        "loc": {
          "start": {
            "line": 5,
            "column": 0
          },
          "end": {
            "line": 5,
            "column": 24
          }
        },
        "declarations": [
          {
            "type": "VariableDeclarator",
            "start": 58,
            "end": 78,
            "loc": {
              "start": {
                "line": 5,
                "column": 4
              },
              "end": {
                "line": 5,
                "column": 24
              }
            },
            "id": {
              "type": "Identifier",
              "start": 58,
              "end": 61,
              "loc": {
                "start": {
                  "line": 5,
                  "column": 4
                },
                "end": {
                  "line": 5,
                  "column": 7
                },
                "identifierName": "foo"
              },
              "name": "foo"
            },
            "init": {
              "type": "CallExpression",
              "start": 64,
              "end": 78,
              "loc": {
                "start": {
                  "line": 5,
                  "column": 10
                },
                "end": {
                  "line": 5,
                  "column": 24
                }
              },
              "callee": {
                "type": "Identifier",
                "start": 64,
                "end": 73,
                "loc": {
                  "start": {
                    "line": 5,
                    "column": 10
                  },
                  "end": {
                    "line": 5,
                    "column": 19
                  },
                  "identifierName": "operation"
                },
                "name": "operation"
              },
              "arguments": [
                {
                  "type": "NumericLiteral",
                  "start": 74,
                  "end": 75,
                  "loc": {
                    "start": {
                      "line": 5,
                      "column": 20
                    },
                    "end": {
                      "line": 5,
                      "column": 21
                    }
                  },
                  "extra": {
                    "rawValue": 6,
                    "raw": "6"
                  },
                  "value": 6
                },
                {
                  "type": "NumericLiteral",
                  "start": 76,
                  "end": 77,
                  "loc": {
                    "start": {
                      "line": 5,
                      "column": 22
                    },
                    "end": {
                      "line": 5,
                      "column": 23
                    }
                  },
                  "extra": {
                    "rawValue": 8,
                    "raw": "8"
                  },
                  "value": 8
                }
              ]
            }
          }
        ],
        "kind": "let"
      }
    ],
    "directives": []
  },
  "comments": []
}
```

</details>

We can observe that even for this small little program, the AST representation is incredibly verbose. It's composed of different types of nodes (`FunctionDeclaration`s, `ExpressionStatement`s, `Identifier`s, `CallExpression`s etc.), and many nodes also have a sub node. To transform the AST, we'll be making use of the Babel traverse package to recursively traverse the tree and modify nodes.

## Phase 2: Coming Up With The Transformation Logic/Pseudo-code

This isn't an obfuscated file, but we'll still write a plugin to demonstrate the traverse package's functionality.

Let's assign ourselves an arbitrary goal of transforming the script to replace all occurences of arithmetic addition operators (`+`) with arithmetic multiplication operators (`*`). That is, the final script should look like this:

```javascript
function operation(arg1, arg2) {
  return arg1 * arg2;
}

let foo = operation(6, 8);
```

### Determining the Target Node Type(s)

First, we need to determine what our node type(s) of interest are. If we highlight a section of the code, AST explorer will automatically expand that node on the right-hand side. In our case, we want to focus on the `arg1 + arg2` operation. After highlughting that piece of code, we'll see this:

![A closer look at the nodes of interest](intro2.png)

We can see that `arg1 + arg2` has been parsed into a `BinaryExpression` node. This node has the following properties:

- `type` stores the node's type, in this case: `BinaryExpression`
- `left` stores the information for the left side of the expression, in this case: the `arg1` identifier.
- `right` stores the information for the right side of the expression, in this case: the `arg2` identifier.
- `operator` stores the operator, in this case: `+`.

Our goal is to replace all `+` operators in the script with a `*` operator, so it makes sense that our node type of interest is a `BinaryExpression`.

Now that we have our target node type, we need to figure out how we'll transform them

### Transformation Logic

To reiterate: we know that we're looking for `BinaryExpression`s. Each `BinaryExpression` has a property, `operator`. We want to edit this property to `*` if it is a `+`.

The logical proccess would therefore look like this:

1. Parse the code to generate an AST.
1. Traverse the AST in search of `BinaryExpression`s.
1. If one is encountered, check that it's operator is currently equal to `+`. If it isn't, skip that node.
1. If the operator is equal to `+`, set the operator to `*`.

Now that we understand the logic, we can write it as code

## Phase 3: Writing the Transformation Code

To parse the tree, we will use the `@babel/parser` package as previously demonstrated. To traverse the generated AST and modify the nodes, we'll make use of `@babel/traverse`.

To target a specific node type during traversal, we'll use a visitor[https://github.com/jamiebuilds/babel-handbook/blob/master/translations/en/plugin-handbook.md#visitors].

From the Babel Plugin Handbook:

> Visitors are a pattern used in AST traversal across languages. Simply put they are an object with methods defined for accepting particular node types in a tree.

To target nodes of type `BinaryExpression`, our visitor would like like this:

```javascript
const changeOperatorVisitor = {
  BinaryExpression(path) {
    // transformations here ...
  },
};
```

Now, every time a `BinaryExpression` is encountered, the `BinaryExpression(path)` method will be called.

Inside of the `BinaryExpression(path)` method of our visitor, we can add code for any checks and transformations.

Each visitor method takes in a parameter, `path`, which holds the path to the node being visited. To access the actual properties of the node, we must use `path.node`.

Our first step in our transformation would be to check that the `operator` property of the node is a `+`. We can do that like this:

```javascript
const changeOperatorVisitor = {
  BinaryExpression(path) {
    if (path.node.operator == "+") {
      // continue with transformations...
    } else {
      return; // Skip the node
    }
  },
};
```

If it is a `+`, we can set it to `*`.

```javascript
const changeOperatorVisitor = {
  BinaryExpression(path) {
    // Check if operator is +
    if (path.node.operator == "+") {
      // Set operator as *
      path.node.operator = "*";
    } else {
      return; // Skip the node
    }
  },
};
```

And our visitor is complete! Now we just need to call it on the generated AST. But first, let's generate the AST:

```javascript
const parser = require("@babel/parser");
const generate = require("@babel/generator").default;
const traverse = require("@babel/traverse").default;
const types = require("@babel/types");
// Set the source code
const code = `
function operation(arg1, arg2) {
  return arg1 * arg2;
}
let foo = operation(6, 8);
`;
// Parse the source code into an AST
let ast = parser.parse(code);
```

After that, we can paste our visitor into the source code. To taverse the AST using the visitor, we'll use the `traverse` method from the `@babel/traverse` package. That would look like this:

```javascript
const parser = require("@babel/parser");
const generate = require("@babel/generator").default;
const traverse = require("@babel/traverse").default;
const types = require("@babel/types");
// Set the source code
const code = `
function operation(arg1, arg2) {
  return arg1 * arg2;
}
let foo = operation(6, 8);
`;
// Parse the source code into an AST
let ast = parser.parse(code);

// Visitor for modifying operator of BinaryExpression
const changeOperatorVisitor = {
  BinaryExpression(path) {
    // Check if operator is +
    if (path.node.operator == "+") {
      // Set operator as *
      path.node.operator = "*";
    } else {
      return; // Skip the node
    }
  },
};

traverse(ast, changeOperatorVisitor);
```

Finally, we'll use the `generate` method from the `@babel/generator` package to generate the final code from the modified AST. We can also output the reuslting code to a file, but I'll just log it to the console for simplicity.

So, our final deobfuscation script looks like this:

```javascript
const parser = require("@babel/parser");
const generate = require("@babel/generator").default;
const traverse = require("@babel/traverse").default;
const types = require("@babel/types");
// Set the source code
const code = `
function operation(arg1, arg2) {
  return arg1 * arg2;
}
let foo = operation(6, 8);
`;
// Parse the source code into an AST
let ast = parser.parse(code);

// Visitor for modifying operator of BinaryExpression
const changeOperatorVisitor = {
  BinaryExpression(path) {
    // Check if operator is +
    if (path.node.operator == "+") {
      // Set operator as *
      path.node.operator = "*";
    } else {
      return; // Skip the node
    }
  },
};

traverse(ast, changeOperatorVisitor);

let finalCode = generate(ast).code;

console.log(finalCode);
```

This will output the following to the console:

```javascript
function operation(arg1, arg2) {
  return arg1 * arg2;
}

let foo = operation(6, 8);
```

And we can see that the code has been successfully transformed to replace `+` operators with `*` operators!

# Why use Babel for Deobfuscation?

So, why should we use use Babel as a deobfuscation tool as opposed to other static analysis tools like Regex?

Here's a few reasons:

1. **Ast is less error-prone.**

   - For large chunks of code, writing transfomrations can become incredibly tedious due to the edge cases. For example, a it's difficult to account for scope and state of variables when using regex. For example, two different variables can share the same name if they're in different scopes:

```javascript
//Scope 1:
{
  let foo = 123;
  {
    let foo = 321;
    console.log(foo);
  }
  console.log(foo);
}
```

Eventually, regular expressions will become very convoluted when you have to account for edge cases; whether it be scope, or tiny variations in syntax. Babel doesn't have this problem, as you can use built it functionality to make transformations with respect to scope and state.

2. **The Babel API has a lot of useful features.**

   Here's a few useful things you can do with the built-in Babel API:

   - Easily target certain nodes
   - Handle scope when renaming/replacing variables
   - Easily get initial values and references of variables
   - Node validation, generation, cloning, replacement, removal
   - Find paths to ancestor and descendant nodes based on test conditions
   - Containers/Lists: Check if a node is in a container/list, and get all of its siblings

3. **Good for static and dynamic analysis**
   - Inherently, parsing the code into an AST and applying transformations will not execute the code. But Babel also has functionality to evaluate nodes (ex. BinaryExpressions) and return their actual value. Babel can also generate code from nodes, which can be evaluated in a with `eval` or the NodeJS VM.

# Conclusion + Additional Resources

That was a short demonstration of transforming a piece of code with Babel! The next articles will be more in-depth, and include practical cases of reversing obfuscation techniques you might encounter in the wild.

For the sake of time, I didn't go to deep into the behind the scenes of Babel or all of it's API methods. In the future, I may decide to update this article or write a new one with more detailed explanations, examples, and documentation. But, I really recommend getting a solid fundamental understanding of Babel's features before continuing on in this series. Most notably, I didn't cover usage of the `@babel/types` package in this article, but it will be utilized in future ones. I'd recommend giving these resources a look:

[Official Babel Docs](https://babeljs.io/docs/en/)
[Babel Plugin Handbook](https://github.com/jamiebuilds/babel-handbook/blob/master/translations/en/plugin-handbook.md)
[Video: @babel/how-to](https://www.youtube.com/watch?v=UeVq_U5obnE)

Here are links to the other articles in this series:


- [Deobfuscating Javascript via AST: Reversing Various String Concealing Techniques](http://SteakEnthusiast.github.io/2022/05/22/Deobfuscating-Javascript-via-AST-Manipulation-Various-String-Concealing-Techniques/)

- [Deobfuscating Javascript via AST: Converting Bracket Notation => Dot Notation for Property Accessors](http://SteakEnthusiast.github.io/2022/05/28/Deobfuscating-Javascript-via-AST-Manipulation-Converting-Bracket-Notation-Dot-Notation-for-Property-Accessors/)

- [Deobfuscating Javascript via AST: Constant Folding/Binary Expression Simplification](http://SteakEnthusiast.github.io/2022/05/28/Deobfuscating-Javascript-via-AST-Manipulation-Constant-Folding/)

- [Deobfuscating Javascript via AST: Constant Folding/Binary Expression Simplification ](http://SteakEnthusiast.github.io/2022/05/28/Deobfuscating-Javascript-via-AST-Manipulation-Constant-Folding/)

- [Deobfuscating Javascript via AST: Replacing References to Constant Variables with Their Actual Value](http://SteakEnthusiast.github.io/2022/05/31/Deobfuscating-Javascript-via-AST-Replacing-References-to-Constant-Variables-with-Their-Actual-Value/)

- [Deobfuscating Javascript via AST: Removing Dead or Unreachable Code](http://SteakEnthusiast.github.io/2022/06/04/Deobfuscating-Javascript-via-AST-Removing-Dead-or-Unreachable-Code/)


You can also view the source code for all my deobfuscation tutorial posts in [this repository](https://github.com/SteakEnthusiast/Supplementary-AST-Based-Deobfuscation-Materials/)

Okay, that's all I have for you today. I hope that this article helped you learn something new. Thanks for reading, and happy reversing!
