---
layout: 'post'
title: 'Finding Joy in Node with Neo Geo'
---

The title is a bit of a misnomer. Anyway, I recently purchased an old Neo Geo arcade cabinet. I was bored and wanted to implement
the looping I noticed that it did between FBI warnings and game carts. Doing this oddly made we realize that Node isn't all that bad.
It's fun to work with. Here's the simple program that mimics what I noticed my new toy does before you enter coins.

I plan to actually optimize this code at some point. Maybe ditch all the recursion stuff.

```javascript
const readline = require('readline');
readline.emitKeypressEvents(process.stdin);
process.stdin.setRawMode(true);

// Demos
msgDemo = () => {
  setTimeout(() => {
    console.log('METAL SLUG DEMO!');
  }, 2000);
};

// Globals
let count = 0;
let slots = [{
  name: 'FBI WARNING',
  msg: 'Winners Don\'t Do Drugs!'
}, {
  name: 'Metal Slug',
  msg: 'Super Vehicle-001! メタルスラッグ!',
  demo: msgDemo
}, {
  name: 'Baseball Stars 2',
  msg: 'SNK Neo Geo Baseball for super fun time!'
}];

process.stdin.on('keypress', (str, key) => {
  if (key.ctrl && key.name === 'q') {
    if (count === 0) {
      // ignore the key press
      return
    } else {
      // display current slot on the keypress
      console.log(slots[count].name);
      count = -1;
    }
  } else if (key.ctrl && key.name === 'c') {
    process.exit();
  }
});

// Main program
const demoLoop = () => {
  if (count >= slots.length) {
    // restart the loop
    count = 0;
  };
  console.log(slots[count].name + '\n------' + '\n' + slots[count].msg + '\n');
  if (slots[count].demo) {
    console.log('demo found...play it...');
    slots[count].demo();
  }
  setTimeout(() => {
    count++
    demoLoop();
  }, 4000);
};

// call main program
demoLoop();

```
