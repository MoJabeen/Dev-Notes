---
author:
  - Mo D Jabeen
date: 2023-10-01
title: Julia Cheat Sheet
---
# General

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
```

## Constants

Julia has lexical scope, allowing to access the scope in which the function is defined not from where its called. Global values are looked up at run time making state tracking difficult if multiple places are updating the value. Looking up global values is also slow.

Using `const` skips this issue and allows the complier to fix the variable type giving faster and easier to manage code.

```julia

const L = 2

```

## Loops

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

## Functions

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

# Pass function as param

function meh(hem::Function)

#; enforces keyword argument when calling function
function meh(;x::Int)

#ANON funcs, used in place of func arguments
args -> body

#do end

foreach(list) do
	body of func
	end

```

## Struct

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

```
## Tenancy

```julia

# If the condition is true 1 is returned else -1 is
x > 0 ? 1 : -1 
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

abstract type «name» end
abstract type «name» <: «supertype» end

abstract type Number end 
abstract type Real <: Number end

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

# MULTI-DISPATCH 

- Will use the narrowest available subtype.
- Careful of ambiguities from unclear crossover within subtype structures (detect_ambiguities tool available)

```julia

function blah(n::Int, d::Int) = println('meh')

function blah(x::Int, y::name) = println(x*y.num)
#This func now has two methods (multi dispatch)

```

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

# Interfaces

Frameworks modules with functions and abstract types than need to be implanted and extended. *Great for evolvability and extendability*.

- Can be Hard (required) or soft (optional)
	- Hard might use an error in the interface to force implementation
	- Soft might return nothing as its not necessary
- Extended by creating a new datatype that extends the main super type in the interface.
- Check funcs used to check if a data type has a certain characteristic or behaviour - **Trait**
	- Normally response is boolean


# Modules

**Dont overdo modules, should represent a single app/project. Only further separate if make sense as stand alone sections.**

Modules allow for better namespace control and cleaner structure.

They are not attached to a file, can have multiple modules in a file and multi files for the same module.

**using modulename:** Includes all code and exported variables.
**import modulename:** Includes only the code.

Can use submodules which are accessed via . operator.

# Differences from Python

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

**Benefits:**
- Might give more concise code (ie using string literals for custom domain specific lang)
- May improve performance as similar to direct source code so remove overhead of higher level stuff ie loops (@unroll).
	- Executed only once during compile so better than a constructor being called many times in a func.
- Access global scope from within local scope without specifiying 

Compiled code as an expression not executed on runtime but during parsing.

```julia
macro name()

end

@name() # Run using the @ operator.

@time func #Used to measure the time taken to run a func
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

# Plotting

The Julia plot pkg used is **Gadfly**, git: https://github.com/GiovineItalia/Gadfly.jl.

Used for plotting and visualization, using the browser to supply interactive plots, also supports svg,png,postscript and pdf (done via draw command).

## General

```julia
plot(data, elements ....; mapping)
#data is a dataframe
#elements are the plot ie x= :col, color = blue etc
#mapping is the type of graph ie Geom.point, Geom.line

map(display,plot) 
#Or can use REPL to output to default media viewer (browser)

plot(func(),lower,upper,elements;mapping)
#lower upper are x bounds

plot(Vector{funcs()},lower,upper,elements;mapping)
plot(func(),xmin,xmax,ymin,ymax,elements;mapping)

# Can also add elements to plot via push!
```

**To output the plot as a html file, run code in REPL using include(\"filename.jl\").**

*Widerform is when cols use the sames measurment type but are grouped in
cols.* 

Elements have a Scale (continous or discrete) and Guide class:

## Aesthetics

Data is binded to aesthetics, defined by a number of elements: Scales, Coordinates, Guides and Geometrics.

### Themes

Allow easy setting of a bunch of parameters, the theme overrides the default values. Style will override the current theme.

### Geometries

Are the types of graph: Point, line, bar, candle etc.

### Scale

Scale allows a characteristic to be used to show the different grouping of values ie alpha(transparency),color, shape, linetype.

```julia
plot(x=xdata, y=ydata, color=cdata,
Scale.color_continuous(minvalue=-1, maxvalue=1))

plot(x=x, y=y, color=x+y, Geom.rectbin,
Scale.color_continuous(colormap=p->RGB(0,p,0)))

plot(x=xdata, y=ydata, color=repeat([1,2,3], 
outer=[4]), Scale.color_discrete)

plot(x=rand(12), y=rand(12), color=repeat(["a","b","c"],
 outer=[4]),
 Scale.color_discrete_manual("red","purple","green"))

