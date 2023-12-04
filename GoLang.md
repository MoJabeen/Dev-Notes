---
author:
  - Dainish Jabeen
bibliography: []
date: 2023-10-01
title: Golang Cheat Sheet
---

# Packages

Go uses packages, which can contain multiple files. **The app will start running in the main application.** Names exported outside the packages, must use a Capital letter. **Golang will auto connect any package in different files.**

```go
	package main

	import "fmt"

	func main(){
		fmt.Println("Hello World")
	}
```

**Can use go run . to run a package but should build for production.**

# Basics

## Types

```go

:= //Declare and initialize non explicit type
>> //Shift bitwise right
<<  //Shift bitwise left

//ARRAY
name []string 
var := []string{"blah","meh"}

//MAPS
map[key type]val type //Dict has to be made
m := make(map[string]string)

m["pi"] = 3.14 //Add to map

elem, ok = m[key] // check key exists

// Initialization

var i int // initializes as 0

// Constants

const(
	x=1 // the type of this can change on context
)

p := &i //point to i
*p //value of i, changes will change also change i

type Name struct{
	x int
	y int
}

// Pointers to structs

v := StructName{1,2}

p = &v
p.x = //Will change the value of v

// Struct constructors

v := StructName{x:1} //Others members made 0

//SLICES

//Slices acts as pointers

a[1:] // slice to end
s := a[:3] // slice start to 3

cap(s) // Capacity, elements in underlying array

// Dynamically size arrays

a := make(type,len,cap)
append(arr,val)

```

## Functions

```go

func Name(name type) type{}

//FUNC PARAMS

func name(x int,y int)
func name(x,y int)

//NAKED RETURN

func() (x,y int) {
	x:=1
	y:=2
	return
} // Will return x and y 

```

### Funcs as params

```go
func compute(fn func(float64, float64) float64) float64 {
return fn(3, 4)
}

func main() {
	hypot := func(x, y float64) float64 {
		return math.Sqrt(x*x + y*y)
	}
	fmt.Println(hypot(5, 12))

	fmt.Println(compute(hypot))
	fmt.Println(compute(math.Pow))
}
```

### Funcs as closures

Reference var from outside the body and be bound to that value, which when called includes any prev changes to those vars.

Allows a state of memory via an assigned variable to a func of the internal variables of that func.

```go

func adder() func(int) int {
	sum := 0
	return func(x int) int {
		sum += x
		return sum
	}
}

func main() {
	pos, neg := adder(), adder()
	for i := 0; i < 10; i++ {
		fmt.Println(
			pos(i),
			neg(-2*i),
		)
	}
}

```

## Control

```go

for i:=0;i<10;i++{}

for x<100 {} //Same as while loop

if statement; cond {}

switch statement; val {

	case x: //x same as val == x

	case y: //y same as val == y

	default:
}

defer expr //execute expr at the end of func, can stack defers

for i,v := range arr {} // Loops through array (can do _,v or i,_)

```

# Methods

Function with a receiver argument, ie accessed through an object (normally a struct). Generally a type through which the function allows for polymorphism.

```go

type blah struct{
	x int
}

func (v blah) Name() int {} // v can access all members of the struct.

func (v *blah) Name() int {} //Pointer allows changing of structs.

// v.Name() interpreted as &v.Name()

```

-   Pointers good for performance as copying is avoided
-   **Should avoid mixing methods of val and pointers**

## Interfaces

Interfaces are a type of data through which you can access a set defined number of methods. An interface types has the methods listed implemented.

This can:

-   Define behaviour of an object (a type that uses a chosen interface must also have accompanying methods)
-   Decouple code, allow use of methods without needing to know the details of the objects

```go

type Meh interface {
	name() string
}

type Float float64
type Strig string

func (f Float) name() string {
	return string(f)
}

func (s Strig) name() string{
	return s
}

func main(){

	//This variable can now be used to access either methods name
	var a Meh

	a = 99.99
	a.name()

	a = "meh"

	a.name()
}

```

