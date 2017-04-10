## Flux Standard Ajax Action
A human-friendly standard for Ajax Flux action objects. Feedback welcome.

### Motivation
Ajax seems to be a area where new learners get stuck, whether they're learning to use thunks, sagas or epics. One area where they don't seem to have much difficulty is creating actions - as they're just plain old Javascript objects, and we have the added clarity provided by the Flux Standard Actions spec. This line of thinking led to the question - Why not develop a Flux Standard Ajax Action?

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

### Estimated Benefits
- a dramatic decrease in the amount of async actions required in your average application (as the ajax related async actions would fall away)
- it's declarative, implementation details no longer the concern of the developer
- different FSAA compliant middleware could be swopped in/out without breakages
- it's idiomatic Redux (action log still makes sense, serializable etc)

### Details
```js
function likePost(postId) {
  return {
    type: 'LIKE_POST_REQUEST',
    // Ajax request setup
    ajax: {
      url: `/api/posts/${postId}/likes`,
      method: 'POST',

      // Optional data attribute. Translated into query string params for GET requests
      data: { ... },

      // Optional ajax meta attributes
      meta: {
        // Amount of times to retry the request on failure
        retry: 2,

        // Amount of time before timing out
        timeout: 10000,  

        // The action type that cancels the request
        cancelType: 'LIKE_POST_CANCELLATION',

        // Amount of time to debounce the execution of the request
        debounce: 400,

        // Request resolving strategy when multiple LIKE_POST_REQUESTs are in flight
        resolve: 'LATEST',

        // Fine tune debouncing and request resolving within the action type.
        // E.g. you could debounce LIKE_POST_REQUESTs by post id
        group: postId,

        // Broaden debouncing and request resolving across related action types.
        // E.g. you could debounce LIKE_POST_REQUEST and UNLIKE_POST_REQUESTs by postId
        groupUid: `like-post-${postId}`,
      },

      // Optional headers. "Content-Type": "application/json" added by default
      headers: { ... },

      // Optional response callback
      response: data => normalize(data, posts),
    }
  };  
}
```
