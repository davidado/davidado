+++
title = 'Go Programming Notes'
date = 2024-02-17T10:59:49-08:00
draft = false
+++

This isn't meant to be a comprehensive set of notes so I won't be including much of the obvious stuff.

## Installation

1. Download archive from https://go.dev/dl/
2. Remove any previous Go installation by deleting the /usr/local/go folder (if it exists), then extract the archive you just downloaded into /usr/local, creating a fresh Go tree in /usr/local/go: ` rm -rf /usr/local/go && tar -C /usr/local -xzf go1.22.0.linux-amd64.tar.gz`
3. Add /usr/local/go/bin to the `PATH` environment variable. You can do this by adding the following line to your $HOME/.profile or /etc/profile (for a system-wide installation): `export PATH=$PATH:/usr/local/go/bin`
4. Run `source $HOME/.profile` to apply changes immediately.
5. Verify that you've installed Go by opening a command prompt and typing the following command: `go version`

## Environment

- `go build -o hello_world hello.go` builds a binary named *hello_world* that can be distributed to other people.
- Install third party tools using `go install`.
- `go install github.com/rakyll/hey@latest` installs the *hey* tool that load tests an HTTP endpoint (ex: `hey https://www.google.com`).
- `goimports` is the enhanced version of `go fmt` that not only automatically formats your Go code to standard formatting, but also cleans up your import statements (sorts in alphabetical order, removes unused imports, and guesses at unspecified imports).
- Install with `go install golang.org/x/tools/cmd/goimports@latest` then run `goimports -l -w .` across your project (`-l` prints the files with incorrect formatting to the console, `-w` modifies the files in-place, `.` specifies the current directory and all subdirectories).
- The Go linter enforces code style. Install with `go install golang.org/x/lint/golint@latest` and run with `golint ./...` which runs the linter over your entire project.
- `go vet` catches mistakes such as passing the wrong number of parameters or unused variables.
- `golangci-lint run` combines `golint` and `go vet` and can be configured to run multiple linters.

**Makefiles**

- Makefiles help automate tasks. Sample makefile:

```
.DEFAULT GOAL := build

fmt:
	go fmt ./...
.PHONY: fmt

lint: fmt
	golint ./...
.PHONY: lint

vet: fmt
	go vet ./...
.PHONY:vet

build: vet
	go build hello.go
.PHONY:build
```

- `build: vet` specifies that `vet` must run first before `go build`.
- .PHONY keeps *make* from getting confused if a directory with the same name as the target is ever created in the project.

## Primitive Types and Declarations

- Avoid declaring package level variables whose values change sit it can be difficult to track the changes made to it. Generally, only declare variables in the package blog that are effectively immutable.

## Composite Types

**Arrays**

- To declare an array of 3 *ints* pre-filled with zero values: `var x [3]int`.
- Arrays are rarely used especially since Go considers the *size* of the array to be part of the *type* of the array so a [3]int is a different type from a [4]int.
- Also, a variable can't be used to specify the size of an array because types must be resolved at compile time.

**Slices**

- To declare a slice literal: `var x = []int{10, 20, 30}`.
- Declaring a slice literal with `var x = []int{1, 3:6}` creates a slice of 5 ints with the following values: `[1, 0, 0, 6]`.
- `len(slice)` returns the length of a slice.
- `x = append(x, 5, 6, 7)` appends 5, 6, 7 to the end of the slice `x`.
- `x = append(x, y...)` appends slice `y` to the end of slice `x`.
- `cap(x)` returns the capacity of slice `x` but is used far less often than `len`.
- `make` can set length and capacity.
- `x := make([]int, 5)` creates an int slice with a length and capacity of 5 so all elements are initialized to 0.
- If you try to do `x = append(x, 10)` to above, you will end up with the slice [0 0 0 0 0 10].
- `x := make([]int, 0, 10)` creates an int slice of length 0 and capacity 10 so appending 10 to this slice would result in the expected slice of [10].
- **Slice Declarations**
  - `var data []int` declares a slice that might stay *nil*.
  - `var x = []int{}` declares a zero-length slice which is non-nil where `x == nil` returns *false*. This way of slice declaration is most useful when converting a slice to JSON.
- Slices share storage in most cases so taking a slice from a slice means that changing one slice, changes the other.
- Slicing slices combined with *append* can be very confusing so try to avoid it.
- Use a three-part slice expression to prevent *append* from sharing capacity between slices (`z := x[2:4:4]`)
- A slice taken from an array also shares its memory.
- **copy** can create a slice independent from the original. `z := copy(y, x)` copies `x` into `y`.

**Strings, Runes, and Bytes**

- Be careful of indexing into a string where you aren't sure if every character is a byte long. Trying to access a multi-byte character using an index may result in garbled text.
- A string can be converted back and forth to a slice of bytes or a slice of runes (`[]bytes(str)` or `[]rune(str)`).
- Most data in Go is read and written as a sequence of bytes to the most common string type converstion is back and forth with a slice of bytes. Slices of runes are uncommon.
- Instead of using slice and index expressions with strings, use the functions in the *strings* and *unicode/utf8* packages in the standard library.

**Maps**

- `var nilMap map[string]int` results in a map with the zero value of *nil* and attempting to write to a *nil* map causes a panic.
- `totalWins := map[string]int{}` results in an empty map literal which is not the same as a *nil* map because you can read and write to it.
- `ages := make(map[name]int, 10)` creates a map with length 0 but with the default size of 10. It can also grow past its initial size.
- The keys for a map can be of any comparable type so it can't be a slice or another map.
- Reading the value of a non-existent key returns the zero value of the type of value stored so that a map of ints would return 0 for a non-existent key.
- Maps return two values: `v, ok := m["hello"]` where if *ok* is *false*, the key is not present.
- `delete(m, "hello")` deletes the *hello* key from *m*.
- To use a *map* as a *set*, enter all values as a map key and either set the value as a bool or an empty struct.
- The bool is simpler to work with but uses a byte of memory whereas an empty struct does not. Use bool for simplicity unless you have a very large set and saving a byte of data per entry makes a difference.

**Structs**


```
type person struct {
	name string
	age int
}

// Order matters
julia := person{
	"Julia",
	40,
}

// The preferred way
beth := person{
	age: 30,
	name: "Beth",
}

beth.age = 31
```

**Anonymous Structs**


```
pet := struct{
	name string
	kind string
}{
	name: "Spot",
	kind: "dog",
}
```

- Anonymous structs are most often used for:
  - Converting external data into a struct.
  - Converting a struct into external data like JSON or a protocol buffer. (*marshaling*)

**Comparing and Converting Structs**

- Structs can be comparable if they are entirely composed of comparable types, have the same field names, order of those name, and types.
- Anonymous structs can be comparable if they also have the same field names, order, and types.


```
f := person{
	name: "Bob",
	age: 50,
}
var g struct {
	name string
	age int
}
g = f
fmt.Println(f == g)
```

## Blocks, Shadows, and Control Structures

