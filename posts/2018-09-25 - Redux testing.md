# Advanced Redux TDD: Reducers

We do TDD for most development at Transparent Classroom, including our frontend business logic written in Redux. Doing TDD for Redux revealed certain frustrating aspects of structuring and testing Redux. I set out to make it better. This post offers a slightly different structure for your reducers to make them more testable and easier to statically type (for Flow or TypeScript). Let's dive in.

(Note: we also use Flow for type checking, which adds additional motivation for what we did. Where appropriate, I'll be adding Flow typing for discussion purposes, otherwise I'll strip it.)

# The Status Quo

[In the offical docs](https://redux.js.org/recipes/writing-tests#reducers), reducers are structured using a `switch` statement, and then the branches of the reducer are tested using actions:

```javascript
it('should handle ADD_TODO', () => {
  expect(reducer(initialState, addTodo('Run the tests'))) .toEqual([
    {
      text: 'Run the tests',
      completed: false,
      id: 0
    }
  ])
})
```

There were a couple issues that didn't sit right with me:

1. **Navigation was a pain.** We couldn't navigate directly to that part of the reducer from the spec by going into a function definition. At best, we'd have to look for usages of the constant, then find it that way.
1. **Tests didn't read naturally**, with all the extra setup for the reducer, action object, etc. We wanted to write functions, because functions are easy to test.
1. We're also **testing the action creators, which are a separate concern**; not a big deal, but also not my favorite.
1. **Type checking with Flow was a miserable experience** based on what was recommended by the [official Flow docs](https://flow.org/en/docs/react/redux/). We were essentially creating the same type definition in multiple places, getting obtuse errors, and finding little benefit. (That's saying a lot for being one of the maintainers of [flow-typed](https://github.com/flowtype/flow-typed/).)

# Finding a Better Way: Handlers -- Take 1

An alternative to using a switch statement is to have a "handlers object," [mapping action types to handlers](https://redux.js.org/recipes/reducing-boilerplate#generating-reducers), like so:

```javascript
export const todos = createReducer([], {
  [ActionTypes.ADD_TODO]: (state, action) => {
    const text = action.text.trim()
    return [...state, text]
  }
})
```

(Amusingly, I thought of this idea on my own after ruminating on how to improve on the switch statement, then discovered it was already discussed in the Redux docs. Turns out coming up with original ideas is hard.)

It was a nice start, but not enough. It didn't allow for easy navigation between implementation and test, and the dynamic key object syntax felt a bit weird and limiting.

This approach can be taken one step further to something that is truly useful, however. The functions on that "handlers object" can be pulled out and exported. At last, we have a solution where we're testing functions rather than a system. I call these exported functions *handlers*:

```javascript
// reducer.js
export const addTodo = ({ todo }, state) => /* ... */

// reducer_spec.js
describe('#addTodo', () => {
  it('should add a todo to the state', () => {
    expect(addTodo({ todo: 'Hello' }, [])).toEqual(['Hello'])
  })
})
```

The need for an action object to be passed in was still dissatisfying and not natural to write. Consider a handler that takes a bunch of pieces of data that are necessary, i.e. an action object with many keys:

```javascript
it('should do a thing', () => {
  const action = {
    thing1: 'foo',
    thing2: 'bar',
    thing3: 'baz',
    thing4: 'qux',
  }
  const state = myHandler(action, defaultState)
  /* ... */
})

it('should do another thing', () => {
  const action = {
      thing1: 'argh',
      thing2: 'whyMustI',
      thing3: 'keepWritingOut',
      thing4: 'theseDarnKeys',
    }
})
```

# Finding a Better Way: Handlers -- Take 2

I wanted something that reads more naturally, like an ordinary function that has multiple parameters:

```javascript
myHandler('foo', 'bar', 'baz', 'qux', state)
```

In its current form, the handlers were just an extraction of the internals of how Redux and reducers work, and I wasn't interested in being beholden to those details. I wanted to extract the arguments from the action object and just give them to the handler. Turns out, you can do just that.

```javascript
// Let's say you have an action object...
const actionObject = { type: 'ADD_TODO', title: 'Write a cool blog post about reducers', description: 'It better be good...' }

// ...you can safely extract the values and pass them in as parameters to a handler 
const { type, ...payload } = actionObject
const newState = myHandler(...Object.values(payload), state)
```

**A brief disclaimer about `Object.values` and object iteration order, before you flip a table...**

I thought using `Object.values` in this way would be dangerous: we always hear about how [the iteration order of an object's keys is not guaranteed](https://stackoverflow.com/a/5525820/2672869). I tried a couple alternatives, including having the action creators generate `Map`s rather than objects (where the order is guaranteed). Using `Map`s were unpleasant to debug and it bothered me to step further away from how Redux is normally structured (i.e. using plain objects).

ES6, however, [introduced a spec](http://2ality.com/2015/10/property-traversal-order-es6.html) for the iteration order of object keys. For objects with string (non-`Symbol`) keys that can't be parsed as integers, the iteration order is simply the insertion order.

```javascript
Object.values({ foo: 1, bar: 2 })
// [1, 2]
```

Adding keys that are symbols (`{ [Symbol('first')]: true }`) or ones that can be parsed as integers (`{ '10': true }`) behave differently and make the picture more complex, but that doesn't matter for our purposes, because we want the string keys on the object to generally reflect parameter names, which cannot be integers or Symbols. Put another way, we have no need for `Symbol`s or strings that can be parsed as integers to be used as keys on our action objects.

Assuming consistent iteration order for object keys is totally safe given the above restrictions, which I think are reasonable. We've been doing it in our production code for almost a year without issue. Even older browsers seem fine, because `Object#values` is polyfilled by our Babel setup, so we can safely rely on the newer standard for how it behaves. (No promises though.)

**Now then, disclaimer aside...**

With this new pattern to extract the values off the action object and apply them to a handler function, we could easily translate handler functions into a reducer that fits with Redux. Remember, *a reducer is a function that takes a state and an action object and returns a state*:

```javascript
export const addTodo = (title, description, state) => { /* ... */ }
export const removeTodo = (id, state) => { /* ... */ }

const handlers = {
  [ADD_TODO]: addTodo,
  [REMOVE_TODO]: removeTodo,
}

const reducer = (state = defaultState, { type, ...args }) => {
  const handler = handlers[type]
  if (!handler) return state
  return handler(...Object.values(args), state)
}
export default reducer
```

With all this in place, we could test our reducers as a series of functions, easily navigate from its usage in the test to the implementation, and have a bunch of other handy IDE intellisense features (e.g. easily seeing code structure, auto-imports, etc.).

```javascript
// actions.js
export const addTodo = (title, description) => ({ type: ADD_TODO, title, description })

// reducer.js
export const addTodo = (title, description, state) => { /* ... */ }

// reducer_spec.js
it('should do a thing', () => {
  const state = addTodo('My First Todo', 'I feel so proud!', defaultState)
  /* ... */
})
```

Ah, the sweet smell of victory.

A bit on the nose, but worth reminding: the parameter names of the `addTodo` handler do not have to match the key names of the action object; only the order of them matters. We use the same names for convenience. If you had something like this:

```javascript
// actions.js
export const addTodo = (title, description) => ({ type: ADD_TODO, description, title }) // <-- notice the order

// reducer.js
export const addTodo = (title, description, state) => { /* ... */ } // <-- vs. here
```

... you've got a bug. That translation does feel brittle, and also it hasn't come up yet as a problem. We're talking about other possible solutions such as using [ducks](https://github.com/erikras/ducks-modular-redux) to co-locate our action creators and reducers, or even auto-generate our action creates based on our reducers, for an even better developer experience.

# Wiring the Handlers to the Reducer

The tricky part then became how to tell the reducer to actually use these handlers. You saw one above, where I created the handlers object, and just before the reducer I associated handlers to their action types. I didn't like it that much; it wasn't obvious when a handler was actually being registered with the reducer. After running through a few different approaches, I settled on the concept of "registering" a handler on the handlers object, then giving the object to the reducer. I'm going to omit the actual helper code for now and focus on its usage:

```javascript
// reducer.js

const handlers = {}
const registerHandler = setupRegistration(handlers)

export const addTodo = (title, description, state) => { /* ... */ }
registerHandler(ADD_TODO, addTodo)

export const removeTodo = (id, state) => { /* ... */ }
registerHandler(REMOVE_TODO, removeTodo)

const initialState = { /* ... */ }
export default createReducerWithHandlers(handlers, initialState)
```

This approach made it clear which handlers were actually employed by the reducer (rather than helpers) and colocated the action type with the handler. Super handy for navigation and reasoning about whether a handler was being used directly to respond to an action type.

# Wait, Why is the State a Parameter at the End?

I structured the handlers to take the state object *at the end* rather than at the beginning (like in the actual reducer). Answering the question of "why" requires diving into a bit of functional programming. 

I made this design choice to reflect a common principle of functional programming: put the intended recipient of the operation at the end of the parameter list. So, if you were to write a standalone map function:

```javascript
// do this
map(someFn, xs)

// not this
map(xs, someFn)
```

This principle is useful because you can make that concept of "map someFn" into another abstraction:

```javascript
const addOne = x => x + 1
// this is the abstraction
const incrementList = map(addOne) // this assumes `map` is curried
// which can be applied to various lists
const list1 = incrementList([1, 2, 3])
const list2 = incrementList([4, 5, 6])
```

Trying to do it the other way around is... really hard to do. So, to reiterate, in FP you often want to structure your functions such that the most commonly changing parameter comes at the end, so you can create more useful abstractions on the parameters that come before it.

(By the way, if `const incrementList = map(addOne)` throws you and you don't know about currying, I strongly recommend you check out one of the many excellent blog posts out there that explains this concept.)

Anyway. I moved the state to the end to make piping state from one handler to the next a breeze. If you curry your handlers, the tests become wonderfully simple and expressive, piping results from one handler into another one.

**A brief aside on piping**

`pipe` takes any number of functions, takes the last parameter given to it and gives it to the first fn, which then pipes the output into the second one, and so on. FPers call it "left-to-right composition." If you have something like `pipe(f1, f2, f3, x)`, this translates to `f3(f2(f1(x)))`. Or, in an imprecise graphical sort of way, `f1(x) -> f2 -> f3`.

I strongly recommend using [Ramda](https://ramdajs.com/) to give you an insanely powerful FP toolkit for Javascript. I hope to write more about Ramda some other day.

So, with `pipe`, we can easily build up a state through a sequence of handler uses.

```javascript
const state = pipe(
  addTodo('First Todo', 'This was my first todo!'),
  addTodo('Make TDD better', 'Focus first on reducers'),
)(defaultState)
```

Compare this with the alternative, where state is the first parameter:

```javascript
const state = pipe(
  x => addTodo(x, 'First Todo', 'This was my first todo!'),
  x => addTodo(x, 'Make TDD better', 'Focus first on reducers'),
)(defaultState)
```

It's not terrible, but I vastly prefer the flexibility of the former. Also, the handlers can then be used to create other abstractions, in the same way as `incrementList` was created:

```javascript
const someComplicatedHandler = curry((arg1, arg2, arg3, arg4, arg5, state) => { /* ... */ })
const mostlyFigureOutHandler = someComplicatedHandler(1, 2, 3, 4)
```

With all this setup, you can at last achieve TDD nirvana and test the various parts of your reducer functionality like they were functions that are composable and modular. (Yes, you functional programmers out there can be all 🙌 here.)

```javascript
it('#completeTodo should mark a todo at an index as completed', () => {
  const state = pipe(
    addTodo('A'),
    addTodo('B'),
    addTodo('C'),
    completeTodo(1),
  )(defaultState)

  expect(state.map(x => x.completed)).toEqual([false, true, false])
})
```

# Static Typing for Reducers

Using this approach of exporting handlers and registering them on the reducer provides the added benefit of dramatically simplifying the type checking for reducers. I'll discuss it using Flow, but I'm sure TypeScript would be similar.

Setting up type checking based off the Flow docs is a mouthful, requiring a lot of extra typing:

```javascript
// types.js
type FooAction = { type: "FOO", foo: number }
type BarAction = { type: "BAR", bar: boolean }

type Action =
  | FooAction
  | BarAction

// actions.js
const fooAction: FooAction = (foo: number) => ({ type: 'FOO', foo })
const barAction: BarAction = (bar: boolean) => ({ type: 'BAR', bar})

// reducer.js
function reducer(state: State, action: Action): State {
  switch (action.type) { /* ... */ }
}
```

It works, but not perfectly. Flow sometimes behaves strangely with disjoint unions because it's not a perfect system. It's also hard to understand, requiring a deeper knowledge of Flow and typing. Finally, inside the reducer, you don't actually know your types at the various branches of your switch statement; you have to refer to a different file entirely to figure out the types of the action object for that branch.

Fortunately, using handlers circumvents all these issues, because the handlers are simply functions whose parameters can be easily typed. The inputs to the function are explicitly typed right with the definition, making it a breeze to read.

```javascript
const addTodo = (title: string, description: string, state: TodosStateT): TodosStateT => { /* ... */ }
```

(We append type names with `T` to avoid naming conflicts, which has worked out *really* well for us.)

We don't bother typing our actions because it hasn't served any utility. There's a temptation to "Type All The Things" with Flow, but we've decided to be more strategic around what we type to maximize benefit and not waste time.

# TL;DR

To recap: you can write your reducers as a collection of handlers that are registered to the reducer. Structuring them this way keeps with the patterns of Redux while improving testability, navigation, and general developer experience. Here's the code in a nutshell, including the helper functions extracted into a separate file:

```javascript
// reducer_helpers.js

// this mutates the handler object! That's okay though.
export const setupRegistration = (handlers) => (actionType, handler) => {
  handlers[actionType] = handler
}

export const createReducerWithHandlers = (handlers, defaultState, state, { type, ...args }) => {
  const handler = handlers[type]

  if (!handler) return state || defaultState

  // This is safe for empty args arrays, the spread will still come out to be [state], not [null, state]
  return handler(...Object.values(args), state)
}

// reducer.js

const handlers = {}
const registerHandler = setupRegistration(handlers)

export const addTodo = (title, description, state) => { /* ... */ }
registerHandler(ADD_TODO, addTodo)

export const removeTodo = (id, state) => { /* ... */ }
registerHandler(REMOVE_TODO, removeTodo)

const initialState = { /* ... */ }
export default createReducerWithHandlers(handlers, initialState)
```

Writing our reducers in this way has vastly improved the experience of TDD for Redux, simplified our static typing of reducers using Flow, and made our reducer files easier to read and understand. I hope you will give it a try and see if you get similar benefits. I'd love to hear your thoughts and how it works out in your projects.
