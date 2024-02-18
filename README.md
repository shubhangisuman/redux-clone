# Redux 
* Redux is a library for managing global application state
* Small stand-alone JS library
* Redux uses a "one-way data flow" app structure

Action --> State --> View 

* State is read-only
* Changes are made through pure reducer functions. 
* Redux expects that all state updates are done immutably

# Pure functions
Any function that - 
* Doesn’t alter input data.
* Doesn’t depend on external state (like a database, DOM, or global variable)
* Provides the same output for the same input.
* Only depends on its input arguments.

# Redux app flow
* State describes the condition of the app at a specific point in time
* UI is rendered based on state
* When something happens, state gets updated
* UI re-renders based on the new state

# When to use?
* Large amounts of application state that are needed in many places in the app
* App state is updated frequently over time
* Logic to update that state may be complex
* App has a medium or large-sized codebase, and might be worked on by many people

# React-redux
* It allows React components interact with a Redux store by reading pieces of state and dispatching actions to update the store.

# Immutability
* "Mutable" means "changeable". If something is "immutable", it can never be changed.
* JavaScript objects and arrays are all mutable by default.
* In order to update values immutably, your code must make copies of existing objects/arrays, and then modify the copies.

# Redux Terms

# Action
* An action is a plain JavaScript object having - type and payload.
* Describes something that happened in the application.
* Type - "domain/eventName", first part is the feature or category that this action belongs to, and the second part is the specific thing that happened.
* Payload - additional information about what happened

# Action Creator
* An action creator is a function that creates and returns an action object. 

# Reducer
* A reducer is a function that receives the current state and an action object, decides how to update the state if necessary, and returns the new state: (state, action) => newState. 
* They are not allowed to modify the existing state. Instead, they must make immutable updates.
* Return a new state or return the existing state unchanged.

# Why new state? Why no mutation?
There is only one way to know if two JavaScript objects have the same properties. To deep-compare them.
But this becomes extremely expensive in real-world apps, because of the typically large objects and the number of times they need to be compared.

# Store
* JS object.
* The current Redux application state lives in an object called the store .
* The store is created by passing in a reducer. 
* It has a method - "getState" returns the current state value

# Dispatch
* The Redux store has a method called dispatch.
* The only way to update the state is to call store.dispatch() and pass in an action object. 
* The store will run its reducer function and save the new state value inside.
* Use getState() to retrieve the updated value.
* Call action creators in dispatch.


* Dispatching actions are like "triggering an event" - to tell something has happened.
* Reducers act like event listeners. Hear the action and update the state.

# Selectors
* Functions that know how to extract specific pieces of information from a store state value.
* As an application grows bigger, this can help avoid repeating logic as different parts of the app need to read the same data

# Flow

# Initial
* Store is created using root reducer 
* Store calls root reducer once and saves returned value as initial state
* UI is first rendered, UI components access the current state of the Redux store.
* They also subscribe to any future store updates.

# Update
* Something happened, dispatch action.
* Store runs the reducer function with the previous state and the current action, and saves the return value as the new state.
* Store notifies all parts of the UI that are subscribed that the store.
* Each UI component that needs data from the store checks to see if the parts of the state they need have changed.
* Each component that sees its data has changed forces a re-render with the new data.

--------------------------

# createStore
* To initialise a store, use createStore. Take a root reducer and a preloadedState.
const store = createStore(rootReducer, preloadedState)

# Slice
The reducer for a specific section/feature of redux is usually written as a single file, known as a "slice" file.
eg - todosSlice.js, filtersSlice.js

# combineReducers
* Reducers can be combined together with the Redux combineReducers function


````` 
export default function rootReducer(state = {}, action) {
  // always return a new object for the root state
  return {
    // the value of `state.todos` is whatever the todos reducer returns
    todos: todosReducer(state.todos, action),
    // For both reducers, we only pass in their slice of the state
    filters: filtersReducer(state.filters, action)
  }
} 
`````

using combine reducers:

`````
const rootReducer = combineReducers({
  // Define a top-level state field named `todos`, handled by `todosReducer`
  todos: todosReducer,
  filters: filtersReducer
})
`````

Key names will become the keys in your root state object, and the values are the slice reducer functions


* store.getState() - access to current state.
* store.dispatch(action) - allows state to be updated.
* store.subscribe(render) - takes a callback function. Pass render function, to know the store upates.
* Handles unregistering of listeners via the unsubscribe function returned by store.subscribe(listener).


# Miniature redux store

`````
function createStore(reducer, preloadedState) {
  let state = preloadedState
  const listeners = []

  function getState() {
    return state
  }

  function subscribe(listener) {
    listeners.push(listener)
    return function unsubscribe() {
      const index = listeners.indexOf(listener)
      listeners.splice(index, 1)
    }
  }

  function dispatch(action) {
    state = reducer(state, action)
    listeners.forEach(listener => listener())
  }

  dispatch({ type: '@@redux/INIT' })

  return { dispatch, subscribe, getState }
}
`````

# Middlewares or Enhancers
Middleware are added using the "applyMiddleware" enhancer
Middleware are written as three nested functions inside each other
Middleware run each time an action is dispatched
Middleware can have side effects inside

A middleware can do anything it wants when it sees a dispatched action:
* Log something to the console
* Set timeouts
* Make asynchronous API calls
* Modify the action
* Pause the action or even stop it entirely

``````
const alwaysReturnHelloMiddleware = storeAPI => next => action => {
  const originalResult = next(action)
  // Ignore the original result, return something else
  return 'Hello!'
}

const middlewareEnhancer = applyMiddleware(alwaysReturnHelloMiddleware)
const store = createStore(rootReducer, middlewareEnhancer)

const dispatchResult = store.dispatch({ type: 'some/action' })
console.log(dispatchResult)
// log: 'Hello!'
``````

# useSelector

``````
// BAD: this selector returns the entire state, meaning that the component will rerender unnecessarily
const { count, user } = useSelector((state) => state)

// GOOD: instead, select only the state you need, calling useSelector as many times as needed
const count = useSelector((state) => state.count.value)
const user = useSelector((state) => state.auth.currentUser)

const a = { foo: "bar" }
const b = { foo: "bar" }

console.log( a === b ) // will log false
console.log( shallowEquals(a, b)) // will log true
``````
useSelector() uses strict === reference equality checks.
shallowEquals when an object is similar in contents, but different by reference.

# connect
The connect() function connects a React component to a Redux store.
It returns a wrapper function that takes your component and returns a wrapper component with the additional props it injects.
Takes 4 args - mapStateToProps, mapDispatchToProps, mergeProps (merge - stateProps, ownProps and dispatchProps), options (areStatesEqual, areOwnPropsEqual, forwardRef)


# mapStateToProps
It allows the new wrapper component will subscribe to Redux store updates. 
The results of mapStateToProps must be a plain object, which will be merged into the wrapped component’s props. 
Pass null or undefined in place of mapStateToProps if you dont want to subscribe to store updates

``````
const mapStateToProps = (state, ownProps) => ({
  todo: state.todos[ownProps.id],
})
``````

# mapDispatchToProps
Component will receive dispatch by default, even if i dont pass this.
Returns an object


1. Used as a function
(dispatch, ownProps?) => Object

2. Used as an object
```
import { addTodo, deleteTodo, toggleTodo } from './actionCreators'
const mapDispatchToProps = {
  addTodo,
  deleteTodo,
  toggleTodo,
}
export default connect(null, mapDispatchToProps)(TodoApp)
```
