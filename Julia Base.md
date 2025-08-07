---
author:
  - Mo D Jabeen
date: 2023-10-01
title: Julia Base
Description: Notes on Julia's base design
---

# Constants

Julia has lexical scope, allowing to access the scope in which the function is defined not from where its called. Global values are looked up at run time making state tracking difficult if multiple places are updating the value. Looking up global values is also slow.

Using `const` skips this issue and allows the complier to fix the variable type giving faster and easier to manage code.

```julia

const L = 2

```

# Loops

```julia
for i in 1:5 # This calls the iterate func
println(i)
end

#Avoid using len for iteration (better saftey ,extension and efficency!)
The rule of thumb: if you need indices, use `eachindex`. If you need to work with specific dimensions, use `axes`. Only use `length`/`size` when you actually need the numeric size value, not for iteration.

#DONT DO
for i in 1:length(arr)

#DO
for i in eachindex(arr)

a = collect(1:20) # convert into an array

a = map((x) -> x^2, [1,3,5,3]) # map performs func on each array element

x = hcat(map((x) -> x^2, [1,3,5,3])...)

#The ... will seperate each result of the func for hcat

#Example

julia> hcat([[1,2],[1,3]]...)
2×2 Matrix{Int64}:
 1  1
 2  3

julia> hcat([[1,2],[1,3]])
2×1 Matrix{Vector{Int64}}:
 [1, 2]
 [1, 3]

#... essentially removes the outer vector layer

ColumnData = map(x -> "meh", collect(1:10);

foreach(func, collection) #operate func on each val of collection

for (key,val) in dict # to iterate through dict
```

# Functions

```julia
function name()
code
end 

function name()
r=3

r,r+2 #Omit the return keyword for tuple return
end 

# Varargs, unknown number of params
function meh(a,b,x...)
# x is a collection of all the other params used in calling the func (slurping)

function meh(a,b,c)
meh(x...)# tuple converted into multiple args (splatting)

# Keyword arg ;
function meh(a;b)
meh(1,b=3)

# Pass function as param
function meh(hem::Function)

#; enforces keyword argument when calling function
# Keyword args have slight increase in overhead 
function meh(;x::Int)

#ANON funcs, used in place of func arguments
args -> body

#Can wrap a function using anon function (will return new wrapped func)
function meh(f)

	x -> begin
		if x ...
			f(x)
		else 
			return 1
		end
	end
end

#Multipe var wrap
function meh(f)

	(args,kwargs) -> begin
		x = (args,kwargs)
		if x ...
			f(args...;kwargs...)
		else 
			return 1
		end
	end
end

#do end

foreach(list) do
	body of func
	end

```

# Struct

```julia

# Use union for combinations
const Numbs = Union{Int,Float}

mutable struct name 
	string::AbstractString
	boolean::Bool 
	age::Int
	height::Numbs
	a::Array{Int,5}
end

newstruct = name(...)

#Keyword struct

Base.@kwdef struct TextStyle
	font_family
	font_size
	font_weight = "Normal"
	foreground_color = "black"
	background_color= "white"
	alignment = "center"
	rotation = 0
end

# Internal constructors are used to place constraints on the code

mutable struct name
	meh::AbstractString
	numb::Int

	name(blah::AbstractString)= new(meh,4) 
	# this enforces if a struct without 
	#a number is given 4 is placed
end

getfield(struct,symbol) #get field value using symbol instead of direct access
setfield!(struct,symbol,value) #set field value using symbol instead of direct access

#Built in anon struct into a vector

res = []
push!(results, (
            base_metric = base_metric,
            avg_position = avg_position,
            positions = positions,
            count = length(positions)
        ))

# Callable structs
struct PredicateFunction{T,S}
	f::Function
end

(pred::PredicateFunction{T,S})(x::T; kwargs...) where {T,S} = pred.f(x; kwargs...)

```   

## Struct with Separate Fields

