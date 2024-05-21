# 资料链接

1. CodeQL官方使用说明 		[CodeQL overview — CodeQL (github.com)](https://codeql.github.com/docs/codeql-overview/)

   1.1 CLI

   CodeQL CLI 安装[Setting up the CodeQL CLI - GitHub Docs](https://docs.github.com/en/code-security/codeql-cli/getting-started-with-the-codeql-cli/setting-up-the-codeql-cli)

   创建db[Preparing your code for CodeQL analysis - GitHub Docs](https://docs.github.com/en/code-security/codeql-cli/getting-started-with-the-codeql-cli/preparing-your-code-for-codeql-analysis)

   查询分析[Analyzing your code with CodeQL queries - GitHub Docs](https://docs.github.com/en/code-security/codeql-cli/getting-started-with-the-codeql-cli/analyzing-your-code-with-codeql-queries)

   1.2 查询语句

   指南[CodeQL language guides — CodeQL (github.com)](https://codeql.github.com/docs/codeql-language-guides/)

   库[CodeQL standard libraries (github.com)](https://codeql.github.com/codeql-standard-libraries/)

   1.3 代码示例

   go仓库 [codeql/go/ql/examples/snippets at main · github/codeql](https://github.com/github/codeql/tree/main/go/ql/examples/snippets)

   还有其他语言示例
2. 一些学习链接

[SummerSec/learning-codeql: CodeQL Java 全网最全的中文学习资料 (github.com)](https://github.com/SummerSec/learning-codeql)

[使用 CodeQL 分析闭源 Java 程序 (seebug.org)](https://paper.seebug.org/1324/)

# 方法流程示例

```
# auto.sh
# 代码项目目录下

# 创建数据库
codeql database create db --language=python

# 创建ql文件夹
mkdir ql

# 在ql文件夹中创建qlpack.yml文件
echo "name: dyl/myquery
version: 0.0.0
dependencies: 
    codeql/python-all: \"*\"
    codeql/python-queries: \"*\"" > ql/qlpack.yml

# 在ql文件夹中创建test.ql文件
echo "
/**
 * Query metadata
 *
 * @id python/test
 * @kind problem
 * @severity error
 */


import python

from
  Value val, CallNode call, string call_location, string caller_scope_location,
  string callee_definition_location
where
  call = val.getACall() and
  (
    callee_definition_location = val.(CallableValue).getScope().getLocation().toString()
    or
    callee_definition_location = val.(ClassValue).getScope().getLocation().toString()
  ) and
  caller_scope_location = call.getScope().getLocation().toString() and
  call_location = call.getLocation().toString()
select call_location, caller_scope_location, callee_definition_location" > ql/test.ql


# codeql database analyze db /ql/test.ql --format=csv --output=results/results.csv

codeql query run ql/test.ql -d db


echo "completed!
```

![1699938336589](image/CodeQL_/1699938336589.png)

![1699938304015](image/CodeQL_/1699938304015.png)

# 现有且已测试过的查询代码

![1699935757245](image/CodeQL_/1699935757245.png)

现有代码是东拼西凑的，需要改进

## cpp

```
import cpp

from
  FunctionCall call, Function function, string call_location, //, string caller_scope_location
  string callee_definition_location
where
  call.getTarget() = function and
  // (
  //   function.hasName("add") or
  //   function.hasName("multiply")
  // ) and
  //function.getLocation().getFilePath().startsWith("file:///dyl/c-test1/")
  //caller_scope_location = call.getParentScope().getLocation().toString() and
  callee_definition_location = function.getLocation().toString() and
  call_location = call.getLocation().toString() //and
select call_location, call, function as function_name, callee_definition_location
```

不足?：会显示调用的库函数

![1699939204006](image/CodeQL_/1699939204006.png)

## python

```
/**
 * Query metadata
 *
 * @id python/test
 * @kind problem
 * @severity error
 */


import python

from
  Value val, CallNode call, string call_location, string caller_scope_location,
  string callee_definition_location
where
  call = val.getACall() and
  (
    callee_definition_location = val.(CallableValue).getScope().getLocation().toString()
    or
    callee_definition_location = val.(ClassValue).getScope().getLocation().toString()
  ) and
  caller_scope_location = call.getScope().getLocation().toString() and
  call_location = call.getLocation().toString()
select call_location, caller_scope_location, callee_definition_location"
```

![1699940051315](image/CodeQL_/1699940051315.png)

## js

```
import javascript
import semmle.javascript.explore.CallGraph
import DataFlow

from InvokeNode invoke, FunctionNode function //, string invoke_location
where
  callEdge*(invoke, function) and
  isStartOfCallPath(invoke) //and
//invoke_location = invoke.getLocation().toString()
select invoke, function //, invoke_location
```

![1699941789063](image/CodeQL_/1699941789063.png)

还不知道用什么方式getlocation显示位置

## Golang

```
import go

from Function function, string call_location, Location defi_location
where
  call_location = function.getACall().getStartLine().toString() and
  defi_location = function.getDeclaration().getLocation()
select function as function_name, call_location, defi_location
```

![1699943523290](image/CodeQL_/1699943523290.png)

# 其他事项

在qlpack.yml中需要填的

![1699935790180](image/CodeQL_/1699935790180.png)

# todo

不同语言查询语句不同，需要学习

query run 和 analyze的区别

query metadata是什么

# tips

调试query代码用vscode插件更快
