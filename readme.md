# Engineering Management
## Process Planning (SDLC)
## Estimation

# Requirements
## Software Requirements Engineering

# Design
## OOD :bulb:
### GoF Design Patterns
#### Creational
##### Singleton
Asserts that the whole application will have single instance of certain object
and provides global access point to that object.
The access to the object goes through the "GetInstance" method.

Just a fancy global - unsafe and has side effects, hides the dependencies of
application in code instead of exposing them

![img](https://refactoring.guru/images/patterns/diagrams/singleton/structure-en.png)

```go
func GetInstance() *Singleton {
	once.Do(func() {
		instance = &Singleton{}
	})
	return instance
}
```

##### Factory Method
Defines an interface (with method "Create") that allows to create different
implementations of object (Product).

![img](https://refactoring.guru/images/patterns/diagrams/factory-method/structure.png)

##### Builder
Allows to create objects with different options. Every Builder's method
returns Builder itself, so it's convenient to chain methods. There is
also might be a Director - struct that contains builder and has method "Construct"
that creates Builder's instance with predefined options, but this is optional.

![img](https://refactoring.guru/images/patterns/diagrams/builder/structure.png)

##### Prototype
Allows to copy object with it's state. Defines an interface with method "Clone"
that returns same interface. Under the hood implementation creates new instance with
same fields as Prototype.

![img](https://refactoring.guru/images/patterns/diagrams/prototype/structure-prototype-cache.png)

##### Abstract Factory
Just like Factory Method, AF defines an interface, but instead of single method it has
bunch of methods like "CreateCarcass", "CreateEngine", "CreateWheel" of course
it has to be implemented in various ways (sportcar, pickup, truck).

The main point is that end users are not aware of what exact implementation they use.
Instead, correct concrete factory will be chosen at runtime at the initialization stage.
The app must select the factory type depending on the configuration or the environment settings.

![img](https://refactoring.guru/images/patterns/diagrams/abstract-factory/structure.png)

#### Structural
##### Adapter
The main point: it transforms object of one type to another.

Adapter doesn't have any strict form (interface or something). And can be implemented
in various ways.

E.g it might be a struct that contains existing and target objects via composition
and owerrides target object's methods, after that access to the target performs through
that struct.

Can be a struct that implements target interface and contains existing. (most popular)
Can be a simple function that accepts existing and returns target.

```go
// Target provides an interface with which the system should work.
type Target interface {
	Request() string
}

// Adaptee implements system to be adapted.
type Adaptee struct {
}

// NewAdapter is the Adapter constructor.
func NewAdapter(adaptee *Adaptee) Target {
	return &Adapter{adaptee}
}

// SpecificRequest implementation.
func (a *Adaptee) SpecificRequest() string {
	return "Request"
}

// Adapter implements Target interface and is an adapter.
type Adapter struct {
	*Adaptee
}

// Request is an adaptive method.
func (a *Adapter) Request() string {
	return a.SpecificRequest()
}
```

![img](https://refactoring.guru/images/patterns/diagrams/adapter/structure-object-adapter.png)
![img](https://refactoring.guru/images/patterns/diagrams/adapter/structure-class-adapter.png)

##### Bridge
Mockery
Lets split a large class or a set of closely related classes into separate
hierarchies-abstraction and implementation which can be developed independently of each other.

Structure contains different objects over composition and interacts with them. Use the pattern
when you need to extend a class in several orthogonal (independent) dimensions.

![img](https://refactoring.guru/images/patterns/diagrams/bridge/structure-en.png)

##### Composite
Filetree.
Fuck you composite

```go
// Component provides an interface for branches and leaves of a tree.
type Component interface {
	Add(child Component)
	Name() string
	Child() []Component
	Print(prefix string) string
}
```

##### Decorator
Lets attach new behaviors to objects
by placing these objects inside special wrapper objects that contain the behaviors

```go
// implements io.Writer
type TransportWriter struct{}
func (TransportWriter) Write(p []byte) (n int, err error) { /*...*/ }

func NewCompressionWriter(w io.Writer) io.Writer {
	return CompressionDecorator{w}
}
type CompressionDecorator struct{ writer io.Writer }
func (c CompressionDecorator) Write(p []byte) (n int, err error) {
	// compress than write
	return c.Write(p)
}
```

![img](https://refactoring.guru/images/patterns/diagrams/decorator/structure.png)

##### Proxy
Proxy intercepts calls to the real object before or after and can extend, perform validation, cache etc.
Similar to decorator, but Decorator get reference for decorated object (usually through constructor)
while Proxy responsible to do that by himself. Proxy implements objects interface.

```go
// implements io.Writer
type TransportWriter struct{}
func (TransportWriter) Write(p []byte) (n int, err error) { /*...*/ }

// creates transportWriter
func NewCompressionProxy() io.Writer {
	transportWriter := new(TransportWriter)
	return CompressionProxy{transportWriter}
}

type CompressionProxy struct{ writer io.Writer }
func (c CompressionProxy) Write(p []byte) (n int, err error) {
	// compress than write
	return c.Write(p)
}
```

![img](https://refactoring.guru/images/patterns/diagrams/proxy/structure.png)

##### Facade
Incapsulation of complex staff (hierarchy) behind simple interface.

![img](https://refactoring.guru/images/patterns/diagrams/facade/structure.png)

##### Flyweight
Just a fancy cache.
Let's say you have ton of similar objects. You are creating a Flyweight (factory) that
responsible for creatint objects with given parameters, but before return such object it
saves it to underlying array and next time you'll need it, Flyweight
will get it from that array.

#### Behavioral
##### Chan of responisibility
lets pass requests along a chain of handlers. Upon receiving a request, each handler
decides either to process the request or to pass it to the next handler in the chain.

- If one part of chain fails, no further processing performs.
- Different way - stop processing on success.

![img](https://refactoring.guru/images/patterns/diagrams/chain-of-responsibility/structure.png)

##### Command
Sender -> Command -> Receiver

Creates middle layer set of commands where every command represented by its own class
with common interface. (usually with one method “Execute()”) It requires additional layer
to bound Commands to Senders. On initialization, Command gets Receiver’s instance.
After that command may be assigned to Sender. Sender stores a list of Commands, and methods
to add, remove and execute them, this allows dinamically change Sender’s behavior.

```
Senders
[] [] [] [] [] []
    Commands
    [] [] []
       Receiver
       []
```

##### Iterator
Moves iteration logic to its own object. Allows to easily implement different iteration
methods for COMPLEX DATA.

There is usually just interface with methods like `Next()`, `HasNext()`, `GetValue()` etc.

I guess Go's `sort.Interface` is pretty fits into this pattern. (But actually not really)

```go
// A type, typically a collection, that satisfies sort.Interface can be
// sorted by the routines in this package. The methods require that the
// elements of the collection be enumerated by an integer index.
type Interface interface {
	// Len is the number of elements in the collection.
	Len() int
	// Less reports whether the element with
	// index i should sort before the element with index j.
	Less(i, j int) bool
	// Swap swaps the elements with indexes i and j.
	Swap(i, j int)
}
```

##### Mediator
This one is FAAAT.

The pattern restricts direct communications between the objects and forces them to collaborate only via a mediator object.

Mediator contains all initialized objects that has to communicate with each other and each object contains Mediator's instance.

The Mediator is about the interactions between "colleague" objects who don't know each other.
encapsulates the interaction between several colleague objects in order to isolate them from each other.

![img](https://refactoring.guru/images/patterns/diagrams/mediator/structure.png)

##### Memento
Lets save and restore the previous state of an object.

![img](https://refactoring.guru/images/patterns/diagrams/memento/structure1.png)

##### Observer
Lets you define a subscription mechanism to notify multiple objects about any events that happen to the object they’re observing.

![img](https://refactoring.guru/images/patterns/diagrams/observer/structure.png)

##### State
Lets an object alter its behavior when its internal state changes. It appears as if the object changed its class.

There is method like `SetState(IState)` and everything method does goes through it's state.

E.g phone call mode (silent, vibration, sound).

##### Strategy
Ugh the same as state.

- The Strategy pattern is really about having a different implementation that accomplishes (basically) the same thing, so that one implementation can replace the other as the strategy requires. For example, you might have different sorting algorithms in a strategy pattern. The callers to the object does not change based on which strategy is being employed, but regardless of strategy the goal is the same (sort the collection).
- The State pattern is about doing different things based on the state, while leaving the caller relieved from the burden of accommodating every possible state. So for example you might have a getStatus() method that will return different statuses based on the state of the object, but the caller of the method doesn't have to be coded differently to account for each potential state.

- The Strategy pattern deals with HOW an object performs a certain task -- it encapsulates an algorithm.
- The State pattern deals with WHAT (state or type) an object is (in) -- it encapsulates state-dependent behavior, whereas

##### Template Method

## DB Design
## Modeling
## Security
## Algorithms

# Core
## Programming language :bulb:
### Interfaces: Interface Values, Type Assertions
https://research.swtch.com/interfaces
https://github.com/teh-cmc/go-internals/blob/master/chapter2_interfaces/README.md

Interface is an abstract data type, that doesn't expose the representation
of internal structure. When you have a value of an interface type, you know
nothing about what it is, you know only what it can do.

```go
// src/runtime/runtime2.go
type iface struct {
    tab  *itab
    data unsafe.Pointer // underlying data
}

type itab struct {
    inter *interfacetype
    _type *_type     // metadata describing type
    hash  uint32     // copy of _type.hash. Used for type switches.
    _     [4]byte
    fun   [1]uintptr // variable sized. fun[0]==0 means _type does not implement inter.
                     // (describes if interface is implemented) (but i'm not sure)
}

// src/runtime/type.go
type interfacetype struct {
    typ     _type
    pkgpath name
    mhdr    []imethod // probably methods and their types
}
```

Interface values may be compared using == and !=. Two interface values are equal if both are
nil, or if their dynamic types are identical and their dynamic values are equal according to the
usual behavior of == for that type. Because interface values are comparable, they may be used
as the keys of a map or as the operand of a switch statement. However, if two interface values
are compared and have the same dynamic type, but that type is not comparable (a slice, map or
function for instance), then the comparison fails with a panic.

A nil interface value, which contains no value at all, is not the same as an interface value
containing a pointer that happens to be nil:

``` go
type xi interface{}
var a *string = nil
var x xi = a
fmt.Println(x == nil) // false
```

### Stack and heap
https://segment.com/blog/allocation-efficiency-in-high-performance-go-services/

Heap - global programm memory. If a function creates a variable and returns reference to it,
the variable allocates in heap.

Stack - local memory allocated per function. It has its own top that moves up and down for each nested
call. This memory is freed once function is returned.

Go prefers allocation on the stack — most of the allocations within a given Go program will be on the
stack. It’s cheap because it only requires two CPU instructions: one to push onto the stack for allocation,
and another to release from the stack. On the other side, heap allocations are expensive, `malloc` has to
search for a chunk of free memory large enough to hold the new value and the garbage collector
scans the heap for objects which are no longer referenced.

Compiler performs `escape analysis` - set of rules that variable must pass on compilation stage
to be allocated in stack. Stack allocation requires that the lifetime and memory footprint of a
variable can be determined at compile time.

### Maps
https://dave.cheney.net/2018/05/29/how-the-go-runtime-implements-maps-efficiently-without-generics

https://github.com/golang/go/blob/master/src/runtime/map.go
```go
// A map is just a hash table. The data is arranged
// into an array of buckets. Each bucket contains up to
// 8 key/elem pairs. The low-order bits of the hash are
// used to select a bucket. Each bucket contains a few
// high-order bits of each hash to distinguish the entries
// within a single bucket.
```

Default underlying array size == 8, so default LOB == 3 (2^3 = 8), as map grows
it doubles it underlying array and increments LOB (2^4 = 16) and so on.

```go
// A bucket for a Go map.
type bmap struct {
	// tophash generally contains the top byte of the hash value
	// for each key in this bucket. If tophash[0] < minTopHash,
	// tophash[0] is a bucket evacuation state instead.
	tophash [bucketCnt]uint8
	// Followed by bucketCnt keys and then bucketCnt elems.
	// NOTE: packing all the keys together and then all the elems together makes the
	// code a bit more complicated than alternating key/elem/key/elem/... but it allows
	// us to eliminate padding which would be needed for, e.g., map[int64]int8.
	// Followed by an overflow pointer.
}
```
**Why are they unordered?**
- It's more efficient, runtime don't need to remember order.
- They intentionally made it random starting with Go 1 to make developers not rely on it. Since the release of Go 1.0, the runtime has randomized map iteration order. Programmers had begun to rely on the stable iteration order of early versions of Go, which varied between implementations, leading to portability bugs. Iteration order which order may change from release-to-relase, from platform-to-platform, or may even change during a single runtime of an app when map internals change due to accommodating more elements.
- Regarding the last point: during map growth its buckets gets reordered that would cause to different orders during runtime. Anyway, even without reordering if it would loop like `bucket -> elements -> next bucket -> elements...`, than after adding new element it would appear in the middle of order because you newer know in which bucket it will go.

**Where actual values are stored in memory?**
The "where" is specified by hmap.buckets. This is a pointer value, it points to an array in memory, an array holding the buckets. (NOTICE: it isn't `*bmap` or `bmap` it's `unsafe.Pointer` what says that it a bit more complicated than i thought)

```
[8]tophash -> [8]key -> [8]value -> overflowpointer
```

## Tools & Ecosystem :bulb:
## Go Concurrency :bulb:
## Networking :bulb:

# Back-End
## Web and Application Servers
## Cloud-based Deployment Services

# DB
## SQL :bulb:
## NoSQL
## Database (practice) :bulb:

# Verification
## Code quality
## Refactoring
## Tests, Trace, Profile :bulb:
### Benchmark
### Profiling concepts
### Profiling tools (pprof, http/pprof, profile)
There are two pprof implementations available:

- net/http/pprof
- runtime/pprof

``` go
import "runtime/pprof"

file, _ := os.Create(filepath.Join("cpu.pprof")) // create a file for profiler output
pprof.StartCPUProfile(file)                      // start CPU Profiler
pprof.StopCPUProfile()                           // write results to the file
```

Web view:

``` shell
go tool pprof -http=:8080 cpu.pprof
```

# Configuration Management
## Product builds and Continuous Integration
## Managing versions