```
func main() {
	x := 10
	if x > 5 {
		fmt.Println(x) // Prints 10
		x := 5
		fmt.Println(x) // Prints 5
		x, y := 7, 20
		fmt.Println(x, y) // Prints 7 20
	}
	fmt.Println(x) // Prints 10
}
```

- The *x* in the inner *if* block shadows the *x* in the outer block.
- Accidental shadowing frequently occurs when assigning to multiple variables since you only need on new variable to use the := assignment operator.

## Detecting Shadowed Variables

- `go install golang.org/x/tools/go/analysis/passes/shadow/cmd/shadow@latest`
- `shadow ./...`
- Makefile:

```
vet:
	go vet ./...
	shadow ./...
.PHONY:vet
```

- Running this will give you the line: `declaration of "x" shadows declaration at line 6`

**if**

```
if n := rand.Intn(10); n == 0 {
	fmt.Println("That's too low")
} else if n > 5 {
	fmt.Println("that's too big: ", n)
} else {
	fmt.Println("That's a good number:", n)
}
fmt.Println(n) // Results in error undefined: n
```

- *n* is scoped to the entirety of *if* block and nowhere else.

**for**

- To mimic a do-while loop:

```
for {
	// Do stuff
	if !condition {
		break
	}
}
```

- Consider using the *continue* statement to break up nested *if*s in a *for* loop.
- To range over just the map keys, just leave off the value key: `for k := range m`. Most commonly used for sets.
- Ranging over a map will not result in the same order of keys every time.
  - This is done to remind programmers that map keys are not stored in order.
  - This is also a security fix for an attack called a *Hash DoS* where an attacker sends data where all of the keys hash to the same bucket to slow down a server.
- However, using *fmt.Println* will output maps with their keys in ascending order to ease debugging.
- When ranging over strings, the index and the number of bytes from the beginning of the string but is of type *rune*.

```
str := "hello"
for i, r := range str {
	fmt.Println(i, r, string(r))
}

// Prints:
// 0 104 h
// 1 101 e
// 2 108 l
// 3 108 l
// 4 111 o
```

**switch**

- Switches don't require the *break* statement in Go.
- Regular switches only allows you to check a value for equality.
- Blank switches allows you to use any boolean comparison for each case.
- However, if you find yourself using a blank switch where every comparison is for equality, you're better off just having a switch with an expression:

```
// Don't do this
switch {
case a == 2:
	fmt.Println(a)
case a == 3:
	fmt.Println(a)
}

// Do this instead
switch a {
case 2:
	fmt.Println(a)
case 3:
	fmt.Println(a)
}
```

## Functions

- Functions that take another function as an input parameter or a return value are *higher order functions*.

**defer**

- The code within *defer* closures run after the return statement.

```
tx, err := db.BeginTx(ctx, ni)
if err != nil {
	return err
}
defer func() {
	if err == nil {
		err = tx.Commit()
	}
	if err != nil {
		tx.Rollback()
	}
}()
```

- The parentheses after the anonymous function is what executes it. It's a compile time error to leave it out.
- *defer* is something like the *finally* clause in other languages. The downside to these resource cleanup blocks is that they create another level of indentation in your function making code harder to read.
	- In a 2017 research paper, out of 11 code characteristics, only nesting depth and lack of structure markedly influence complexity.

**Go Is Call By Value**

- Everything passed as parameter to a function is a value. Even structs changed within a function do not affect the original struct.
- The behavior is different for maps and slices even though those are also passed in as values. Maps and slices are implemented as pointers to structs in the underlying Go platform so the values passed are the values of those pointers.

## Pointers

- A pointer is a variable whose contents are the address in memory where another variable is stored.
- Pointers always take up the same size in memory because all it stores is a location in memory where values are stored.
- A pointer that doesn't point to anything has zeroes as values and the zero value of a pointer is *nil*.
- The *&* is the *address* operator. It precedes a value type and returns the memory address of where the value is stored.
- The * is the *indirection* operator. It precedes a variable of pointer type and returns the pointed-to value (also called *dereferencing*).
	- Make sure that a pointer is non-nil before dereferencing it or the program panics in the attempt.
- If a struct has a field that takes a pointer, you can either create a variable as a pointer and set it in the struct, or you can create a helper function that returns a pointer type.

```
type person struct {
	FirstName string
	MiddleName *string
}

func stringp(s string) *string {
	return &s
}

p := person{
	FirstName: "Pat",
	MiddleName: stringp("Perry"),
}
```
- Most of the time, you should pass in values instead of pointers since it makes it easier to understand how and when your data is modified. It also reduces the amount of work that the garbage collector has to do.

**Pointers Indicate Mutable Parameters**

- The lack of immutable declarations in Go might seem problematic, but the ability to choose between value and pointer parameter types addresses the issue.
- Since Go is a call by value language, the values passed to functions are copies and the immutability of the original data is guaranteed (maps and slices notwithstanding).
- When you pass a *nil* pointer to a function, you cannot make the value non-nil. You can only reassign the value if there was a value already assigned to the pointer.
	- Remember that Go is call by value and the values passed are copies so trying to change the pointer in the function is just changing a copy of it.
- Since the pointer passed into a function is just a copy of the original, you have to *dereference* the pointer to set it to a new value. Dereferencing puts the new value in the memory location pointed to by both the original and the copy.

```
// Doesn't work
func failedUpdate(px *int) {
	x2 := 20
	px = &x2 // you just updated the copy
}

// Works
func update(px *int) {
	// dereferencing takes you to the original memory location
	// then the value in that location is changed.
	*px = 20
}
```

**Pointers Are a Last Resort**

- Rather than passing a pointer to a struct as a return value of a function, have the function instantiate and return the struct.
```
// Don't do this
func MakeFoo(f *Foo) error {
	f.Field1 = "val"
	f.Field2 = 20
	return nil
}

// Do this instead
func MakeFoo() (Foo, error) {
	f := Foo{
		Field1: "val",
		Field2: 20,
	}
	return f, nil
}
```
- The only time you should use pointer parameters to modify a variable is when the function expects an interface. You see this pattern when unmarshalling JSON.
	- This pattern seems common since JSON operations are done frequently but this is the exception.

**Pointer Passing Performance**

**Passing a pointer into a function**

- The time to pass a pointer into a function is constant for all data sizes, roughly one nanosecond, because the size of a pointer is the same for all data types.
- Passing a value into a function takes longer as the data gets larger. It takes about a millisecond once the value gets to around 10 megabytes of data.

**Returning a pointer from a function**

- For data structures that are smaller than a megabyte, it is *slower* to return a pointer type than a value type. A 100-byte data structure take around 10 nanoseconds to be returned but a pointer to that data structure takes about 30 nanoseconds.
- For data structures larger than a megabyte, the performance flips. It takes nearly 2 milliseconds to return 10 megabytes of data but around half a millisecond to return a pointer to it.
- If you are passing megabytes of data between functions, consider using a pointer even if the data is meant to be immutable.