#### Pros:

1. **Type safety**: Fields have explicit types, giving you compile-time type checking
2. **Performance**: Direct field access is faster than dictionary lookups
3. **Auto-completion**: IDEs can provide field name suggestions
4. **Documentation**: Field names and types can be documented in the struct definition
5. **Clear interface**: The available fields are explicitly defined in the code
6. **Memory efficiency**: Fixed memory layout, no overhead for hash tables

#### Cons:

1. **Less flexible**: Adding/removing fields requires changing the struct definition
2. **Code repetition**: Can lead to boilerplate code when many similar structs are needed
3. **Refactoring overhead**: Changing field names requires updating all references

## Dictionary Approach

#### Pros:

1. **Flexibility**: Easy to add or remove keys dynamically
2. **Generic code**: Can write functions that work with any keys without changing struct definitions
3. **Metaprogramming friendly**: Easier to generate or manipulate keys programmatically
4. **Unified interface**: All metrics can share exactly the same type

#### Cons:

1. **No type safety** for keys: Misspelled keys don't cause compile-time errors
2. **Performance overhead**: Hash lookups are slower than direct field access
3. **Memory overhead**: Hash tables have more memory overhead than structs
4. **Less clear interface**: Available keys aren't obvious from the type definition
5. **No auto-completion**: IDEs can't suggest available keys

### Parametric Structs

- Helps enforce type consistency within subtypes.

```julia

# Allows creating same name structs with different data types
mutable struct name{T <: Real}
	string::T
	boolean::T 
	age::T
	height::T
	a::T
end

#Multiple types + using a single value of the type to define a new type (N)
abstract type Color{T, N} <: Colorant{T,N} end
abstract type AbstractRGB{T} <: Color{T,3} end

```

# Misc

-   printf for formatted prints uses the module Printf and is macro with syntax \@printf
-   %3f : used to show 3 sig fig
-   ë : scientific notation
-   index starts at 1 !
-   Strings can be indexed like arrays
-   try, catch : for error handling
-   Avoid globals
- Immutable vars use more memory but safer for multi threading.
-   Locals scope is defined by code blocks ie func, loop not if
-   Built in funcs such as iterate can be extended via multi-dispatch
-   Use the Profiling package for measuring performance.

```julia 
#String comb uses *
comb = "str" * "ing"
 
d = Dict(1=>"one", 2 => "two")
        d[3] = "three" # Add to the dict
#Loops and funcs can also be placed in dicts

@enum Options A B C
#Creates type options with values A=0, B=1, C=2 etc

# || as if mechanism (false returns y)
bool_func(x) || return y

# Create array using func and loop
arr = [func() for x in list]
```

# Tenancy

```julia

# If the condition is true 1 is returned else -1 is
x > 0 ? 1 : -1 

#Set value type based on condition
x = s=="Int64" ? Vector{Int64}(undef,n) : Vector{Float64}(undef, n)

```

# Push

```julia

pushfirst! #Can cause signficantly larger operations up to O(N^2) if a larger block of memory needs to be allocated

push! #Stays at O(1) even combined with a sort its at O(N) 

```

# Abstract types

Abstract types dont dictate how data is stored instead for controlling behaviour!

- **Hierarchical organization**: They allow you to create a logical type hierarchy that represents relationships between different types.
- **Code reuse through multiple dispatch**: You can write methods that operate on an abstract type, and these methods will work with any concrete subtype. (Overlapping behaviour built into each subtype, if subtype missing will use supertype func.)
- **Interface definition**: Abstract types define a conceptual interface that concrete subtypes are expected to implement, similar to interfaces in other languages. (Code control)
- **Parametric polymorphism**: They enable parametric types to be constrained to a family of related types.
- **Extension without modification**: New concrete subtypes can be added without changing existing code.

