[![Build Status](https://travis-ci.org/davelondon/jennifer.svg?branch=master)](https://travis-ci.org/davelondon/jennifer) [![Go Report Card](https://goreportcard.com/badge/github.com/davelondon/jennifer)](https://goreportcard.com/report/github.com/davelondon/jennifer) [![codecov](https://codecov.io/gh/davelondon/jennifer/branch/master/graph/badge.svg)](https://codecov.io/gh/davelondon/jennifer)

# Jennifer
Jennifer is a code generator for Go.

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-NewFile):
```go
package main

import (
    "fmt"

    . "github.com/davelondon/jennifer/jen"
)

func main() {
	f := NewFile("main")
	f.Func().Id("main").Params().Block(
		Qual("fmt", "Println").Call(
			Lit("Hello, world"),
		),
	)
	fmt.Printf("%#v", f)
}
```
Output:
```go
package main

import fmt "fmt"

func main() {
	fmt.Println("Hello, world")
}
```

# Install
```
go get -u github.com/davelondon/jennifer/jen
```

# Examples
Jennifer has a comprahensive suite of examples - [see godoc.org](https://godoc.org/github.com/davelondon/jennifer/jen#pkg-examples) for an index.

The code that powers jennifer is generated by jennifer itself, see the 
[genjen package](https://github.com/davelondon/jennifer/tree/master/genjen) - 
it uses data in [data.go](https://github.com/davelondon/jennifer/blob/master/genjen/data.go), which is processed by [render.go](https://github.com/davelondon/jennifer/blob/master/genjen/render.go) to create [generated.go](https://github.com/davelondon/jennifer/blob/master/jen/generated.go).

A much larger implementation of jennifer [can be found in the kego project](https://github.com/kego/ke/blob/master/process/generate/structs.go).

# Rendering
For testing, a File or Statement can be rendered with the fmt package 
using the %#v verb.

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-Call-fmt):
```go
c := Id("a").Call(Lit("b"))
fmt.Printf("%#v", c)
// Output:
// a("b")
```

This is not recommended for use in production because any error will cause a 
panic. For production use, [File.Render](#render) or [File.Save](#save) are 
preferred.

# Id
Id renders an identifier.

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-Id):
```go
c := If(Id("i").Op("==").Id("j")).Block(
	Return(Id("i")),
)
fmt.Printf("%#v", c)
// Output:
// if i == j {
// 	return i
// }
```

# Qual
Qual renders a qualified identifier.

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-Qual):
```go
c := Qual("encoding/gob", "NewEncoder").Call()
fmt.Printf("%#v", c)
// Output:
// gob.NewEncoder()
```

 Imports are automatically added when
used with a File. If the path matches the local path, the package name is
omitted. If package names conflict they are automatically renamed.

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-Qual-file):
```go
f := NewFilePath("a.b/c")
f.Func().Id("init").Params().Block(
	Qual("a.b/c", "Foo").Call().Comment("Local package - name is omitted."),
	Qual("d.e/f", "Bar").Call().Comment("Import is automatically added."),
	Qual("g.h/f", "Baz").Call().Comment("Colliding package name is renamed."),
)
fmt.Printf("%#v", f)
// Output:
// package c
// 
// import (
// 	f "d.e/f"
// 	f1 "g.h/f"
// )
// 
// func init() {
// 	Foo()    // Local package - name is omitted.
// 	f.Bar()  // Import is automatically added.
// 	f1.Baz() // Colliding package name is renamed.
// }
```

# Op
Op renders the provided operator / token.

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-Op):
```go
c := Id("a").Op(":=").Id("b").Call()
fmt.Printf("%#v", c)
// Output:
// a := b()
```

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-Op-star):
```go
c := Id("a").Op("=").Op("*").Id("b")
fmt.Printf("%#v", c)
// Output:
// a = *b
```

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-Op-variadic):
```go
c := Id("a").Call(Id("b").Op("..."))
fmt.Printf("%#v", c)
// Output:
// a(b...)
```

# Summary
Many of the language constructs that jennifer emits are presented as functions 
taking zero or more items as parameters. For example, here the Append function 
takes two items and renders them appropriately:

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-Append-more):
```go
c := Id("a").Op("=").Append(Id("a"), Id("b").Op("..."))
fmt.Printf("%#v", c)
// Output:
// a = append(a, b...)
```

Below we summarize most of the language constructs, while examples follow.

Language constructs taking zero items:

