# BenchBnB Bonus: Make The Login And Signup Form Pages Into Modals (Redux Version)

Rather than redirecting to the sign up page when there isn't a current user,
(for instance on the `BenchForm`), it would be nice if there was a session modal
that just overlaid the page. Then, after signing up or logging in, the modal
could close and the user's context would not be lost.

In this bonus Phase, you will enhance the user experience by using a modal for
log in / sign up to preserve context.

**Note:** There are many ways to implement modals. The steps below use Redux,
but you could just as easily use React context.

## Step 1: Create modal action creators & reducer

Create a __modals.js__ file inside of __frontend/src/store/__. You are going to
need two action creators, one to show the correct modal and one to close it.
Start by defining the `SHOW_MODAL` and `HIDE_MODAL` constants:

```js
//frontend/src/store/modals.js

const SHOW_MODAL = 'modal/showModal';
const HIDE_MODAL = 'modal/hideModal';

export const showModal = (modalType) => ({
  type: SHOW_MODAL,
  modalType
});

export const hideModal = () => ({
  type: HIDE_MODAL
});
```

`showModal` will take in the correct modal type as a string. The reducer will
use this `modalType` to return a new object for the modal slice of state. That
object will look like `{ type: 'login' }`, `{ type: 'signup' }`, or `{ type:
'null' }`. `hideModal` does not need an argument: the reducer will just set
`type` back to `null` for the `HIDE_MODAL` action type.

Here is an example of what the reducer could look like:

```js
//frontend/src/store/modals.js

// ...
function modalsReducer(state = { type: null }, action) {
  switch (action.type) {
    case SHOW_MODAL: {
      return { type: action.modalType };
    }
    case HIDE_MODAL:
      return { type: null };
    default:
      return state;
  }
}

export default modalsReducer;
```

Ensure the default value is set to `{ type: null }` to maintain state
consistency. By doing so, when the application initially loads, this slice of
state will be initialized with `{ type: null }` instead of an empty object,
preventing the rendering of any modals. Modals will only be displayed when this
key has a truthy value. When it's truthy, the application can check the value of
this key to determine which modal should be rendered.

## Step 2: Add reducer to rootReducer and test actions on the browser

Import the `modalsReducer` in __frontend/src/store/store.js__ and add it to the
`rootReducer`.

```js
// frontend/src/store/store.js

// ...
import modals from './modals';

const rootReducer = combineReducers({
  session,
  benches,
  reviews,
  users,
  modals
});
// ...
```

Import the actions in __frontend/src/main.jsx__ and attach them to the window
inside the if statement that checks if the environment is not production:

```js
// frontend/src/main.jsx

// ...
import * as modalActions from './store/modals';

const store = configureStore();

if (import.meta.env.MODE !== "production") {
  restoreCSRF();
  window.store = store;
  window.csrfFetch = csrfFetch;
  window.benchActions = benchActions;
  window.reviewActions = reviewActions;
  window.modalActions = modalActions;
}
// ...
```

To test on the browser, inspect and run the following functions:

```js
store.dispatch(modalActions.showModal("login"))
// Open the next state and check if your modals slice of state looks like
// modals: { type: 'login' }
store.dispatch(modalActions.showModal("signup"))
//  modals : { type: 'signup' }
store.dispatch(modalActions.hideModal())
// Open the next state and check if your modals slice of state looks like
// modals: { type: null }

// Refresh the page and check if the slice of state looks like
// modals: { type: null }
```

## Step 3: Create a `Modal` component

Make a __frontend/src/components/Modal/__ directory with a __Modal.jsx__ file
for a single Modal component that should be able to render any children
components it's given.

Create a function component called `Modal`, destructure `children` from its
props. Export it as the default export. You can also add an __index.js__ file to
the __Modal__ directory if you want.

Render a `div` with an `id` of `modal`. Inside, nest two `div`s:

1. A `div` with an `id` of `modal-background`
2. Another `div` with an `id` of `modal-content`

In the `modal-content` div, render the `children`.

The `modal-background` div needs to be rendered **before** the `modal-content`
because it will naturally be placed "behind" the depth of the `modal-content` if
it comes before the `modal-content` in the DOM tree.

