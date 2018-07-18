# Functional programming idioms

## Accumulators
We usually can turn a linear recursive process into an iterative one by introducing an accumulator.
```racket
(define (max-1 l)
  (if (null? (cdr l))
      (car l)
      (max (car l) (max-1 (cdr l)))))

(define (max-2 l)
  (let loop ([l l] [current-max (car l)])
    (if (null? l)
        current-max
        (loop (cdr l) (max (car l) current-max)))))
```

Rewriting a function to use an accumulator is not needed when it doesn't benefit space complexity.
Take list concatenation:
```racket
(define (append l1 l2)
  (if (null? l1)
      l2
      (cons (car l1) (append (cdr l1 l2))))
```
We could rewrite it to use an accumulator, however the space complexity would still be ϴ(n) and this
version is cleaner. In some environments however it may be necessary to use accumulator even for
such cases, because, for example, the call stack is stored in the stack, not in the heap, and therefore
it can not grow too deep.


## TODO
* iterating with a helper function
* relaying on recursion - easily expressing non-trivial computations
* pattern matching and simplifying problems
* generators
* continuations
* mutual recursion
* simulating laziness