| Construct       | Name |
| --------------- | ---- |
| Keywords        | Break, Default, Func, Select, Chan, Else, Const, Fallthrough, Type, Continue, Var, Goto, Defer, Go, Range |
| Types           | Bool, Byte, Complex64, Complex128, Error, Float32, Float64, Int, Int8, Int16, Int32, Int64, Rune, String, Uint, Uint8, Uint16, Uint32, Uint64, Uintptr |
| Constants       | True, False, Iota, Nil |
| Helpers         | Err |

Built-in functions taking one or more items:

| Construct | Name |
| --------- | ---- |
| Functions | Append, Cap, Close, Complex, Copy, Delete, Imag, Len, Make, New, Panic, Print, Println, Real, Recover |

Some keywords are always followed by another construct. These take one or more 
items and render them as follows:
 
| Keyword                          | Opening       | Separator | Closing | Usage                                   |
| -------------------------------- | ------------- | --------- | ------- | --------------------------------------- |
| [Return](#return)                |               | `,`       |         | `return a, b`                           |
| [If](#if-for)                    |               | `;`       |         | `if i, err := a(); err != nil { ... }`  |
| [For](#if-for)                   |               | `;`       |         | `for i := 0; i < 10; i++ { ... }`       |
| [Switch](#switch-case-caseblock) |               | `;`       |         | `switch a { ... }`                      |
| [Case](#switch-case-caseblock)   |               | `,`       |         | `case a, b: ...`                        |
| [Interface](#interface-struct)   | `{`           | `\n`      | `}`     | `interface { ... }`                     |
| [Struct](#interface-struct)      | `{`           | `\n`      | `}`     | `struct { ... }`                        |
| [Map](#map)                      | `[`           |           | `]`     | `map[string]`                           |

Groups accept a list of items and render them as follows:

| Group             | Opening       | Separator | Closing | Usage                             |
| ----------------- | ------------- | --------- | ------- | --------------------------------- |
| [Sel](#sel)       |               | `.`       |         | `foo.bar[0].baz()`                |
| [List](#list)     |               | `,`       |         | `a, b := c()`                     |
| [Call](#call)     | `(`           | `,`       | `)`     | `fmt.Println(b, c)`               |
| [Params](#params) | `(`           | `,`       | `)`     | `func (a *A) Foo(i int) { ... }`  |
| [Index](#index)   | `[`           | `:`       | `]`     | `a[1:2]` or `[]int{}`             |
| [Values](#values) | `{`           | `,`       | `}`     | `[]int{1, 2}`                     |
| [Block](#block)   | `{`           | `\n`      | `}`     | `func a() { ... }`                |
| [Defs](#defs)     | `(`           | `\n`      | `)`     | `const ( ... )`                   |
| [CaseBlock](#switch-case-caseblock) | `:`           | `\n`      |         | `case a: ...`                     |

These groups accept a single item:

| Group             | Opening  | Closing | Usage                        |
| ----------------- | -------- | ------- | ---------------------------- |
| [Parens](#parens) | `(`      | `)`     | `[]byte(s)` or `a / (b + c)` |
| [Assert](#assert) | `.(`     | `)`     | `s, ok := i.(string)`        |

# GroupFunc methods
All constructs that accept a variadic list of items are paired with GroupFunc 
functions that accept a func(*Group). These are used to embed logic.

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-BlockFunc):
```go
increment := true
c := Func().Id("a").Params().BlockFunc(func(g *Group) {
	if increment {
		g.Id("a").Op("++")
	} else {
		g.Id("a").Op("--")
	}
})
fmt.Printf("%#v", c)
// Output:
// func a() {
// 	a++
// }
```

# Special keywords

### Interface, Struct
Interface and Struct render the keyword followed by a statement list enclosed 
by curly braces.

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-Interface-empty):
```go
c := Var().Id("a").Interface()
fmt.Printf("%#v", c)
// Output:
// var a interface{}
```

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-Interface):
```go
c := Type().Id("a").Interface(
	Id("b").Params().String(),
)
fmt.Printf("%#v", c)
// Output:
// type a interface {
// 	b() string
// }
```

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-Struct-empty):
```go
c := Id("c").Op(":=").Make(Chan().Struct())
fmt.Printf("%#v", c)
// Output:
// c := make(chan struct{})
```

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-Struct):
```go
c := Type().Id("foo").Struct(
	List(Id("x"), Id("y")).Int(),
	Id("u").Float32(),
)
fmt.Printf("%#v", c)
// Output:
// type foo struct {
// 	x, y int
// 	u    float32
// }
```

### Switch, Case, CaseBlock
Switch, Case and CaseBlock are used to build switch statements:

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-Switch):
```go
c := Switch(Id("a")).Block(
	Case(Lit("1")).CaseBlock(
		Return(Lit(1)),
	),
	Case(Lit("2"), Lit("3")).CaseBlock(
		Return(Lit(2)),
	),
	Case(Lit("4")).CaseBlock(
		Fallthrough(),
	),
	Default().CaseBlock(
		Return(Lit(3)),
	),
)
fmt.Printf("%#v", c)
// Output:
// switch a {
// case "1":
// 	return 1
// case "2", "3":
// 	return 2
// case "4":
// 	fallthrough
// default:
// 	return 3
// }
```

### Map
Map renders the keyword followed by a single item enclosed by square brackets. Use for map definitions.

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-Map):
```go
c := Id("a").Op(":=").Map(String()).String().Values()
fmt.Printf("%#v", c)
// Output:
// a := map[string]string{}
```

### Return
Return renders the keyword followed by a comma separated list.

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-Return):
```go
c := Return(Id("a"), Id("b"))
fmt.Printf("%#v", c)
// Output:
// return a, b
```

### If, For
If and For render the keyword followed by a semicolon separated list.

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-If):
```go
c := If(
	Err().Op(":=").Id("a").Call(),
	Err().Op("!=").Nil(),
).Block(
	Return(Err()),
)
fmt.Printf("%#v", c)
// Output:
// if err := a(); err != nil {
// 	return err
// }
```

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-For):
```go
c := For(
	Id("i").Op(":=").Lit(0),
	Id("i").Op("<").Lit(10),
	Id("i").Op("++"),
).Block(
	Qual("fmt", "Println").Call(Id("i")),
)
fmt.Printf("%#v", c)
// Output:
// for i := 0; i < 10; i++ {
// 	fmt.Println(i)
// }
```

# Groups

### Sel
Sel renders a period sep arated list. Use for a chain of selectors.

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-Sel):
```go
c := Sel(Qual("a.b/c", "Foo").Call(), Id("Bar").Index(Lit(0)), Id("Baz"))
fmt.Printf("%#v", c)
// Output:
// c.Foo().Bar[0].Baz
```

### List
List renders a comma separated list. Use for multiple return functions.

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-List):
```go
c := List(Id("a"), Id("b")).Op(":=").Id("c").Call()
fmt.Printf("%#v", c)
// Output:
// a, b := c()
```

### Call
Call renders a comma separated list enclosed by parenthesis. Use for function calls.

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-Call):
```go
c := Qual("fmt", "Printf").Call(
	Lit("%#v: %T\n"),
	Id("a"),
	Id("b"),
)
fmt.Printf("%#v", c)
// Output:
// fmt.Printf("%#v: %T\n", a, b)
```

### Params
Params renders a comma separated list enclosed by parenthesis. Use for function parameters and method receivers.

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-Params):
```go
c := Func().Params(Id("a").Id("A")).Id("foo").Params(Id("b").String()).String().Block(
	Return(Id("b")),
)
fmt.Printf("%#v", c)
// Output:
// func (a A) foo(b string) string {
// 	return b
// }
```

### Index
Index renders a colon separated list enclosed by square brackets. Use for array / slice indexes and definitions.

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-Index):
```go
c := Var().Id("a").Index().String()
fmt.Printf("%#v", c)
// Output:
// var a []string
```

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-Index-index):
```go
c := Id("a").Op(":=").Id("b").Index(Lit(0), Lit(1))
fmt.Printf("%#v", c)
// Output:
// a := b[0:1]
```

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-Index-empty):
```go
c := Id("a").Op(":=").Id("b").Index(Lit(1), Empty())
fmt.Printf("%#v", c)
// Output:
// a := b[1:]
```

### Values
Values renders a comma separated list enclosed by curly braces. Use for slice literals.

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-Values):
```go
c := Index().String().Values(Lit("a"), Lit("b"))
fmt.Printf("%#v", c)
// Output:
// []string{"a", "b"}
```

### Block
Block renders a statement list enclosed by curly braces. Use for code blocks.

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-Block):
```go
c := Func().Id("foo").Params().Block(
	Id("a").Op("=").Id("b"),
)
fmt.Printf("%#v", c)
// Output:
// func foo() {
// 	a = b
// }
```

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-Block-if):
```go
c := If(Id("a").Op(">").Lit(10)).Block(
	Id("a").Op("=").Id("a").Op("/").Lit(2),
)
fmt.Printf("%#v", c)
// Output:
// if a > 10 {
// 	a = a / 2
// }
```

