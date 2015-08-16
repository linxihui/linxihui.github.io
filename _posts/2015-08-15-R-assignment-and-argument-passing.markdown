---
layout: post
title:  "R variable assignment and copy"
date:   2015-08-15

---

## Assignment: copy-on-modify

When you do `y = x`,  `y` only binds to the memory of data of `x`. R does not actually allocate new memory and copy `x` to `y`.  But if you then do `x[1] <- 1`,  R detects the modification and allocate memory for `x` by copy so that `y` is not changed.  R is said to copy-on-modify. There is a package for checking memory address and number of bindings.

```r
> library(pryr)
> x <- 1:10
> c(address(x), refs(x))
> [1] "0x103100060" "1"

> y <- x
> c(address(y), refs(y))
[1] "0x103100060" "2"
```

```r
> x <- 1:10
> y <- x
> c(address(x), address(y))
[1] "0x6336498" "0x6336498"

> x[5] <- 6L
> c(address(x), address(y))
[1] "0x63b5318" "0x6336498"
```

You can see that y, x have the same address, but it has 2 names (`x`,`y`) binding to the same memory chunk in the first example (1st example), and addresses differ when one changed (2nd example). 


## Argument passing: pass-by-promise

For argument passing in R,  it is _pass-by-promise_ (un-evaluated expression), and it is evaluated to the actual data only when it is needed (__R is lazy and smart!__). Sometimes, you want an un-evaluated expression and you call `substitute(x)` inside the function, so that you can manipulated the expression. Check `match.call` as an example.

```r
> # Make a BIG matrix
> x = matrix(runif(10000*10000), 10000);
> y = x; 
> # very fast! because no memory allocation happens.
> y[1,1] = -1;  
> # very slow, because y is re-allocated and copy happens.

> address(x)
[1] "0x7f94508e7010"

> ff = function(z)  address(z)
> ff(x)  # very fast and you know it is not copied. 
[1] "0x3ab6798"  
```

Why not the same? I guess that is the address of the _promise_. 

```r
> fff = function(z)  {z2 = z; address(z2)} 
> fff(x)  # evaluate promise z to z2, but not copied
[1] "0x7f94508e7010"  
```

Awesome, it is the same as `x` now! `fff` "evaluates" promise of `z` to `z2`. Indeed, what R does is just binding the memory to `z2`.
