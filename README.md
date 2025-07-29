# Code notes

Coding diary with the most interesting things I learn each day.
<br />
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
const arr = [1, 2, 3, 4];
let doubleArr = [];

for (let i = 0; i < arr.length; i++) {
  doubleArr[i] = arr[i] * 2;
}
```

<br />

**Declerative code** - you tell **WHAT** you want to achive, and let computer handle the rest

```js
const arr = [1, 2, 3, 4]
const doubleArr = arr.map((el) => el * 2)
```