### Defs
Defs renders a statement list enclosed in parenthesis. Use for definition lists.

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-Defs):
```go
c := Const().Defs(
	Id("a").Op("=").Lit("a"),
	Id("b").Op("=").Lit("b"),
)
fmt.Printf("%#v", c)
// Output:
// const (
// 	a = "a"
// 	b = "b"
// )
```

### Parens
Parens renders a single item in parenthesis. Use for type conversion or to specify evaluation order.

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-Parens):
```go
c := Id("b").Op(":=").Index().Byte().Parens(Id("s"))
fmt.Printf("%#v", c)
// Output:
// b := []byte(s)
```

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-Parens-order):
```go
c := Id("a").Op("/").Parens(Id("b").Op("+").Id("c"))
fmt.Printf("%#v", c)
// Output:
// a / (b + c)
```

### Assert
Assert renders a period followed by a single item enclosed by parenthesis. Use for type assertions.

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-Assert):
```go
c := List(Id("b"), Id("ok")).Op(":=").Id("a").Assert(Bool())
fmt.Printf("%#v", c)
// Output:
// b, ok := a.(bool)
```

# Add
Add appends the provided items to the statement.

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-Add):
```go
ptr := Op("*")
c := Id("a").Op("=").Add(ptr).Id("b")
fmt.Printf("%#v", c)
// Output:
// a = *b
```

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-Add-var):
```go
a := Id("a")
i := Int()
c := Var().Add(a, i)
fmt.Printf("%#v", c)
// Output:
// var a int
```

