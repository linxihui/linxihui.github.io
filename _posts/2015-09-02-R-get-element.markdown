---
layout: post
title:  "`[`, `[[`, and `$`"
date:   2015-09-02

---

Perhaps `[`, `[[` and `$` are the most frequently used functions/operators in R for everyone. But how much do you know about them?

# Behaviors for vector, matrix and data.frame

## for vector
I mean, atomic vector like `c(5, 1, 10)` or recursive vector like list.

```r
> x <- c(A = 'T', C = 'G', G = 'C', T = 'A')
> x
  A   C   G   T 
"T" "G" "C" "A" 

> lx <- list(AA = 1:4, BB = 3:5)
> lx
$AA
[1] 1 2 3 4
$BB
[1] 3 4 5
```

1. `[` keeps attributes (names, class, etc), while `[[` does not.

```r
> x[1] # or x['A'], name kept
  A 
"T" 
> x[[1]]  # or x[['A']], name dropped
[1] "T"

> lx[1]  # or lx['AA']. still a list. Class kept
$AA
[1] 1 2 3 4

> lx[[1]]  # or lx[['AA']].  NOT a list
[1] 1 2 3 4
```

2. `[` takes a vector of indices, while `[[` cannot  (of course as from the above property)

```r
> x[c('A', 'T')]                                                                       
  A   T 
"T" "A" 
> x[[c('A', 'T')]]
Error in x[[c("A", "T")]] : attempt to select more than one element
```

3. `[` accepts negative integer to extract "all-but-these" element, while `[[` does not accept negative integer.
4. `$` can be used in recursive type like list, data frame and environment, but not atomic type like c(1,2,3). Conceptually `list$i` is equivalent to `list[['i', exact = FALSE]]`, where `i` must be literature names (string).

```r
> lx[['A']]  # lx has elements of AA, BB, but not A.
NULL
> lx[['A', exact = FALSE]]                                                                                                                                                   
[1] 1 2 3 4
> lx$A  # use `` to wrap over the name if it contains an invalid name character(e.g., space), e.g., lx$`var name`
[1] 1 2 3 4
```

So obviously, you will choose `$` over `[[` when you want partial match or simpler syntax.  But you may want to use `[[` over `$` if you want exact match, or if `i` is an expression or variable.


## for matrix

First,  matrix is an atomic object, but is not of a vector type.

1. `$` cannot be used.  Both `[` and `[[`, with one argument like `matrix[1]` and `matrix[[1]]` is valid and is exactly the same, with both does not keep attributes.  
2. `matrix[vectorOfIndices]` behaviours identical to `as.vector(matrix)[vectorOfIndices]`, except when `vectorOfIndices` is missing (which does nothing but return the object itself)
3. `matrix[i, j]`:  arguments are matched by position, not keyword, i.e., `matrix[j = 2, i = 1]` returns `matrix[2, 1]` NOT `matrix[1,2]`.
4. `matrix[1:3, 1]` or `matrix[1, 1:3]` returns a vector, NOT matrix.  To request a matrix, use `matrix[1:3, 1, drop = FALSE]`, etc.


## for data.frame

Data frame is an interest object, which is kind of a mix of matrix and list.  Indeed,

1. data.frame is a list, with additional matrix-like attributes and it is always named (column names).   `is.list(data.frame)` will return true. Therefore, `[` (one indices vector), `[[`, `$` behave exactly the same as a list.  Thus, `data.frame[1]`, `data.frame[[1]]` returns the first column, but one is a data.frame of one column, and the other is vector of first column.
> Note: Indeed, there is a difference between a data.frame and a list.  data.frame is NOT a vector (similar to matrix) while list is. Try `is.vector(data.frame)` and `is.vector(list)`.
2. `data.frame[i, j]` behaves identical to `matrix[i,j]`, except when `i` is vector of integer of length 1, while `j` is vector of length > 1, e.g, `i = 2` where `matrix[2, 1:3]` returns a vector but `data.frame[2, 1:3]` gives a data.frame. (Think about why this is reasonable? Hint,  data.frame is a list)


# Use `[`, `[[` or `$` as function to apply-functions

Example, get the first entry or first 2 entries of each element in a list.  Use them, surround them by back-ticks `. Again, bear in mind the difference between these operators. 

```r
> ly = list(AA = c('a' = 1, 'b' = 2), BB = c(3, 4, 'a' = 5, 6))
> ly
$AA
a b 
1 2 
$BB
    a   
3 4 5 6 

> sapply(ly, `[`, 1) # In the outcome, the name of first element is 'AA.a' as `[` retains names
AA.a   BB 
   1    3 
> sapply(ly, `[[`,1)
AA BB 
 1  3 
> sapply(ly, `[`, i = 1:2)
  AA BB
a  1  3
b  2  4
> sapply(ly, `[[`, 'a')
AA BB 
 1  5 
```

# Define `[`, `[[` or `$` for your own object

```r
> lz <- structure(list(AA = c('a' = 1, 'b' = 2), BB = c(3, 4, 'a' = 5, 6)), class = 'myclass')
> lz[1] # use `[` method for list
$AA
a b 
1 2 

# redefine `[` to return the i-th entry of each element of a list

> `[.myclass` <- function(x, i) sapply(x, `[[`, i)

> lz[1]
AA BB 
 1  3 
> lz['a']
AA BB 
 1  5 
```
