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

## Relaying on recursion
Recursive call doesn't need to be the last one (tail call). Using recursion earlier in
a procedure can lead to beautiful, expressive code (although with space complexity of
ϴ(n) beacuse of growing stack).

Implementation of a parser and evaluator of a simple language, consisting only of constants and
subtraction of two expressions:
```ocaml
exception Error of string;;

type exp = Const of int | Diff of exp * exp;;
type token = Num of int | Minus;;

let rec parse_exp = function
    | [] -> raise (Error "End of input")
    | Num i :: tokens -> (Const i, tokens)
    | Minus :: tokens -> let (exp1, tokens_left) = parse_exp tokens in
                         let (exp2, tokens_left) = parse_exp tokens_left in
                         (Diff (exp1, exp2), tokens_left)
;;

let rec eval = function
    | Const i -> i
    | Diff (exp1, exp2) -> (eval exp1) - (eval exp2)
;;

let ts = [Minus; Minus; Num 12; Num 2; Minus; Minus; Num 6; Num 1; Num 3];;
let (exp, _) = parse_exp ts in eval exp;;
```
```ocaml
val ts : token list =
  [Minus; Minus; Num 12; Num 2; Minus; Minus; Num 6; Num 1; Num 3]
- : int = 8
```

## Generators
In functional programming, functions are treated as black boxes and we cannot see
inside them or change their contents. We can, however, "edit" a function by wrapping
it in a lambda that will handle the new behavior and leave the rest to the original
procedure.

```racket
;; empty environment has no keys in it
(define (empty-env)
  (λ (key)
    (error "No such key:" key)))
;; extending an enviroment is a matter of handling only one additional case
(define (extend-env key value env)
  (λ (k)
    (if (eq? k key)
        value
        (env k))))
        
(define env (extend-env 'a 10
                        (extend-env 'b 20
                                    (empty-env))))
(env 'a)    ; 10
(env 'b)    ; 20
(env 'name) ; error
```

```racket
(define (id x) x)

(define (prepender l)
  (let loop ([l l] [f id])
    (match l
      ['() f]
      [(cons x xs) (loop xs (λ (l) (f (cons x l))))])))

(define (reverse-prepender l)
  (let loop ([l l] [f id])
    (match l
      ['() f]
      [(cons x xs) (loop xs (λ (l) (cons x (f l))))])))

(define (rec-prepender l)
  (match l
    ['() id]
    [(cons x xs) (λ (l) (cons x ((rec-prepender xs) l)))]))

(define (rec-reverse-prepender l)
  (match l
    ['() id]
    [(cons x xs) (λ (l) ((rec-reverse-prepender xs) (cons x l)))]))
```

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
* pattern matching and simplifying problems
* continuations
* simulating laziness
