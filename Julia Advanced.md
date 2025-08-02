---
author:
  - Mo D Jabeen
Date Created: 2025-07-30
title: Julia Advanced
Description: Notes on Julia's Advanced mechanics/ patterns and tips
---
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

# Reusability Patterns

## Delegation Pattern

Instead of using inheritance on a object creating a dependancy on the entire implementation wrap it in a new object and forward onto original object any methods.

*When to use:*
- When original code is not easy to change and has a high reuse %.

*Benefits:*
- Better than inheritance as can be selective in choice of methods to forward or data to access.
- Use accessor to control data access
- Can update easily without effecting old methods

*Negatives*:
- Increases complexity via indirection.

```julia

struct wrapObject
	orgObj :: orginalObject
	extra data ...
end

#Forward
func_to_forward(wO::wrapObject) = func_to_forward(wO.orgObj)

```

**Pkg: Lazy.jl**

## Holy traits Pattern

Avoid doing a conditional branch with type checks especially on abstract trees for behaviour control. Instead use **traits** which are new data types related to the behaviour in a small hierarchy  that return a data type based on the desired behaviour of the original type set.

*Benefits:*
- To allow for easier understanding and extension of code (Dont need to rewrite a conditional loop each time)
- Can use a single trait to help define and control many different behaviours
- The conditional will now be based on behaviour which can be extended to any new object with a given type.
- Also used as interface to enforce implementation of new object (example below.)

```julia

#Traits 
abstract type LiquidityStyle end

struct IsLiquid <: LiquidityStyle end
struct IsIlliquid <: LiquidityStyle end

# Cash is always liquid
LiquidityStyle(::Type{<:Cash}) = IsLiquid()
# Any subtype of Investment is liquid
LiquidityStyle(::Type{<:Investment}) = IsLiquid()

# Trait behaviour example: The thing is tradable if it is liquid

tradable(x::T) where {T} = tradable(LiquidityStyle(T), x)

tradable(::IsLiquid, x) = true
tradable(::IsIlliquid, x) = false

#Interface example

marketprice(x::T) where {T} = marketprice(LiquidityStyle(T), x)

marketprice(::IsLiquid, x) = error("Please implement pricing function for
", typeof(x))
marketprice(::IsIlliquid, x) = error("Price for illiquid asset $x is not
available.")
```


## Parametric Type Pattern

With different typed structs the same structure can be reused in different ways. Allowing for via multiple dispatch a single function to be used. Can be taken to a further complexity by layering in parametric structs inside structs.

Great if the different type structs have very similar behaviour.

```julia

struct SingleTrade{T <: Investment} <: Trade
type::LongShort
instrument::T
quantity::Int
price::Float64
end


# GENERIC OPTION: Calculate payment amount for the trade
function payment(t::SingleTrade{T}) where {T}
	return sign(t) * t.quantity * t.price
end

# SPECIFIC Calculate payment amount for option trades (100 shares per contract)
function payment(t::SingleTrade{StockOption})
	return sign(t) * t.quantity * 100 * t.price
end
```

# Performance Patterns

## Global constants

Can lead to very poor performance, in simple pure examples a untyped global var can reduce speed by 900x. As the complier has to add a lot of overhead to keep the flexibility of an untyped global var. *In complex funcs with large overhead difference may be negligible*
 
**Easier Solutions: (in order of best)**
- Make it a constant
- Send as argument with $ operator
- Make variable typed inside any function using it

**Placeholder**

Using Ref object can hide the variable in a global constant that can be updated. Not as good as real constant but better.


```julia
function add_using_global_variable_typed(x)
	return x + variable::Int
end

# Initialize a constant Ref object with the value of 10
const semi_constant = Ref(10)
```

## Struct of arrays

Placing data in contiguous memory allows best use of CPU. Therefore for performance Struct of arrays >> array of structs.

**Struct Arrays.jl pkg** (Unwrap option allows for layered structs in arrays.)

If to make use of the package you are creating a copy of the current data, there is a trade of between speed and memory.

## Shared Array Pattern

