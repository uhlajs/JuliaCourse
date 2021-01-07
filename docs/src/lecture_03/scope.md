
# Scope of Variables

The scope of a variable is the region of a code where a variable is visible. In Julia, there are two main types of scopes: **global** and **local**. A global scope can contain multiple local scope blocks, and moreover, local scope blocks can be nested. There is also a distinction in Julia between constructs which introduce a *hard scope* and those which only introduce a *soft scope*, which affects whether shadowing a global variable by the same name is allowed or not.

The following table shows constructs that introduce scope blocks

| Construct                                          | Scope type   | Allowed within local|
| :--                                                | :--          | :-:                 |
| `module`, `baremodule`                             | global       | ✗                   |
| `struct`                                           | local (soft) | ✗                   |
| `macro`                                            | local (hard) | ✗                   |
| `for`, `while`, `try`                              | local (soft) | ✔                   |
| `let`, `functions`, `comprehensions`, `generators` | local (hard) | ✔                   |

In the table above, there are several constructs that we have not introduced yet. Some of these constructions will be discussed later in the course (modules and structures) and the rest is described in the official [documentation](https://docs.julialang.org/).

## Local scope

A function declaration introduces a new (hard) local scope. It means, that all variables defined inside a function body, can be accessed and modified inside the function body However, it is not possible to access these variables outside the function, i.e. all these variables are local

```@repl local
function f()
    z = 42
    return
end
f()
z
```
Thanks to this property we can use the names most suitable for our variables (x, y, z, etc.) without the risk of clashing with the declaration of other functions. It is possible to specify a global variable inside a function using the `global` keyword

```@repl
function f()
    global z = 42
    return
end
f()
z
```

However, this is not recommended.  If we really need a variable defined inside a function, we should probably return that variable as an output of the function

```@repl
function f()
    z = 42
    return z
end
z = f()
z
```

In the example above, the `z` variable in the function is local and the `z` variable outside of the function is global. These two variables are not the same.


## Global scope

Each module introduces a new global scope, separate from the global scope of all other modules. Note that the interactive prompt (aka REPL) is in the global scope of the module `Main`

```@repl global
module A
    a = 1 # a global in A's scope
    b = 2 # b global in A's scope
end
a # errors as Main's global scope is separate from A's
```

Modules can introduce variables of other modules into their scope through the using or import statements or through qualified access using the dot-notation

```@repl global
using .A: b # make variable b from module A available
A.a
b
```

Note that while variable bindings can be read externally, they can only be changed within the module to which they belong

```@repl global
b = 4
```

Note, that variables in the global scope can be accessed anywhere inside the global scope even in the local scopes defined in that global scope. In the following example, we define a variable `c` in the Main's global scope and then we define a function `foo` (that introduces a new local scope inside the Main's global scope) and inside this function, we use the variable `c`

```@repl
c = 10
foo(x) = x + c
foo(1)
```

However, it is not recommended to use global variables in this way. The reason is, that global variables can change their type and value at any time and therefore they cannot be properly optimized by the compiler. We can see the performance drop in a simple test

```@repl global_test
x = rand(10);
y = rand(10);
f_global() = x .+ y
f_local(x, y) = x .+ y

hcat(f_global(), f_local(x, y))
```
In the example above, we defined two functions that do the same thing. The first function has no arguments and returns a sum of two global variables `x` and `y`. The second function also returns a sum of variables `x` and `y`, however, in this case, these variables are local since they are introduced as the inputs to the function. If we use the `@time` macro, we can measure the time needed to call these two functions

```@repl global_test
@time f_global();
@time f_local(x, y);
```

We can see that the second function is faster and also needs fewer allocations. The reason is, that when we call the `f_local` function for the first time, with arguments `x` and `y`, the function was optimized for the arguments of these types. Each time a function is called for the first time with new types of arguments, it is compiled. This can be seen in the following example, where the first call is slower due to compilation

```@repl global_test
a, b = 1:10, 11:20;

@time f_local(a, b);
@time f_local(a, b);
```

On the other hand, the `f_global` function cannot be optimized because it contains two global variables, and these two variables can change at any time.