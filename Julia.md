---
author:
  - Mo D Jabeen
date: 2023-10-01
title: Julia Cheat Sheet
---

# General

```julia
function name()
code
end 

function name()
r=3

r,r+2 #Omit the return keyword for tuple return
end 
```

-   printf for formatted prints uses the module Printf and is macro with syntax \@printf
-   %3f : used to show 3 sig fig
-   ë : scientific notation
-   index starts at 1 :O
-   Strings can be indexed like arrays
-   Combine strings using \*
-   try, catch : for error handling

```julia
 d = Dict(1=>"one", 2 => "two")
        d[3] = "three" # Add to the dict

        #Loops and funcs can also be placed in dicts
```

## Loops

```julia
for i in 1:5 # This calls the iterate func
println(i)
end

a = collect(1:20) # convert into an array

a = map((x) -> x^2, [1,3,5,3]) # map performs func on each array element

foreach(func, collection) #operate func on each val of collection
```

## Struct

```julia
mutable struct name 
	string::AbstractString
	boolean::Bool 
	age::Int
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

```   

## Tenancy

```julia
x > 0 ? 1 : -1 
```

# If the condition is true 1 is returned else -1 is

-   Avoid globals
-   Locals scope is defined by code blocks ie func, loop not if
-   Built in funcs such as iterate can be extended via multi-dispatch
-   Use the Profiling package for measuring performance.

# Objects/Methods

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


# MULTI-DISPATCH FUNCTION

function blah(n::Int, d::Int) = println('meh')

function blah(x::Int, y::name) = println(x*y.num)
#This func now has two methods (multi dispatch)

# The T is the type inside the vector which refed in the func
function myfunc(x::Vector{T}) where T
	b::Vector{T} = x.*3
end
```

# Modules

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

Julia code is represented after compiling as a data struct of type Expr.

- **\$:** Used as interpolation for literal expression in a macro.
- **eval:** Executes the code from Expr data type.
- **:** Turns code into an expression (can also used quote for blocks)

Can use Expr data types as inputs to functions.

## Macros

Compiled code as an expression not executed on runtime but during
parsing.

```julia
macro name()

end

@name() # Run using the @ operator.
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

Designed around environments which can be local to inviduals or projects. Allows exact set of packages and their version in an environment to be controlled and repeated. All tracked in the manifest file.

A better version of python virtualenv.

```julia
activate [project name]

develop --local [package name] #Allows you to use a local version of a 
# package to develop

free [package name] #Go back to the registered version

get "url" or "local path"

pin [package name] #Never update

# Activate another persons project
activate .
instantiate
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

Can instead use \@view to reference a subarray of the original array. Also contiguous are better and if possible arrays should be converted into a contiguous array before operation.

## Other

-   Static array pkg if array is to be fixed size
-   Avoid string interpolation for I/O (dont use \$a)
-   Fix all warnings
-   Avoid unnecessary arrays
-   Use fld(x,y) and cld(x,y) instead of floor and ceil
-   Annotations available to reduce Julia functions for sake of speed ie don't check boundaries.
-   Use \@code_warntype to show non concrete types in red.