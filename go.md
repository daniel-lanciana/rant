## Go

## Install

```bash
# 1. Install Go
brew install go
# 2. Install GoLand
# 3. Go!
```

## Learnings

* Place project under `$GOPATH/src/github.com/your-username/your-repo/` (otherwise import issues)
* Import using absolution path (e.g. `github.com/foo/bar`). Checks local project path first, then globally installed packages.
* Web server built-in (`net/http`)
* Lots of single-character variable names! Also prefer `foo.Bar()` to `foo.GetBar()` and `foo.String()` over `foo.ToString()`. KISS.
* Functions need to start with an uppercase to be exportable (i.e. public) -- little bit of implicit magic..
* Set `$GOBIN` to use terminal commands
* Use `deps` for package management (places dependencies under `/vendor`) -- otherwise everything will be placed alongside your project in `$GOPATH/src/`
* Running tests not in the root `go test -v ./...`
* GoLand has good support for running, debug, tests
* Handlers == middleware. Handle and HandlerFunc are essentially the same.
* To install packages globally `go get -u github.com/package/to/install`
* gorilla libraries for logging, gzip, routing
* See all Go paths and variables: `go env`
* JSON marshaller (i.e. serializer) only serializes struct properties with capitals. Magic..
* Dep visual dependency graph: `brew install graphviz` then `dep status -dot | dot -T png | open -f -a /Applications/Preview.app`
* Mux routing allows regular expressions
* Testify assertion library
* Use `[[override]]` in `Gopkg.toml` for dev dependencies you want to include, but aren't explicity referenced in the code (to avoid it being removed on `dep ensure`)
* No ternary operator!
* Reason statements are defined as name_type (e.g. foo int) is because of simple semicolon insertion rules of compiler
* Liveloading with `realize start` -- but no GoLand run config or debugging available..
* Interfaces are good (allow changing of underlying implementation, can be mocked during testing)
* Gorm
  * https://github.com/carrollgt91/gorm/pull/1/files (foreign key doesn't work natively)
  * Use of sql.NullXYZ() for persisting null values (https://golang.org/pkg/database/sql/#NullInt64)
* `http.Client` does not specify a timeout -- will just wait! (https://medium.com/@nate510/don-t-use-go-s-default-http-client-4804cb19f779)
* Benchmarking concurrency with Apache ab tool: `ab -c 500 -n 500 http://local.tryroll.com:8080/` (may need to raise socket limit with `ulimit -n 10000`)
* Named return parameters (auto initialized, serves as documentation)
* `defer` for "after" hooks in functional, not block, way
* Gotcha: `struct` values initialized (like any `new` call) to zeroed (e.g. `int = 0`). Have to use pointers to distinguish between no value and valid `0`.
* `new(Foo)` or `&Foo{}` are equivelant and returns a pointer, `var` returns value
* `make([]int, 100)` applies to maps, slices and channels -- does not return pointer. In Go, generally use slices instead (pointers to underlying arrays, more powerful/flexible)
* `map[string]int{"key":123}` unusual declaration -- would expect `map[string, int]`!
* Print statement messy and verbose, use `glog`. Can use `fmt.Printf("%+v\n", t)` to print object `t` in full. 
* Constants use `iota` enumerator integer
* Implicit interfaces -- object checked at compile time to see if implements all require methods. Unusual for such an explicit language (as opposed to `type Cat struct implements Animal`.
* Each file can have an `init()` func to setup state. Types can implement multiple interfaces.
* Can define a method for any type except pointers and interfaces. Can write a method for a function!
* `import _ "pg"` for the package side-effects, don't cleanup on `dev ensure`
* Subclassing interfaces within interfaces, handled by imported type. Embedding methods into structs from elsewhere (forwarding).
* `interface{}` for generic objects (e.g. generic map `make(map[string]interface{}, 0)`) 
* Default JSON unmarshal types: `bool`, `float64` for numbers, `string`, `[]interface{}` for arrays, `map[string]interface{}` for JSON objects, `nil`
* `<nil>` can occur when redeclaring between incompatable types (e.g. assign `err` to `error`, then reasign to `common.Error`). Causes problems because `err != nil` will be `true` even though value is `<nil>`!
* Use interfaces (e.g. `connect()`), encapsulated structs (e.g. wrap `*sql.Sql` as `DB` struct), embedded types (mixins), and type methods (e.g. `func (d DB) connect()...`) A LOT. Makes swapping concrete implementations easy, testable with mocks. 
* big.Int
  * Only `*big.Int` has the JSON marshall method
  * Parsing from string: `i, ok := new(big.Int).SetString(intString, 10)`
  * Not compatable with Gorm (must explicitly define column as `bigint`)
  * Needed for working with Ethereum uint256 types of smart contracts 

```go
// Structs are like classes (i.e. objects)
type Person struct {
  name  string
}
p := Person{ Name:"dan" }

// Or if just containing one embedded types
type Counter struct {
    n int
}
// Same as
type Counter int

// And can have embedded (unnamed) types
type Foo struct {
  Bar
}
b := Bar{} // has method stuff()
f := Foo{Bar{}}
f.stuff() // Bar's methods are embedded at the root-level (i.e. mixins)

// Structs can have methods (i.e. behaviors)
// Parameters after 'func' define it as an extension to that struct
func (p *Person) Speak() *Person {
  // Access to self calling object, p
  fmt.Println("Hello my name is " + p.name)
  // Return self to allow for functional chaining of calls
  return p
}

// Testing a range
func TestCalculate(t *testing.T) {
	assert := assert.New(t)

	var tests = []struct {
		input    int
		expected int
	}{
		{2, 4},
		{-5, -3},
		{99999, 100001},
	}

	for _, test := range tests {
		assert.Equal(Calculate(test.input), test.expected)
	}
}
```

## Resources

* https://golang.org/doc/effective_go.html (book)
* https://www.sohamkamani.com/blog/2017/09/13/how-to-build-a-web-application-in-golang/ (nice getting started guide)
* https://gowebexamples.com (web server examples)
* https://github.com/avelino/awesome-go (curated list of libraries)
* https://blog.questionable.services/article/testing-http-handlers-go/
* https://auth0.com/blog/authentication-in-golang/ (Oauth, react)
* https://medium.com/orbs-network/big-integers-in-go-14534d0e490d
* https://golang.org/pkg/builtin (data types)
* https://flaviocopes.com/go-date-time-format/
* http://gorm.io/docs/index.html and http://doc.gorm.io/advanced.html (Gorm)
* https://www.alexedwards.net/blog/organising-database-access (db patterns)
