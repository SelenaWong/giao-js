## 前言

> 在这篇文章中,我们将通过 JS 构建我们自己的 JS 解释器,用 JS 写 JS,这听起来很奇怪,尽管如此,这样做我们将更熟悉 JS,也可以学习 JS 引擎是如何工作的!

## 什么是解释器 (Interpreter) ?

> 解释器是在运行时运行的语言求值器，它动态地执行程序的源代码。它不同于编译器。编译器将语言源代码翻译成机器代码。
> 解释器解析源代码，从源代码生成 AST(抽象语法树)，遍历 AST 并逐个计算它们。

## 解释器 (Interpreter) 工作原理

![Interpreter](./assets/giao.png)

- 词法分析 (Tokenization)

- 语法解析 (Parsing)

- 求值 (Evaluating)

### 词法分析 (Tokenization)

> 将源代码分解并组织成一组有意义的单词,这一过程即为词法分析(Token)。

在英语中,当我们遇到这样一个语句时:

```js
Javascript is the best language in the world
```

我们会下意识地把句子分解成一个个单词:

```js
+----------------------------------------------------------+
| Javascript | is | the | best | language | in |the |world |
+----------------------------------------------------------+
```

这是分析和理解句子的第一阶段。

词法分析是由**词法分析器（Lexical Analyser）**完成的,词法分析器会扫描（scanning）代码,提取词法单元。

```js
var a = 1;

[
  ("var": "keyword"),
  ("a": "identifier"),
  ("=": "operator"),
  ("1": "literal"),
  (";": "separator"),
];
```

词法分析器将代码分解成 Token 后,会将 Token 传递给解析器进行解析,我们来看下解析阶段是如何工作的。

### 语法解析 (Parsing)

> 将词法分析阶段生成的 Token 转换为抽象语法树(Abstract Syntax Tree),这一过程称之为语法解析(Parsing)。

在英语中,Javascript is the best language 被分解为以下单词:

```js
+------------------------------------------+
| Javascript | is | the | best | language  |
+------------------------------------------+
```

这样我们就可以挑选单词并形成语法结构:

```js
"Javascript": Subject
"is the best language": Predicate
"language": Object
```

Javascript 在语法中是一个主语名词,其余的是一个没有什么意义的句子叫做谓语,language 是动作的接受者,也就是宾语。结构是这样的:

```js
Subject(Noun) -> Predicate -> Object
```

语法解析是由**语法解析器 (Syntax Parser)**完成的,它会将上一步生成的 Token,根据语法规则,转为抽象语法树(AST)。

```js
{
  type: "Program",
  body: [
    {
      type: "VariableDeclaration",
      declarations: [
        {
          type: "VariableDeclarator",
          id: {
            type: "Identifier",
            name: "sum"
          },
          init: {
            type: "Literal",
            value: 30,
            raw: "30"
          }
        }
      ],
      kind: "var"
    }
  ],
}

```

### 求值阶段 (Evaluating)

> 解释器将遍历 AST 并计算每个节点。- 求值阶段

```js
1 + 2
|
    |
    v
+---+  +---+
| 1 |  | 2 |
+---+  +---+
  \     /
   \   /
    \ /
   +---+
   | + |
   +---+
{
    lhs: 1,
    op: '+'.
    rhs: 2
}
```

解释器解析 Ast,得到 LHS 节点,接着收集到操作符(operator)节点+,+操作符表示需要进行一次加法操作,它必须有第二个节点来进行加法操作.接着他收集到 RHS 节点。它收集到了有价值的信息并执行加法得到了结果,3。

```js
{
  type: "Program",
  body: [
    {
      type: "ExpressionStatement",
      expression: {
        type: "BinaryExpression",
        left: {
          type: "Literal",
          value: 1,
          raw: "1"
        },
        operator: "+",
        right: {
          type: "Literal",
          value: 2,
          raw: "2"
        }
      }
    }
  ],
}
```

## 实践

前面我们已经介绍了解释器的工作原理,接下来我们来动动手松松筋骨吧,实现一个 Mini Js Interpreter~

### 实践准备

- Acorn.js

> A tiny, fast JavaScript parser, written completely in JavaScript. 一个完全使用 javascript 实现的，小型且快速的 javascript 解析器

本次实践我们将使用 acorn.js ,它会帮我们进行词法分析,语法解析并转换为抽象语法树。

Webpack/Rollup/Babel(@babel/parser) 等第三方库也是使用 acorn.js 作为自己 Parser 的基础库。(站在巨人的肩膀上啊!)

- The Estree Spec

