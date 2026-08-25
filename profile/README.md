# go-by-value

Go analyzers about value semantics: when a copy is wrong, and when a pointer is unnecessary.

| Analyzer                                              | Kind       | Reports                                                                              |
|-------------------------------------------------------|------------|--------------------------------------------------------------------------------------|
| [deadmut](https://github.com/go-by-value/deadmut)     | Bug finder | Mutations of range loop value copies that have no effect                             |
| [pointless](https://github.com/go-by-value/pointless) | Style      | Pointer receivers, pointer return types, and slices of pointers that could be values |

## Why both

Go copies a struct on assignment, and the copy is silent. deadmut catches the bug where a copy is mutated by mistake:

```go
for _, user := range users {
    user.Name = strings.TrimSpace(user.Name) // deadmut: write to user.Name has no effect
}
```

pointless removes the pointers that exist only out of habit, so there are fewer places where sharing can go wrong in the first place:

```go
type Point struct{ X, Y int } // pointless: methods of Point can use value receivers

func (p *Point) Len() int { return p.X*p.X + p.Y*p.Y }
```

deadmut is meant to run everywhere and stay quiet unless the mutation is certainly lost. pointless is a suggestion with a configurable threshold; it tells you where a pointer is not doing anything inside the declaration, and leaves the call sites to you.

## Shared design

Both are [`go/analysis`](https://pkg.go.dev/golang.org/x/tools/go/analysis) analyzers built on the same idea: analyze what every function does with its pointer receivers and pointer parameters (writes to the pointee, writes through shared memory, or retains the pointer) and export the result as a fact. The analysis crosses package boundaries, so both know that `bytes.Buffer.Len` only reads and `bytes.Buffer.WriteString` writes. Neither uses `init()`, package-level state, or configuration files; settings come from flags or from golangci-lint.

## Using them

```sh
go install github.com/go-by-value/deadmut/cmd/deadmut@latest
go install github.com/go-by-value/pointless/cmd/pointless@latest

deadmut ./...
pointless ./...
```

Both work as `go vet -vettool=...` tools and ship a [golangci-lint module plugin](https://golangci-lint.run/docs/plugins/module-plugins/); see each README for the configuration.

## License

MIT, per repository.
