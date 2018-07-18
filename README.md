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

## Functional dispatch
Many data structures can be implemented by hiding data in closures and returning functions operating on
this data.
```racket
(define (cons head tail)
  (define (dispatch msg)
    (match msg
      ['car head]
      ['cdr tail]
      [else (error "Unknown operation")]))
  dispatch)
(define (car pair) (pair 'car))
(define (cdr pair) (pair 'cdr))
```
```racket
(define (empty-set)
  (define (make-set elements) ; assume that the list contains no duplicates
    (λ (msg)
      (match msg
        ['member? (λ (x) (not (not (member x elements))))]
        ['insert (λ (x) (make-set (if (member x elements)
                                      elements
                                      (cons x elements))))]
        ['remove (λ (x) (make-set (remove x elements)))]
        [else (error "Unknown operation")])))
  (make-set '()))
(define (set-member? s x) ((s 'member?) x))
(define (set-insert s x) ((s 'insert) x))
(define (set-remove s x) ((s 'remove) x))
```

## Mutual recursion
Mututal recursion can be helpful for breaking complex code apart.

Implementation of a finite state machine only accepting gammar `(a b)* a`:

```racket
(define (need-a lst)
  (cond
    [(null? lst) #f]
    [(eq? 'a (car lst)) (need-b (cdr lst))]
    [else #f]))

(define (need-b lst)
  (cond
    [(null? lst) #t]
    [(eq? 'b (car lst)) (need-a (cdr lst))]
    [else #f]))
    
(define accept need-a)
```
In some languages, like SML or OCaml, you cannot refer to a function that hasn't been defined yet
(above in the file or imported) and have to use special syntax:
```ocaml
let is_even n = if n = 0
                then true
                else is_odd (n - 1)
and is_odd n = if n = 0
               then false
               else is_even (n - 1)
```
However, you never need those special constructs.
```ocaml
let is_even_h is_odd n = if n = 0
                         then true
                         else is_odd (n - 1)
let rec is_ odd n = if n = 0
                    then false
                    else is_even_h is_odd (n - 1)
let is_even = is_even_h is_odd
```

## TODO
* iterating with a helper function
* relaying on recursion - easily expressing non-trivial computations
* pattern matching and simplifying problems
* generators
* continuations
* simulating laziness