```julia
#To allow grouping of objects, to define a common type for a function USE ABSTRACT TYPES

abstract type supertype end
abstract type subtype <: supertype end

abstract type Number end 
abstract type Real <: Number end

#Can return values using an abstract type ie

struct implement <: supertype end 

#Used supertype to return concrete type
supertype(::Type(<:Real)) = implement()

# Use inferred type to dispatch, 
func(x::T) where {T} = func(supertype(T),x)
func(::Type,x) = do stuff

```

# Strings

## Regex

Sometimes you are not looking for an exact string, but a particular pattern this can be done using a regex which is a string key used to search through text. (ie . signifies any character)

Julia uses version 2 of Perl-compatible regular expressions (regexes).

```julia

#Regex follow r"" format
re = r""

#Determine whether the first argument matches the second.
occursin(regex,string)  

# Search for the first match of the regular expression `r` in `s` and return a RegexMatch object containing the match, or nothing if the match failed.
match(regex,string)

# Search for all matches of the regular expression `r` in `s` and return an iterator over the matches
eachmatch(regex,string,overlap::Bool=false)

```

# Constructors

Structs mainly used to create new data type objects.
Inner and outer constructor methods for structs define how a new object is created based on data input.

Inner constructors enforce the same checks for multiple data types.

```julia
struct name{T<:Integer} <: Real 
# <: shows all values are included in that set
# {for arg} outer for object

	num::T
	den::T #ensures both are of type T

	#Function checks if the input numbers are empty for every object
	function name{T}(num::T, den::T) where T <: Integer

		if num == 0 && den == 0
			error("invalid")
		end
		new(num,den)
	end     
end 

name(n::Int, d::Float) = name(promote(n,d)...)
#Outer constructor
#Promote converts values of a single type to the same type 
#choosing the type to work with both

```

# Multiple Dispatch

- Will use the narrowest available subtype.
- Careful of ambiguities from unclear crossover within subtype structures (detect_ambiguities tool available)

```julia

function blah(n::Int, d::Int) = println('meh')

function blah(x::Int, y::name) = println(x*y.num)
#This func now has two methods (multi dispatch)

```

Different to method overloading which is done on specified values at compile time, whereas dispatch works at run time. 
# Parametric Methods

Use variable to define types of arguments.

- Benefit is can enforce type consistency within subtype structures as T will need to be the same across args but can still be within a subtype of the <: abstract.

```julia

# The T is the type inside the vector which refed in the func
function myfunc(x::Vector{T}) where T <: Any
	b::Vector{T} = x.*3
end

function meh(a::T,b::T) where T <: Real
	stuff
end

```

# Modules

**Dont overdo modules, should represent a single app/project. Only further separate if make sense as stand alone sections.**

Modules allow for better namespace control and cleaner structure.

They are not attached to a file, can have multiple modules in a file and multi files for the same module.

**using modulename:** Includes all code and exported variables.
**import modulename:** Includes only the code.

Can use submodules which are accessed via . operator.

# Interfaces

Frameworks modules with functions and abstract types than need to be implanted and extended. *Great for evolvability and extendability*.

- Can be Hard (required) or soft (optional)
	- Hard might use an error in the interface to force implementation
	- Soft might return nothing as its not necessary
- Extended by creating a new datatype that extends the main super type in the interface. Not required tho. Can be purely functional (example below).
- Check funcs used to check if a data type has a certain characteristic or behaviour - **Trait**
	- Normally response is boolean


```julia
module Vehicle

# 1. Export/Imports
export go!

# 2. Interface documentation

# A vehicle (v) must implement the following functions:
# power_on!(v) - turn on the vehicle's engine
# power_off!(v) - turn off the vehicle's engine
# turn!(v, direction) - steer the vehicle to the specified direction
# move!(v, distance) - move the vehicle by the specified distance
# position(v) - returns the (x,y) position of the vehicle

# 3. Generic definitions for the interface

function power_on! end
function power_off! end
function turn! end
function move! end
function position end

# 4. Game logic

# Returns a travel plan from current position to destination
function travel_path(position, destination)
	return round(π/6, digits=2), 1000 # just a test
end

# Space travel logic
function go!(vehicle, destination)

	power_on!(vehicle)
	direction, distance = travel_path(position(vehicle),
	destination)
	turn!(vehicle, direction)
	move!(vehicle, distance)
	power_off!(vehicle)
	
	nothing
end

end

```
# Metaprogramming

