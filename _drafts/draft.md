---
title: 'C Things'
layout: post
---

Yes, I worked very hard thinking of the title to this post.

I'd like to quote K&R today (as we are taking a step back from SI204):

> "Call by value is an asset, however, not a liability. It usually leads to more compact programs with fewer extraneous variables, because parameters can be treated as conveniently initialized local variables in the called routine."

Here's an example (from K&R):

```
/* power: rause base to n-th power; n>=0; version 2 */

int power(int base, int n)
{
	int p;

	for (p = 1; n > 0; --n) {
		p = p* base;
	}
	return p;
}
```

I have added the brackets because I come from JavaScript (forgive me!) and for me, `{}` offer reabaility.

Whatever is done to n inside power has no effect on the argument power was called with! Very nice. 

here's version 1:

```
#include <stdio.h>

/* K&R says functions can appear in any order? 
 * but that is against C99?
 * */

/* power: raise base to n-th power; n >= 0 */
int power(int base, int n) {
	int i, p;

	p = 1;
	for (i = 1; i <= n; ++i) {
		p = p * base;
	}
	return p;
}


/* test power function */
int main() {
	int i;

	for (i = 0; i < 10; ++i) {
		printf("%d %d %d\n", i, power(2, i), power(-3, i));
	}
	// "programs should return status to their environment" p. 26, s 1.8;
	return 0;
}
```
This does not work the same for arrays, however. When an array is used as an argument to a function, the value passed is the location or address if the beginning of said array.

Outline for proragm to print longest text line:

```
while (*there's another line*)
	if (*it's longer than the previous longest*)
		*save it*
		*save its length*
	*print longest file*
```