**The Zero Value Versus No Value**

- Resist the temptation to use a pointer field to indicate no value. If you are not going to modify the value, you should use a value type instead, paired with a boolean (ex. comma ok, comma error idioms).

**The Difference Between Maps and Slices**

**Maps**

- Maps are implemented as a pointer to a struct in the Go runtime so passing a map to a function means you are copying a pointer.
- Avoid using maps for input parameters or return values because they say nothing about what values are contained within and the only way to know is to trace through the code.
- They are also bad from an immutability standpoint because the only way to know what ended up in the map is to trace through all of the functions that interact with it.
- Rather than a map, use a struct.

**Slices**

- Slices have complicated behavior: slices modified within a function is reflected in the original variable, but using *append* fails to change the length in the original.
- Slices are implemented as structs with three fields: length (int), capacity (int), and a pointer to a block of memory.
    - When slices are passed to a function, the length, capacity, and pointer are copied.
    - Using *append* on the slice within the function, adds an element in the block of memory through the pointer and increases the length of the slice *copy*.
    - The original slice can't see the new elements added into memory because the length and capacity of the original slice are unchanged so the new elements are effectively hidden.
- By default, you should assume that a slice is not modified by a function and if it does, it should be documented in the function.

**Slices as Buffers**

- When reading data from an external resource, many languages use code like this:
```
r = open_resource()
while r.has_data() {
	data_chunk = r.next_chunk()
	process(data_chunk)
}
close(r)
```
- The problem is that every loop allocates another *data_chunk* even though each one is only used once creating unnecessary memory allocations.
- We can create a slice of bytes once and use it as a buffer instead:
```
file, err := os.Open(filename)
if err != nil {
	return err
}
defer file.Close()
data := make([]byte, 100)
for {
	count, err := file.Read(data)
	if err != nil {
		return err
	}
	if count == 0 {
		return nil
	}
	process(data[:count])
}
```

**Reducing the Garbage Collector's Workload**

- To store something on the stack, you have to know exactly how big it is at compile time.
- To allocate data a pointer points to onto the stack, there are several conditions:
    - It must be a local variable whose data size is known at compile time.
    - The pointer cannot be return from the function.
    - If the pointer is passed into a function, the compiler must be able to ensure that these conditions still hold.
- If the pointer variable is returned, the memory that the pointer points to will no longer be valid when the function exits.
- If the compiler determines that data can't be stored on the stack, the data *escaped* the stack and is stored on the heap.
- Once there are no more pointers pointing to data on the heap, the data is garbage collected.
- *Escape analysis* by the Go compiler isn't perfect and is tuned for lower latency (finish garbage scan fast) instead of higher throughput (find as much garbage as possible in a single scan) to keep response times low.
- Each garbage collection cycle is designed to take less than 500 microseconds.
- It's best to generate less garbage to be collected due to the garbage collector's emphasis on speed instead of throughput so it's best to store as much as possible on the stack.
- A slice of structs in Go can be read faster from memory than a slice of pointers to structs because the slice of structs are laid out sequentially in memory whereas pointer data is scattered across RAM. The speed difference is roughly two orders of magnitude.
- Contrast this with Java where objects are implemented as pointers meaning only the pointers are stored on the stack. Lists are actually a pointer to an array of pointers and though it looks like a linear data structure, reading it actually involves bouncing around through memory.
- Go encourages you to use pointers sparingly to reduce garbage collector workload by storing as much as possible on the stack.

## Types, Methods, and Interfaces

**Pointer Receiver and Value Receivers**

- If your method modifies the receiver, you *must* use a pointer receiver.
- If your method needs to handle *nil* instances, it *must* use a pointer receiver.
- If your method doesn't modify the receiver, you *can* use a value receiver.
- However, when a type has *any* pointer receiver methods, a common practice is to be consistent and use pointer receivers for all methods.
- Avoid writing getters and setters. Go encourages you to directly access a field.

**Code Your Methods for nil Instances**

- If a method uses a pointer receiver, handle the possibility of a *nil* instance.
```
// Binary tree in Go
type IntTree struct {
	val         int
	left, right *IntTree
}

func (it *IntTree) Insert(val int) *IntTree {
	if it == nil {
		return &IntTree(val: val)
	}
	if val < it.left {
		it.left = it.left.Insert(val)
	} else if val > it.val {
		it.right = it.right.Insert(val)
	}
	return it
}

func (it *IntTree) Contains(val int) bool {
	switch {
	case it == nil:
		return false
	case val < it.val:
		return it.left.Contains(val)
	case val > it.val:
		return it.right.Contains(val)
	default:
		return true
	}
}
```
- Pointer receivers work just like pointer function parameters; it's a copy of the pointer that's passed into the method. So if you have a *nil* pointer, you can't make the original pointer non-nil.

**Functions Versus Methods**

- If your logic depends on values configured at startup or that change while the program is running, the those values should be stored in a struct and the logic be implemented as a method.

**Types Are Executable Documentation**

- It's clearer for someone reading your code when a method has a parameter of type *Percentage* than of type *int* and it's harder to be invoked with an invalid value.

**iota is for Enumerations (Sometimes)**

- When using *iota*, the best practice is to first define a type based on *int* that will represent all of the valid values.
```
type MailCategory int

const (
	Uncategorized MailCategory = iota
	Personal
	Spam
	Social
	Advertisements
)
```
- Use *iota* only if the constants will only be referred to internally in the program. If the values are explicitly defined elsewhere, use those values instead.

**Use Embedding for Composition**

```
type Employee struct {
	Name    string
	ID      string
}

func (e Employee) Description() string {
	return fmt.Sprintf("%s (%s)", e.Name, e.ID)
}

type Manager struct {
	Employee // This is an embedded field
	Reports []Employee
}

func (m Manager) FindNewEmployees() []Employee {
	// do business logic
}

func main() {
	m := Manager {
		Employee: Employee{
			Name: "Bob Loblaw",
			ID:   "12345",
		},
		Reports: []Employee{},
	}
	fmt.Println(m.ID)            // prints 12345
	fmt.Println(m.Description()) // prints Bob Loblaw (12345)
}
```
- Any fields or methods declared on an embedded field are promoted to the containing struct and can be invoked directly on it (ex. `m.Description()`).
- If a containing struct has fields or methods with the same name as an embedded field, you need to refer to the embedded field:
```
type Inner struct {
	X int
}

type Outer struct {
	Inner
	X int
}

o := Outer {
	Inner: Inner {
		X: 10,
	},
	X: 20,
}
fmt.Println(o.X)      // prints 20
fmt.Printn(o.Inner.X) // prints 10
```

**Interfaces**

- Interfaces are usually named with "er" endings (ex. fmt.Stringer, io.Reader, io.Closer).

**Interfaces Are Type-Safe Duck Typing**