## Expressions

Julia code is represented after compiling as a data struct of type Expr.

- **\$:** Used as interpolation for literal expression in a macro.
- **eval:** Executes the code from Expr data type.
- **:** Turns code into an expression (can also used quote for blocks)

Can use Expr data types as inputs to functions.

## Abstract syntax tree

Source codes structure is represented as an abstract syntax tree which is made of expressions. The expression x + y is shown as a tree below.

![[Screenshot 2025-07-16 at 10.23.41.png]]

## Macros

Macros accept expressions manipulate them and return expressions. Therefore they adjust the abstract syntax tree effectively altering the source code. 

**They do not access values only expressions.**

**Benefits:**
- Might give more concise code (ie using string literals for custom domain specific lang)
- May improve performance as similar to direct source code so remove overhead of higher level stuff ie loops (@unroll).
	- Executed only once during compile so better than a constructor being called many times in a func.
- Access global scope from within local scope without specifying global variable (cleaner)

```julia
macro name()

end

@name() # Run using the @ operator.

@time func #Used to measure the time taken to run a func

esc() #To access local scope variables 
```

Macros are used in code when an expression is required in multiple places before it is evaluated.

```julia
struct MyNumber
	x::Float64
end
# output

for op = (:sin, :cos, :tan, :log, :exp)
	@eval Base.$op(a::MyNumber) = MyNumber($op(a.x))
end
```

### Compilation

Macros can help run code at different points along the execution pipeline instead of just at the end runtime. Moving constant computation known at compile time to earlier in the pipeline can allow for a more optimised operation created via a macro that is then used at runtime.  

![[Screenshot 2025-08-07 at 11.46.10.png]]

### Generated Functions

Generated funcs are like macros but with access to data types instead of expressions, if that is required when meta programming.

Once the compiler has inferred the types can be used to move constant computation based on type to a more optimised place in the pipeline.

```julia

function prod_dim(x::Array{T, N}) where {T, N}
	s = 1

	#Loop depends on type N (known before runtime)
	for i = 1:N
		s = s * size(x, i)
	end
	
	return s
end

# Example of loop unrolling as will be optimised into a inline calc
@generated function prod_dim_gen{T, N}(x::Array{T, N})

	ex = :(1)
	
	for i in 1:N
		ex = :(size(x, $i) * $ex)
	end
	
	return ex
end


```

Converting a loop into a inline calculation is known as unrolling. Made easy by using N as a type via gen funcs. 
### String Literals

Used as mini domain specific language, making things more concise. Can be used to return defined empty data structures ie DataFrame as not a predefined variable.

```julia

macro macro_name(string)
	stuff
end

macro_name"arg"

```
# Concurrency

Julia combines multi threads and cores using the same memory space as threading. CPUs using separate memory spaces are defined as multi-processor or distributed computing.

- **mutex:** Single lock mechanism for controlling accessing to data.
- **semaphore:** Value signifying what are the resources being used on, for process synchronization.

Julia code tends to be purely functional and avoids mutation, generally opting for only local mutation. If there is a shared states locks should be used or a local state (an object shared by all threads.) A shared local state gives higher performance.

## Asynchronous

### What are Tasks ?

**Tasks** are used for asynchronous calls, ie waiting for external signals. Tasks allow switching at any point in the execution between them and don't use extra memory space (call stack).

wait(t) - waits for the task

