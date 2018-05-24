
# JavaScript Asynchronous Context

## Overview
This document defines a model for "JavaScript Asynchronous Context".  This model includes terminology & definitions, programmatic "types" that result from these definitions, and relations between these types.  These utlimately form an API that expose the model, and allow us to reason about and solve common JavaScript "async code execution" problems

## Principles
We strive to adhere to some basic principles in this document.  We believe that adhereing to these principles will ensure simplicity and help guide the model.

1.  Definitions should flow from well-understood, well-defined constructs in synchronous programming.
2.  Definitions should be independent of the host environment (e.g., Node.js, the browser).
3.  Constructs should enhance JavaScript programmers' understanding of async code flow & async boundaries.
4.  Model should allow for simplified reasoning about async code execution.
5.  Resulting Data Structures and APIs and should support solving common "async understanding" use cases (e.g., long stack traces, continuation local storage, visualizations, async error propogation, user-space queueing...).
6.  Model should allow for implementations at different layers of the stack (VM, host or "monkey-patched"), and implementations in appropriate languages (c++ or JavaScript).

## A Simple Example
We'll start with a simple example that is not controversial. Most JS programmers should have an intuitive understanding of where the async boundaries are in this code.  We'll build off of this example going forward.  Implicit in this example is an undefined concept of "async context".  We'll build stronger definitions later on, but for now, we'll use "async context" to refer to JS programmers' colloquial understanding of asynchonrous function calls.

```javascript
function f1() {
    let i = 0;
    let interval = setInterval(function f2() {
        if ( i < 2) {
            myLoggingAPI(`i is ${i}`);
            ++i;
        } else {
            clearInterval(interval);
        }
    }, 1000);
}
```

There are three interesting constructs in the above example:
  - First, the function `f2` is a function created in one "async context" and passed through an API.  When `f2` is invoked later, it is a "logical continuation" of the "context" in which it was created.
  - Second, the function `setInterval` takes a function as a parameter, and invokes that function later during program execution.  Note that when this parameter is invoked, the call to `setInterval` has completed - i.e., `setInterval` is not on the stack when `f2` is invoked. 
  - Third, `f2` is invoked precisely twice.  These two invocations are distinct.

Let's give these three concepts some names and some initial definitions:

   - A **Continuation** is a JavaScript function that retains a link to the "async context" in which it is created.  Upon invocation, it will establish a new "async context", and will provide a "logical continuation" of the "async context" in which it was created.  In the example above, `f2` is a **Continuation**.
   - A **Continuation Point** - is a function that accepts a **Continuation** as a parameter.  This logically represents an async boundary; functions passed to a **Continuation Point** are invoked at some later point in time.  In our example above, `setInterval` is a **Continuation Point**. 
  -  A **Continuation Invocation** is a specific invocation of a `continuation`; more precisely, a **Continuation Invocation** is the *period of time* where a `continuation` frame is on the stack.  As soon as the continuation frame pops off the stack, then that **Continuation Invocation** is completed. Note that a `continuation` instance can have more than one **Continuation Invocation** instances associated with it.  In our code sample above, there are precisely two **Continuation Invocation** instances associated with the `continuation` f2. 

## A lower-level view

At runtime we can view the example above as two distinct call stacks at specific points in program execution.  These look something like this:

```
---------------------------
|   setTimeout()          |
---------------------------
|   function f1()         |
---------------------------
|    ...host code...      |
---------------------------
|    ...host code...      |
---------------------------
|    host function A()    |
---------------------------
```

and

```
---------------------------
| function x()            |
---------------------------
| function myLoggingAPI() |
---------------------------
|    function f2()        |
---------------------------
|    ...host code...      |
---------------------------
|    host function B()     |
---------------------------
```

Since we previously made a distinction between a regular JavaScript function and a `continuation`, let's make the same distinction in our pictures of the callstacks:

```
--------------------------- 
|   setTimeout()          | 
--------------------------- 
|   function f1()         | 
---------------------------      
|    ...host code...      |      
--------------------------- 
|    ...host code...      | 
===================================
|    Host Continuation A()        |
===================================
```

and

```
---------------------------
| function x()            |
---------------------------
| function myLoggingAPI() |
===================================
|    continuation f2()            |
===================================
|    ...host code...      |
---------------------------
|    ...host code...      |
===================================
|   Host Continuation B()         |
===================================
```

Some notes about the pictures above:
  - We've introduced a "Root Continuation" as the bottom frame.  This  illustrates a basic assumption that all code is executing in the context of a `Continuation Invocation`.
  - Multiple `continuations` can be on the stack at the same time, which result in mulitple `Continuation Invocations`. For any given stack frame, the "context" is the first `Continuation Invocation` below the given frame on the stack.

Here, we've updated our pictures of runtime callstacks, giving  labels to the `Continuation Invocation` instances:

