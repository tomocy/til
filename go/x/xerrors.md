`New`は呼び出しのコールスタックを保持する`error`を返す。`New`等で返されるerrorは`fmt.Formatter`インターフェイスを実装することでスタックをフォーマットに組み込んでいる。  

`Errorf(format string, a ...interface{})`はaの最後の要素がerrorかつ、それに対するフォーマット指定が `: %s`,  `: %v`,  `: %w` であるときに限り、渡されたerr情報を保持する。また `Errorf` のフォーマット指定が `: %w` の場合のみ、渡されたerrは新しいerrにラップされて返ってくる。そして返ってくるerrは以下のインターフェイスを満たす。  
```go
type Wrapper interface {
    Unwrap() error
}
```

`Unwrap(error) error` は渡されたerrが `Wrapper` を満たす場合、その戻り値を返し、満たしていない場合はnilを返す。  

`Is(err, target error) bool` はtargetがerrと一致した場合true、一致しない場合falseを返す。  
このときの一致するとは以下の時のみである  
- `err == target`がtrueの場合
-  errが `Is(error) bool` を満たしており、 `err.Is(target)`がtrueの場合
- `Unwrap(err)` の戻り値が上の場合を満たす時

`Opaque(error) error` は渡されたerrを以下のインターフェイスを満たすものとして返す。  
```go
type noWrapper interface {
    error
}
```
そのため `Opaque` 後のerrは `Unwrap` できなくなり、その結果 `Is` 等の結果も変わる。  

`As(err error, target interface{}) bool`はerrがtargetに代入可能の場合true, 不可能な場合falseを返す。  
`As`がtrueを返す場合は以下の時のみである
- targetにerrが代入可能である
- `Unwrap(err)`の戻り値が上記の場合を満たす時
またtargetはポインタ、インターフェイスあるいは、`error`を満たしていなければならない。  

ラップされたerrはフォーマット指定が`%+v`の場合のみラップしていたerrのスタック情報を表示する。  