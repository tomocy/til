与えられたスライスから最大差になるような組み合わせを見つけるためには、最大-最小=最大差であることを利用する。つまりスライスをイテレートする中で、最小値を常に更新しておき、イテレーションの現在の値とその最大値との差をそれまでの最大値と比べればいい。
```go
func findMaximumDifference(vs []int) int {
    var min, max int
    for i, v := range vs {
        if i == 0 || v < min {
            min = v
        }

        current := v - min
        if max < current {
            max = current
        }
    }

    return max
}
```