---
author:
  - Mo D Jabeen
Date Created: 2025-07-30
title: Julia Advanced
Description: Notes on Julia's Advanced mechanics/ patterns and tips
bibliography:
  - Julia High Performance
---
# Compiler

Dynamic high level languages often use a C based inner kernal for performance, Julia is instead all coded in julia. Julia uses Just in Time (JIT) compilation, where the code is compiled at runtime. Ahead of time (AOT) compilation is compiled prior and stored in memory. The other option is the interpreted languages during runtime, which have a greater overhead and therefore worse performance.

Julia compiles multiple versions of the code optimised for different data types based on the expected use cases and inferred types. Type inference and code specialisation is the secret sauce to Julia's speed. 

**Adding type annotations therefore for arguments and local variables does not help performance!**

***Helps only in storage locations ie composite types and global vars.***

## Inlining Functions

The compiler may place small functions directly into the body of an outer func to allow for better optimisations. It is selective with inlining to ensure the code size does not get too large. 

Can encourage inlining using for performance sensitive section ie inner loop use **@inline** macro. Or stop inlining for example on a barrier function use **@noinline**.

Disable all inlining (if need a direct connection between source code and machine code) using cmd line option -inline = no. 

## Constant Propagation

An argument can be replaced with a constant if its known after the initial compile, this is done on pure funcs that don't alter anything (mutate args or global vars).

Writing small pure funcs is good practise to help compiler improve performance.

# Type stability

**Function:**  Abstract operations
**Methods:** Specialised type implementation 

Julia needs to infer the params and return types of all functions. To do this effectively the code needs to be type stable.

**Type stability:** when the return type only depends on the argument types not the values.

```julia
function pos(x)
	if x < 0
		return 0
	else
		return x
	end
end

# Here the type changes based on input value ! UNSTABLE
julia> typeof(pos(2.5))
	Float64

julia> typeof(pos(-2.5))
	Int64

```


**@code_warntype:** Used to show unstable areas, they will be highlighted in red ambiguous types normally Any or Union. These need to be made certain.