```
---------------------------                    -----------                            
|   setTimeout()          |                              |
---------------------------                              |
|   function f1()         |                              |
---------------------------                              \/
|    ...host code...      |                  Continuation Invocation ci1
---------------------------                              /\  
|    ...host code...      |                              |
===================================                      |
|    Host Continuation A()        |                      |
===================================            -----------
```

and

```
---------------------------                    -----------
| function x()            |                              |
---------------------------                              \/                   
| function myLoggingAPI() |                  Continuation Invocation ci3                        
===================================                      /\
|    Continuation f2()            |                      |
===================================            -----------
|    ...host code...      |                              |
---------------------------                              \/
|    ...host code...      |                   Continuation Invocation ci2                   
===================================                      /\
|   Host Continuation B()         |                      |
===================================            -----------
```

## Link Context
We define the `Link Context` `link-c` of a `Continuation` `c` as a pointer from `c` to the `Continuation Invocation` instance where `c` was constructed.  In the example from above, the `continuation` `f2` has a `link context` pointing to `Continuation Invocation` `ci1`.

## Ready Context
Consider the following example:

```javascript
const p = new Promise((resolve, reject) => {
    setTimeout(function f1() {
       p.then(function f2() {
             myLoggingAPI('hello');
         });
     }, 1000);

     setTimeout(function f3() {
         resolve(true);
     }, 2000)
});
```

In this code example, we have:
  - `setTimeout` and `Promise.then` as our `continuation points`. 
  - four `continuations` - `f1`, `f2` and `f3`, plus the "root continuation" that 
    the initial code is executing in. 
  - four `Continuation Invocations`.