- Interfaces are implemented *implicitly*: as long as a concrete type contains all of the methods of an interface, it automatically implements that interface.
```
type LogicProvider struct {}

func (lp LogicProvider) Process(data string) string {
	// business logic
}

type Logic interface {
	Process(data string) string
}

type Client struct {
	L Logic
}

func (c Client) Program() {
	// get data from somewhere
	c.L.Process(data)
}

func main() {
	c := Client{
		L: LogicProvider{},
	}
	c.Program()
}
```
- Using interfaces encourages the *decorator pattern* where a factory function takes in an instance of an interface and returns another type that implements the same interface.

**Embedding and Interfaces**

```
type Reader interface {
	Read(p []byte) (n int, err error)
}

type Closer interface {
	Close() error
}

type ReadCloser interface {
	Reader
	Closer
}
```

**Accept Interfaces, Return Structs**

- Business logic invoked by your functions should be invoked via interfaces, but the output of your functions should be a concrete type.
- Adding new methods and fields to a returned concrete type won't break existing code.
- Adding a new method to an interface means that all existing implementations of that interface would need to be updated.
- As for garbage collection:
	- Returning a struct is good to avoid a heap allocation.
	- However, a heap allocation occurs for each interface parameter.
	- It's up to you to figure out the trade-off between better abstraction and better performance though it's often better to lean towards the side of readability and maintainability.

**The Empty Interface**

- An empty interface type (*interface{}*) just states that the variable can store any value whose type implements zero or more methods which just happens to match every type in Go.
- If you see a function takes in an empty interface, it's likely using reflection to populate or read the value.
- Avoid using *interface{}* since situations that use it should be rare.

**Type Assertions and Type Switches**

- Type assertions can only be applied to interface types and are checked at runtime while type conversions are applies to both concrete types and interfaces and checked at compile time.
- Conversions change, assertions reveal.
```
type MyInt int

func main() {
	var i interface{}
	var mine MyInt = 20
	i = mine
	i2, ok := i.(MyInt) // type assertion
	if !ok {
		// always check for errors
		return fmt.Errorf("unexpected type for %v", i)
	}
	fmt.Println(i2 + 1)
}
```
- If an interface could be multiple types, use a type switch.
```
func doThings(i interface{}) {
	// it's idiomatic to assign the variable being
	// switched to a variable of the same name.
	switch i := i.(type) {
	case nil:
		// i is nil, type is interface{}
	case int:
		// type int
	case MyInt:
		// type MyInt
	case io.Reader:
		// type io.Reader
	default:
		// unknown so i is of type interface{}
	}
}
```

**Implicit Interfaces Make Dependency Injection Easier**

- Dependency injection is the concept that your code should explicitly specify the functionality it needs to perform its task.

```
// Logger utility function
func LogOutput(message string) {
	fmt.Println(message)
}

// Data store
type SimpleDataStore struct {
	userData map[string]string
}

func (sds SimpleDataStore) UserNameForID(userID string) (string, bool) {
	name, ok := sds.userData[userID]
	return name, ok
}

// Factory function for SimpleDataStore
func NewSimpleDataStore() SimpleDataStore {
	return SimpleDataStore{
		userData: map[string]string{
			"1": "Fred",
			"2": "Mary",
			"3": "Pat",
		},
	}
}

// Business logic
// We don't want to force it to depend on LogOutput or
// SimpleDataStore in case we want to use a different
// logger or data store later. Let's depend on behavior
// instead using interfaces.

type DataStore interface {
	UserNameForID(userID string) (string, bool)
}

type Logger interface {
	Log(message string)
}

// LogOutput doesn't meet the Logger interface since it doesn't
// define the Log function. We can turn the LogOutput function
// into a function type instead to attach the Log function to it.
type LoggerAdapter func(message string)

func (lg LoggerAdapter) Log(message string) {
	lg(message)
}

// Business logic implementation

type SimpleLogic struct {
	l Logger
	ds DataStore
}

func (sl SimpleLogic) SayHello(userID string) (string, error) {
	sl.l.Log("in SayHello for " + userID)
	name, ok := sl.ds.UserNameForID(userID)
	if !ok {
		return "", errors.New("unknown user")
	}
	return "Goodbye, " + name, nil
}

// Factory function for SimpleLogic.
// Pass in interfaces and return a struct.
func NewSimpleLogic(l Logger, ds DataStore) SimpleLogic {
	return SimpleLogic{
		l:  l,
		ds: ds,
	}
}

// API

type Logic interface {
	SayHello(userID string) (string, error)
}

type Controller struct {
	l Logger
	logic Logic
}

func (c Controller) HandleGreeting(w http.ResponseWriter, r *http.Request) {
	c.l.Log("In SayHello")
	userID := r.URL.Query().Get("user_id")
	message, err := c.logic.SayHello(userID)
	if err != nil {
		w.WriteHeader(http.StatusBadRequest)
		w.Write([]byte(err.Error()))
		return
	}
	w.Write([]byte(message))
}


// Factory function for Controller
func NewController(l Logger, logic Logic) Controller {
	return Controller{
		l: l,
		logic: logic,
	}
}

// main: This is the only part of the code that knows what all
// the concrete types are. To swap in different implementations,
// this is the only place that needs to change.
// Try: http://localhost:8000/hello?user_id=1

func main() {
	l := LoggerAdapter(LogOutput)
	ds := NewSimpleDataStore()
	logic := NewSimpleLogic(l, ds)
	c := NewController(l, logic)
	http.HandleFunc("/hello", c.SayHello)
	http.ListenAndServer(":8000", nil)
}
```

- Dependency injection is a great pattern for making testing easier since you can easily swap out different environments where the inputs and outputs are constrained to validate functionality. For example, you can swap out a database implementation to use hard coded values instead resulting in faster tests.
- *Wire*: a dependency injection helper written by Google to automatically crete the concrete type declarations that we wrote ourselves in *main*.

## Errors

- Errors are favored over exception handling because the execution path is clearer.
- Since all variables must be read in Go, returned errors have to either be handled or explicitly ignored.

**Use Strings for Simple Errors**

- `errors.New("only even numbers are processed")` returns an `error`.
- `fmt.Errorf("%d isn't an even number", i)` also returns an `error` but allows you to use formatting.

**Sentinel Errors**

- Sentinel errors are one of the few variables that are declared at the package level.
- By convention, their names start with *Err* (ex. *zip.ErrFormat*).

**Errors Are Values**

- Since *error* is an interface, you can define your own errors.

```
type Status int

const (
	InvalidLogin Status = iota + 1
	NotFound
)

type StatusErr struct {
	Status Status
	Message string
}

func (se StatusErr) Error() string {
	return se.Message
}

func LoginAndGetData(uid, pwd, file string) ([]byte, error) {
	err := login(uid, pwd)
	if err != nil {
		return nil, StatusErr{
			Status: InvalidLogin,
			Message: fmt.Sprintf("invalid credentials for user %s", uid),
		}
	}
	data, err := getData(file)
	if err != nil {
		return nil, StatusErr{
			Status: NotFound,
			Message: fmt.Sprintf("file %s not found", file),
		}
	}
	return data, nil
}
```

**Wrapping Errors**