plot(dataset("datasets", "CO2"), x=:Conc, y=:Uptake,
group=:Plant, linestyle=:Treatment, color=:Type, Geom.line,
Scale.linestyle_discrete(order=[2,1]))
```

Also used for scaling of axis:

-   Scale.x_continous(format=, minvalue,maxvalue)
-   Scale.x_discrete
-   Scale.xlog10 : Added as elements to change entire plot
-   Scale.ylog10

```julia
p2 = plot(mammals, x=:Body, y=:Brain, label=:Mammal, 
	Geom.point, Geom.label,
	Scale.x_log10, Scale.y_log10)

p3= plot(Diamonds, x=:Price, y=:Carat, 
	Geom.histogram2d(xbincount=25, ybincount=25),
	Scale.x_continuous(format=:engineering))
```

### Guides

Types of Guides:

-   Annotation (inc shapes to highlight points)
-   Keys : colour, shape or size
-   Title labels
-   xrug,yrug : show distribution of points on axis
-   ticks : show plot lines

```julia
pb = plot(iris, x=:SepalLength, y=:PetalLength, color=:Species, 
	Geom.point,
	Guide.colorkey(title="Iris", pos=[0.05w,-0.28h]) )

plot(dataset("ggplot2", "diamonds"), x="Price", Geom.histogram,
	Guide.title("Diamond Price Distribution"))
```
### Stats

Can also uses in built stat transformations ie:

-   Density
-   Quantile
-   Doge: Grouped and shown value
## Compositing

Types:

-   Facets: Share same axis dims but seperate
-   Stack: Diff datasets and axis but together
-   Layers: Single plot
### Layers

```julia
l = layer(data,elements;mapping) 
# Can not include Scale,coordinates or guides
#use plot for that

plot(layer1,layer2..)

plot(data
	layer(elements;mapping)
	layer(...))
```

# Profiling

Julia has built in profiling, allowing code lines to be timed to discover code bottlenecks. Checks how often lines appear in a set of backtraces, the line coverage is therefore dependant on the backtrace sampling freq.

**Pros:** No need to change code to measure and little computational overhead.

### Example

```julia
using Profile

@profile myfunc()

Profile.print() #Output to stderr backtraces

#Run test 100 times

@profile for( i=1:100; myfunc();end)

Profile.clear() #Clear buffer

@time func() #Measure how long to run func
```

The number of backtraces shows how expensive that line is.

### Memory allocation

Use \@time or \@allocation to check the memory allocation. Can also check the cost of garbage collection.

To check line by line allocation, use flag trackallocation. Remove compiler overhead before measuring allocation by running Profile.clear_malloc_data() after executing commands. 

GC.enable_logging(true) set to true to check collection cost.

# Performance tips

Performance critical code should be inside a function.
**AVOID:**

-   Untyped global variables, if they must be used annotate the type at point of use.
-   Problems with type stability
-   Many temporary small arrays

At point of use annotation:

```julia
for i in x :: Vector{Float64}
```

## Push

```julia

pushfirst! #Can cause signficantly larger operations up to O(N^2) if a larger block of memory needs to be allocated

push! #Stays at O(1) even combined with a sort its at O(N) 

```

## Wrappers


Better to define the type of the wrapper than only the its elements. As you dont want the high level struct type to be ambiguous:

 ```julia
mutable struct Ty{T<: AbstractFloat}
	a::T
end

#This is better than

mutable struct Ty
	a::Float64 #should be concrete type
end
```

Functions using the same struct but different types of elements should be explicit of the type of elements

```julia
function func(c::Ty{<:AbstractArray{<:Integer}})

#If a type in vector is known should declare it!
function func(a::Vector{Any,1})
	x = a[1]::Int32
```

**However, abstract type annotation for values and elements can hinder performance.** 

Dont use compound functions, ie same function used differently for types use multiple dispatch.

## How should functions be returned?

A function should always return the same type, so if the type is ambiguous convert before return.

```julia
zero(x) #Gives a 0 val in type of x
oftype(x,y) #Gives y as type of x
```

Variables changing type within the function should also be avoided.

## How should functions be broken down?

Generally functions of made of the setup and compute on the setup vars. The compute should broken into its own function.

## Types used as parameters for other types

If a var is created using an unknown type ie a matrix or array then its type will be checked on every access.

```julia
fill(5,ntuple(d->3,N)) 
#If N is unknown then will create an unknown type matrix.
::Val{N} 
#is used a concrete type to specify dimensions, pass as Val(N) in func
```

## What is dangerous when using Multi Dispatch?

If at compile time Julia does not know the type it will be checked at
run time slowing down compute.

```julia
struct Car{:make,:model}
	year::Int
end

Array{Car{:Honda,:Accord},N} 
#This is contiguous and known