```jsx
// frontend/src/components/Modal/Modal.jsx

function Modal({ children }) {
  return (
    <div id="modal">
      <div id="modal-background" />
      <div id="modal-content">
        {children}
      </div>
    </div>
  );
}

export default Modal;
```

Add a CSS file in the __Modal__ folder called __Modal.css__. The `modal` div
should have a `position` `fixed` and take up the entire width and height of the
window. The `modal-background` should also take up the entire width and height
of the window and have a `position` `fixed`. The `modal-content` div should have
a `position` of `absolute` and be centered inside of the `modal` div by flexing
the `modal` div. You may want to give the `modal-background` a
`background-color` of `rgba(0, 0, 0, 0.7)` and the `modal-content` a
`background-color` of `white` just to see them better. Give the `modal-content`
div a `border-radius` of `10px`.

```css
/* frontend/src/components/Modal/Modal.css */

#modal {
  position: fixed;
  top: 0;
  right: 0;
  left: 0;
  bottom: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 999; 
  /* you may have other elements with z-index, this one should be the highest
   in order to be on top of the whole UI */
}

#modal-background {
  position: fixed;
  top: 0;
  right: 0;
  left: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.7);
}

#modal-content {
  position: absolute;
  background-color: white;
  border-radius: 10px;
}
```

Import the __Modal.css__ file into the __Modal.jsx__ context file.

## Step 4: Create a reusable `SessionModal`

To create a reusable `SessionModal`, make a __components/SessionModal__
directory with a __SessionModal.jsx__ file, a __SessionModal.css__ file, and,
optionally, an __index.js__ file. Also, move the `LoginForm` and `SignupForm`
component files to this directory. (You can delete the __session__ folder now.)

Create and export as the default a generic `SessionModal` component in
__SessionModal.jsx__. It should return a `Modal`.

```jsx
// frontend/src/components/SessionModal/SessionModal.jsx

  return (
    <Modal>
      <div className="session-modal">
        {/* Modal content */}
      </div>
    </Modal>
  );
```

For the modal content, `SessionModal` should render either the `LoginForm` or
the `SignupForm`, with a button at the bottom that toggles between the two (so a
user could easily switch forms). To achieve this, you'll need to subscribe the
`SessionModal` component to the `modals` slice of state from the redux store,
and render the right modal according to the value of the type key. In addition,
dispatch the `showModal` action creator with the right modal type when the
button to change modals is clicked.

The `SessionModal` should look like this:

```js
// frontend/src/components/SessionModal/SessionModal.jsx

// ...
import { useSelector, useDispatch } from 'react-redux';
import * as modalActions from '../../store/modals'
import Modal from '../Modal/Modal';
import { useNavigate, useParams } from 'react-router';

function SessionModal({ onSuccess }) {
 const dispatch = useDispatch();
 const modalType = useSelector(state => state.modals.type);
 const sessionUser = useSelector(state => state.session.user);
 const { benchId } = useParams();

 if (!modalType) return null;
 // if the modals slice of state looks like { type: null } 
 // it will not render a modal
 if (sessionUser) return null;
 // don't render the SessionModal if a user is already logged in

 const goBack = (e) => {
    e.preventDefault();
    dispatch(modalActions.hideModal());
    benchId ? navigate(`/benches/${benchId}`) : navigate("/")
 } // a way for the user to go back home if they do not want 
   // to sign in or sign up, or back to the bench show page
   // if that is where they were

 return (
   <Modal >
     <div className="session-modal">
       <button className='back-button' onClick={goBack}>Back</button>
       <h1>{modalType === "login" ? "Log In" : "Sign Up"}</h1>
       {modalType === "login" ? (
         <LoginForm onSuccess={onSuccess} />
       ) : (
         <SignupForm onSuccess={onSuccess} />
       )}
       <button
         className="link"
         onClick={() => dispatch(modalActions.showModal(modalType === "login" ? "signup" : "login"))}
       >
         {modalType === "login" ? "Sign up" : "Log in" } instead
       </button>
     </div>
   </Modal>
 );
}

export default SessionModal;
```

Once you finish, render a `SessionModal` on the `BenchForm` page if there is no
current user by dispatching the `showModal` action creator. Now when you go to
the `BenchForm` page, a `Login` modal should appear over the page if no one is
logged in.