```julia
t = @task func()
#OR
t= @task begin ... end
#OR  
t = Task(func)

schedule(t,[val],error) # Allocate task to scheduler, pass val

# if error true, val passed as an exception

@async func() same as schedule(@task func())

asyncmap(func,collection,ntask,batch) 

# Return collection with the func executed on by ntasks.
# Batch executes on collection in groups set by number of batch.

yield() #Switch to scheduler to allow another task to run
yield(t) #Switch to task t.

Condition() #Edge triggered event source
thread.condition - thread safe version

Event() #Level triggered event source

notify(condition,val,all,error) #Wake up tasks waiting for condition

semaphore(sem_size) #counting with max at semsize

acquire(s) #get a semaphore, blocks if none available
release(s)

#Use below format for locking

lock()
try
...
finally
unlock()
end

bind(channel,task) 

wait([x])

x {
	Channel: Wait for val
	Condition: Wait for notify 
	Process: wait for process to exit
	Task: wait for task to finish
	RawFD: change the file descriptor
}

#if there is no x will wait for schedule to be called
```

### What are Channels?

Channels are a first in first out queue, used to connect tasks in a memory/race safe way. They can be bounded to a task, by being placed as a parameter and therefore do not need closing.

```julia
c = Channel{Type}(limit) #limit is max number of objects in queue

put!(channel,data) # Place data into channel
take!(channel) #Read data from channel

Channel(func()) - Bind a channel with a task
```

-   Readers will block on a take if the channel is empty
-   Writers will block on a put if the channel is full
-   Wait will wait until the channel has data
-   isready test if the channel has data
- CAREFUL OF USING MUTABLE STRUCTS WITH CHANNELS !

## Multi threads

Use atomic vars to ensure expected correct operation when using threads (ie for arithmetics). Careful of finalization (tasks to clean up before garbage collection.)

```julia
Threads.@threads [schedule] for ... end

[schedule] {
	default: :dyanmic assumes equal load per thread, cant 
	guarantee thread id on an iteration
	:static one task for thread, can guarantee same id for an 
	iteration
}

threads.foreach(f,c,ntasks) 
#operate function on channel with n threads

Threads.@spawn func() 
#Create task and schedule to run on any available thread
```

# Logging

Inserting a logging statement creates an event, logging is allows for better control and visibility than print statements.

```julia
@debug #Auto set to not output to stderr
@info # mid level
@warn #Higher level
@error #highest level, generally not needed 
# (use exceptions instead)

@__ msg var x=var y=func() #msg in markdown

@__ "blah $var "
```

## How do you process a log ?

Log creation and processing are separate to allow for module level and app level work.

```julia
ConsoleLogger([steam,] min_level=Info) #stream can be a file io
SimpleLogger([stream,] min_level=Info)

global_logger(logger)
with_logger(logger) do ... end #Local logger

#FILTER

disable_logging(level) #disable below this level
should_log(logger,level) #return true if accepts this level

handle_message(logger,level,message,val ...)

#EXAMPLE

# Open a textfile for writing
io = open("log.txt", "w+")
IOStream(<file log.txt>)

# Create a simple logger
logger = SimpleLogger(io)
SimpleLogger(IOStream(<file log.txt>), Info, Dict{Any,Int64}())

# Log a task-specific message
with_logger(logger) do
		   @info("a context specific log message")
	   end

# Write all buffered messages to the file
flush(io)

# Set the global logger to logger
global_logger(logger)
SimpleLogger(IOStream(<file log.txt>), Info, Dict{Any,Int64}())

# This message will now also be written to the file
@info("a global log message")

# Close the file
close(io)

# Create a ConsoleLogger that prints any log 
#messages with level >= Debug to stderr
debuglogger = ConsoleLogger(stderr, Logging.Debug)

# Enable debuglogger for a task
with_logger(debuglogger) do
		@debug "a context specific log message"
	end

# Set the global logger
global_logger(debuglogger)
```

# Packages

