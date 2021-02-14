# Exception Handling

In many cases, unexpected behavior may occur during running code, which may lead to the situation that some function is unable to return a reasonable value to its caller. In such cases, it may be best for the exceptional condition to either terminate the program while printing a diagnostic error message or, if the programmer has provided code to handle such exceptional circumstances, then allow that code to take the appropriate action.

In the following example, we define a factorial function in the same way as we did in the  [Short-circuit evaluation](@ref Short-circuit-evaluation) section

```jldoctest expections; output = false
function fact(n)
    isinteger(n) && n >= 0 || error("argument must be non-negative integer")
    return n == 0 ? 1 : n * fact(n - 1)
end

# output
fact (generic function with 1 method)
```

Note that we use the `error` function, which will throw the `ErrorException` if the input argument does not meet the given conditions. This function works quite well and returns a reasonable error message for incorrect input arguments

```jldoctest expections
julia> fact(1.4)
ERROR: argument must be non-negative integer

julia> fact(-5)
ERROR: argument must be non-negative integer
```

However, it is better to use error messages that are as descriptive as possible. In this case, the error message only tells us what conditions the argument should meet, but not the value of the argument. Fortunately, Julia provides several predefined types of exceptions that can be used to create more descriptive error messages. In our example, we want to check whether the argument is a non-negative integer. For such a case, we can use more specific `DomainError` as follows

```jldoctest expections; output = false
function fact(n)
    isinteger(n) && n >= 0 || throw(DomainError(n, "argument must be non-negative integer"))
    return n == 0 ? 1 : n * fact(n - 1)
end

# output
fact (generic function with 1 method)
```

Note that we must use the `throw` function because the `DomainError(x, msg)` function will only create an instance of type `DomainError`, but will not raise an error.

```jldoctest expections
julia> fact(1.4)
ERROR: DomainError with 1.4:
argument must be non-negative integer

julia> fact(-5)
ERROR: DomainError with -5:
argument must be non-negative integer
```

The error message now contains a short description, the input argument's value, and the type of expectation. Now imagine that due to an error, the `fact` function is used to calculate the factorial from a string (or any other non-numeric value)

```julia
julia> fact("a")
ERROR: MethodError: no method matching isinteger(::String)
Closest candidates are:
  isinteger(::BigFloat) at mpfr.jl:859
  isinteger(::Missing) at missing.jl:100
  isinteger(::Integer) at number.jl:20
  ...
Stacktrace:
 [1] fact(::String) at ./REPL[1]:2
 [2] top-level scope at REPL[2]:1
```

In this case, the `MethodError` is raised for the `isinteger` function, and the error says nothing about the `fact` function. We can track that the error occurs when calling the `fact` function using the `Stacktrace` section located under the error message. The `Stacktrace` provides us an ordered list of function calls (from the last one) that preceded the error. In this case, we see that the last function call before the error is `fact(:: String)`. It tells us that the error occurs in the function `fact` with a string as the input argument. In this particular case, it makes sense to define factorial function only for real numbers. This can be done simply by entering the type of the input argument in the function declaration as follows

```jldoctest expections; output = false
function fact_new(n::Real)
    isinteger(n) && n >= 0 || throw(DomainError(n, "argument must be non-negative integer"))
    return n == 0 ? 1 : n * fact(n - 1)
end

# output
fact_new (generic function with 1 method)
```

This function declaration will only work for arguments of a type that is a subtype of `Real`. Otherwise, `the MethodError` will occur

```jldoctest expections
julia> fact_new("aaa")
ERROR: MethodError: no method matching fact_new(::String)
[...]
```

Note that `MethodError` provides two important pieces of information. The first is that the `fact_new` function is not defined for arguments of type `String`. The second one is the list of methods closest to the one we tried to call. In this case, the `fact_new` function has only one method, which works for any subtype of `Real`, as can be checked using the `methods` function

```jldoctest expections
julia> methods(fact_new)
# 1 method for generic function "fact_new":
[1] fact_new(n::Real) in Main at none:1
```

A more precise description and a list of all predefined exception types can be found in the official [documentation](https://docs.julialang.org/en/v1/manual/control-flow/#Exception-Handling).