# Code notes

Coding diary with the most interesting things I learn each day.
<br />
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
const arr = [1, 2, 3, 4];
let doubleArr = arr.map((el) => el * 2);
```
