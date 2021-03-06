# redux-saga-async-action

[![Generated with nod](https://img.shields.io/badge/generator-nod-2196F3.svg?style=flat-square)](https://github.com/diegohaz/nod)
[![NPM version](https://img.shields.io/npm/v/redux-saga-async-action.svg?style=flat-square)](https://npmjs.org/package/redux-saga-async-action)
[![Build Status](https://img.shields.io/travis/diegohaz/redux-saga-async-action/master.svg?style=flat-square)](https://travis-ci.org/diegohaz/redux-saga-async-action) [![Coverage Status](https://img.shields.io/codecov/c/github/diegohaz/redux-saga-async-action/master.svg?style=flat-square)](https://codecov.io/gh/diegohaz/redux-saga-async-action/branch/master)

Dispatching an action handled by [redux-saga](https://github.com/redux-saga/redux-saga) returns promise. It looks like [redux-thunk](https://github.com/gaearon/redux-thunk), but with pure action creators.

```js
store.dispatch({ 
  type: 'FOO',
  payload: { title: 'bar' },
  meta: {
    async: true
  }
}).then((detail) => {
  console.log('Yaay!', detail)
}).catch((error) => {
  console.log('Oops!', error)
})
```

> `redux-saga-async-action` uses [Flux Standard Action](https://github.com/acdlite/flux-standard-action) to determine action's `payload`, `error` etc.

## Install

    $ npm install --save redux-saga-async-action

## Basic setup

Add `middleware` to your redux configuration (**before redux-saga middleware**):

```js
import { createStore, applyMiddleware } from 'redux'
import createSagaMiddleware from 'redux-saga'
import { middleware as asyncMiddleware } from 'redux-saga-async-action'

const sagaMiddleware = createSagaMiddleware()
const store = createStore({}, applyMiddleware(asyncMiddleware, sagaMiddleware))
```

## Usage

Add `meta.async` to your actions and receive `key` on response actions:

```js
const resourceCreateRequest = data => ({
  type: 'RESOURCE_CREATE_REQUEST', // you can name it as you want
  payload: data,
  meta: {
    async: true
    ^
  }
})

const resourceCreateSuccess = (detail, key) => ({
                                       ^
  type: 'RESOURCE_CREATE_SUCCESS', // name really doesn't matter
  payload: detail, // promise will return payload
  meta: {
    async: key
           ^
  }
})

const resourceCreateFailure = (error, key) => ({
                                      ^
  type: 'RESOURCE_CREATE_FAILURE',
  error: true, // redux-saga-async-action will use this to determine if that's a failed action
  payload: error,
  meta: {
    async: key
           ^
  }
})
```

`redux-saga-async-action` will automatically transform your request action and inject a `key` into it.

Handle actions with `redux-saga` like you normally do, but you'll need to grab `key` from the request action and pass it to the response actions:

```js
// async will be transformed in something like 'RESOURCE_CREATE_REQUEST_1234567890123456_REQUEST'
// the 16 digits in the middle are necessary to handle multiple async actions with same type
function* createResource() {
  while(true) {
    const { payload, meta } = yield take('RESOURCE_CREATE_REQUEST')
                     ^
    try {
      const detail = yield call(callApi, payload)
      yield put(resourceCreateSuccess(detail, meta.async))
                                              ^
    } catch (e) {
      yield put(resourceCreateFailure(e, meta.async))
                                         ^
    }
  }
}
```

Dispatch the action from somewhere. Since that's being intercepted by `asyncMiddleware` cause you set `meta.async` on the action, dispatch will return a promise.

```js
store.dispatch(resourceCreateRequest({ title: 'foo' })).then((detail) => {
  // detail is the action payload property
  console.log('Yaay!', detail)
}).catch((error) => {
  // error is the action payload property
  console.log('Oops!', error)
})
```

## Usage with selectors

To use `isPending` and `hasFailed` selectors, you'll need to add the `asyncReducer` to your store:

```js
import { combineReducers } from 'redux'
import { reducer as asyncReducer } from 'redux-saga-async-action'

const reducer = combineReducers({
  async: asyncReducer,
  // your reducers...
})
```

Now you can use selectors on your containers:

```js
import { isPending, hasFailed } from 'redux-saga-async-action'

const mapStateToProps = state => ({
  loading: isPending(state, 'RESOURCE_CREATE_REQUEST'),
  error: hasFailed(state, 'RESOURCE_CREATE_REQUEST')
})
```

## API

<!-- Generated by documentation.js. Update this documentation by updating the source code. -->

### isPending

Tells if an action is pending

**Parameters**

-   `state` **State** 
-   `name` **([string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String) \| [Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)&lt;[string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)>)** 

**Examples**

```javascript
const mapStateToProps = state => ({
  fooIsPending: isPending(state, 'FOO'),
  fooOrBarIsPending: isPending(state, ['FOO', 'BAR']),
  anythingIsPending: isPending(state)
})
```

Returns **[boolean](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** 

### hasFailed

Tells if an action has failed

**Parameters**

-   `state` **State** 
-   `name` **([string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String) \| [Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)&lt;[string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)>)** 

**Examples**

```javascript
const mapStateToProps = state => ({
  fooHasFailed: hasFailed(state, 'FOO'),
  fooOrBarHasFailed: hasFailed(state, ['FOO', 'BAR']),
  anythingHasFailed: hasFailed(state)
})
```

Returns **[boolean](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** 

## License

MIT © [Diego Haz](https://github.com/diegohaz)
