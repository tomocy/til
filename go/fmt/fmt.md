`Stringer`は以下の通りである
```go
type Stringer interface {
    String() string
}
```
`Stringer`を実装することで
`%s`、`%v`で独自のフォーマットを指定することができる
ただし`Stringer`では`%#v`のフォーマットを指定することはできない

`GoStringer`は以下の通りである
```go
type GoStringer interface {
    GoString() string
}
```
`GoStringer`を実装することで
`%#v`で独自のフォーマットを指定することができる

`Formatter`は以下の通りである
```go
type Formatter interface {
    Format(s State, verb rune)
}
```
`Formatter`を満たすことで
自由にフォーマットを指定することができる
`verb`にはフォーマット指定子が挿入されるので
独自のフォーマット指定子も作ることができる
例えば以下のようにである
```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	wagashi := wagashi("")
	fmt.Printf("%4.4w", wagashi)
}

type wagashi string

func (w wagashi) Format(s fmt.State, verb rune) {
	if verb != 'w' {
		fmt.Fprint(s, w)
		return
	}

	var ds string
	if w, ok := s.Width(); ok {
		ds = strings.Repeat("🍡", w)
	}
	var ts string
	if p, ok := s.Precision(); ok {
		ts = strings.Repeat("🍵", p)
	}

	fmt.Fprintf(s, "%s%s", gs, ds)
}
```