- You can wrap an error into another error to add additional context to it.

```
func fileChecker(name string) error {
	f, err := os.Open(name)
	if err != nil {
		return fmt.Errorf("in fileChecker: %w", err)
	}
	f.Close()
	return nil
}

func main() {
	err := fileChecker("not_here.txt")
	if err != nil {
		fmt.Println(err)
		if wrappedErr := errors.Unwrap(err); wrappedErr != nil {
			fmt.Println(wrappedErr)
		}
	}
}
```

- You don't usually use *errors.Unwrap* directly. Use *errors.Is* and *errors.As* to find a specific wrapped error instead.

```
func fileChecker(name string) error {
	f, err := os.Open(name)
	if err != nil {
		return fmt.Errorf("in fileChecker: %w", err)
	}
	f.Close()
	return nil
}

func main() {
	err := fileChecker("not_here.txt")
	if err != nil {
		if errors.Is(err, os.ErrNotExist) {
			fmt.Println("That file doesn't exist")
		}
	}
}
```

- The *errors.As* funciton allows you to check if a returned error matches a specific type.

```
err := AFunctionThatReturnsAnError()
var myErr MyErr
if errors.As(err, &myErr) {
	fmt.Println(myErr.Code)
}
```

- Use *errors.Is* when you are looking for a specific instance or specific values.
- Use *errors.As* when you are looking for a specific type.

**panic and recover**

- *panic* and *recover* look like exception handling but they are not intended to be used that way.
- Reserve panics for fatal situations and use recover as a way to gracefully handle those situations.


```
func div60(i int) {
	defer func() {
		if v := recover(); v != nil {
			fmt.Println(v)
		}
	}()
	fmt.Println(60 /i)
}

func main() {
	for _, val := range []int{1, 2, 0, 6} {
		div60(val)
	}
}

// Produces:
// 60
// 30
// runtime error: integer divide by zero
// 10
```

- If a panic is triggered because the computer is out of a resource like memory or disk space, the safest thing to do is use recover to log the situation and shut down with os.Exit(1).
- However, if creating a library for third parties, do not let panics escape the boundaries of your public API. If a panic is possible, a public function should use a recover to convert the panic into an error, return it, and let the calling code decide what to do with them.

**Getting a Stack Trace from an Error**

