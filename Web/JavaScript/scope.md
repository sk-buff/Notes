## Scope in JavaScript

### 1. Lexical Scope

Just as it's name says, lexical scope determines the scope of variables at lex time. Which means all scopes are determined after the program is written. JS doesn't has the concept of code blocks, so the scope of a variable is often determined by function definitions. 

**I think** the scope chain (of functions) works like following:

(1) scope chain is used to search variables, the search process is easy to understand so I didn't write it here.

(2) when the JS program starts to run, the **current scope chain** (we use the term current scope chain to infer the scope chain used to search for variables in the current execution context, we'll see later functions have their scope chain stored in the internal `[[Scope]]` attribute) only contains a single scope, which is the global scope.

(3) scope chain will only change when calling/exiting a function (`with` statement is ignored here).

(4) when a function is defined (no matter defined with the `function` keyword or with a function expression), it has a basic scope chain which is stored in the internal `[[Scope]]` attribute, the basic scope chain is the same as the scope chain used in current (when the function is defined) execution context.

(5) each execution environment has a corresponding **variable object** which stores all variables (including functions). It's easy to guess that elements in the scope chain are exactly these variable objects.

(6) when a function is called and the execution context of it is created in the stack, an corresponding **activation object** is also created as the variable object of this function. When initially created, activation object only contains the **argument object **  which contains all actual parameters of the function when it is called and some other attributes (e.g. `arugument.callee`). As the execution environment has changed, the current scope chain should also change. The JS engine will put the newly created argument object at the head of the function's basic scope chain and use the result as the current scope chain.

(7) when a function is exited, which also changes the execution context, the current scope chain will switch to the scope chain of the down adjacent execution environment in the stack (I guess the scope chain of each execution context is stored in the stack).

(8) when the variable object (actually the activation object) of a function no longer exists in any execution context's (Note there may be many execution contexts in the stack) scope chain or any function's basic scope chain (these basic scope chains have the potential to be part of an execution context's scope chain because the function can be called), all variables in the variable are freed which means you can never access these variables.

(9) the activation variable of a function and the function variable itself have no necessary relationship. Sometimes you can access the function variable but the function doesn't has a activation variable, e.g., you defined a function but did not call it. Sometimes the function variable can no longer be accessed but it's activation variable still exists, the IIFE which is discussed later is a good example. Sometimes a function may has more than one activation variable, this happens when the function is recursively called, here is an example:

```javascript
var i = "abc";

function f(a) {
    if (a > 0) {
        let i = a;
        f(a-1);
        console.log(i);
    }
    else if (a == 0) {
        console.log(i);
    }
}

f(1)
```

the output is:

```
abc
1
```

`f` has two execution context which are `f(1)` and `f(0)`, thus has two activation object. Variable `i` is defined in the execution context of `f(1)` but not defined in that of `f(0)`, so `console.log(i)` in `f(1)` will output the local `i` while the same statement in `f(0)` will output the global `i`.

If you use `var i = a;` instead of using `let` in the above code segment, the output will become:

```
undefined
1
```

the reason can be found in .

### x. Immediately Invoked Function Expression

You can use *immediately invoked function expression (IIFE)* to simulate a code block, the syntax is like following:
```javascript
(function(){
    // this is a code block
})()
```