A task broken down to allow multiple processes to concurrently operate, needs to be combined again, the recombination is called reduction. 

Use **Shared Arrays.jl pkg** for this process used in distributed computing.

There is some overhead for the parallel setup but this is normally negligible compared to the overall speed gains.
## Memoization Pattern

Cache results from a function so that if the same variables are reused the result can be gained straight from cache. 

```julia

#Function wrap using anon function
function memoize(f)

	memo = Dict()
	
	x -> begin
		if haskey(memo, x)
			return memo[x]
		else
			value = f(x)
			memo[x] = value
			return value
		end
	end
end

```

If the arguments used to create the cache ie the keys of the dict are mutable this can cause issues, for a constant unique identifier for a given input in this case can use a hash.

**Memoize.jl pkg** 

## Barrier function pattern

Solve issues due to type instability, if required.

@code_warntype used to show instability issues in a func (shows in red).
@inferred used to compare expected type of return to gathered from func.

When the exact argument types are known inside the func it is compiled for that type this is called specialisation. A barrier func separates out a function into smaller versions to allow the smaller version to specialise. As within the function before the smaller function is called the type is known.

Helps to make sure initialised values match the data they are going to work with, can use parametric of eltype in func.

```julia

function double_sum(data::AbstractVector{T}) where {T <: Number}
	#Initialised value matches type of input
	total = zero(T)

	for v in data
		total += v
	end
	return total

end
```

# Maintainability Pattern 

## Sub Module Pattern

Afferent coupling : External entity that depend on current entity
Efferent coupling: External that the current depends on

High afferent means it has to be very stable, high efferent may cause instability due to external changing parts. Best for both to be minimised. Avoid bidirectional coupling.

## Code Gen Pattern

Can use code generating funcs to produce the most concise code, especially if need many similar methods. 

```julia

for level in (:info, :warning, :error)

	lower_level_str = String(level)
	upper_level_str = uppercase(lower_level_str)
	upper_level_sym = Symbol(upper_level_str)
	
	fn = Symbol(lower_level_str * "!")
	label = " [" * upper_level_str * "] "

	#eval used to define the function in a loop.

	@eval function $fn(logger::Logger, args...)

		if logger.level <= $upper_level_sym
			let io = logger.handle
		
				print(io, trunc(now(), Dates.Second), $label)
				
				for (idx, arg) in enumerate(args)
					idx > 0 && print(io, " ")
					print(io, arg)
				end
		
				println(io)
				flush(io)
			end
		end
	end
end

```

OR Define a flexible func and then write new funcs as defined arguments as versions of that func.

```julia

info! (logger::Logger, msg...) = logme!(INFO, " [INFO] ", logger, msg...)

warning!(logger::Logger, msg...) = logme!(WARNING, " [WARNING] ", logger, msg...)

error! (logger::Logger, msg...) = logme!(ERROR, " [ERROR] ", logger,msg...)

```

Closure technique using a returned anon func (these cannot be extended):

```julia

function make_log_func(level, label)
	
	(logger::Logger, args...) -> begin
		if logger.level <= level
			println(io)
		end
	end
end

info! = make_log_func(INFO, "INFO")
warning! = make_log_func(WARNING, "WARNING")
error! = make_log_func(ERROR, "ERROR")

```

## Domain specific Lang

Using string literal macros can create a domain specific language.

# Robustness Patterns

## Accessor Patterns

More important for user based systems where safeguarding areas of the code is crucial.

Benefit: 
- Setup triggers before direct field access
- Allows the core variable names to be changed without need for code wide changes 
- Allows the core variable type to be changed without need for code wide changes 

The change only happens in the accessor func instead of every use of the field. Not the biggest issue unless very large code base.

```julia

get_heatmap(s::Simulation) = s.heatmap

function heatmap!(s::Simulation{N}, new_heatmap::AbstractArray{Float64, N})  where {N}
	
	s.heatmap = new_heatmap
	s.stats = (mean = mean(new_heatmap), std = std(new_heatmap))
	return nothing
end

```

To enforce use best to overwrite the get and set property funcs.