最开始 [Mozilla JS Parser API](https://developer.mozilla.org/en-US/docs/Mozilla/Projects/SpiderMonkey/Parser_API) 是 Mozilla 工程师在 Firefox 中创建的 SpiderMonkey 引擎输出 JavaScript AST 的规范文档，文档所描述的格式被用作操作 JAvaScript 源代码的通用语言。

随着 JavaScript 的发展，更多新的语法被加入，为了帮助发展这种格式以跟上 JavaScript 语言的发展。[The ESTree Spec](https://github.com/estree/estree) 就诞生了，作为参与构建和使用这些工具的人员的社区标准。

acorn.js parse 返回值符合 ESTree spec 描述的 AST 对象,这里我们使用@types/estree 做类型定义。

- Jest

号称令人愉快的 JavaScript 测试...我们使用它来进行单元测试.

- Rollup

Rollup 是一个 JavaScript 模块打包器,我们使用它来打包,以 UMD 规范对外暴露模块。

### 项目初始化

```js
// visitor.ts 创建一个Visitor类,并提供一个方法操作ES节点。
import * as ESTree from "estree";
class Visitor {
  visitNode(node: ESTree.Node) {
    // ...
  }
}
export default Visitor;
```

```js
// interpreter.ts 创建一个Interpreter类,用于运行ES节点树。
// 创建一个Visitor实例,并使用该实例来运行ESTree节点
import Visitor from "./visitor";
import * as ESTree from "estree";
class Interpreter {
  private visitor: Visitor;
  constructor(visitor: Visitor) {
    this.visitor = visitor;
  }
  interpret(node: ESTree.Node) {
    this.visitor.visitNode(node);
  }
}
export default Interpreter;
```

```js
// vm.ts 对外暴露run方法,并使用acorn code->ast后,交给Interpreter实例进行解释。
const acorn = require("acorn");
import Visitor from "./visitor";
import Interpreter from "./interpreter";

const jsInterpreter = new Interpreter(new Visitor());

export function run(code: string) {
  const root = acorn.parse(code, {
    ecmaVersion: 8,
    sourceType: "script",
  });
  return jsInterpreter.interpret(root);
}
```

### 1+1= ？

我们这节来实现 1+1 加法的解释。首先我们通过[AST explorer](https://astexplorer.net/),看看 1+1 这段代码转换后的 AST 结构。

![1+1 ast](./assets/1+1.png)

我们可以看到这段代码中存在 4 种节点类型,下面我们简单的介绍一下它们:

#### Program

根节点,即代表一整颗抽象语法树,body 属性是一个数组，包含了多个 Statement 节点。

```ts
interface Program {
  type: "Program";
  sourceType: "script" | "module";
  body: Array<Directive | Statement | ModuleDeclaration>;
  comments?: Array<Comment>;
}
```

#### ExpressionStatement

表达式语句节点,expression 属性指向一个表达式节点对象

```ts
interface ExpressionStatement {
  type: "ExpressionStatement";
  expression: Expression;
}
```

#### BinaryExpression

二元运算表达式节点，left 和 right 表示运算符左右的两个表达式，operator 表示一个二元运算符。
本节实现的重点,简单理解,我们只要拿到 operator 操作符的类型并实现,然后对 left,right 值进行求值即可。

```ts
interface BinaryExpression {
  type: "BinaryExpression";
  operator: BinaryOperator;
  left: Expression;
  right: Expression;
}
```

#### Literal

字面量，这里不是指 [] 或者 {} 这些，而是本身语义就代表了一个值的字面量，如 1，“hello”, true 这些，还有正则表达式，如 /\d?/。

```ts
type Literal = SimpleLiteral | RegExpLiteral;

interface SimpleLiteral {
  type: "Literal";
  value: string | boolean | number | null;
  raw?: string;
}

interface RegExpLiteral {
  type: "Literal";
  value?: RegExp | null;
  regex: {
    pattern: string;
    flags: string;
  };
  raw?: string;
}
```

废话少说,开撸!!!

```ts
// standard/es5.ts 实现以上节点方法

import Scope from "../scope";
import * as ESTree from "estree";
import { AstPath } from "../types/index";

const es5 = {
  // 根节点的处理很简单,我们只要对它的body属性进行遍历,然后访问该节点即可。
  Program(node: ESTree.Program) {
    node.body.forEach((bodyNode) => this.visitNode(bodyNode));
  },
  // 表达式语句节点的处理,同样访问expression 属性即可。
  ExpressionStatement(node: ESTree.ExpressionStatement>) {
    return this.visitNode(node.expression);
  },
  // 字面量节点处理直接求值,这里对正则表达式类型进行了特殊处理,其他类型直接返回value值即可。
  Literal(node: ESTree.Literal>) {
    if ((<ESTree.RegExpLiteral>node).regex) {
      const { pattern, flags } = (<ESTree.RegExpLiteral>node).regex;
      return new RegExp(pattern, flags);
    } else return node.value;
  },
  // 二元运算表达式节点处理
  // 对left/node两个节点(Literal)进行求值,然后实现operator类型运算,返回结果。
  BinaryExpression(node: ESTree.BinaryExpression>) {
    const leftNode = this.visitNode(node.left);
    const operator = node.operator;
    const rightNode = this.visitNode(node.right);
    return {
      "+": (l, r) => l + r,
      "-": (l, r) => l - r,
      "*": (l, r) => l * r,
      "/": (l, r) => l / r,
      "%": (l, r) => l % r,
      "<": (l, r) => l < r,
      ">": (l, r) => l > r,
      "<=": (l, r) => l <= r,
      ">=": (l, r) => l >= r,
      "==": (l, r) => l == r,
      "===": (l, r) => l === r,
      "!=": (l, r) => l != r,
      "!==": (l, r) => l !== r,
    }[operator](leftNode, rightNode);
  },
};
export default es5;
```

```js
// visitor.ts
import Scope from "./scope";
import * as ESTree from "estree";
import es5 from "./standard/es5";

const VISITOR = {
  ...es5,
};
class Visitor {
  // 实现访问节点方法,通过节点类型访问对应的节点方法
  visitNode(node: ESTree.Node) {
    return {
      visitNode: this.visitNode,
      ...VISITOR,
    }[node.type](node);
  }
}
export default Visitor;
```

就这样,普通的二元运算就搞定啦!!!