```js
// frontend/src/components/BenchForm/BenchForm.jsx

import { useState } from 'react';
import { useSelector, useDispatch } from 'react-redux';
import SessionModal from '../SessionModal';
import * as modalActions from '../../store/modals';
import './BenchForm.css';
// ...

function BenchForm() {
  const dispatch = useDispatch();
  const sessionUser = useSelector(state => state.session.user);

  // ...
  // Remove useEffect
  // ...

  if (!sessionUser) {
    dispatch(modalActions.showModal("login"));
  } // open the login modal if no user is logged in
  
  return (
    <>
      {!sessionUser && <SessionModal />} 
      {/* This will ensure the SessionModal is not 
      closed when the user clicks on the background */}

      <div className="new-bench-form">
        <h1>Create A Bench!</h1>
      {/* ... */}
```

Add `SessionModal`s anywhere else that requires a current user, don't forget to
dispatch the `showModal` action creator where needed.

## Step 5: Unify login and signup protocols

With `SessionModal`, your `/login` and `/signup` routes are a bit redundant.
Let's unify the session protocol by eliminating those routes and using the
`SessionModal` for all logging in and signing up.

To do this, you'll need to adjust your `Navigation` component so that it opens a
`SessionModal` when a user clicks `Login` or `Signup` instead of navigating to a
route. If you've made it this far, you can figure out the logic for how to do
this on your own. Just keep two points in mind:

1. The modal that opens should correspond to the button that was clicked (i.e.,
   `Login` or `Signup`).
2. The modal should close whenever someone clicks outside the modal or the modal
   successfully completes its task.

Once you get the `Navigation` component working, you can go ahead and eliminate
the redundant `login` and `signup` routes from your router.

## Step 6: Review modal

To create reviews with a modal, you will use the same `Modal` component from
earlier, but with a different form.

When creating the `BenchShow` component, you rendered the form with the click of
a button (if the current user had not reviewed the bench yet). To do that, you
most likely subscribed the `BenchShow` component to the session, bench, and
reviews slices of state like:

```js
// frontend/src/components/BenchShow/BenchShow.jsx

// ... 
function BenchShow() {
  const { benchId } = useParams();
  const sessionUser = useSelector(state => state.session.user);
  const bench = useSelector(state => state.benches[benchId]);
  const reviews = useSelector(getBenchReviews(parseInt(benchId)));

  // ...
  
  const hasReviewed = sessionUser && reviews.some(review => review.authorId === sessionUser.id); // to check if it had been reviewed before

  // ...

  return (
    <div className="bench-show">

    {/* ... */}

    {!hasReviewed && <LeaveReview bench={bench} />}
    {/* render `LeaveReview` if there is no review from the sessionUser */}
    {/* `LeaveReview` returns either a button or the ReviewForm  */}

    {/* ...*/}
  )
```

Your component may look different, but as long as it works it's fine. The magic
for reusing the modal component happens inside of the `LeaveReview` component.
Just like you did with the `SessionModal`, you'll need to subscribe the
`LeaveReview` component to the `modals` slice of state (from the redux store).
Then, render the `ReviewForm` if the `modals` slice looks like: `{ type:
create-review }`, and dispatch the `showModal` function with the argument of
`"create-review"` with the click of a button.

Here is an example of the `LeaveReview` component:

```js
// frontend/src/components/BenchShow/BenchShow.jsx

// ...
import { useDispatch, useSelector } from 'react-redux';
import * as modalActions from '../../store/modals';

function LeaveReview({ bench }) {
  const modalType = useSelector(state => state.modals.type)
  const dispatch = useDispatch()

  return modalType === "create-review" ? (
    <ReviewForm 
      bench={bench} 
      closeForm={() => dispatch(modalActions.hideModal())}
    />
  ) : (
    <button className="button" onClick={() => dispatch(modalActions.showModal("create-review"))}>
      Leave a Review
    </button>
  );
}
```

Last but not least, wrap the form rendered by the `ReviewForm` component with
the `Modal` component from earlier. Now the form should be rendered as a modal
when the button is clicked!

Congratulations! You've added modals to BenchBnB!