```julia

function Base.getproperty(o::Object, s::Symbol)
	return getfield(o, s)
end
```

Can also make private vars throw error when accessed directly. And update var info tools in repl ie chosen fields to hide.

## Error Handling

Highest level is a good location for try as last chance to catch any errors.

Use catch_backtrace for trace output before error.

```julia

function foo1()

	try
	
		foo2()
	
	catch
	
		println("handling error gracefully")
		pretty_print_stacktrace(stacktrace(catch_backtrace()))
	
	end
end

```

Avoid try catch in loops and if performance is key find other ways to deal with errors.

Use retry func with defined delay between retries (default is Exponential back off)

```julia
#Example of retry
retry(do_something, delays=fill(2.0, 3))("John")
```

# General Patterns

## Singleton Pattern

The same constructor in julia will return the same object, if non mutable.

Val type allows used of value to change the type of data, creating single instances for different inputs. Using dispatch to create different methods for different data, with a main dispatcher that makes the function arguments look cleaner (better for evolvability).

Instead of using Val type can use a parametric structs.

```julia

struct Val(x) end

function process_command(::Val{:open}, filename)
	println("opening file $filename")
end

function process_command(::Val{:close}, filename)
	println("closing file $filename")
end

#Main dispatcher
function process_command(command::String, args...)
	process_command(Val(Symbol(command)), args...)
end


#Paramteric struct version

struct Meh{T} end

function process_command(::Meh{:open}, filename)
	println("opening file $filename")
end

```

## Stubbing / Mocking

Create automated unit testing, in which functions can be replaced with a testing version. Small functions enable this more.

Testing double: Replace a component during testing (considered fake) the real one is called the collaborator func.

A stub replaces a function inside the function being tested, the stub will have hard coded values returned for consistency. Mock also replaces func but is focused on the behaviour of the func not just return values, the behaviour is tracked via global vars.

![[Screenshot 2025-08-01 at 12.10.55.png]]

**Mocking.jl pkg**

```julia

using Mocking

function open_account(first_name, last_name, email)

	#Placed where stub will be used
	@mock(check_background(first_name, last_name)) || return
	:failure

	account_number = create_account(first_name, last_name, email)

	notify_downstream(account_number)

	return :success
end

Mocking.activate()

check_background_success_patch =
	
	@patch function check_background(first_name, last_name)
	println("check_background stub ==> simulating success")
	return true
end

check_background_failure_patch =
	
	@patch function check_background(first_name, last_name)
	println("check_background stub ==> simulating failure")
	return false
end

# test background check failure case
apply(check_background_failure_patch) do
	@test open_account("john", "doe", "jdoe@julia-is-awesome.com")
	== :failure
end

#Vars used to test behaviour, modified in patch for mock
let check_background_call_count = 0,

	create_account_call_count = 0,
	notify_downstream_call_count = 0,
	notify_downstream_received_proper_account_number = false
end

check_background_success_patch =

	@patch function check_background(first_name, last_name)
	check_background_call_count += 1
	println("check_background mock is called, simulating success")
	return true
end

```

## Functional Pipes

*Good for visual scenarios in which there is regular editing/prototyping and mixing of different funcs.*

Pass data sequentially from one func to the next, every func must accept only single argument. Use dispatch for conditional situations creating tree branches. Use broadcasting ie . on operations to avoid copying data between each func. 

```julia

:> : right to left pass
\circle : left to right pass
```


![[Screenshot 2025-08-01 at 12.31.16.png]]

# Anti-Patterns

## Piracy

Package extension is dangerous and should be done very carefully and only if very useful. The extension needs to match a similar behaviour to the original function to avoid hard to debug issues.

## Narrow type

Too narrow a func argument type stops it from being more useful. **A func type really doesnt affect performance, used more to ensure accuracy!**

## Concrete structs

Using abstract types in a struct will need pointers to point to the data, if concrete is used the compiler will instead place the data directly into the struct. Skipping the dereference and improving performance. 

Another benefit of using Parametric types is that they give the flexibility of using abstract types but the implementation is concrete based on the exact type given to the struct, much improving performance. 


