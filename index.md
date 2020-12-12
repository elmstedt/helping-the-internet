# "Test if a value is in a region"
From: [u/omit89/](https://www.reddit.com/user/omit89/)  
Source: [https://www.reddit.com/r/rprogramming/comments/k2ni75/i_am_stuck_with_a_simple_r_programming_problem/](https://www.reddit.com/r/rprogramming/comments/k2ni75/i_am_stuck_with_a_simple_r_programming_problem/)

## Problem as presented

> I have a simple data frame with specifies start and end positions within lists. These start and end positions define the number of regions (i). Now I would like to test whether a given position lies within such a region and if yes I need to know in which region (i).
>
> Here is a simple example data frame:


```r
start <- list(c(5,10,15), c(5) ,c(6,11),c(6,11))
end <- list(c(7,11,17), c(10), c(8,12),c(8,12))
imax <- c(3,1,2,2)
position <- c(11,6,9,8)
example <- data.frame(start = I(start), end = I(end), imax = imax, position = position)
```

> When I have only one start and end position it is no problem (as in row 2 of example):
``` r
data.table::between(example$position[[1]], example$start[[1]], example$end[[1]])
```
```
[1] FALSE  TRUE FALSE
```

> How can I turn this into a function which checks this pairwise for every element (from $i=1$ to $i=max$) within `example$start` and `example$end`?
>
> The second step would be to retrieve for which region `i` (`1` to `imax`) this was `TRUE`.
>
> Thank you.

## Initial comments

I think this is actually a very good and interesting problem for beginners as it presents a few opportunities to explore important ideas in programming and analysis specifically the idea of breaking a problem down into digestible pieces.

This is sure to be a very overlong discussion of what is ultimately a fairly simple problem. Many of the ideas and techniques presented here are much heavier than even moderately experienced programmers will want or need to bring to bear on a problem like this. I can already hear the choruses of people clamoring about how it's much simpler, faster, and easier to just write the code rather than trudge through the ponderous processes I do here.

To which I counter, if that were so no one would need bother ask the internet to help with this problem!

My hope is that this proves somewhat instructive for novice programmers about ways they can approach problems when they hit a wall. I often tell students that every minute they spend in deliberate consideration of a problem before they start to write code causes the problem to get done two minutes faster.

Not that they ever believe me, but I keep trying!

In short, when you have a problem you aren't immediately sure how to tackle, I recommend trying to break it into as many bite sized pieces as you can and plan each of them to death.

With that, let's take a glimpse at what the example data looks like and get started!

```r
example
```

```
      start       end imax position
1 5, 10, 15 7, 11, 17    3       11
2         5        10    1        6
3     6, 11     8, 12    2        9
4     6, 11     8, 12    2        8
```

## Digestible pieces

Making digestible pieces out of a problem is often an iterative process, for very complicated problems you might need to break pieces into pieces. But it always starts with very broad strokes. In almost all problem, the broadest strokes might be something like "for each _thing_ do _some task_." As, reductive as this may be, it's often enough to get the ball rolling.

I find it's often better to work in reverse here as the "task" is almost always easier to identify and describe, and once we know what the `task` it becomes more clear what each "thing" needs to be.

So, let's identify the task. We want to know,

> Now I would like to test whether a given position lies within such a region and if yes I need to know in which region (i).

Our task is to take a `position` and determine which, if any, of the `region`s it is contained in. We'll call this idea `task`.

Now, what is an example of a "thing" we are going to do this to? Well, we need a `position` and a collection of `region`s. Each "thing" here is contained somehow in a row of your data frame, so we'll refer to the "things" as `r`s going forward.

So,

> For each pairwise collection of `region`s and `position`s in the data identify which `region` the `position` is in.

can be described now as,

> For each `r` do the `task`.

Now, we can iterate and break down the `task`. What we want to do is identify the "core" problem to solve. Occasionally, if the `task` is simple enough, the "core" problem _is_ the `task`. 

Let's look at one `r` to give ourselves an idea of what we're up against.


```r
(r <- example[1, ])
```

```
      start       end imax position
1 5, 10, 15 7, 11, 17    3       11
```

```r
tibble::glimpse(r)
```

```
Rows: 1
Columns: 4
$ start    <I<list>> 5, 10, 15
$ end      <I<list>> 7, 11, 17
$ imax     <dbl> 3
$ position <dbl> 11
```

Note that we don't have any objects which represent `region`s, rather we have `start`s and `end`s. So, we have a choice to be either rigid or flexible in our thinking.

### Rigid

#### Thinking
If we are rigid and maintain our thinking we'll need to construct a collection of regions from the contents of `r` before we can identify which of those regions the `position` value is contained in. Our "core" problem is then answering the question "is the `position` in this `region`?

Then we'd re-write or task as,

> For each `r`, construct a 'regions' object which is a collection of `region`s, then for each `region`, ask if `position` is contained within and identify which are true.

or, more simply,

> For each `r`, make the `regions` identify which `region`'s answer to the "core" problem is true.

#### Planning  {.tabset}

For each of the bite-sized sub-problems identified, we will go through the following process.

1. Write a _function contract_ which, in plain English, says what the function will accept and what it will return if given acceptable inputs.
2. Write formalized pseudocode, which is language-agnostic. (Mathematical notation can be considered a form of pseudocode.)
3. Write the function to the contract using the pseudocode as a guide.

This process helps keep our solutions laser focused on what they are supposed to do, avoiding mission creep or meandering. Another benefit is that with well defined contracts and clear pseudocode, we could easily break up the coding work among many individuals and be confident the final functions will integrate seamlessly.

##### `which_regions()`

###### Contract
`which_regions()` will take a data frame containing numeric objects `start` and `end` which are numeric objects of the same positive, non-zero, lengths (representing the start and end positions of regions), and a numeric `position` object. For each `position` `which_regions()` will return the index of the region in which `position` is contained or `0` if there is no region which contains `position`.

###### Pseudocode
```
FUNCTION: which_regions()
INPUTS:   df, a data frame of the following form:
            3 columns:
              start   a list of numeric values representing the lowerbounds of a collection of regions
              end     a list of numeric values representing the upperbounds of a collection of regions
                      each element of end must be the same length as the corresponding  element in start
              position    a single numeric value to test
OUTPUT:   a collection of integer values (one for each row of df), identifying the index of the region the
          position object is contained in

IF is_valid_df(df) returns true
  output <- a collection of numeric objects having a length equal to the number of rows in df
  FOR EACH row of df
    where start, end, and position are those elements in the row of interest
    the corresponding index of output <- get_region(start, end, position)
  RETURN output
ELSE
  THROW an ERROR
```

##### `is_valid_df()`
FOR EVERY row of df is_valid_row(row) is true

###### Contract
`is_valid_df()` will accept a data frame object and return true if the structure of the data frame is valid for this problem.

###### Pseudocode
```
FUNCTION: is_valid_df()
INPUTS:   df, a data frame
OUTPUT:   a single true or false value indicating if the data frame is valid for the problem

IF df is a data frame object
  IF (df has a start column AND
      df has an end column AND
      df has a position column AND
      the position column in df is a numeric collection)
    results <- a collection of logical values having a length equal to the number of rows in df
    FOR EACH row in df
      the corresponding value of results <- is_valid_row(row)
    IF all the values in results are true
      RETURN true
  RETURN false
ELSE
  THROW an ERROR
```

##### `is_valid_row()`

###### Contract
`is_valid_row()` will accept a data frame object with a single row and return true if the structure of the row is valid for this problem.

###### Pseudocode
```
FUNCTION: is_valid_row()
INPUTS:   r, a data frame with a single row
OUTPUT:   a single true or false value indicating if the row is valid for the problem

IF (r is a data frame object AND
    r has one row AND
    r has a start column AND
    r has an end column AND
    r has a position column AND
    the length of the start collection equals the length of the end collection AND
    start is a collection of numeric values AND
    end is a collection of numeric values AND
    position is a single numeric value)
  RETURN true
ELSE
  RETURN false
```

##### `get_region()`

###### Contract
`get_region()` will take two collection of numeric objects `start` and `end` which are have the same positive, non-zero, lengths (representing the start and end positions of regions), and a numeric `position` object. `get_region()` will return the index of the which contains the `position` value or `0` if no such region exists.

###### Pseudocode

```
FUNCTION: get_region()
INPUTS:   start   a collection of numeric values representing the lowerbounds of regions
          end     a collection of numeric values representing the upperbounds of a regions
                  end must be the same length as start
          position    a single numeric value
OUTPUT:   an integer value indicating the index of the region which contains position

IF (start is a collection of numeric values AND
    end is a collection of numeric values AND
    start is the same length as end AND
    position is is a single numeric value)
  regions <- make_regions(start, end)
  results <- a collection of logical values the same length as regions
  FOR EACH region in regions
    the corresponding value of results <- in_region(region, position)
  output <- the index of results which is true
  RETURN output
ELSE
  THROW an ERROR
```

##### `make_regions()`

###### Contract
`make_regions()` will take two collection of numeric objects `start` and `end` which are have the same positive, non-zero, lengths (representing the start and end positions of regions). `make_regions()` returns a collection of the same positive, non-zero, length as `start` and `end`. Each element of this collection is a length-two numeric object defining a region from a value in `start` to the corresponding value in `end`.

###### Pseudocode
```
FUNCTION: make_regions()
INPUTS:   start   a collection of numeric values representing the lowerbounds of regions
          end     a collection of numeric values representing the upperbounds of a regions
                  end must be the same length as start
OUTPUT:   a collection the same length as start and end,
          consisting of length-two collections representing the boundaries of the corresponding region.

IF (start is a collection of numeric values AND
    end is a collection of numeric values AND
    start is the same length as end)
  output <- an empty collection the same length as start and end
  FOR EACH element of output
    the element of outout <- a collection consisting of the corresponding values of start and end
  RETURN output
ELSE
  THROW an ERROR
```

##### `in_region()`

###### Contract
`in_region()` accepts `region`, a length-two collection of numeric values and `position` a numeric value. `in_region()` returns true if `position` is in the `region` and false otherwise.

###### Pseudocode
```
FUNCTION: in_region()
INPUTS:   region  a length-two collection of numeric values representing the boundary of a region
          position  a single numeric value
OUTPUT:   a value indicating if position is contained in the region

IF (region is a collection of numeric values AND
    region is length two AND
    position is a single numeric value)
  IF (position >= the smallest value in region AND
      position <= the largest value in region)
    RETURN true
  ELSE
    RETURN false
ELSE
  THROW an ERROR
```

#### Code {.tabset}

##### `which_regions()`

```r
which_regions <- function(df) {
  if (!is_valid_df(df)) {
    stop("df is not valid")
  } else {
    output <- numeric(nrow(df))
    
    for (i in seq_along(output)) {
      output[[i]] <- with(df[i, ], 
                          get_region(unlist(start), unlist(end), position))
    }
    output
  }
}
```

##### `is_valid_df()`

```r
is_valid_df <- function(df) {
  if (is.data.frame(df)) {
    if (!is.null(df[["start"]]) &&
        !is.null(df[["end"]]) &&
        !is.null(df[["position"]]) &&
        is.numeric(df[["position"]])) {
      all(vapply(split(df, row.names(df)),
                 is_valid_row,
                 logical(1)))
    } else {
      FALSE
    }
  } else {
    stop("df must be a data.frame")
  }
}
```

##### `is_valid_row()`
**Note:** As we code this, we can make some assumptions about our inputs as they will have been checked previously by the calling function.

```r
is_valid_row <- function(r) {
  start <- unlist(r[["start"]])
  end <- unlist(r[["end"]])
  position <- unlist(r[["position"]])
  if (nrow(r) != 1) {
    stop("'r' must be a data frame object with a single row")
  }
  length(start) > 0 &&
    length(end) > 0 &&
    length(start) == length(end) &&
    length(position) == 1 &&
    is.numeric(start) &&
    is.numeric(end) &&
    is.numeric(position)
}
```

At this point we can begin testing some of these functions against our data.

```r
is_valid_row(example[1, ])
```

```
[1] TRUE
```

```r
is_valid_df(example)
```

```
[1] TRUE
```

```r
is_valid_df(example[, -1])
```

```
[1] FALSE
```

##### `get_region()`

```r
get_region <- function(start, end, position) {
  if(!(is.numeric(start) &&
       is.numeric(end) && 
       length(start) == length(end))) {
    stop("'start' and 'end' must be numeric vectors of the same length")
  }
  if (!(is.numeric(position) && length(position) == 1)) {
    stop("'position' must be a length-one numeric vector")
  }
  regions <- make_regions(start, end)
  max(which(vapply(regions, in_region, logical(1), position)), 0)
}
```

##### `make_regions()`

```r
make_regions <- function(start, end) {
  if (!(is.numeric(start) &&
        is.numeric(end) &&
        length(start) == length(end))) {
    stop("'start' and 'end' must be numeric vectors of the same length")
  }
  lapply(seq_along(start), function(i) c(start[[i]], end[[i]]))
}
```

##### `in_region()`

```r
in_region <- function(region, position) {
  if (!(is.numeric(region) && length(region) == 2)) {
    stop("'region' must be a length-two numeric vector")
  }
  if (!(is.numeric(position) && length(position) == 1)) {
    stop("'position' must be a length-one numeric vector")
  }
  position >= min(region) && position <= max(region)
}
```

We can test some more of the components now,

```r
start <- c(1, 4, 7)
end <- c(3, 6, 9)
get_region(start, end, 2)
```

```
[1] 1
```

```r
get_region(start, end, 5)
```

```
[1] 2
```

```r
get_region(start, end, 8)
```

```
[1] 3
```

```r
get_region(start, end, 11)
```

```
[1] 0
```

#### Testing

```r
which_regions(example)
```

```
[1] 2 1 0 1
```

### Flexible

Alternately, if we're flexible in our understanding, we can realize that asking "which `region` contains the `position`" we can recognize that the `region` which contains the `position` is the index where `position` >= `start` and `position` <= `end`.

With this new understanding our task becomes,

> For each row `r` in the data frame `df` identify the index where both `position` >= `start` and `position` <= `end` are true.

We can keep the same `which_regions()`, `is_valid_df()`, and `is_valid_row()` functions and simply update `get_region()`. Since the contract doesn't specify _how_ `get_region()` solves the problem, the function contract remains the same, we only need to update the pseudocode to reflect our new understanding of the task we are doing.

##### `get_region()`

###### Contract
`get_region()` will take two collection of numeric objects `start` and `end` which are have the same positive, non-zero, lengths (representing the start and end positions of regions), and a numeric `position` object. `get_region()` will return the index of the region defined by `start` and `end` which contains the `position` value or `0` if no such region exists.

###### Pseudocode

```
FUNCTION: get_region()
INPUTS:   start   a collection of numeric values representing the lowerbounds of regions
          end     a collection of numeric values representing the upperbounds of a regions
                  end must be the same length as start
          position    a single numeric value
OUTPUT:   an integer value indicating the index of the region which contains position

IF (start is a collection of numeric values AND
    end is a collection of numeric values AND
    start is the same length as end AND
    position is is a single numeric value)
  output <- the index where position >= start and position <= end
  RETURN output
ELSE
  THROW an ERROR
```
#### Code {.tabset}

##### `get_region()`

```r
get_region <- function(start, end, position) {
  if(!(is.numeric(start) &&
       is.numeric(end) && 
       length(start) == length(end))) {
    stop("'start' and 'end' must be numeric vectors of the same length")
  }
  if (!(is.numeric(position) && length(position) == 1)) {
    stop("'position' must be a length-one numeric vector")
  }
  max(which(position >= start & position <= end), 0)
}
```

#### Testing

```r
which_regions(example)
```

```
[1] 2 1 0 1
```
