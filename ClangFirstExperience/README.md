# Clang 初体验

## 编译过程

在 Xcode 中以 macOS 命令行工具为模板新建一个项目，语言选择 Objective-C，创建好后自动生成 main.m 文件，其内容如下：

```objc
//
//  main.m
//  Main
//
//  Created by Janlor on 4/7/26.
//

#import <Foundation/Foundation.h>

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        // insert code here...
        NSLog(@"Hello, World!");
    }
    return EXIT_SUCCESS;
}
```

打开终端 Terminal，进入存放 main.m 的目录

```bash
cd ./ClangFirstExperience/ClangFirstExperience
```

通过 -ccc-print-phases 命令查看整个编译过程

```bash
janlor@MacBook-Pro ClangFirstExperience % clang -ccc-print-phases main.m
               +- 0: input, "main.m", objective-c
            +- 1: preprocessor, {0}, objective-c-cpp-output
         +- 2: compiler, {1}, ir
      +- 3: backend, {2}, assembler
   +- 4: assembler, {3}, object
+- 5: linker, {4}, image
6: bind-arch, "arm64", {5}, image
```

解释上述过程：

* 0 - 输入 main.m 源文件
* 1 - 预处理输出 cpp 文件
* 2 - 生成中间语言 IR
* 3 - 后端优化
* 4 - 生成汇编
* 5 - 链接
* 6 - 绑定系统架构

## 编译阶段

上面对程序的编译过程可以分为以下几个阶段，使用 Clang 指令描述：

### 1. 词法分析

```dash
janlor@MacBook-Pro ClangFirstExperience % clang -fmodules -fsyntax-only -Xclang -dump-tokens main.m
annot_module_include		Loc=<main.m:8:1>
int 'int'	 [StartOfLine]	Loc=<main.m:10:1>
identifier 'main'	 [LeadingSpace]	Loc=<main.m:10:5>
l_paren '('		Loc=<main.m:10:9>
int 'int'		Loc=<main.m:10:10>
identifier 'argc'	 [LeadingSpace]	Loc=<main.m:10:14>
comma ','		Loc=<main.m:10:18>
const 'const'	 [LeadingSpace]	Loc=<main.m:10:20>
char 'char'	 [LeadingSpace]	Loc=<main.m:10:26>
star '*'	 [LeadingSpace]	Loc=<main.m:10:31>
identifier 'argv'	 [LeadingSpace]	Loc=<main.m:10:33>
l_square '['		Loc=<main.m:10:37>
r_square ']'		Loc=<main.m:10:38>
r_paren ')'		Loc=<main.m:10:39>
l_brace '{'	 [LeadingSpace]	Loc=<main.m:10:41>
at '@'	 [StartOfLine] [LeadingSpace]	Loc=<main.m:11:5>
identifier 'autoreleasepool'		Loc=<main.m:11:6>
l_brace '{'	 [LeadingSpace]	Loc=<main.m:11:22>
identifier 'NSLog'	 [StartOfLine] [LeadingSpace]	Loc=<main.m:13:9>
l_paren '('		Loc=<main.m:13:14>
at '@'		Loc=<main.m:13:15>
string_literal '"Hello, World!"'		Loc=<main.m:13:16>
r_paren ')'		Loc=<main.m:13:31>
semi ';'		Loc=<main.m:13:32>
r_brace '}'	 [StartOfLine] [LeadingSpace]	Loc=<main.m:14:5>
return 'return'	 [StartOfLine] [LeadingSpace]	Loc=<main.m:15:5>
numeric_constant '0'	 [LeadingSpace]	Loc=<main.m:15:12 <Spelling=/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk/usr/include/_stdlib.h:122:22>>
semi ';'		Loc=<main.m:15:24>
r_brace '}'	 [StartOfLine]	Loc=<main.m:16:1>
eof ''		Loc=<main.m:16:2>
```

### 2. 语法分析并生成 AST