-   polymorphism same func does different things based on interface

```go

type Name interface{
	method()
}

interface{} // Empty interface can hold any type

//Use known type from interface object with assertion
val.(type)

// Type Assertion

var := interface{} = "h"

s := var.(string) // Gives the value in the interface
s := var.(int) // Causes panic

s,ok := i.(string) // Check

switch v := i.(type){
	case int:

	case string:
}

```

Fmt uses Stringer interface, that allows the method String() to be accessed via new types.

- Allow use of unknown types by using an empty interface as all types implement no methods
	- To access the underlying value need to use a type assertion

```go

func ex(i interface{}) {

	//i can be any type
	fmt.Println(i)
	
	s := i.(string)
	fmt.Println(s)
}
```

# Error

Error method can be extended to use new types, which is auto called if the return type is error.

```go

// Kind of a struct with only a float stored
type ErrNegativeSqrt float64 

func (e ErrNegativeSqrt) Error() string {
	return fmt.Sprint("cannot Sqrt negative number:",
	float64(e))
}

func Sqrt(x float64) (float64, error) {
	if x > 0{
		return x*x,nil
	}else {
		return 0,ErrNegativeSqrt(x)
	}
}

func main() {
	fmt.Println(Sqrt(2))
	fmt.Println(Sqrt(-2))
}

```

# Type Parameters

Can use T as a generic type in parameters with a constraint.

```go

//s can be any type that is comparable
func Name[T comparable](s T) int{} 

type[T any] struct{
	next *List[T]
	val
} 

v := List[int]{nil,3} // Before use need to define T
```

# Concurrency

## Go routines

Lightweight thread managed by the Go routine.

$$go\;f(x,y,z)$$

Eval of f happens in current routine, execution happens in new routine. Happens in the same address space.

## Channels

Type conduit to send/receive values through the routines.

$$ch\;<-v,\;v:=<-ch$$

The data flows in arrows direction. Like maps and slices need to make before use, send and receive will block until channel is ready. Add length to make buffered lengths. Sends can close the channel so no more vals will be sent. Use ,ok to check if channel is closed, sending on a closed channel causes a panic.

```go 

func fibonacci(n int, c chan int) {
x, y := 0, 1
for i := 0; i < n; i++ {
	c <- x
	x, y = y, x+y
}
close(c)
}

func main() {
	c := make(chan int, 10)
	go fibonacci(cap(c), c)
	for i := range c { // Receives until c is closed
		fmt.Println(i)
	}
}

```


Select with a channel will block until one of its cases can run (a channel is not empty), unless theres a default.

## Mutual exclusion

```go

var mu sync.Mutex

mu.Lock
mu.Unlock

```

Lock will wait for another lock to unlock before continuing, can use defer to unlock. Locks are queued in order.

# Testing

## Unit Tests

A test file name, should end with \_test.go.

```go

package example

import "testing"

func TestExample(t *testing.T){
	if testResult != example {
		test.Errorf{"This test has failed "}
	}
}
```


Use go test to run the file.

## Table Driven Tests

Helps reduce repetition. 

```go
// Defining the columns of the table

var tests = []struct {

name string
input int
want string
}{
	// the table itself
	{"9 should be Foo", 9, "Foo"},
	{"3 should be Foo", 3, "Foo"},
	{"1 is not Foo", 1, "1"},
	{"0 should be Foo", 0, "Foo"},
}

// The execution loop
for _, tt := range tests {

	// Use the test func as param in loop
	t.Run(tt.name, func(t *testing.T) {
	ans := Fooer(tt.input)
	
	if ans != tt.want {
		t.Errorf("got %s, want %s", ans, tt.want)
	}
	})
}
```

## Reports

```cmd
go test -cover
```

How much of the code is covered by tests.

## Benchmarking

