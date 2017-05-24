## Flux Standard Ajax Action (exploratory)
A human-friendly standard for Ajax Flux action objects. Feedback welcome.

### Motivation
The motivation behind this is two-fold and was sparked by the [Redux discussion](https://github.com/reactjs/redux/issues/2295) about potential abstractions to ease its learning curve. **Firstly**, Ajax seems to be an area where new learners have difficulty, whether they're learning to use thunks, sagas or epics. One area where they don't seem to have much difficulty is creating actions - as they're just plain old Javascript objects™️, and we have the added clarity provided by the Flux Standard Actions spec. **Secondly**, as the Redux eco-system grows, more libraries will have the need to make requests on your behalf e.g. [redux-offline](https://github.com/jevakallio/redux-offline). Currently there is no standard way to make requests, which forces these libraries to roll their own implementation.

This line of thinking led to the question - Why not develop a Flux Standard Ajax Action?

A basic Flux Standard Ajax Action:
```js
function fetchPosts(userId) {
  return {
    type: 'FETCH_POSTS_REQUEST',
    ajax: {
      url: `/api/users/${userId}/posts`,
      method: 'GET',
    }
  };  
}
```
The ajax action would be handled by FSAA compliant middleware (which would react to the ajax key in the action).

### Design Goals
- Should be easy to read and write - easing the learning curve of writing Ajax in Redux applications
- Should be flexible - common Ajax request requirements should be covered including request cancellation, debouncing, timeouts, retries
- Should enable the creation of useful tools and abstractions
- Should be an extension of FSA 
- Should be idiomatic Redux (action log should still make sense, serializable etc)

### Estimated Benefits
- a dramatic decrease in the amount of async actions required in your average application (as the ajax related async actions would fall away)
- it's declarative, implementation details no longer the concern of the developer
- different FSAA compliant middleware could be swopped in/out without breakages, which should allow for some competition

## Details
```js
function likePost(postId) {
  return {
    type: 'LIKE_POST_REQUEST',

    ajax: {
      url: `/api/posts/${postId}/likes`,
      method: 'POST',

      // Optional request data
      data: { ... },

      // Optional request meta attributes
      meta: {
        // Amount of times to retry the request on failure
        retries: 2,

        // Amount of time before timing out
        timeout: 10000,  

        // The action type that cancels the request
        cancelType: 'LIKE_POST_CANCELLATION',

        // Amount of time to debounce the execution of the request
        debounce: 400,

        // Request resolving strategy when multiple LIKE_POST_REQUESTs are in flight
        resolve: 'LATEST',

        // Fine tune debouncing and request resolving.
        // E.g. you could debounce LIKE_POST_REQUESTs by post id
        group: postId,

        // Broaden debouncing and request resolving across related action types.
        // E.g. you could debounce LIKE_POST_REQUEST and UNLIKE_POST_REQUESTs together
        // if both have the same groupUid
        groupUid: `like-unlike-${postId}`,
      },

      // Optional request headers
      headers: { ... },

      // Optional response callback
      response: data => normalize(...),
  
      // Optional response type
      responseType: 'json',

      // Optional chain of FSAAs to dispatch sequentially
      chain: [
        res => fetchFoo(...),
        res => fetchFooBars(...)
      ],

      // Optional cross domain flag
      crossDomain: false,

      // Optional credentials flag
      withCredentials: false,

      // Optional username and password to be used with XMLHttpRequest
      username: '...',
      password: '...',
    }
  };  
}
```
### GET Request Data
- Should be encoded into query string params
- Should handle encoding arrays in ```indices```, ```brackets``` or ```repeat``` format
- Should allow array encoding format to be configured

### Non-GET Request Data
- Should be sent as JSON

### Request Headers
- Should add ```'Content-Type': 'application/json; charset=UTF-8'``` by default if not provided

### Response Callbacks

- Should be removed from the action in the middleware so that they don't make their way into the action log or the reducers

### Response Types

- Should be auto generated

```
LIKE_POST_REQUEST -> LIKE_POST_SUCCESS | LIKE_POST_FAILURE
```

- Should be configurable when creating the middleware. E.g.

```js
createAjaxMiddleware({ 
  requestSuffix: 'REQUEST',
  successSuffix: 'SUCCESS',
  failureSuffix: 'FAILURE'
});
```
### Successful/Fulfilled Requests

- Should copy over the ```ajax``` attribute from the request into its ```meta``` attribute

```js
{
  type: 'LIKE_POST_SUCCESS',
  // The response data
  payload: { ... },
  meta: {
    ajax: { 
      url: '/api/posts/42/likes',
      method: 'POST'
    }
  }
}
```
### Failed Requests

- Should follow the approach outlined in the FSA spec for errors
- Should include HTTP error information in the payload
- Should copy over the ```ajax``` attribute from the request into its ```meta``` attribute

```js
{
  type: 'LIKE_POST_FAILURE',
  error: true,
  payload: {
    status: [status code],
    text: [status text]
  },
  meta: {
    ajax: { 
      url: '/api/posts/42/likes',
      method: 'POST'
    }
  }
}
```
### Request Chains
- Should be removed from the action in the middleware so that they don't make their way into the action log or the reducers

**General thoughts on chaining requests**
- Should only be used for unique data flows (edge cases)
- If you find yourself using these chains often, then it could be time to look into [redux-observable](https://github.com/redux-observable/redux-observable) / [redux-saga](https://github.com/redux-saga/redux-saga) for setting up these side effects

### FSAA  Compliant Middleware
[redux-ajaxable](https://github.com/mcoetzee/redux-ajaxable) built with RxJS

### Prior Art
[redux-api-middleware](https://github.com/agraboso/redux-api-middleware) also introduced the concept of having a standard action, which they called Redux Standard API-calling Action. There are some important differences between it and FSAA including:
- it doesn't have the Ajax meta attributes dealing with request cancellation, retries, resolve strategies etc
- the action isn't an extension to FSA, so doesn't naturally fit in with your other FSA action creators
- all three types (request, success, failure) need to be provided for each request

The project's future also seems in doubt as activity has died down and new maintainers are required.
