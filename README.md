# Code notes

Coding diary with the most interesting things I learn each day.
<br />
<br />

## Day 15

React Query (TanStack Query) have a plugin to automatically sync your cached data with localStorage, sessionStorage or IndexedDB (so colled persisters). More in [docs.](https://tanstack.com/query/v4/docs/framework/react/plugins/persistQueryClient)

```js
import { QueryClient } from '@tanstack/react-query'
import { persistQueryClient } from '@tanstack/query-persist-client-core'
import { createSyncStoragePersister } from '@tanstack/query-persist-client-sync-storage'

const queryClient = new QueryClient()

const persister = createSyncStoragePersister({
  storage: window.localStorage, // could be sessionStorage or IndexedDB
})

persistQueryClient({
  queryClient,
  persister,
})
```

<br />

## Day 14

If you create `{children}` of inside of an array that is changing - they are going to be new every time (even thought we did not change this particular item). 

```jsx
{users.map((user) => (
  <User key={user.id} user={user}>
    <button onClick={() => removeUser(user.id)}>
      Remove
    </button>
  </User>
))}
```
In this example if we delete one user - all of them will be updated, even if they are wrapped into memo. Reason - prop `children` changes.

<br />

## Day 13

Children (`ExpensiveComponent`) belongs to the component where they are declared (`App`), and won't be re-rendered where they are used, even if  `Component` gets re-rendered.

```jsx
const App = () => {
 // ... 

  return (
    <div>
      {/* ... */}
      <Component>
        <ExpensiveComponent />
      </Component>
    </div>
  )
}

const Component = ({ children }) => {
  // ... some states here

  return (
    <div>
      {/* ... */}
      {children}
    </div>
  )
}
```

We could achieve the same memoizing `ExpensiveComponent` and using it inside of `Component`, but not in `App`, but it's better to solve it architecturally only, as it's free-ish.

If `App` re-renders - `ExpensiveComponent` will be re-rendered, even if `Component` is memoized. Reason - props changes (children is a prop).

<br />

## Day 12

React performance tip #4:

Context re-renders every single component subscribed to it, if the value (`{ items, dispatch }`) is not primitive

```jsx
export const ItemsContext = createContext({})

const ItemsProvider = ({ children }) => {
  const [items, dispatch] = useReducer(reducer, getInitialItems())

  return (
    <ItemsContext.Provider value={{ items, dispatch }}>
      {children}
    </ItemsContext.Provider>
  )
}

export default ItemsProvider
```

To stop component that subscribed to `dispatch` only from re-rendering you can split the context into two:

```jsx
export const ItemsContext = createContext({})
export const ActionsContext = createContext({})

const ItemsProvider = ({ children }) => {
  const [items, dispatch] = useReducer(reducer, getInitialItems())

  return (
    <ActionsContext.Provider value={dispatch}>
      <ItemsContext.Provider value={items}>
        {children}
      </ItemsContext.Provider>
    </ActionsContext.Provider>
  );
};

export default ItemsProvider
```

Order matters - the outer context never changes. You can create more than two contexts that way.

<br />

## Day 11

React performance tip #3:

React memo has a second parameter - the comparison function. It case you want to have some props to be (not-memoized) objects - you can use it to trigger the update only when some particular parts of this objects change.

```jsx
memo(Component, ((prev, next) => prev.x === next.x))
```

In fact, memo works with no-primitive props, if they are mutated during an update, but not re-created, which is possible with immer.

Some people consider it to be hacky, and confusing for the future maintainers trying to scale the app. So probably it’s more explicit to just use memoized objects. Yet, in case of really large objects, or not mutable states, it can be a more performant solution.

<br />

## Day 10

React performance tip #2:

If you need some code to run on the very first render (before React commits to the DOM), but you want to avoid `useEffect` or `useLayoutEffect`for some reason, you can use this `useRef` hack:

```jsx
function Component() {
  const ref = useRef(true)

  if (ref.current) {
    // run some logic here ...
    // ...
    ref.current = false
  }

  return (
    <div>...</div>
  )
}
```

Note: if you need to manipulate real DOM nodes - you need `useEffect` or `useLayoutEffect`. Otherwise it’s used here just to detect the very first render of this component.

Many libs use this hack, so refusing to use it because it’s “hacky” doesn’t make much sense anyway.

<br />

## Day 9

`if (!canvas) return` is a common boilerplate that does not really do anything - useEffect
ensures that by the time that it runs the jsx does render, so the canvas never be `null`.

```js
useEffect(() => {
  const canvas = canvasRef.current
  if (!canvas) return
  const ctx = canvas.getContext('2d')
  drawLine(ctx)
}, [])
```

It can be `null` in some complex cases (conditional rendering + dependency array is not empty, SSR hydration, portals, etc.), but not in a simple `useEffect(() => {}, [])` case.

<br />

## Day 8

React performance tip #1:

Wrap a function into an anonymous function inside of a useState()

```js
const [items, setItems] = useState(() => generateItems())
```

because this would call `generateItems()` on every re-render, even though the value
returned from this function will be ignored.

```js
const [items, setItems] = useState(generateItems())
```

<br />

## Day 7

[View Transition API](https://developer.chrome.com/docs/web-platform/view-transitions/)

New native way to build a transitions between two html documents (pages) without building SPA.

```css
@view-transition {
  navigation: auto;
}

::view-transition-old(root),
::view-transition-new(root) {
  animation: fade 0.3s ease both;
}

@keyframes fade {
  from { opacity: 0; }
  to { opacity: 1; }
}
```

For example building a thumbnails for shoping cards you can do:

```html
<a href='/product/red-shoes'>
  <img src='/images/red-shoes-thumb.jpg' style='view-transition-name: product-image;' />
</a>
```

And the product detail page:

```html
<img src='/images/red-shoes-large.jpg' style='view-transition-name: product-image;' />
```

The browser matches and animates the elements between navigations.

Also, there is [Speculation Rule](https://developer.chrome.com/docs/web-platform/implementing-speculation-rules/). This lets the browser preload or prerender full pages based on user behaviour – like hovering or touching a link – before they click.

```html
<script type='speculationrules'>
{
  'prerender': [
    {
      'where': {
        'selector_matches': 'a'
      }
    }
  ]
}
</script>
```

<br />

## Day 6

Implementing a vanilla analogy to a 'virtual dom' - something that updates only textNodes,
if they have changed. First, generate a new markup string:

```js
const newMarkup = `
  <div class='some-class' id='some-id'>
    <div>Some sibling</div>
    <div class='some-class'>
      <div>hey, I won't be changed and re-rendered!</div>
      <div>And I will be, as I have some new updated text inside ${someText}</div>
    </div>
  </div>
`
```

Than create a virtual DOM fragment to compare with a real one:

```js
const newElemDOM = document.createRange().createContextualFragment(newMarkup)
const prevElemDOM = document.querySelector('#some-id')

// create arrays to loop over and compare
const newElemArray = Array.from(newElemDOM.querySelectorAll('*'))
const prevElemArray = Array.from(prevElemDOM.querySelectorAll('*'))
```

Now let's derive only the elements with a change in text nodes, but not their parents:
`isEaqualNode()` returns true is anything (children or text nodes) has changed, so we need one extra check:

```js
newElemArray.forEach((el, i) => {
  const prevEl = prevElemArray[i]
  if (!el.isEaqualNode(prevEl) && newEl.firstChild?.nodeValue?.trim() !== '') {
    // finally update the DOM!
    prevEl.textContent = el.textContent
  }
})
```

`elem.nodeValue()` returns `null` if that's a real element, and a text - if element is a text node.
Be carefull with comments or text nodes that are sibblings of DOM elements, it requires some extra work!
Read more about nodeValue on [MDN.](https://developer.mozilla.org/en-US/docs/Web/API/Node/nodeValue)

<br />

## Day 5

There is an `<use>` html element that duplicates svg nodes to somewhere else. Can be used
within one `<svg>` element:

```html
<svg viewBox='0 0 30 10' xmlns='http://www.w3.org/2000/svg'>
  <circle id='my-circle' cx='5' cy='5' r='4' stroke='blue' />
  <use href='#my-circle' x='10' fill='blue' />
  <use href='#my-circle' x='20' fill='white' stroke='red' />
</svg>
```

of to represent an external file (from public directory or from external link):

```html
<svg width='24' height='24'>
  <use href='/icons.svg#circle-icon' />
</svg>
```

Note that it can be CORS and MIME type issues when uploaded form external url, otherwise browser support is good. Read more on [MDN's <use> documentation](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/use).

<br />

## Day 4

To stop trying on a request after N seconds, create a helper function:

```js
function timer(s) {
  return new Promise((_, reject) => {
    setTimeout(() => {
      reject(new Error(`Request took too long. Quit after ${s} seconds.`))
    }, s * 1000)
  })
}
```

then use Promise.race() to get the first resolved responce:

```js
const res = await Promise.race([fetchFuncion, timer(5)])
```

<br />

## Day 3

**Hashchange event:**

```js
window.addEventLitener('hashchange', callback)
```

The event is fired when the url hash part changes. For example, if you have two links on the page:

```html
<a href='#12345'>hash 1</a>
<a href='#67890'>hash 2</a>
```

clicking first one will add '/#12345' to the current url. That's how we can use 'native' version of router params and, let's say, make a request using **location.hash** property inside of a callback to get a current hash.

<br />

## Day 2

**Side effects** are literally everything that affects the world outside of a function - a browser, outer variables, a server, etc.

```js
function noSideEffects(a, b) {
  return a + b
}

function sideEffects(sideEffect) {
  console.log(sideEffect)
}
```

<br />

## Day 1

**Imperative code** - you tell the machine **HOW** to execute your code step by step:

```js
const arr = [1, 2, 3, 4]
let doubleArr = []

for (let i = 0; i < arr.length; i++) {
  doubleArr[i] = arr[i] * 2
}
```

<br />

**Declerative code** - you tell **WHAT** you want to achive, and let computer handle the rest

```js
const arr = [1, 2, 3, 4]
const doubleArr = arr.map((el) => el * 2)
```