Designed around environments which can be local to individuals or projects. Allows exact set of packages and their version in an environment to be controlled and repeated. All tracked in the manifest file.

A better version of python virtualenv.

```julia
activate [project name]

develop --local [package name or path] #Allows you to use a local version of a 
# package to develop

free [package name] #Go back to the registered version

get "url" or "local path"

pin [package name] #Never update

# Activate another persons project
activate .
instantiate
```

## Creating package

Add PkgTemplates

Execute:

```julia

using PkgTemplates 

t = Template(; 
	user="your-GitHub-username", 
	authors=["your-name"],
	plugins=[ License(name="MIT"), Git(), GitHubActions(), 
	], 
)


t("YourPackageName")
```

Source files stored at:  `~/.julia/dev`

- Add `.jl` to the name of the git repository.
- Keep the options of **Initialize this repository with** unselected.

### Code

Add includes for files exports for functions to be accessed in the main package file 

Main File (PackageName.jl):
```julia
module YourPackageName 
export greet_your_package_name 
include("functions.jl") 
end
```

Add tests in test/runtests.jl

**Run tests:**
```
julia> ] # Go to the package mode (v1.8) 
pkg> activate . 
(YourPackageName) pkg> test
```

## CI

Adjust CI.yml to include macOS-latest

```yml
        os:
          - macOS-latest
          - ubuntu-latest
        arch:
```

# Profiling

## Basic

For more accurate comparison use BenchmarkTools.jl pkg.

```julia
@time func() #Measure how long to run func
@timev func() #Extra memory stats

@time func(); #; stops return value given to console
x = @elapsed func() #Gives value that can be further processed ie tests

```
## Sampling

Julia has built in profiling, allowing code lines to be timed to discover code bottlenecks. Checks how often lines appear in a set of backtraces, the line coverage is therefore dependant on the backtrace sampling freq.

**Pros:** No need to change code to measure and little computational overhead.

Don't want to profile JIT so best to run code once allowing it to compile before profiling. Check if inference.jl shows up to see if JIT was profiled.

The output will be saved in memory, the output will show the number against each line being how many times it was sampled.

```julia
using Profile

@profile myfunc()

Profile.print() #Output to stderr backtraces
Profile.print(format = :flat) #If too nested

#Run test 100 times

@profile for(i=1:100; myfunc();end)

Profile.clear() #Clear buffer
Profile.init(delay=.01) #Adjust sampling delay, too low will increase overhead.

```

The number of backtraces shows how expensive that line is.

## ProfileView.jl

Package that gives graphical output of profiling (flame graph).

Time elapsed shown left to right and call stack top to bottom.
## TimerOutputs.jl

Be more selective in choice of parts of the program to profile.

## Memory allocation

```julia

julia track-allocation=user
 
Profile.clear_malloc_data() #after executing commands

```

Use \@time or \@allocation to check the memory allocation. Can also check the cost of garbage collection.

To check line by line allocation, use flag trackallocation. Remove compiler overhead before measuring allocation by running. 

GC.enable_logging(true) set to true to check collection cost.

# Appendix

## Variables

Two types of concrete: primitive and composite. Composite have field names.

## Macros

Macro Hygiene : Compiler labels macro variables to avoid conflict with normal code.

## Differences from Python

-   Use immutable Vector (same data type) instead of arrays (python
    would use list)
-   Indents start with 1
-   Include end when slicing ie \[1:end\] not \[1:\]
-   Use \[start;stop;step\] format
-   Matrix indexing creates submatrix not tuple ie X\[\[1:2\]\[2:3\]\]
-   To create a tuple from a matrix use (like python) X\[CartesianIndex(1,1), CartesianIndex(2,3)\]
-   Variable assignment is not pointer assignment ie a= b creates new variable so they remain separate.
-   push! is the same as append
-   \% is remainder not modulus
-   Int is not an unknown size its int32
-   nothing instead of null