ar = [Car{:Honda,:Accord}(2),Car{:Honda,:Buggy}(3)] 
#Non contiguous will be determined at runtime
```

The above situation would be bad if used as method of multi dispatch as many lookups on each type as the array is iterated will be done. Will also take a lot more compile time memory as all versions of functions will need to be compiled for each type ie for push!.

## What are the array types used in Julia?

Arrays are column major, the first index changes most rapidly (row). Meaning the inner most loop should be iterated on rows, the outer loop on cols. Cols are faster to get than rows.

## Should outputs for func allocated before?

The output of a function should be preallocated with a defined type and the function should be used to change its values.

## How should nested dot calls be used?

Nested dot calls are fusing, they are combined at the syntax level into a single loop. And use preallocated arrays instead of temp ones (SHOULD BE USED WHENEVER POSSIBLE!).\

```julia
f(x) = 3x.^2 + 4x + 7x.^3
fdot(x) = @. 3x^2 + 4x + 7x^3 

# @. converts all vector ops into a dot operation
# fdot(x) is 10x faster
```

## How should views be used?

A slice creates a copy of the array, which is okay if many operations are to be done on the copy, however, could be worse if the cost of the copy outweighs the operations to be done on it.

Can instead use \@view to reference a subarray of the original array without making a copy. Also contiguous are better and if possible arrays should be converted into a contiguous array before operation.

```julia

A = zeros(3, 3); 

#@views converts each operations return into a view, like a wrapper
@views for row in 1:3 
	b = A[row, :] # b is a view, not a copy 
	b .= row # assign every element to the row index 
end


```

## Other

-   Static array pkg if array is to be fixed size
-   Avoid string interpolation for I/O (dont use \$a)
-   Fix all warnings
-   Avoid unnecessary arrays
-   Use fld(x,y) and cld(x,y) instead of floor and ceil
-   Annotations available to reduce Julia functions for sake of speed ie don't check boundaries.
-   Use \@code_warntype to show non concrete types in red.

# Style

- Better to use abstract types than concrete
- Use ! for argument modifying funcs
- Avoid complicated container types, better to use Any
- Use _ prefix for internal elements (implementation details)
- modules and types (structs) should be capital and camel case
- functions should be lower case and if easy to ready squashed as a single word, otherwise use underscore to show a combination of concepts.
- Avoid abbreviations 
- Dont () conditions 

```julia
#Not (a==b)
if a==b

end
```

- Dont overuse ... (separates  elements into array)
- Dont overdo macros, funcs are better
- Dont allow easy access to overwrite pointers
- Avoid having functions with specialised types of base containers (ie Vector, Array), will be confusing to work with
- For testing types use isa and <: instead of ==
- Avoid Float literals if possible use Int or Rational (fractions ie 1//2 is 0.5)

```julia
f(x) = 2.0 * x #meh
g(x) = 2 * x #Best
h(x) = 2//1 * x #Good
```
## Exported methods 

Use exported methods of a module instead of direct access (unless an API).

- Allows use of more abstract funcs instead of always needing implementation details of the modules type.
- Generally allows for easier code reuse and abstraction of funcs

## Type Piracy

Avoid type piracy, ie dont extend package methods without using your own custom types.

```julia
module A 
#Dont do this !
import Base.* 
*(x::Symbol, y::Symbol) = Symbol(x,y) 
end
```

## Anon funcs

Avoid trivial anon funcs, pass funcs directly as arguments instead of wrapping them as anon.

```julia
#Dont do
map(x-f(x),a)
#Pass func directly
map(f,a)
```

# Docs

Package: Documenter.jl

Combines md docs and inline code docstrings to create a single inter-linked doc.

**Folder structure:**
```
Example/ 
├── docs/ 
│   └── ... 
├── src/ 
│    └── Example.jl ...
```

The docs folder contains a src folder to hold all markdown files to use and a make script

```
docs/ 
├── src/
│   └── index.md
└── make.jl
```

**Never** `git commit` the contents of `build` (or any other content generated by Documenter) to your repository's `master` branch.

## Index

To link doc string add to the index.md file:

with code line:

```
code block @docs
func(x)

```

then run make.jl which will replace the block for you.

Note that _more than one_ object can be referenced inside a `@docs` block – just place each one on a separate line.

To change module use the @meta block and set CurrentModule = to chosen module.

### Cross reference

index.md file

```

- link to [Example.jl Documentation](@ref) // links to header
- link to [`func(x)`](@ref) // links to doc string func(x)

```

### Navigation

```

code block @contents

code block @index

```


Use makedocs func to control sidebar.
## Docstrings

In md format.

"""
	func(parmas) -> (return...)

Description

\`\`\` julia
	example code
\`\`\`
... 
\# Arguments 
 \-\`n::Integer\`: the number of elements to compute. 
 \-\`dim::Integer=1\`: the dimensions along which to perform the computation. 
...

"""

# Appendix

## Variables

Two types of concrete: primitive and composite. Composite have field names.
