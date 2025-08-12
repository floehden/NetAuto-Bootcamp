# **Day 9: Structuring Go Projects and Modules**

## **Introduction:** 
Go modules for dependency management. Project structure (`main.go`, `go.mod`, utility packages). Best practices for organizing Go code.

## **Code Example: Simple Module**

```
myproject/
├── go.mod
├── main.go
└── utils/
    └── network.go
```

`myproject/go.mod`:

```
module example.com/myproject

go 1.22
```

`myproject/utils/network.go`:

```go
package utils

import "fmt"

func GreetNetworkEngineer(name string) {
    fmt.Printf("Hello, %s! Happy automating!\n", name)
}
```

`myproject/main.go`:

```go
package main

import (
    "example.com/myproject/utils" // Import your custom package
    "fmt"
)

func main() {
    fmt.Println("Starting network automation project.")
    utils.GreetNetworkEngineer("Alice")
}
```

* To run: `cd myproject && go mod tidy && go run .`

## **Challenge 9:** 
Refactor your `configureInterface` function from Day 4 into a `config` package within your project. Update `main.go` to import and use it.

