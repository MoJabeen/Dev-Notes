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

# Patterns

## Reusability Patterns

### Delegation Pattern

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

### Holy traits Pattern

Avoid doing a conditional branch with type checks especially on abstract trees for behaviour control. Instead use **traits** which are new data types related to the behaviour in a small hierarchy  that return a data type based on the desired behaviour of the original type set.

*Benefits:*
- To allow for easier understanding and extension of code (Dont need to rewrite a conditional loop each time)
- Can use a single trait to help define and control many different behaviours
- The conditional will now be based on behaviour which can be extended to any new object with a given type.
- Also used as interface to enforce implementation of new object (example below.)

```julia

abstract type LiquidityStyle end

struct IsLiquid <: LiquidityStyle end
struct IsIlliquid <: LiquidityStyle end

# Cash is always liquid
LiquidityStyle(::Type{<:Cash}) = IsLiquid()

# Any subtype of Investment is liquid
LiquidityStyle(::Type{<:Investment}) = IsLiquid()


# The thing is tradable if it is liquid

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


### Param