Variables in loop should also not be changing types during the loop to avoid instability. Some issues can be solved by breaking the function down giving hints to the compiler: [[#Barrier function pattern]].

```julia

function string_zeros(s::AbstractString)

	n=1000_000
	
	x = s=="Int64" ? Vector{Int64}(undef,n) : Vector{Float64}(undef, n)

	# Type of x is unknown to the compiler as it works on func boundaries
	# Therefore adding a boundary here will help
	for i in 1:length(x)
		x[i] = 0	
	end
	
	return x
end

function string_zeros(s::AbstractString)

	n=1000_000
	
	x = s=="Int64" ? Vector{Int64}(undef,n) : Vector{Float64}(undef, n)

	#Barrier func helping make x known as its become a param instead of a
	# local var
	return fill_zero(x)
end

```

### How should functions be returned?

A function should always return the same type, so if the type is ambiguous convert before return.

```julia
zero(x) #Gives a 0 val in type of x
oftype(x,y) #Gives y as type of x
```

Variables changing type within the function should also be avoided.

### Types used as parameters for other types

If a var is created using an unknown type ie a matrix or array then its type will be checked on every access.

```julia
fill(5,ntuple(d->3,N)) 
#If N is unknown then will create an unknown type matrix.
::Val{N} 
#is used a concrete type to specify dimensions, pass as Val(N) in func
```

### What is dangerous when using Multi Dispatch?

If at compile time Julia does not know the type it will be checked at run time slowing down compute.

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

# Numbers

Julia does not do any overflow or underflow checking on numbers, if a number overflows it will often wrap from max to min value. Underflows will set the number to 0. It has no auto-upgrade to larger type feature other dynamic languages have that do check for overflow (python).

BigInt is a type used for very arbitrarily large integer values, it does increase overhead. Floats can also be used to hold large values and avoid overflow as they will adjust precision if a value gets too large. Up to 4 quadrillion (52 bits) the precision for ints and floats is the same.

## Floats

Default float is float64 (64 bits) uses the IEEE 754 binary standard. 

Floats are calculated by splitting the binary into 3 sections the first bit is the sign (+/-). The next 11 are the exponent calculated using $2^{n-1023}$ where where is the decimal value of the 11 bits.

The next 53 bits is the significand and is a fraction value calculated using the formula: $1 + \frac{b_n}{2^n} ...$ where n is pos and b is binary value.

Using this formula not all decimal values will be able to be represented exactly. The example below shows how 0.1 is not represented exactly in float form.

```bash
julia>0.1 > 1//10
true
```

Machine epsilon (eps) shows the diff between two floating point values that can be represented in float form. Therefore the eps value is all the values + given float that cant be represented in float. The next float that can be represented is a result of adding 1 to the unit in last place (ULP) or the least significant bit.

### Conversion

To skip bounds checking when converting to a different type use %.

```julia

x = UInt32(4294967297)

# Faster with no bounds checking
x = 4294967297 % UInt32

```

## Fast Math

Use **@fastmath** to loosen IEEE constrains potentially reducing accuracy for performance improvement. 

Advice is to test and measure extensively before use.

## K-B-N

For a more accurate method of summing floats use KahanSummation pkg, will negatively effect performance. Standard sum method has $O(\sqrt{n})$ and KBN has $O(n)$ error.  Generally fine to use sum().

## Subnormal

Subnormals numbers are smaller than can be represented without using any leading 0 in the significand. If the exponent cannot represent the number as its to small for the power formula the significand will have leading zeros which is a subnormal.

Subnormals are a significant performance slow down, however are useful for very small calculations to stop underflow. 

Improve performance by setting small values to 0 or at an extreme use the cmd line option -set_zero(subnormal) setting all subnormals to 0 reducing accuracy. A subnormal example often found is via exponential decay toward 0.

# Arrays

Arrays are of the form Array{T,N}; T is type and N is number of elements.

In Julia elements of the array are stored within the arrays primary memory locations, it does not use like other dynamic languages (that do not have enough type info) for concrete types pointers. Improving performance !

Column ordered arrays, meaning cols are stored together in memory. C is row ordered.

![[Screenshot 2025-08-07 at 16.21.24.png]]

Sequential reads should access contiguous areas in memory therefore col by col. 

```julia
function col_iter(x)
	s=zero(eltype(x))
	
	for i in 1:size(x, 2)
		
		for j in 1:size(x, 1)
		
			s = s + x[j, i] ^ 2
			
			x[j, i] = s
			
		end
	end
end
```

## Adjoint

Adjoint type allows the array to be flipped to use efficiently row ordered functions. 

```julia

#b will be Adjoint type
b=a'

```

## Bounds checking

Can remove bounds checking with **@inbounds**, to be used when loop limits depends directly on size.

```julia
function prefix_inbounds(a, b)

	@inbounds for i in 2:size(a, 1)
		a[i] = b[i-1] + b[i]
	end
end
```

Cmd line flag *-check-bounds* set to yes will ignore inbounds macros and when no will turn off all bound checking.
## Sizehint()!

Define expected size of a dynamic array - probably does not give much benefit as new julia is fast with dynamic allocation.

## Broadcasting

Writing vectorised code is not a performance gain unlike numpy as julia loops are already very fast.

When using chained functions broadcasting (due to optimisations via loop fusion) can give a performance boost especially with an in place vector fusion assignment and preallocated array.

```julia

# .= in place assigmnent
b .= cos.(sin.(a))

```

Nested dot calls are fusing, they are combined at the syntax level into a single loop. And use preallocated arrays instead of temp ones (SHOULD BE USED WHENEVER POSSIBLE!).\

```julia
f(x) = 3x.^2 + 4x + 7x.^3
fdot(x) = @. 3x^2 + 4x + 7x^3 

# @. converts all vector ops into a dot operation
# fdot(x) is 10x faster
```

## Slices

A slice creates a copy of the array, which is okay if many operations are to be done on the copy, however, could be worse if the cost of the copy outweighs the operations to be done on it.

Can instead use \@view to reference a subarray of the original array without making a copy.  SubArrays are almost as fast as a dense array. 

Also contiguous are better and if possible arrays should be converted into a contiguous array before operation.

```julia

A = zeros(3, 3); 

#@views converts each operations return into a view, like a wrapper
@views for row in 1:3 
	b = A[row, :] # b is a view, not a copy 
	b .= row # assign every element to the row index 
end

function sum_cols_matrix_views(x::Array{Float64, 2})

	num_cols = size(x, 2); num_rows = size(x, 1)
	
	s = zeros(num_cols)
	
	for i = 1:num_cols
		s[i] = sum_vector(@view(x[:, i]))
	end
	
	return s
end

```

## SIMD

Single instruction multiple data parallelisation allows for parallel calc on the CPU improving performance. Can encourage its use using **@simd**, however should meet the following criteria:

- Each iteration of the loop is independent of the others. That is, no iteration of the loop uses a value from a previous iteration or waits for its completion. The significant exception to this rule is that certain reductions are permitted.
- The arrays being operated upon within the loop do not overlap in memory.
- The loop body is straight-line code without branches or function calls.
- The number of iterations of the loop is obvious. In practical terms, this means that the loop should typically be expressed on the length of the arrays within it.
- The subscript (or index variable) within the loop changes by one for each iteration. In other words, the subscript is unit stride.
- Bounds checking is disabled for SIMD loops. (Bound checking can cause branches due to exceptional conditions.)

Can force SIMD by using the pkg and its special types.

## StaticArrays.jl

For fixed size arrays, recommended for arrays up to 100 elements.

## StructArrays.jl

Gives the benefits of using structs (type usage via dispatch) and the speed of arrays. [[#Struct of arrays]].

Increase use on memory as creates copy of array.
## Indexing

Linear indexing (single value index) is fast but not always the best option, cartesian index ([x,y,z]) maybe better (SubArrays work better with cartesian). 

Best to use **eachindex()** func instead of direct indexing or iterate directly over the list so that the optimal choice is made for you.
# Performance tips

Performance critical code should be inside a function.
**AVOID:**

- Initially always look to optimise the algorithm !
- Untyped global variables, if they must be used annotate the type at point of use.
- Problems with type stability
- Many temporary small arrays
- Keyword args have slight increase in overhead 

At point of use annotation:

```julia
# good if x is global var
for i in x :: Vector{Float64}
```

## In place functions

Preallocating and mutating existing vars via in place function can improve performance by removing unnecessary new allocations especially in loops.

```julia

function xpow_loop(n)

	s = 0
	
	for i = 1:n

		# xpow has many new allocations returning a new array
		s = s + xpow(i)[2]
	
	end
	
	return s

end

function xpow_loop_noalloc(n)

	r = [0, 0, 0, 0]
	s = 0
	
	for i = 1:n

		#New allocations removed by in place func
		xpow!(r, i)
		
		s = s + r[2]
	end
	
	return s
end

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

## How should functions be broken down?

Generally functions of made of the setup and compute on the setup vars. The compute should broken into its own function.

## Other

-   Avoid string interpolation for I/O (dont use \$a)
-   Fix all warnings
-   Avoid unnecessary arrays
-   Use fld(x,y) and cld(x,y) instead of floor and ceil

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

# Misc

## Composition

Julia does not support implementation inheritance (data/subtype structure) only does interface inheritance. 

- All concrete types are final, abstracts cant have a concrete type.

**Benefits:**
- Avoids base class changes breaking things
- Avoids need for a lot of overwriting to get behaviour you need which is bad for peformance (square rectangle problem)

## Variance

Covariant: A is part of B
Contravariant: B is part of A. 
 
Parametric types are invariant ie even if Mammal <: Animal does not mean Array{Mammal,1} <: Array{Animal,1}.

This can be fixed using Array{T,1} where T <: Animal.

Methods are covariant.

## Diagonal Rule 

In a parametric when T is used for consistency the match must be of concrete type not abstract. 

Be aware things wont always work as expected in complex situations with accessing the defined type in parametric methods.
