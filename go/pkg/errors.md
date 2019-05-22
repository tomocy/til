`Wrap(error, string) error`は引数のerrをcauseとした新しいerrorを返す  
`Cause() error`はもし `errors.causer` インターフェイス、つまり `Cause(err error) error` を満たしていればその戻り値を返す  
もしそのインターフェイスを満たさなければそのままのerrを返す  