インターフェイスを満たすかどうかは構造体が要求されているメソッドを持っているかどうかである  
ここではそのインターフェイスがexportedされていようが、unexporetedされていようが関係ない  
しかしunexportedな関数を要求するインターフェイスに対して、外部パッケージはその関数の独自の実装をすることはできない  

例えば以下のようなパッケージfooがあるとする  
```go
package foo

func Foo(fooer Fooer) {
    fooer.Foo()
}

type Fooer interface {
    foo()
}
```

このときmainパッケージでfooを持つ構造体を定義してもfoo.Fooにその構造体を渡すことはできない  
なぜならFooインターフェイスはfoo()ではなくfoo.foo()を要求しているからである  

これは外部パッケージで独自の実装を持った構造体を受け付けないようにすることに使える  

Alias declarationsとは既存のtypeに別の名前をつけることである。
文法は以下である
```go
type After = Before
```
既存のAPIを新しくする場合
新しい機能の追加、古いものから新しい機能を使用するよう修正、古い機能を削除
を行う必要がある
Goでは以下のようにすることで
後方互換性を保ちながらリファクタリングをすることができる
```go
var Old = new.New
const Old = new.New
func Old() { new.New() }
type Old = new.New
```