We define the `Ready Context` (//TODO, close on naming - `Causal Context`, `Resolve Context`???) for a promise as a reference from a `Continuation Invocation` associated with a Promise.then call, to the `Continuation Invocation` in which the preceding promise in the promise chain was resolved.

For non-promise-based APIs, the `Ready Context` is the same as the `Link Context` (//TODO or should it be null/undefined?)

In our example above, the `Continuation Invocation` associated with `continuation f2()` has as its `Ready Context`, a reference to the `Continuation Invocation` associated with `continuation f3()`.  

// TODO:  add stacks w/ `continuation invocations` to illustrate `ready context`.

## Crisper Definitions

  - **Continuation Point** - a function that accepts a `Continuation` as a parameter.  This logically represents an async boundary; functions passed to a `Continuation Point` are invoked at some later point in time.

  - **Continuation** -  a function that, when invoked, creates a new `Continuation Invocation` instance, and maintains references to other related `Continuation Invocation` instances. Using TypeScript for descriptive purposes, a `Continuation` has the following shape:

    ```TypeScript
    interface Continuation {
        linkingContext: ContinuationInvocation;
    }
    ```

  -  **Continuation Invocation** - a specific invocation of a `Continuation`. A `Continuation Invocation` has the following shape in TypeScript:

    ```TypeScript
    interface ContinuationInvocation {
        invocationID:  number;
        continuation: Continuation;
        readyContext: ContinuationInvocation;
    }
    ```

  - **Async Call Graph** - A directed acyclic graph comprised of `Continuations` and `Continuation Invocations` instances as nodes, and the `linkingContext` and `readyContext` references as edges.

### Continuation Invocation States
A `Continuation Invocation` can be in a number of states:
  - `pending` - A  `Continuation Invocation` instance has been created, but is not yet executing, nor is it ready to execute.
  - `ready` - A `Continuation Invocation` is ready to execute, but is not yet executing
  - `executing` - `Continuation Invocation` is currently executing on the stack
  - `paused` - A `Continuation Invocation` has started execution, but is not currently on the stack.  E.g., for generator functions. // todo - is this necessary for async/await & generators.  what is tc39 terminology?
  - `completed` - A `Continuation Invocation` instance is completed.  However, the instance, during execution, may have called other `Continuation Points` and those `Continuation Invocations` are still pending execution.
  - `collectable` - A `Continuation Invocation` instance is completed and all child-spawned continuations are in the collectable state.

## Where are the Continuation Points?
The `Continuation Points` are defined by convention by the host environment.  The host needs logic to determine if a given argument is a function or a continuation, and if not yet a continuation, needs to "continuify" the parameter.  The host is also responsible for "pinning" any `Continuation` instances to prevent premature garbage collection.

## A User-Space API
Here is a user-space API that provides runtime access to the concepts defined above.  At any given time, there is a "current" `Continuation Invocation`, precisely defined as the `Continatuation Invocation` associated with the `Continuation` nearest to the top of the stack.  (Note: there can be more than one `Continuation` on the stack at any given time, for example, in "user space queueing" examples).

```typescript
    interface ContinuationInvocation {
        invocationID:  number;
        continuation: Continuation
        readyContext: ContinuationInvocation;
        static GetCurrent() : ContinuationInvocation;
    }

    interface Continuation {
        linkingContext: ContinuationInvocation;
    }
```

## An Event Stream
For some post-mortem analysis scenarios, an event stream of async context events is useful.  Such an event stream can be used to define the graph.  The event stream is comprised of the following events: 

  - **invocationCreated** - emitted when a `Continuation Invocation` instance is created.
  - **executeBegin** - emitted when a  `Continuation Invocation` instance transitions to the `executing` state.  
  - **executeEnd** - emitted when a  `Continuation Invocation` instance transitions to the `completed` state.
  - **link** - emitted when a new `Continuation` instance is created.
  - **ready** - emitted when a `Continuation Invocation` instance transitions to the `ready` state.

The events above are JSON events and their schema is illustrated in the following example:

```json
{"event":"invocationCreated","invocationID":1, "continuationId":0}
{"event":"executeBegin","invocationID":1}
{"event":"link","continuationID":1}
{"event":"invocationCreated", "invocationID":2, "continuationId":3}
{"event":"ready","invocationId":2}
{"event":"executeEnd", "invocationID":1}
```

// TODO: can this be simplified to support  state-change events on the continuation invocation 
// TODO: add better examples here. 

## Applications

### Continuation Local Storage
```typescript
/**
 *  A simple Continuation Local Storage construction that utilizes the
 *  Async Call Graph.  For setting values, we set a property at the current
 *  ContinuationInvocation.  For getting values, we follow the "linking context"
 *  path from the current "Continuation Invocation" to the 
 *  root of the graph.  At each node, we check for the presence of the key we're looking for.
 */
class ContinuationLocalStorage {
    static set(key: string, value: any) {
        let curr: ContinuationInvocation = ContinuationInvocation.GetCurrent();
        let props  = curr['CLS'];
        if (!props) {
             props = {};
             curr['CLS'] = props;
        }
        props[key] = value;
    }

    static get(key: string): any {
        let curr: ContinuationInvocation = ContinuationInvocation.GetCurrent();
        while (curr) {
            let props  = curr['CLS'];
            if (props && key in props) {
                return props[key];
            }
            curr = curr.continuation.linkingContext;
        }
        return undefined;
    }
}
```

### Async Error Propogation
```typescript
/**
 *  simple error propagation across async boundaries, following the "linking context"
 */
class AsyncErrorPropogationStrategy {
    static AsyncThrow(err: Error) {
        let curr: ContinuationInvocation = ContinuationInvocation.GetCurrent();
        while (curr) {
          let handler  = curr['AsyncErrorHandler'];
          if (handler) {
              handler(t);
              return;
          }
          curr = curr.continuation.linkingContext;
    }
}
```

### Long Call Stacks
A Long Call Stacks is a list of call stacks.  The entries in the list are captured at various times deemed interesting by the host environment.  Given that the Continuation implementation will be aware of when various events occur (e.g., `Continuation` instance construction), the implementation will be aware of when stacks need captured, and will expose APIs to toggle stack capture on & off.

### Step-Into Async
A debugger wants to perform a "step-into" action across an async boundary. Consider the following code:

```javascript
processHttpRequest(req, res) {
    // currently debugger is paused on doSomethingAsync below
    doSomethingAsync(function c() {
        console.log('I want to break here');
    });
}    
```

A trivial approach is for the IDE to simply set a breakpoint inside the continuation.  However, that breakpoint may hit for a different http request than the one currently under inspection.  To address this, the debugger client, when responding to a "step into async" gesture, will execute the followign code

```typescript
const location = ...; // location of where to set the conditional breakpoint
const currentContext = debugger.Evaluate(`ContinuationInvocation.GetCurrent()`);
const condition =  `ContinuationInvocation.GetCurrent().continuation.linkingContext.invocationID == ${currentContext.invocationID}`;
debugger.setConditionalBreakpoint(location, condition);
debugger.continue();
```

### Step-back Async
A "step-back" debugger wants to step backwards across an async boundary.  Consider the following code:

```javascript
processHttpRequest(req, res) {
    doSomethingAsync(function c() {
        // currently debugger is paused on console.log below
        console.log('');
    });
}    
```

A trivial approach is for the IDE to simply set a breakpoint outside of the scope of the current continuation.  However, that breakpoint may hit for a different http request than the one currently under inspection.  To address this, the debugger client, when responding to a "step back async" gesture, will execute the followign code

```typescript
const location = ...; // location of where to set the conditional breakpoint
const currentContext = debugger.Evaluate(`ContinuationInvocation.GetCurrent()`);
const condition =  `ContinuationInvocation.GetCurrent().invocationID == ${currentContext.continuation.linkingContext.invocationID}`;
debugger.setConditionalBreakpoint(location, condition);
debugger.reverseContinue();
```