```cpp
janlor@MacBook-Pro ClangFirstExperience % clang -fmodules -fsyntax-only -Xclang -ast-dump main.m   
TranslationUnitDecl 0xbb508b008 <<invalid sloc>> <invalid sloc> <undeserialized declarations>
|-TypedefDecl 0xbb51f4018 <<invalid sloc>> <invalid sloc> implicit __int128_t '__int128'
| `-BuiltinType 0xbb508b5d0 '__int128'
|-TypedefDecl 0xbb51f4088 <<invalid sloc>> <invalid sloc> implicit __uint128_t 'unsigned __int128'
| `-BuiltinType 0xbb508b5f0 'unsigned __int128'
|-TypedefDecl 0xbb51f4128 <<invalid sloc>> <invalid sloc> implicit SEL 'SEL *'
| `-PointerType 0xbb51f40e0 'SEL *' imported
|   `-BuiltinType 0xbb508b790 'SEL'
|-TypedefDecl 0xbb51f4210 <<invalid sloc>> <invalid sloc> implicit id 'id'
| `-ObjCObjectPointerType 0xbb51f41b0 'id' imported
|   `-ObjCObjectType 0xbb51f4180 'id' imported
|-TypedefDecl 0xbb51f4300 <<invalid sloc>> <invalid sloc> implicit Class 'Class'
| `-ObjCObjectPointerType 0xbb51f42a0 'Class' imported
|   `-ObjCObjectType 0xbb51f4270 'Class' imported
|-ObjCInterfaceDecl 0xbb51f4358 <<invalid sloc>> <invalid sloc> implicit Protocol
|-TypedefDecl 0xbb51f46f0 <<invalid sloc>> <invalid sloc> implicit __NSConstantString 'struct __NSConstantString_tag'
| `-RecordType 0xbb51f44c0 'struct __NSConstantString_tag'
|   `-Record 0xbb51f4420 '__NSConstantString_tag'
|-TypedefDecl 0xbb51f4758 <<invalid sloc>> <invalid sloc> implicit __SVInt8_t '__SVInt8_t'
| `-BuiltinType 0xbb508b7b0 '__SVInt8_t'
|-TypedefDecl 0xbb51f47c0 <<invalid sloc>> <invalid sloc> implicit __SVInt16_t '__SVInt16_t'
| `-BuiltinType 0xbb508b7d0 '__SVInt16_t'
|-TypedefDecl 0xbb51f4828 <<invalid sloc>> <invalid sloc> implicit __SVInt32_t '__SVInt32_t'
| `-BuiltinType 0xbb508b7f0 '__SVInt32_t'
|-TypedefDecl 0xbb51f4890 <<invalid sloc>> <invalid sloc> implicit __SVInt64_t '__SVInt64_t'
| `-BuiltinType 0xbb508b810 '__SVInt64_t'
|-TypedefDecl 0xbb51f48f8 <<invalid sloc>> <invalid sloc> implicit __SVUint8_t '__SVUint8_t'
| `-BuiltinType 0xbb508b830 '__SVUint8_t'
|-TypedefDecl 0xbb51f4960 <<invalid sloc>> <invalid sloc> implicit __SVUint16_t '__SVUint16_t'
| `-BuiltinType 0xbb508b850 '__SVUint16_t'
|-TypedefDecl 0xbb51f49c8 <<invalid sloc>> <invalid sloc> implicit __SVUint32_t '__SVUint32_t'
| `-BuiltinType 0xbb508b870 '__SVUint32_t'
|-TypedefDecl 0xbb51f4a30 <<invalid sloc>> <invalid sloc> implicit __SVUint64_t '__SVUint64_t'
| `-BuiltinType 0xbb508b890 '__SVUint64_t'
|-TypedefDecl 0xbb51f4a98 <<invalid sloc>> <invalid sloc> implicit __SVFloat16_t '__SVFloat16_t'
| `-BuiltinType 0xbb508b8b0 '__SVFloat16_t'
|-TypedefDecl 0xbb51f4b00 <<invalid sloc>> <invalid sloc> implicit __SVFloat32_t '__SVFloat32_t'
| `-BuiltinType 0xbb508b8d0 '__SVFloat32_t'
|-TypedefDecl 0xbb51f4b68 <<invalid sloc>> <invalid sloc> implicit __SVFloat64_t '__SVFloat64_t'
| `-BuiltinType 0xbb508b8f0 '__SVFloat64_t'
|-TypedefDecl 0xbb51f4bd0 <<invalid sloc>> <invalid sloc> implicit __SVBfloat16_t '__SVBfloat16_t'
| `-BuiltinType 0xbb508b910 '__SVBfloat16_t'
|-TypedefDecl 0xbb51f4c38 <<invalid sloc>> <invalid sloc> implicit __clang_svint8x2_t '__clang_svint8x2_t'
| `-BuiltinType 0xbb508b930 '__clang_svint8x2_t'
|-TypedefDecl 0xbb51f4ca0 <<invalid sloc>> <invalid sloc> implicit __clang_svint16x2_t '__clang_svint16x2_t'
| `-BuiltinType 0xbb508b950 '__clang_svint16x2_t'
|-TypedefDecl 0xbb51f4d08 <<invalid sloc>> <invalid sloc> implicit __clang_svint32x2_t '__clang_svint32x2_t'
| `-BuiltinType 0xbb508b970 '__clang_svint32x2_t'
|-TypedefDecl 0xbb51f4d70 <<invalid sloc>> <invalid sloc> implicit __clang_svint64x2_t '__clang_svint64x2_t'
| `-BuiltinType 0xbb508b990 '__clang_svint64x2_t'
|-TypedefDecl 0xbb51f4dd8 <<invalid sloc>> <invalid sloc> implicit __clang_svuint8x2_t '__clang_svuint8x2_t'
| `-BuiltinType 0xbb508b9b0 '__clang_svuint8x2_t'
|-TypedefDecl 0xbb51f4e40 <<invalid sloc>> <invalid sloc> implicit __clang_svuint16x2_t '__clang_svuint16x2_t'
| `-BuiltinType 0xbb508b9d0 '__clang_svuint16x2_t'
|-TypedefDecl 0xbb51f4ea8 <<invalid sloc>> <invalid sloc> implicit __clang_svuint32x2_t '__clang_svuint32x2_t'
| `-BuiltinType 0xbb508b9f0 '__clang_svuint32x2_t'
|-TypedefDecl 0xbb51f4f10 <<invalid sloc>> <invalid sloc> implicit __clang_svuint64x2_t '__clang_svuint64x2_t'
| `-BuiltinType 0xbb508ba10 '__clang_svuint64x2_t'
|-TypedefDecl 0xbb51f4f78 <<invalid sloc>> <invalid sloc> implicit __clang_svfloat16x2_t '__clang_svfloat16x2_t'
| `-BuiltinType 0xbb508ba30 '__clang_svfloat16x2_t'
|-TypedefDecl 0xbb51f5000 <<invalid sloc>> <invalid sloc> implicit __clang_svfloat32x2_t '__clang_svfloat32x2_t'
| `-BuiltinType 0xbb508ba50 '__clang_svfloat32x2_t'
|-TypedefDecl 0xbb51f5068 <<invalid sloc>> <invalid sloc> implicit __clang_svfloat64x2_t '__clang_svfloat64x2_t'
| `-BuiltinType 0xbb508ba70 '__clang_svfloat64x2_t'
|-TypedefDecl 0xbb51f50d0 <<invalid sloc>> <invalid sloc> implicit __clang_svbfloat16x2_t '__clang_svbfloat16x2_t'
| `-BuiltinType 0xbb508ba90 '__clang_svbfloat16x2_t'
|-TypedefDecl 0xbb51f5138 <<invalid sloc>> <invalid sloc> implicit __clang_svint8x3_t '__clang_svint8x3_t'
| `-BuiltinType 0xbb508bab0 '__clang_svint8x3_t'
|-TypedefDecl 0xbb51f51a0 <<invalid sloc>> <invalid sloc> implicit __clang_svint16x3_t '__clang_svint16x3_t'
| `-BuiltinType 0xbb508bad0 '__clang_svint16x3_t'
|-TypedefDecl 0xbb51f5208 <<invalid sloc>> <invalid sloc> implicit __clang_svint32x3_t '__clang_svint32x3_t'
| `-BuiltinType 0xbb508baf0 '__clang_svint32x3_t'
|-TypedefDecl 0xbb51f5270 <<invalid sloc>> <invalid sloc> implicit __clang_svint64x3_t '__clang_svint64x3_t'
| `-BuiltinType 0xbb508bb10 '__clang_svint64x3_t'
|-TypedefDecl 0xbb51f52d8 <<invalid sloc>> <invalid sloc> implicit __clang_svuint8x3_t '__clang_svuint8x3_t'
| `-BuiltinType 0xbb508bb30 '__clang_svuint8x3_t'
|-TypedefDecl 0xbb51f5340 <<invalid sloc>> <invalid sloc> implicit __clang_svuint16x3_t '__clang_svuint16x3_t'
| `-BuiltinType 0xbb508bb50 '__clang_svuint16x3_t'
|-TypedefDecl 0xbb51f53a8 <<invalid sloc>> <invalid sloc> implicit __clang_svuint32x3_t '__clang_svuint32x3_t'
| `-BuiltinType 0xbb508bb70 '__clang_svuint32x3_t'
|-TypedefDecl 0xbb51f5410 <<invalid sloc>> <invalid sloc> implicit __clang_svuint64x3_t '__clang_svuint64x3_t'
| `-BuiltinType 0xbb508bb90 '__clang_svuint64x3_t'
|-TypedefDecl 0xbb51f5478 <<invalid sloc>> <invalid sloc> implicit __clang_svfloat16x3_t '__clang_svfloat16x3_t'
| `-BuiltinType 0xbb508bbb0 '__clang_svfloat16x3_t'
|-TypedefDecl 0xbb51f54e0 <<invalid sloc>> <invalid sloc> implicit __clang_svfloat32x3_t '__clang_svfloat32x3_t'
| `-BuiltinType 0xbb508bbd0 '__clang_svfloat32x3_t'
|-TypedefDecl 0xbb51f5548 <<invalid sloc>> <invalid sloc> implicit __clang_svfloat64x3_t '__clang_svfloat64x3_t'
| `-BuiltinType 0xbb508bbf0 '__clang_svfloat64x3_t'
|-TypedefDecl 0xbb51f55b0 <<invalid sloc>> <invalid sloc> implicit __clang_svbfloat16x3_t '__clang_svbfloat16x3_t'
| `-BuiltinType 0xbb508bc10 '__clang_svbfloat16x3_t'
|-TypedefDecl 0xbb51f5618 <<invalid sloc>> <invalid sloc> implicit __clang_svint8x4_t '__clang_svint8x4_t'
| `-BuiltinType 0xbb508bc30 '__clang_svint8x4_t'
|-TypedefDecl 0xbb51f5680 <<invalid sloc>> <invalid sloc> implicit __clang_svint16x4_t '__clang_svint16x4_t'
| `-BuiltinType 0xbb508bc50 '__clang_svint16x4_t'
|-TypedefDecl 0xbb51f56e8 <<invalid sloc>> <invalid sloc> implicit __clang_svint32x4_t '__clang_svint32x4_t'
| `-BuiltinType 0xbb508bc70 '__clang_svint32x4_t'
|-TypedefDecl 0xbb51f5750 <<invalid sloc>> <invalid sloc> implicit __clang_svint64x4_t '__clang_svint64x4_t'
| `-BuiltinType 0xbb508bc90 '__clang_svint64x4_t'
|-TypedefDecl 0xbb51f57b8 <<invalid sloc>> <invalid sloc> implicit __clang_svuint8x4_t '__clang_svuint8x4_t'
| `-BuiltinType 0xbb508bcb0 '__clang_svuint8x4_t'
|-TypedefDecl 0xbb51f5820 <<invalid sloc>> <invalid sloc> implicit __clang_svuint16x4_t '__clang_svuint16x4_t'
| `-BuiltinType 0xbb508bcd0 '__clang_svuint16x4_t'
|-TypedefDecl 0xbb51f5888 <<invalid sloc>> <invalid sloc> implicit __clang_svuint32x4_t '__clang_svuint32x4_t'
| `-BuiltinType 0xbb508bcf0 '__clang_svuint32x4_t'
|-TypedefDecl 0xbb51f58f0 <<invalid sloc>> <invalid sloc> implicit __clang_svuint64x4_t '__clang_svuint64x4_t'
| `-BuiltinType 0xbb508bd10 '__clang_svuint64x4_t'
|-TypedefDecl 0xbb51f5958 <<invalid sloc>> <invalid sloc> implicit __clang_svfloat16x4_t '__clang_svfloat16x4_t'
| `-BuiltinType 0xbb508bd30 '__clang_svfloat16x4_t'
|-TypedefDecl 0xbb51f59c0 <<invalid sloc>> <invalid sloc> implicit __clang_svfloat32x4_t '__clang_svfloat32x4_t'
| `-BuiltinType 0xbb508bd50 '__clang_svfloat32x4_t'
|-TypedefDecl 0xbb51f5a28 <<invalid sloc>> <invalid sloc> implicit __clang_svfloat64x4_t '__clang_svfloat64x4_t'
| `-BuiltinType 0xbb508bd70 '__clang_svfloat64x4_t'
|-TypedefDecl 0xbb51f5a90 <<invalid sloc>> <invalid sloc> implicit __clang_svbfloat16x4_t '__clang_svbfloat16x4_t'
| `-BuiltinType 0xbb508bd90 '__clang_svbfloat16x4_t'
|-TypedefDecl 0xbb51f5af8 <<invalid sloc>> <invalid sloc> implicit __SVBool_t '__SVBool_t'
| `-BuiltinType 0xbb508bdb0 '__SVBool_t'
|-TypedefDecl 0xbb51f5b60 <<invalid sloc>> <invalid sloc> implicit __clang_svboolx2_t '__clang_svboolx2_t'
| `-BuiltinType 0xbb508bdd0 '__clang_svboolx2_t'
|-TypedefDecl 0xbb51f5bc8 <<invalid sloc>> <invalid sloc> implicit __clang_svboolx4_t '__clang_svboolx4_t'
| `-BuiltinType 0xbb508bdf0 '__clang_svboolx4_t'
|-TypedefDecl 0xbb51f5c30 <<invalid sloc>> <invalid sloc> implicit __SVCount_t '__SVCount_t'
| `-BuiltinType 0xbb508be10 '__SVCount_t'
|-TypedefDecl 0xbb508bf28 <<invalid sloc>> <invalid sloc> implicit __builtin_ms_va_list 'char *'
| `-PointerType 0xbb508bee0 'char *'
|   `-BuiltinType 0xbb508b0b0 'char'
|-TypedefDecl 0xbb508bf98 <<invalid sloc>> <invalid sloc> implicit __builtin_va_list 'char *'
| `-PointerType 0xbb508bee0 'char *'
|   `-BuiltinType 0xbb508b0b0 'char'
|-ImportDecl 0xbb5693ab0 <main.m:8:1> col:1 implicit Foundation
`-FunctionDecl 0xbb5693d88 <line:10:1, line:16:1> line:10:5 main 'int (int, const char **)'
  |-ParmVarDecl 0xbb5693b08 <col:10, col:14> col:14 argc 'int'
  |-ParmVarDecl 0xbb5693c30 <col:20, col:38> col:33 argv 'const char **'
  `-CompoundStmt 0xbb580ace8 <col:41, line:16:1>
    |-ObjCAutoreleasePoolStmt 0xbb580aca0 <line:11:5, line:14:5>
    | `-CompoundStmt 0xbb580ac88 <line:11:22, line:14:5>
    |   `-CallExpr 0xbb580ac48 <line:13:9, col:31> 'void'
    |     |-ImplicitCastExpr 0xbb580ac30 <col:9> 'void (*)(id, ...)' <FunctionToPointerDecay>
    |     | `-DeclRefExpr 0xbb580ab30 <col:9> 'void (id, ...)' Function 0xbb5693ed8 'NSLog' 'void (id, ...)'
    |     `-ImplicitCastExpr 0xbb580ac70 <col:15, col:16> 'id' <BitCast>
    |       `-ObjCStringLiteral 0xbb580aba8 <col:15, col:16> 'NSString *'
    |         `-StringLiteral 0xbb580ab80 <col:16> 'char[14]' lvalue "Hello, World!"
    `-ReturnStmt 0xbb580acd8 <line:15:5, /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk/usr/include/_stdlib.h:122:22>
      `-IntegerLiteral 0xbb580acb8 <col:22> 'int' 0
```