TBA

## Fuzz

TBA
# Modules

Dependencies (packages) are managed via modules, the go.mod file tracks them. Modules essentially bundle together all dependencies needed for the code. After adding an import to the package the mod file should be tidied.

```bash
go mod init [path] // Initalise setup
go mod tidy // Add imports to mod file

go mode -replace [mod name] = [path]
// replace path of mod to local file
```


- Module path is normally: \<prefix\>/\<descriptive-text\>\ 
- Prefix: The location ie github.com\
- Descriptive text: Project name
## Workspaces

Use work spaces to control multiple and locally change modules. Allows reference to packages inside a module outside the module in the workspace.

```bash
go work init ./module_name
go work use ./module_name
go mod edit -replace [module_name] = [fullpath]

```

Ensure the local module has the same name .mod file as the package name and then replace the workspace path to local.

# APIs

Gin is a HTTP web framework written in golang. TBA

## JSON/HTTP

Client errors need to be unmarshalled and then acted upon.

-   Decode into struct with 'json: \"meh\"' markers
-   If data comes in as array will need to unmarshal as an interface array

# Fuzzing

A method of testing with random injections of data. TBA

# Logging

Using the package Zap.

```go
import (
"go.uber.org/zap"
)

logger, _ = zap.NewProduction()
sugar := logger.Sugar() 
//Sugar - Less performance intense version
//New Production is a built-in preset

defer logger.Sync() 
// Will flush any buffered logs

//NewProduction():

{"level":"info","ts":1686135419.8329651,
"caller":"GoTests/test.go:13","msg":"Meh"}

//NewDevelopment():

2023-06-07T11:57:16.804+0100    INFO    
GoTests/test.go:13  Meh

//NewExample():

{"level":"info","msg":"Meh"}

sugar.Info("Hello World")
.Error("Not able to reach blog.",
	   zap.String("url", "codewithmukesh.com"))

//example with key-val data pairs for context 
sugar.Infow("failed to fetch URL",
"url", url,
"attempt", 3,
"backoff", time.Second,)

logger.Info("failed to fetch URL",
  // Structured context as strongly typed Field values.
  zap.String("url", url),
  zap.Int("attempt", 3),
  zap.Duration("backoff", time.Second),
)

//Using fmt.Sprintf
sugar.Infof("failed to fetch URL: %s", "http://example.com")
```


**Levels:**

-   DPanic : Causes logger to panic after log (in development)
-   Debug
-   Error
-   Fatal
-   Info
-   Panic: Causes logger to panic after log
-   Warn

**Level Endings:**

-   None: Uses fmt.Sprint
-   f: Uses fmt.Sprintf
-   ln: Uses fmt.Sprintln
-   w: allows extra values to be added in log

# Prometheus

Host metrics locally:

```go
        package main

        import (
           "net/http"

            "github.com/prometheus/client_golang/prometheus/promhttp"
        )

        func main() {
            http.Handle("/metrics", promhttp.Handler())
            http.ListenAndServe(":2112", nil)
        }
```


Add counter/gauge:

```go
var (
	counterExample = promauto.NewCounter(prometheus.CounterOpts{
			Name: "Counter_Example",
			Help: "The total number of processed events",
	})
	gaugeExample = promauto.NewGauge(prometheus.GaugeOpts{
		Namespace: "",
		Subsystem: "",
		Name: "gauge_example",
		Help: "Gauge example"
	})

)

counterExample.Inc() // Increment counter
gaugeExample.Set(99.9)
gaugeExample.Inc()
gaugeExample.Dec() //Decrement value
gaugeExample.Add(10) //Add 10
gaugeExample.Sub(10) //Subtract 10

gaugeExample.SetToCurrentTime()
```


Add histogram, counts observations from configurable static buckets in a burst sample. Which can be used to calculate quantiles.

Data points will be separated out into a chosen bucket.

