> The overall goal of the Go 2 effort is to address the most significant ways that Go fails to scale to large code bases and large developer efforts.

> As part of re-entering “design mode” for the Go 2 effort, we are again attempting to find a design for generics that we feel fits well into the language while providing enough of the flexibility and expressivity that users want.

> Instead of attempting that at the start, we spent our time on features more directly applicable to Go’s initial target of networked system software (now “cloud software”), such as concurrency, scalable builds, and low-latency garbage collection.

> If we are to add generics, we want to do it in a way that gets as much flexibility and power with as little added complexity as possible.

### Problem
> They would prefer to let the implementation manage its own array, but Go does not permit expressing that in a type-safe way. The closest one can come is to make a priority queue of interface{} values and use type assertions after fetching each element. (The standard container/list and container/ring implementations take this approach.)

> Our goal is to address the problem of writing Go libraries that abstract away needless type detail, such as the examples in the previous section, by allowing parametric polymorphism with type parameters.

> In particular, in addition to the expected container types, we aim to make it possible to write useful libraries for manipulating arbitrary map and channel values, and ideally to write polymorphic functions that can operate on both []byte and string values.

> Polymorphism in Go must fit smoothly into the surrounding language, without awkward special cases and without exposing implementation details. For example, it would not be acceptable to limit type parameters to those whose machine representation is a single pointer or single word. As another example, once the general Keys(map[K]V) []K function contemplated above has been instantiated with K = int and V = string, it must be treated semantically as equivalent to a hand-written non-generic function. In particular it must be assignable to a variable of type func(map[int]string) []int.

> Polymorphism in Go should be implementable both at compile time (by repeated specialized compilation, as in C++) and at run time, so that the decision about implementation strategy can be left as a decision for the compiler and treated like any other compiler optimization. This flexibility would address the generic dilemma we’ve discussed in the past.

### Draft Design
type parameter
```go
type List(type T) []T
func Keys(type K, V)(ms map[K]V) []K
```
contract
```go
contract Equal(t T) {
    t == t
}
```
example declaration
```go
contract Addable(t T) {
    t + t
}

func Sum(type T AdDable)(ts T) T {
    var sum T
    for _, t := range ts {
        sum += t
    }

    return sum
}
```
example usage
```go
var xs []int
sum := Sum(int)(xs)
```
The two invocations can be splited.
```go
sumInts := Sum(int)
var xs []int
sum := sumInts(xs)
```
The call with type arguments can be ommitted when they can be inferred from input values. If you provide untyped contants, the default types of the values are used.
```go
var xs []int
sum := Sum(xs)
```
Parameterized type can be compiled.
Parameterized type can have methods, and the receiver type of a method must list the type parameters without `type` or contracts.
```go
type Vector(type Element) []Element

type v Vector(int)

func (v *Vector(Element)) Push(elem Element) {
    *v = append(*v, elem)
}
```
Parameterized types can refer themselves in cases where the types can ordinally refer to themselves, but they cannot in cases where they cannot.
```go
// This is OK because the type of List.next *List(Element) can refer to List(Element)
type List(type Element) struct {
    next *List(Element)
    value Element
}

// This is invalid because type of Tree.Children *Tree(Right, Left) cannot refer to Tree(Left, Right).
// This restriction aviods infinite recursions of type inspection.
type Tree(type Left, Right) struct {
    Children *Tree(Right, Left)
}
```

contracts may embed another contracts.
```go
contract stringPrinter(x X) {
    stringer(x)
    x.Print()
}

contract stringer(x X) {
    var s string = x.String()
}
```
contract check mainly about type. So if you wan to check that type X has method `Equal(X) bool`, there is no need to take two arguments of X.
```go
// This is enough.
contract equalable(x X) {
    var isEqual bool = x.Equal(x)
}

// This is not neccesarry.
contract equable(x, y X) {
    var isEqual bool = x.Equal(y)
}
```
It might be hard to see how different codes using contract and using interface. The benefit of contract is that it can ensure not only methods, but alos concreate types. The exmple contract below proves that the elements of `[]Node` and `[]Edge` are `*Vertex` and `*FromTo`, ensuring that those types have required methods.
```go
func New(type Node, Edge Grapher)(nodes []Node) *Graph(Node, Edge) { ... }

type Graph(type Node, Edge Grapher) struct { ... }

func (g *Graph(Node, Edge)) ShortestPath(from, to Node) []Edge { ... }

contract Grapher(n Node, e Edge) {
    var _ []Edge = n.Edges()
    var _, _ []Node = e.Nodes()
}

type Vertex struct { ... }

func (v *Vertext) Edges() []*FromTo { ... } 

type FromTo struct { ... }

func (ft *FromTo) Nodes() (*Vertex, *Vertex) { ... }
```
```go
graph := New(*Vertex, *FromTo)([]*Vertex{ ... })
```

Type inference is done without regard to contract. The check that the types implement contracts is done after typp inference.

The parser does not know whether it is seeing type conversion or composite literal. So to avoid this ambiguity the parser always parse `[]T(v1)` as type convertion, and the composite literal with generic types should always b `[](T(v2))`
```go
// This is always type convertion to []T from v1
x1 := []T(v1)
// This is always composite literal of slice of T 
x2 := [](T(v2)){}
```