### 3. 生成中间代码文件 main.ll

```dash
janlor@MacBook-Pro ClangFirstExperience % clang -S -fobjc-arc -emit-llvm main.m -o main-3.ll
```

[生成的 main.ll 文件](./ClangFirstExperience/main-3.ll)

### 4. 编译优化 main.ll 文件

```dash
janlor@MacBook-Pro ClangFirstExperience % clang -O3 -S -fobjc-arc -emit-llvm main.m -o main.ll
```

[生成的 main.ll 文件](./ClangFirstExperience/main.ll)

### 5. 如果开启了 bitcode，那么再进一步优化，生成 bitcode 文件 main.bc

```dash
janlor@MacBook-Pro ClangFirstExperience % clang -emit-llvm -c main.m -o main.bc
```

[生成的 main.bc 文件](./ClangFirstExperience/main.bc)

### 6. 生成汇编语言 main.s

```dash
janlor@MacBook-Pro ClangFirstExperience % clang -S -fobjc-arc main.m -o main.s
```

[生成的 main.s 文件](./ClangFirstExperience/main.s)

### 7. 生成目标代码文件 main.o

```dash
janlor@MacBook-Pro ClangFirstExperience % clang -fmodules -c main.m -o main.o 
```

[生成的 main.o 文件](./ClangFirstExperience/main.o)