```go
temps := prometheus.NewHistogram(prometheus.HistogramOpts{
	Name:    "pond_temperature_celsius",
	Help:    "The temperature of the frog pond.", 
	Buckets: prometheus.LinearBuckets(20, 5, 5),  
	// Start at 20, 5 buckets, each 5 centigrade wide.
})

// Simulate some observations.
for i := 0; i < 1000; i++ {
	temps.Observe(30 + 
	math.Floor(120*math.Sin(float64(i)*0.1))/10)
}

//Example output

histogram: <
sample_count: 1000
sample_sum: 29969.50000000001
bucket: <
cumulative_count: 192
upper_bound: 20
>
bucket: <
cumulative_count: 366
upper_bound: 25
>
bucket: <
cumulative_count: 501
upper_bound: 30
>
bucket: <
cumulative_count: 638
upper_bound: 35
>
bucket: <
cumulative_count: 816
upper_bound: 40
>
```


# Cobra - CLI

Cobra is a CLI framework. Built around commands (actions), args (things) and flags (modifiers).

*APPNAME COMMAND ARG --FLAG*

**Structure:**

appName/
	-- cmd/
		--- files.go
		-- main.go

The main files will simply initalise Cobra.

## Example Command

```go
//Placed in file called ex.go in package cmd

var exCmd = &cobra.Command{
	Use:   "ex",
	Short: "Hugo is a very fast static site generator",
	Long: `A Fast and Flexible Static Site Generator built 
	with love by spf13 and friends in Go.
	Complete documentation is available at 
	https://gohugo.io/documentation/`,

	Run: func(cmd *cobra.Command, args []string) {
	  // Do Stuff Here
	},
}

func Execute() {
	if err := rootCmd.Execute(); err != nil {
		fmt.Fprintln(os.Stderr, err)
		os.Exit(1)
	}
}

[
caption= main.go, % Caption above the listing
language=go, % Use Julia functions/syntax highlighting
frame=single, % Frame around the code listing
showstringspaces=false, % Don't put marks in string spaces
numbers=left, % Line numbers on left
numberstyle=\large, % Line numbers styling
]

func main() {
	cmd.Execute()
}
```

## Sub commands

Can create sub commands using the Add command function:

## Flags

Two types: Persistent flag (available to that command and all subcommands), local command (only applies to that specific command).

```go
var Verbose bool

//Persistent

rootCmd.PersistentFlags().BoolVarP(&Verbose, "verbose", 
"v", false, "verbose output")
rootCmd.Flags().StringVarP(&Verbose, "verbose",
 "v", false, "verbose output")

// Flag required
rootCmd.MarkFlagRequired("verbose")
rootCmd.MarkPersistentFlagRequired("verbose")

// Flags required together
rootCmd.MarkFlagsRequiredTogether("username", "password")

// Flags mutually exclusive
rootCmd.MarkFlagsMutuallyExclusive("json", "yaml")
```

## Arguments

Using the Args field in cobra.Command(), can add validators:

-   NoArgs - report error if any args used
-   MinimumNArgs(int) - report an error if less than N positional args are provided.
-   MaximumNArgs(int) - report an error if more than N positional args are provided.
-   ExactArgs(int) - report an error if there are not exactly N positional args.
-   OnlyValidArgs - report an error if there are any positional args not specified in the ValidArgs field of Command, which can optionally be set to a list of valid values for positional args.

**MatchAll()** is used to combine validators. Can also create custom validator funcs.

## Help

If using subcommands, cobra will auto generate command help, will give list of all subcommands.
Can also get help on specific commands by using command help subcommand.

## Version

**cmd.SetVersionTemplate()**

## Cobra Generator

Use: **cobra-cli init** to create new cli app.

*Edit details in cmd/root.go*.

Use: **cobra-cli add command** to create new command files. Add -p 'parentCmd' for a subcommand.

# Viper

A configurator (TBA)