- One of the reasons why new Go developers are tempted by panic and recover is that they want to get a stack trace when something goes wrong.
- You can use error wrapping to build a call stack by hand, but there are third-party libraries with error types that generate those stacks automatically.
- The best known third-party library (https://github.com/pkg/errors) provides functions for wrapping errors with stack traces.
- If you want to to see the stack trace, use *fmt.Printf* and the verbose output verb (%+v).
- The stack trace output includes the full path to the file. Use *-trimpath* flag when building your code to replace the full path.

## Modules, Packages, and Imports

- Library management in Go is based around three concepts: repositories, modules, and packages.
- A repository is a place in a version control system where the source code for a project is stored.
- A module is the root of a Go library or application consisting of one or more packages which gives the module organization and structure.
- Before you can use code from packages outside of the standard library, make sure that you have declared that your project is a module with a globally unique identifier.
	- In go, this is the path to the module repository (ex. github.com/pkg/errors).

**go.mod**

- A collection of Go source code becomes a module when there's a valid go.mod file in its root directory.
- Run `go mod init <module_path>` to create the go.mod file that makes the current directory the root of a module.
- The *require* section lists the modules that your module depends on and the minimum version required for each one.

**Imports and Exports**

- *import* allows you to access exported constants, variables, functions, and types in another package.
- Go uses capitalization to determine if a package-lvel identifier is visible outside of the package where it is declared.
- Identifiers whose name starts with an uppercase letter is exported.
- Anything you export is part of your package's API so ensure that you intend to expose it to clients.

**Naming Packages**

- Package names should be descriptive.
- Don't create a package called *util*, for example. If it contained two functions referred to as *util.ExtractNames* and *util.FormatNames*, the package name tells you nothing about what the functions do.
- It's better to create one function called *Names* in a package called *extract* and a second function called *Names* in a package called *format* with the function calls becoming *extract.Names* and *format.Names*.

**How to Organize Your Module**

- There is no official way to structure Go packages but several patterns have emerged over the years.
- When your module is small, keep all of your code in a single package.
- If your module consists of one or more applications, create a directory called *cmd* at the root of your module.
	- Within *cmd*, create one directory for each binary built from your module.
	- For example, you might have a module that contains both a web application and a command-line tool that analyzes data in the web application's database.
	- Use *main* as the package name within each of these directories.
- If your module's root directory contains many files for managing the testing and deployment of your project (Dockerfiles, shell scripts, etc), place all of your Go code (besides the *main* packages under *cmd*) into packages under a directory called *pkg*.
- Within the *pkg* directory, organize your code to limit the dependencies between packages.
	- One common pattern is to organize your code by slices of functionality.
	- For example, in a shopping site, you might place all of the support customer management code in one package and all of the inventory management code in another.
	- This helps limit dependencies between packages making it easier to refactor into multiple microservices, if needed.

**Overriding a Package's Name**

- If packages contain conflicting names, you can override them.

```
import (
	crand "crypto/rand"
	"encoding/binary"
	"fmt"
	"math/rand"
)

func seedRand() *rand.Rand {
	var b [8]byte
	_, err := crand.Read(b[:])
	if err != nil {
		panic("cannot seed with cryptographic random number generator")
	}
	r := rand.New(rand.NewSource(int64(binary.LittleEndian.Uint64(b[:]))))
	return r
}
```
- Using the package name "." places all of the exported identifiers in the imported package into the current package's namespace but this is discouraged for clarity.

**The internal Package**

- If you need to share a function, type, or constant between packages but don't want to make them part of the API, use the *internal* package name.
- Exported identifiers in the *internal* package are only accessible to its sibling package and the direct parent package.

**The init Function: Avoid if Possible**

- Some packages, like database drivers, use *init* functions to register the database driver through the use of a blank import which uses the underscore as the package name.
	- The underscore is necessary since Go requires you to read all variables when all we need is to trigger the *init* function within the imported package and don't require access to any of the exported identifiers.
- This pattern is considered obsolete since it's unclear that a registration operation is being performed. Instead, register your plugins directly.

**Circular Dependencies**

- Circular dependencies are sometimes caused by splitting packages up too finely. If two packages depend on each other, there is a good chance they should be merged into a single package.
- If there are good reasons to keep the packages separate, move just the items causing the circular dependency to one of the two packages or to a new package.

**Gracefully Renaming and Reorganizing Your API**

- For functions or methods, just declare a function or method that calls the original.
- For constants, declare a new constant with the same type and value but with a different name.
- When you want to rename or move and exported type, use an *alias*.
```
type Foo struct {
	x int
	S string
}
```
- If you want to allow users to access Foo by the name Bar, all you need is: `type Bar = Foo`.

**Working with Versions**

- To declare a specific version in your *go.mod* file:
```
module region_tax

go 1.15

require (
	github.com/shopspring/decimal v1.2.0
)
```
- To see what versions of the module are available, use the `go list` command.
- Use *tags* in *git* to version your modules using the `git tag` command. Delete a tag using the `git tag -d` command. Checkout a *tag* using `git checkout`.
	- Tagging is generally used to capture a point in history that is used for a marked version release (ex. v1.0.1). A tag is like an immutable branch that doesn't change and will have not further history of commits.
- You can change the version of a package by running: `go get github.com/shopspring/decimal@v1.0.0`. This will change the dependency in *go.mod*.
- Dependencies labeled as *// indirect* in your *go.mod* file can be caused by dependencies on older modules that don't have *go.mod* files themselves or their *go.mod* files have and error and is missing some of its dependencies.
- Another cause of an indirect declaration is if a direct dependency properly specifies an indirect dependency, but it specifies an older version than what's installed in your project. This happens when you explicitly update an indirect dependency with `go get` or downgrade a dependency's version.

**Minimum Version Selection**

- You will always get the lowest version of a dependency that is declared to work in all of the *go.mod* files across all of your dependencies.
- If the *go.mod* files of modules A, B, and C require a module D of versions v1.1.0, v1.2.0, and v1.2.3, Go will import module D only once and will choose v1.2.3 as that is the most recent specified version.
	- This works because Go uses semantic versioning (SemVer) which states that only updates to a major version can break backward compatibility. v1 modules should be compatible whereas v2 modules may not be.

**Updating to Incompatible Versions**

- A module may have a different directory to store a new major version (ex. v2) and you can import this into your program as `import github.com/learning-go-book/simpletax/v2`.
- Your *go.mod* file will have new and old versions if different parts of your program require both modules.
- To remove unused versions, run `go mod tidy`.

**Vendoring**

- To ensure that a module always builds with identical dependencies, you can keep copies of your dependencies using *vendoring*.
- Enable vendoring by running `go mod vendor` which creates a directory called *vendor* at the top level of your modules containing all of your module's dependencies.
- If new dependencies are added to *go.mod* or versions are updated with `go get`, you need to run `go mod vendor` to update the *vendor* directory or you will get an error message on build.
- This practice is falling out of favor since it dramatically increases your project size and proxy servers now exist to cache your dependencies.

## Concurrency in Go

- Concurrency is not parallelism. Whether or not concurrent code runs in parallel depends on the hardware and if the algorithm allows it.
- Use concurrency when you want to combine data from multiple operations that can operate independently.

**Goroutines**

- A *process* is an instance of a program that's being run by a computer's operating system. The OS associates resources, such as memory, with the process and makes sure that other processes can't access them.
- A process is composed of one or more *threads*. A thread is a unit of execution that is given some time to execute by the OS scheduler. Threads within a process share acess to resources.
- Goroutines are lightweight processes managed by the Go runtime.
- Go has its own runtime scheduler to assign goroutines to OS threads. There are benefits to Go having its own scheduler:
	- Goroutine creation is faster than thread creation since you aren't creating an OS-level resource.
	- Goroutine initial stack sizes are smaller than thread stack sizes and can grow as needed making goroutines more efficient.
	- Switching between goroutines is faster than switching between threads because it happens entirely within the process instead of using OS calls that are relatively slower.
	- The scheduler can optimize its decisions since it's part of the Go process.
- You can launch thousand of goroutines simultaneously while launching thousands of threads will slow the program.

**Channels**

- Goroutines communicate using *channels*: `ch := make(chan int)`.
- When you pass a channel to a function, you are passing a pointer to the channel.
- The zero value for a channel is *nil*.

**Reading, Writing, and Buffering**

```
a := <-ch // reads a value from ch and assigns it to a
ch <- b // write the value in b to ch
```
- Channels are unbuffered by default. A read from an open, unbuffered channel causes the reading goroutine to pause until another goroutine writes to the same channel. This means that you cannot write to or read from an unbuffered channel without at least two concurrently running goroutines.
- Buffered channels allow for a limited number of writes without blocking. If the buffer fills before there are any reads from the channel, a subsequent write to the channel pauses the writing goroutine until the channel is read.
- Reading from a channel with an empty buffer also blocks.
- To create a buffered channel: `ch := make(chan int, 10)`.
- Most of the time, use unbuffered channels.

**for-range and Channels**

- You can read from a channel using a for-range loop:
```
for v := range ch {
	fmt.Println(v)
}
```
- The loop continues until the channel is closed or a break or continue statement is reached.

**Closing a Channel**

- When done writing to a channel, close it: `close(ch)`.
- Attempting to write to a closed channel will trigger a panic but reads will always succeed.
- Reading an empty channel will result in the zero value for the channel's type.
- To determine if a channel is closed, use the comma ok idiom: `v, ok := <-ch` where if ok is true, the channel is open. Use this any time you are reading from a channel that might be closed.
- The responsibility for closing a channel lies with the goroutine that writes to the channel.

**select**

- The *select* statement is the control structure for concurrency in Go.
- The *select* keyword allows a goroutine to read from or write to one of a set of multiple channels.
```
select {
case v := <-ch:
	fmt.Println(v)
case v := <-ch2:
	fmt.Println(v)
case ch3 <- x:
	fmt.Println("wrote", x)
case <-ch4:
	fmt.Println("got value on ch4, but ignored it")
}
```
- Each case in a *select* is a read or a write to a channel and each case creates its own block.
- A *select* checks its cases at random to see if any can proceed, which helps mitigate deadlocks.
- *select* statements are often run in a *for* loop often called a *for-select* loop:
```
for {
	select {
		case <-done:
			return
		case v := <-ch:
			fmt.Println(v)
		default:
			// defaults in a for-select loop are almost always the wrong thing to do
			// since it will be triggered every time through the loop causing the
			// loop to run constantly, taking up CPU cycles.
			fmt.Println("no value written to ch")
	}
}
```

**Concurrency Practices and Patterns**

**Keep your APIs Concurrency-Free**

- Never expose (export) channels or mutexes in your API's types, functions, and methods.
- If you expose a channel, you put the responsibility of channel management on the users of your API so now they have to be concerned about whether or not a channel is buffered or closed or nil.

**Goroutines, for Loops, and Varying Variables**

- If you want to avoid shadowing and make the data flow more obvious, you can pass a value to a goroutine as a parameter:
```
for _, v := range a {
	go func(val int) {
		ch <- val * 2
	}(v)
}
```
- This behavior changed in Go version 1.22 where variables in for loops, like in *v* above, have per-iteration scope instead of per-loop scope since this is a common bug that developers make.

**Always Clean Up Your Goroutines**

- When launching a goroutine, make sure that it will eventually exit since the Go runtime can't detect if a goroutine will never be used again so it will remain active. This is called a *goroutine leak*.
```
func countTo(max int) <-chan int {
	ch := make(chan int)
	go func() {
		for i := 0; i < max; i++ {
			ch <- i
		}
		close(ch)
	}()
	return ch
}

func main() {
	for i := range countTo(10) {
		if i > 5 {
			break
		}
		fmt.Println(i)
	}
}
```
- The *break* statement exits the loop early causing the goroutine to block forever, waiting for a value to be read from the channel.

**The Done Channel Pattern**

- Provides a way to signal a goroutine that it's time to stop processing by using a channel to signal that it's time to exit.
```
func searchData(s string, searchers []func(string) []string) []string {
	done := make(chan struct{}) // using an empty struct since the value is unimportant
	result := make(chan []string)
	// We only want the result from the fastest function
	// so close done as soon as a result is read.
	for _, searcher := range searchers {
		go func(searcher func(string) []string) {
			select {
			case result <- searcher(s):
			case <- done:
			}
		}(searcher)
	}
	r := <-result
	close(done)
	return r
}
```

**Using a Cancel Function to Terminate a Goroutine**

- You can also use the *done channel pattern* to return a cancellation function alongside a channel.
```
func countTo(max int) (<-chan int, func()) {
	ch := make(chan int)
	done := make(chan struct{})
	cancel := func() {
		close(done)
	}
	go func() {
		for i := 0; i < max; i++ {
			select {
			case <-done:
				return
			default:
				ch <- i
			}
		}
		close(ch)
	}()
	return ch, cancel
}

func main() {
	ch, cancel := countTo(10)
	for i := range ch {
		if i > 5 {
			break
		}
		fmt.Println(i)
	}
	cancel()
}
```

**When to Use Buffered and Unbuffered Channels**

- Buffered channels are useful when you known how many goroutines you have launched, want to limit the number of goroutines you will launch, or want to limit the amount of work that is queued up.
```
func processChannel(ch chan int) []int {
	const conc = 10
	results := make(chan int, conc)
	for i := 0; i < conc; i++ {
		go func() {
			results <- process(v)
		}()
	}
	var out []int
	for i := 0; i < conc; i++ {
		out = append(out, <-results)
	}
	return out
}
```

**Backpressure**

- You can use a buffered channel and a *select* statement to limit the number of simultaneous requests in a system.
- We can create a struct that contains a buffered channel with a number of "tokens" and a function to run. Every time a goroutine wants to use the function, it calls *Process*. The *select* tries to read a token from the channel. If it can, the function runs, and the token is returned. If it can't read a token, the *default* case runs and an error is returned instead.
```
type PressureGauge struct {
	ch chan struct{}
}

func New(limit int) *PressureGauge {
	ch := make(chan struct{}, limit)
	for i := 0; i < limit; i++ {
		ch <- struct{}{}
	}
	return &PressureGauge{
		ch: ch,
	}
}

func (pg *PressureGauge) Process(f func()) error {
	select {
	case <-pg.ch:
		f()
		pg.ch <- struct{}{}
		return nil
	default:
		return errors.New("no more capacity")
	}
}

func doThingThatShouldBeLimited() string {
	time.Sleep(2 * time.Second)
	return "done"
}

func main() {
	pg := New(10)
	http.HandleFunc("/request", func(w http.ResponseWriter, r *http.Request) {
		err := pg.Process(func() {
			w.Write([]byte(doThingThatShouldBeLimited()))
		})
		if err != nil {
			w.WriteHeader(http.StatusTooManyRequests)
			w.Write([]byte("Too many requests"))
		}
	})
	http.ListenAndServe(":8000", nil)
}
```

**Turning Off a case in a select**

- If one of the cases in a *select* is reading a closed channel, it will always be successful, returning the zero value. Every time that case is selected, you need to check to make sure that the value is valid and skip the case. If reads are spaced out, your program is going to waste a lot of time reading junk values. For this case, you can use a *nil* channel to disable a *case* in a *select*.
- When you detect that a channel has been closed, set the channel's variable to *nil*. The associated case will no longer run because the read from the *nil* channel never returns a value:
```
// in and in2 are channels, done is a done channel.
for {
	select {
	case v, ok := <-in:
		if !ok {
			in = nil // the case will never succeed again!
			continue
		}
		// process the v that was read from in
	case v, ok := <-in2:
		if !ok {
			in2 = nil // the case will never succeed again!
			continue
		}
		// process the v that was read from in2
	case <-done:
		return
	}
}
```

**How to Time Out Code**

- Any time you need to limit how long an operation takes in Go, you'll see a variation on this pattern:
```
func timeLimit() (int, error) {
	var result int
	var err error
	done := make(chan struct{})
	go func() {
		result, err = doSomeWork()
		close(done)
	}()
	select {
	case <-done:
		return result, err
	case <-time.After(2 * time.Second):
		return 0, errors.New("work timed out")
	}
}
```

**Using WaitGroups**

- If you are waiting on several goroutines, you need to use a *WaitGroup*:
```
func main() {
	var wg sync.WaitGroup
	wg.Add(3)
	go func() {
		defer wg.Done()
		doThing1()
	}()
	go func() {
		defer wg.Done()
		doThing2()
	}()
	go func() {
		defer wg.Done()
		doThing3()
	}()
	wg.Wait()
}
```
- Don't explicitly pass the *sync.WaitGroup* since we need to ensure that every place using it is using the same instance. Also, passing the *sync.Waitgroup* to a goroutine function passes a copy if a pointer is not used so the call to *Done* won't decrement the original sync.WaitGroup count.
- Use a close to capture the *sync.WaitGroup* to ensure that every goroutine is referring to the same instance.
- When you have multiple goroutines writing to the same channel, you need to make sure that the channel being written to is only used once:
```
func processAndGather(in <-chan int, processor func(int) int, num int) []int {
	out := make(chan int, num)
	var wg sync.WaitGroup
	wg.Add(num)
	for i := 0; i < num; i++ {
		go func() {
			defer wg.Done()
			for v := range in {
				out <- processor(v)
			}
		}()
	}
	go func() {
		wg.Wait()
		close(out)
	}()
	var result []int
	for v := range out {
		result = append(result, v)
	}
	return result
}
```
- In the above example, a monitoring goroutine waits until all of the processing goroutines exit. When they do, the monitoring goroutine calls *close* on the output channel. The *for-range* channel loop exits when *out* is closed and the buffer is empty. Finally, the function returns the processed values.
- While *WaitGroups* are handy, they shouldn't be your first choice when coordinating goroutines. Use them only when you have something to clean up (like closing a channel they all write to) after all of your worker goroutines exit.

**Running Code Exactly Once**

- Sometimes you want to lazy load, or call some initialization code exactly once after program launch time. This is usually because the initialization is relatively slow and may not even be needed every time your program runs. The *sync* package includes a handy type called *Once* that enables this functionality.
```
type SlowComplicatedParser interface {
	Parse(string) string
}
var parser SlowComplicatedParser
var once sync.Once

func Parse(dataToParse string) string {
	once.Do(func() {
		parser = initParser()
	})
	return parser.Parse(dataToParse)
}

func initParser() SlowComplicatedParser {
	// do all sorts of setup and loading here
}
```
- In the above example, we want to ensure that *parser* is only initialized once, so we set the value of *parser* from within a closure that's passed to the *Do* method on *once*. If *Parse* is called more than once, *once.Do* will not execute the closure again.

**When to Use Mutexes Instead of Channels**

- The main problem with mutexes is that they obscure the flow of data through a program. In contrast, when a value is passed from goroutine to goroutine over a series of channels, the data flow is clearer.
- When a mutex is used to protect a value, there is nothing to indicate which goroutine currently has ownership of the value because access to the value is shared by all of the concurrent processes making it hard to understand the order of processing.
	- "Share memory by communicating; do not communcate by sharing memory."
- However, sometimes it is clearer to use a mutex.
	- The most common case is when your goroutines read or write a shared value, but don't process the value.
- How to decide whether to use channels or mutexes:
	- If you are coordinating goroutines or tracking a value as it is transformed by a series of goroutines, use channels.
	- If you are sharing access to a field in a struct, use mutexes.
	- If you discover a critical performance issue when using channels and you cannot find any other way to fix the issu, modify your code to use a mutex.
- Mutexes must never be copied. If a mutex is copied, its lock won't be shared.
- Never try to access a variable from multiple goroutines unless you acquire a mutext for that variable first since it can cuase odd errors that are hard to trace.

## The Context

- Context is a way to handle metadata for individual requests.
- It can be used to identify a chain of requests with a tracking ID or to set a timer that ends a request that takes too long.
- A context is an instance that meets the *Context* interface defined in the context package.
- Idiomatic Go encourages passing the context explicitly and convention has it as the first parameter of a function.
- If an existing context doesn't exist, you can create an empty one with *context.Background* which returns a variable of type *context.Context*.
- Each time you add metadata to the context, you do so by wrapping the existing context with *WithContext*.
- *WithContext* takes in a *context.Context* and returns a new http.Request with the old request's state combined with the supplied *context.Context*.
```
type userKey int
const key userKey = 1

// The name of the function that creates a context with a value should
// start with ContextWith.
func ContextWithUser(ctx context.Context, user string) context.Context {
	return context.WithValue(ctx, userKey, user)
}

// The function that returns the value from the context should have a
// name that ends with FromContext.
func UserFromContext(ctx context.Context) (string, bool) {
	user, ok := ctx.Value(userKey).(string)
	return user, ok
}

// Not a real implementation for user authentication.
func extractUser(req *http.Request) (string, error) {
	userCookie, err := req.Cookie("user")
	if err != nil {
		return "", err
	}
	return userCookie.Value, nil
}

func Middleware(handler http.Handler) http.Handler {
	return http.HandlerFunc(func(rw http.ResponseWriter, req *http.Request) {
		user, err := extractUser(req)
		if err != nil {
			rw.WriteHeader(http.StatusUnauthorized)
			return
		}
		ctx := req.Context()
		ctx = ContextWithUser(ctx, user)
		req = req.WithContext(ctx)
		handler.ServeHTTP(rw, req)
	})
}

func (c Controller) handleRequest(rw http.ResponseWriter, req *http.Request) {
	ctx := req.Context()
	user, ok := identity.UserFromContext(ctx)
	if !ok {
		rw.WriteHeader(http.StatusInternalServerError)
		return
	}
	data := req.URL.Query().Get("data")
	result, err := c.Logic.businessLogic(ctx, user, data)
	if err != nil {
		rw.WriteHeader(http.StatusInternalServerError)
		rw.Write([]byte(err.Error()))
		return
	}
	rw.Write([]byte(result))
}
```
- One more situation that uses the *WithContext* method is when passing a context to an outgoing request:
```
req, err := http.NewRequest(http.MethodGet, "http://example.com?data="+data, nil)
if err != nil {
	return "", err
}
req = req.WithContext(ctx)
resp, err := sc.client.Do(req)
if err != nil {
	return "", err
}
// Do other stuff
```

**Cancellation**

- Create a cancellable context with *context.WithCancel* which takes a *context.Context* as a parameter and returns a child context of type *context.Context* and a *context.CancelFunc* that is used to cancel the context.
- Any time a cancellable context is created, the cancel function must be called. It's fine to call it multiple times since subsequent calls after the first are ignored.
- Use a *defer* to ensure the call.
```
var client = http.Client{}
func callBoth(ctx context.Context, errVal string, slowURL string, fastURL string) {
	ctx, cancel := context.WithCancel(ctx)
	defer cancel()
	var wg sync.WaitGroup
	wg.Add(2)
	go func() {
		defer wg.Done()
		err := callServer(ctx, "slow", slowURL)
		if err != nil {
			cancel()
		}
	}()
	go func() {
		defer wg.Done()
		err := callServer(ctx, "fast", fastURL+"?error="+errVal)
		if err != nil {
			cancel()
		}
	}()
	wg.Wait()
	fmt.Println("done with both")
}
```

**Timers**

- A server can manage its load by:
	- Limiting simultaneous requests.
	- Limiting how many requests are queued waiting to run.
	- Limiting how long a request can run.
	- Limiting the resources a request can use (such as memory or disk space).
- The context provides a way to control how long a request runs with *context.WithTimeout* and *context.WithDeadline*.
	- *context.WithTimeout* takes an existing context and a *time.Duration* that specifies the duration until the context is canceled.
	- *context.WithDeadline* takes an existing context and a *time.Time* that specifies the time when the context is canceled.
- You control how long an individual call takes by creating a child context that wraps a parent context using *context.WithTimeout* or *context.WithDeadline*.
	- Any timeout that you sent on the child context is bounded by the timeout set on the parent context; if a parent context times out in two seconds and the child context times out in three seconds, the child will still time out when the parent does.
```
ctx := context.Background()
parent, cancel := context.WithTimeout(ctx, 2*time.Second)
defer cancel()
child, cancel2 := context.WithTimeout(parent, 3*time.Second)
defer cancel2()
start := time.Now()
<-child.Done()
end := time.Now()
fmt.Println(end.Sub(start))
// prints "2s"
```

**Values**

- Values are most commonly entered into the context in middleware.
	- Possible situations are extracting a user from a JWT or creating a per-request GUID that is passed through multiple layers of middleware and into your handler and business logic.
- You enter values into the context using *context.WithValue* which takes in a context to wrap, a key to look up the value, and the value itself.
- To check if a value is in a context or any of its parents, use the *Value* method on *context.Context*.
	- Use the comma ok idiom to type assert the returned value to the correct type.

**Keys**

- Like the key for a map, the key for a context value must be comparable.
- If you use a string or another public type for the type of the key, different packages could create identical keys, resulting in collisions.
- Instead, create a new, unexported type for the key, based on an int then declare an unexported constant of that type:
```
type userKey int
cont key userKey = 1
```
- With both the type and the constant of the key being unexported, no code from outside of your package can put data into the context that would cause a collision.
- If you need multiple keys, use the *iota* pattern.




























