### 8. 生成可执行的二进制文件 main

```dash
janlor@MacBook-Pro ClangFirstExperience % clang main.o -o main
```

[生成的 main 文件](./ClangFirstExperience/main)

### 9. 执行 main 文件

```dash
janlor@MacBook-Pro ClangFirstExperience % ./main
2026-04-07 14:43:18.221 main[2242:151082] Hello, World!
```

## 常用指令集

| 指令  | 说明 |
|:---------------:| :-------------|
| -dump-tokens |   运行预处理器，拆分内部代码段为各种 token |
| -ast-dump       |   构建抽象语法树 AST，然后对其进行拆解和调试 |
| -emit-llvm       |    使用 LLVM 描述汇编和对象文件 |
| -fmodules | 允许 modules 的语言特性 |
| -fsyntax-only | 防止编译器生成代码，只是语法级别的说明和修改 | 
| -fobjc-arc | 在 ARC 环境下，为 Objective-C 对象生成 retain 和 release 的调用 |
| -fno-objc-arc | 在 MRC 环境下使用 |
| -framework | 添加库 |
| -rewrite-objc | 将 Objective-C 源码重写为 C++ |
| -Xclang | 向 Clang 编译器传递参数 |
| -c | 只运行预处理，编译和汇编步骤 |
| -C | 在预处理输出中包含注释 |
| -g | 在可执行程序中包含标准调试信息，用于 gdb 调试 |
| -l | 在头文件的搜索路径列表中添加 dir 目录 |
| -L | 在库文件的搜索路径中添加 dir 目录 |
| -o | 输出 .o 目标文件 |
| -S | 生成 .s 汇编代码文件 |