# Do
Do calls the provided function with the statement as a parameter. Use for
embedding logic.

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-Do):
```go
f := func(name string, isMap bool) *Statement {
	return Id(name).Op(":=").Do(func(s *Statement) {
		if isMap {
			s.Map(String()).String()
		} else {
			s.Index().String()
		}
	}).Values()
}
fmt.Printf("%#v\n%#v", f("a", true), f("b", false))
// Output:
// a := map[string]string{}
// b := []string{}
```

# Lit, LitFunc
Lit renders a literal, using the format provided by the fmt package %#v
verb.

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-Lit):
```go
c := Id("a").Op(":=").Lit("a")
fmt.Printf("%#v", c)
// Output:
// a := "a"
```

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-Lit-float):
```go
c := Id("a").Op(":=").Lit(1.5)
fmt.Printf("%#v", c)
// Output:
// a := 1.5
```

 LitFunc generates the value to render by executing the provided
function.

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-LitFunc):
```go
c := Id("a").Op(":=").LitFunc(func() interface{} { return 1 + 1 })
fmt.Printf("%#v", c)
// Output:
// a := 2
```

# Dict, DictFunc
Dict takes a map[Code]Code and renders a list of colon separated key value
pairs, enclosed in curly braces. Use for map literals.

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-Dict):
```go
c := Id("a").Op(":=").Map(String()).String().Dict(map[Code]Code{
	Lit("a"):	Lit("b"),
	Lit("c"):	Lit("d"),
})
fmt.Printf("%#v", c)
// Output:
// a := map[string]string{
// 	"a": "b",
// 	"c": "d",
// }
```

DictFunc executes a func(map[Code]Code) to generate the value.

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-DictFunc):
```go
c := Id("a").Op(":=").Map(String()).String().DictFunc(func(m map[Code]Code) {
	m[Lit("a")] = Lit("b")
	m[Lit("c")] = Lit("d")
})
fmt.Printf("%#v", c)
// Output:
// a := map[string]string{
// 	"a": "b",
// 	"c": "d",
// }
```

Note: the items are ordered by key when rendered to ensure repeatable code.

# Tag
Tag renders a struct tag

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-Tag):
```go
c := Type().Id("foo").Struct(
	Id("A").String().Tag(map[string]string{"json": "a"}),
	Id("B").Int().Tag(map[string]string{"json": "b", "bar": "baz"}),
)
fmt.Printf("%#v", c)
// Output:
// type foo struct {
// 	A string `json:"a"`
// 	B int    `bar:"baz" json:"b"`
// }
```

Note: the items are ordered by key when rendered to ensure repeatable code.

# Null
Null adds a null item. Null items render nothing and are not followed by a
separator in lists.

In lists, nil will produce the same effect.

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-Null-and-nil):
```go
c := Func().Id("foo").Params(
	nil,
	Id("s").String(),
	Null(),
	Id("i").Int(),
).Block()
fmt.Printf("%#v", c)
// Output:
// func foo(s string, i int) {}
```

# Empty
Empty adds an empty item. Empty items render nothing but are followed by a
separator in lists.

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-Empty):
```go
c := Id("a").Op(":=").Id("b").Index(Lit(1), Empty())
fmt.Printf("%#v", c)
// Output:
// a := b[1:]
```

