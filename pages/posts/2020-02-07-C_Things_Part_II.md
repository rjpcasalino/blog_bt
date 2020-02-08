---
title: 'C Things Part II'
layout: post
---

I'd like to quote K&R today (as we are taking a step back from SI204):

> "Call by value is an asset, however, not a liability. It usually leads to more compact programs with fewer extraneous variables, because parameters can be treated as conveniently initialized local variables in the called routine."

Here's an example (from K&R):

```
/* power: raise base to n-th power; n>=0; version 2 */

int power(int base, int n)
{
	int p;

	for (p = 1; n > 0; --n) {
		p = p* base;
	}
	return p;
}
```

I have added the brackets because I come from JavaScript (forgive me!) and for me `{}` offers readability.

Whatever is done to n inside power has no effect on the argument power was called with! Very nice. 

Here is version 1:

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

Outline for program to print longest text line:


```
while (*there's another line*)
	if (*it's longer than the previous longest*)
		*save it*
		*save its length*
	*print longest file*
```

An aside here to further clarify K&R. K&R is a style. It is also known as C78, for the year of "The C Programming Language" publication.

>    After the publication of the book mentioned before, the American National
     Standards Institute (ANSI) started to work on standardizing the language,
     and they announced ANSI X3.159-1989 in 1989.  It is usually referred as
     ANSI C or C89.  The main difference in this standard were the function
     prototypes, which is a new way of declaring functions.  With the old-
     style function declarations, the compiler was unable to check the sanity
     of the actual parameters at a function call.  The old syntax was highly
     error-prone because incompatible parameters were hard to detect in the
     program code and the problem only showed up at run-time.

>    In 1990, the International Organization for Standardization (ISO) adopted
     the ANSI standard as ISO/IEC 9899:1990 in 1990.  This is also referred as
     ISO C or C90.  It only contains negligible minor modifications against
     ANSI C, so the two standards often considered to be fully equivalent.
     This was a very important milestone in the history of the C language, but
     the development of the language did not stop.

>    Since then new standards have not been published, but the C language is
     still evolving.  New and useful features have been showed up in the most
     famous C compiler: GNU C.  Most of the UNIX-like operating systems use
     GNU C as a system compiler, but those addition in GNU C should not be
     considered as standard features.

Anyway, that answers my question on C99 and FreeBSD

SI204 has veered into discussion on `pointers` and `dereferancing`.

Here is an interlude from Master Ueshiba:

> If you have not

> Linked yourself

> To true emptiness,

> You will never understand

> The Art of Peace.

```
#include <stdio.h>

#define MAXLINE  1000 	/* maximum input line size */

int grabline(char line[], int maxline);
void copy(char to[], char from[]);

/* print the longest line */
// what should main do if line is bigger than limit?
int main() {
	int len;
	int max;
	char line[MAXLINE];
	char longest[MAXLINE];

	max = 0;
	// grab line returns an int 
	while((len = grabline(line, MAXLINE)) > 0) {
		if (len > max) {
			if (len > 80)
				printf("more than 80: %s\n", line);
			max = len;
			copy(longest, line);
		}
	}
	/* there was a line */
	if (max > 0) {
		printf("Max: %i\n", max);
		printf("This is longest: %s\n", longest);
	}
	return 0;
}

int grabline(char s[], int lim) {
	int c, i;

	for (i = 0; i < lim - 1 && (c = getchar()) != EOF && c != '\n'; ++i) {
		s[i] = c;
	}
	/* newline new char */
	if (c == '\n') {
		s[i] = c;
		++i;
	}
	/* denote the end of the array */
	s[i] = '\0';
	return i;
}

void copy(char to[], char from[]) {
	int i;

	i = 0;
	/* recall: end of the array in C is always `'\0'` */
	while ((to[i] = from[i]) != '\0') {
		++i;
	}
}
```