# Line
Line inserts a blank line.

# Comment, Commentf
Comment adds a comment. If the provided string contains a newline, the
comment is formatted in multiline style.

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-Comment):
```go
f := NewFile("a")
f.Comment("Foo returns the string \"foo\"")
f.Func().Id("Foo").Params().String().Block(
	Return(Lit("foo")).Comment("return the string foo"),
)
fmt.Printf("%#v", f)
// Output:
// package a
// 
// // Foo returns the string "foo"
// func Foo() string {
// 	return "foo" // return the string foo
// }
```

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-Comment-multiline):
```go
c := Comment("a\nb")
fmt.Printf("%#v", c)
// Output:
// /*
// a
// b
// */
```

 If the comment string starts
with "//" or "/*", the automatic formatting is disabled and the string is
rendered directly.

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-Comment-formatting-disabled):
```go
c := Id("foo").Call(Comment("/* inline */")).Comment("//no-space")
fmt.Printf("%#v", c)
// Output:
// foo( /* inline */ ) //no-space
```

Commentf adds a comment, using a format string and a list of parameters.

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-Commentf):
```go
name := "Foo"
output := "foo"
f := NewFile("a")
f.Commentf("%s returns the string \"%s\"", name, output)
f.Func().Id(name).Params().String().Block(
	Return(Lit(output)),
)
fmt.Printf("%#v", f)
// Output:
// package a
// 
// // Foo returns the string "foo"
// func Foo() string {
// 	return "foo"
// }
```

# File
File represents a single source file. Package imports are managed
automaticaly by File.

### NewFile
NewFile Creates a new file, with the specified package name.

### NewFilePath
NewFilePath creates a new file while specifying the package path - the
package name is inferred from the path.

### NewFilePathName
NewFilePathName creates a new file with the specified package path and name.

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-NewFilePathName):
```go
f := NewFilePathName("a.b/c", "main")
f.Func().Id("main").Params().Block(
	Qual("a.b/c", "Foo").Call(),
)
fmt.Printf("%#v", f)
// Output:
// package main
// 
// func main() {
// 	Foo()
// }
```

### PackageComment
PackageComment adds a comment to the top of the file, above the package
keyword.

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-File-PackageComment):
```go
f := NewFile("c")
f.PackageComment("a")
f.PackageComment("b")
f.Func().Id("init").Params().Block()
fmt.Printf("%#v", f)
// Output:
// // a
// // b
// package c
// 
// func init() {}
```

### Anon
Anon adds an anonymous import:

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-File-Anon):
```go
f := NewFile("c")
f.Anon("a")
f.Func().Id("init").Params().Block()
fmt.Printf("%#v", f)
// Output:
// package c
// 
// import _ "a"
// 
// func init() {}
```

### PackagePrefix
If you're worried about package aliases conflicting with local variable
names, you can set a prefix here. Package foo becomes {prefix}_foo.

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-File-PackagePrefix):
```go
f := NewFile("c")
f.PackagePrefix = "pkg"
f.Func().Id("main").Params().Block(
	Qual("fmt", "Println").Call(),
)
fmt.Printf("%#v", f)
// Output:
// package c
// 
// import pkg_fmt "fmt"
// 
// func main() {
// 	pkg_fmt.Println()
// }
```

### Save
Save renders the file and saves to the filename provided.

### Render
Render renders the file to the provided writer.

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-File-Render):
```go
f := NewFile("a")
f.Func().Id("main").Params().Block()
buf := &bytes.Buffer{}
err := f.Render(buf)
if err != nil {
	fmt.Println(err.Error())
} else {
	fmt.Println(buf.String())
}
// Output:
// package a
// 
// func main() {}
```

# Clone
Be careful when passing *Statement. Consider the following... 

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-Statement-Clone-broken):
```go
a := Id("a")
c := Block(
	a.Call(),
	a.Call(),
)
fmt.Printf("%#v", c)
// Output:
// {
// 	a()()
// 	a()()
// }
```

Id("a") returns a *Statement, which the Call() method appends to twice. To 
avoid this, use Clone. Clone makes a copy of the Statement, so further tokens can be appended
without affecting the original.  

[Example](https://godoc.org/github.com/davelondon/jennifer/jen#example-Statement-Clone-fixed):
```go
a := Id("a")
c := Block(
	a.Clone().Call(),
	a.Clone().Call(),
)
fmt.Printf("%#v", c)
// Output:
// {
// 	a()
// 	a()
// }
```