---
title: 'C Things Part I'
layout: post
---

Over the past week I've discovered that several institutions of higher learning freely share lectures and notes on a variety of courses. The ones I was interested in were, of course, computer science.

I decided to start with the federal government and a war academy. The U.S. Military is the godhead (as it were)  and gave birth to DARPA; they are one and the same. They are the source of any number of computer science related innovation and have been connected not just via funding. 

In any case, I stumbled over a U.S. Naval Academy offering related to data structures as I began my search for decent info on linked lists. I clicked the sixth link down and was dumped into the middle of [SI204](https://www.usna.edu/Users/cs/roche/courses/s17si204/index.php). I jumped right to the third or fourth section without a care. I knew all, see. Upon sober reflection I saw that [I knew nothing](https://en.wikipedia.org/wiki/I_know_that_I_know_nothing). Thus, I started with Unit 1 and declared right then and THERE that I was going to ["keyboard cowboy"](https://en.wikiquote.org/wiki/Hackers#Eugene_%22The_Plague%22_Belford) the crap out of the examples. One has to get one's fingers into recall mode; ["punch the keys!"](https://www.urbandictionary.com/define.php?term=ptkfgs) and one will more easily type and further instill the discipline into one's brain -- the great tissue! 

Here is the first example:

	#include "si204.h"

	int main() {
		/* Read name into a variable */
		fputs("What is your name ?", stdout);
		cstring name;
		readstring(name, stdin);

		/* Read age into a variable */
		fputs("How old are you now? ", stdout);
		int age = readnum(stdin);

		/* Make a crucial decision */
		if (age >= 21) {
			fputs(name, stdout);
			fputs(" can... rent a car!\n", stdout);
		}
		else {
			fputs("Sorry, no fun for ", stdout);
			fputs(name, stdout);
			fputs(" (yet).\n", stdout);
		}
		
		return 0;
	}

Right off the bat I could tell the syntax was a bit different than what I was used to with reading basic C. For example,

	#include "lib.h"

diff 

	#include <stdio.h>:


The small SI204 library wraps some of the "annoying things" one sometimes has to learn first. With the library it's easier to approach C and get started quickly: 

Example 2:

	#include "si204.h"

	int main() {
		int x;		// DECLARE x of type int
		x = 523 - 248;	// ASSIGN x to the difference
		writenum(x*x*x, stdout); // PRINT x cubed
		fputs("\n", stdout); // print a newline at the end
		return 0; // success!
	}

Basic declaration and assignment. And lots of writing to `stdout` (i.e., one's terminal).

Here's a look at the SI204 library:

	/* Simple C typedefs and helper functions for strings and I/O
	 * SI204, Spring 2017
	 * Dr. Roche
	 */

	/* Make sure nothing bad happens if you #include this file twice */
	#ifndef SI204_H
	#define SI204_H

	/* Standard header files */
	#include <stdio.h>
	#include <stdlib.h>
	#include <string.h>

	/* Define a special type "cstring" which is just a char array
	 * of a predetermined, fixed length.
	 * XXX: If you change this, be sure to update CSTRINGSCAN below also. */
	#define CSTRINGLEN (128)
	typedef char cstring[CSTRINGLEN];

	/* The scanf argument for a cstring.
	 * Note the length is CSTRINGLEN-1 for the null character. */
	#define CSTRINGSCAN "%127s"

	/* Typedef for stream, which is the same as FILE* */
	typedef FILE* stream;

	/* Function prototypes defined here.
	 * Note: they are declared "static" because the implementations are
	 * below, not in a separately-compiled file.
	 */
	static char readchar(stream in);
	static void readstring(cstring s, stream in);
	static double readnum(stream in);
	static void writenum(double x, stream out);

	/* Function definitions below */

	char readchar(stream in) {
	  char c;

	  /* when reading from stdin, make sure all pending stdout output
	   * is written first */
	  if (in == stdin) fflush(stdout);

	  /* The fscanf is the most important part!
	   * It returns the number of items successfully read. */
	  if (fscanf(in, " %c", &c) != 1) {
	    fprintf(stderr, "\nERROR in readchar. Maybe end-of-file?\n");
	    abort();
	  }

	  return c;
	}

	void readstring(cstring s, stream in) {
	  /* when reading from stdin, make sure all pending stdout output
	   * is written first */
	  if (in == stdin) fflush(stdout);

	  /* The fscanf is the most important part!
	   * It returns the number of items successfully read. */
	  if (fscanf(in, CSTRINGSCAN, s) != 1) {
	    fprintf(stderr, "\nERROR in readstring. Maybe end-of-file?\n");
	    abort();
	  } else if (strlen(s) >= CSTRINGLEN - 1) {
	    fprintf(
		stderr,
		"\nERROR: Input string is too long (must be at most %d characters).\n",
		CSTRINGLEN - 2);
	    abort();
	  }
	}

	double readnum(stream in) {
	  double x;

	  /* when reading from stdin, make sure all pending stdout output
	   * is written first */
	  if (in == stdin) fflush(stdout);

	  /* The fscanf is the most important part!
	   * It returns the number of items successfully read. */
	  if (fscanf(in, "%lf", &x) != 1) {
	    if (feof(in)) fprintf(stderr, "\nERROR in readnum: EOF reached.\n");
	    else fprintf(stderr, "\nERROR in readnum: Maybe ill-formatted number?\n");
	    abort();
	  }

	  return x;
	}

	void writenum(double x, stream out) {
	  /* The fprintf is the most important part!
	   * It returns the number of characters actually printed. */
	  if (fprintf(out, "%g", x) <= 0) {
	    fprintf(stderr, "\nERROR on writenum. Maybe the stream is invalid?\n");
	    abort();
	  }
	}

	#endif /* ifdef SI204_H */

Jumping somewhat ahead...

Here's example 5:

	#include "si204.h"

	int main() {
		cstring str1, str2;
		int m1, m2, s1, s2;
		char c1, c2;
		readstring(str1, stdin);
		m1 = readnum(stdin);
		c1 = readchar(stdin);
		s1 = readnum(stdin);
		readstring(str2, stdin);
		m2 = readnum(stdin);
		c2 = readnum(stdin);
		s2 = readnum(stdin);
	}

And there's `stdin` and all that fun jazz. Recall: `cstring`, `readstring`, `readnum` are part of SI204 library.

I am going to interject with an "aside of code":

	#include "si204.h"

	int main() {
		int x = 10;
		writenum(x, stdout);
		fputs("\n", stdout);
		writenum(++x, stdout);
		fputs("\n", stdout);
		writenum(x, stdout);
		fputs("\n", stdout);
		
		int y = 10;
		writenum(y, stdout);
		fputs("\n", stdout);
		writenum(y++, stdout);
		fputs("\n", stdout);
		writenum(y, stdout);
		fputs("\n", stdout);

		return 0;

	}

While writing or reading C one will often see this kind of syntax in `for` loops: 

	// note here that `int` is optional 
	for(int i = 0; i < n; ++i)

What does `++i` mean? Well, from what I can gather, when a programmer decides to increment in this way the expression is evaluated and the result is pointed elsewhere. The value of `x` thus will still be 10 during that call to `writenum(++x)`.

This was one of the things I learned last week that really fascinated me and put "more wood in the fire" as the saying goes.

Now at this point I decided to supplement the course I was going through with some good old K&R. K&R are big on programmers knowing and understanding `ASCII` and it's high time we took an extended detour and wandered down the lane to a simpler time:

[Remember ASCII art?!](https://en.wikipedia.org/wiki/ASCII_art)

Note, dear reader, I am using FreeBSD and trying to stay as close to Unix as an amateur can. So behavior might differ under Linux or any other `*BSD`.

To get up to speed quickly:

	#include <stdio.h>

	int main(void) {
		puts("Printable ASCII:");
		for (int i = 32; i < 127; i++) {
			putchar(i);
			putchar(i % 16 == 15 ? '\n' : ' ');
		}
	}

start a new line on any number modulus 16 == 15 otherwise space since we want our table to be 15 chars wide.
This is printing the `ASCII` values of given C chars.
`ASCII` values can be between 0 and 127.

Here's the result:

	Printable ASCII:
	  ! " # $ % & ' ( ) * + , - . /
	0 1 2 3 4 5 6 7 8 9 : ; < = > ?
	@ A B C D E F G H I J K L M N O
	P Q R S T U V W X Y Z [ \ ] ^ _
	` a b c d e f g h i j k l m n o
	p q r s t u v w x y z { | } ~ 

Read up on `putchar` function.

Since we're messing with tables we can jump back to SI204 with a Hankel Matrix:

	#include "si204.h"

	int main() {

		int n;
		fputs("Please enter n: ", stdout);
		n = readnum(stdin);

		for (int row = 1; row <= n; row++) {
			for (int col = 1; col <= n; col++) {
				int val = row + (col - 1);
				if (val < 10) {
					// padding on lefthand for smaller numbers
					fputs(" ", stdout);
				}
				writenum(val, stdout);
				// padding regardless for larger
				fputs(" ", stdout);
			}
			fputs("\n", stdout);
		}
		return 0;
	}

Here's the result:

	Please enter n:  10  
	 1  2  3  4  5  6  7  8  9 10 
	 2  3  4  5  6  7  8  9 10 11 
	 3  4  5  6  7  8  9 10 11 12 
	 4  5  6  7  8  9 10 11 12 13 
	 5  6  7  8  9 10 11 12 13 14 
	 6  7  8  9 10 11 12 13 14 15 
	 7  8  9 10 11 12 13 14 15 16 
	 8  9 10 11 12 13 14 15 16 17 
	 9 10 11 12 13 14 15 16 17 18 
	10 11 12 13 14 15 16 17 18 19 

Time for some basic algorithms, kids!

We will start with the Euclidean algorithm:

	#include "si204.h"

	int main() {
		int a, b;
		fputs("Enter two numbers: ", stdout);
		a = readnum(stdin);
		b = readnum(stdin);
		while (b != 0) {
			int r;
			r = a % b;
			a = b;
			b = r;
		}
	writenum(a, stdout);
	fputs(" is the GCD.\n", stdout);
	}

Besides performing division, what other uses make sense? (hint: I don't know)
Or better yet, let us cry slow tears and review the Caesar-shift as it is presented in SI204:

	/************************************
	 Ceasar-Shift Encryption on 4-Letter Messages

	 The "Ceasar-Shift" method of encryption scrambles
	 a message a letter at a time. First we view each
	 letter as being represented by its distance from
	 the letter 'a' when we write the alphabet in a line.
	 So, a->0, b->1m c->2, ..., z->25. Given a key k,
	 which is just an integer between 0 and 25, we map
	 each letter to a new one as follows:

		If the letter is i away from 'a', then it maps
		to the letter i + k away from 'a'. If i + k is
		greater than 25, we simply wraparound. The mod
		operator (%) does this wraparound for us.

	So, the rule becomes:

		If letter L1 is i away from letter 'a', then it
		maps to the letter that is (i + k) % 26 away from
		the letter 'a'.

	The thing to notice is that (L1 - 'a') tells you
	precisely how far away letter L1 is from letter 'a',
	and for distance d, char('a' + d) is the letter that
	is d away from the letter 'a'.

	Ex: key = 1, message = busy, encrypted message = cvtz
	*****************************************************/
	#include "si204.h"

	int main() {
		// Read in key value
		int k;
		fputs("Enter key value: ", stdout);
		k = readnum(stdin);

		// Read in 4-letter message
		fputs("Enter 4-letter message (lower-case): ", stdout);
		char c1, c2, c3, c4;
		c1 = readchar(stdin);
		c2 = readchar(stdin);
		c3 = readchar(stdin);
		c4 = readchar(stdin);

		// Compute distances for original letters
		int d1, d2, d3, d4;
		d1 = c1 - 'a';
		d2 = c2 - 'a';
		d3 = c3 - 'a';
		d4 = c4 - 'a';
		
		// Compute distances for encrypeted letters
		int ed1, ed2, ed3, ed4;
		ed1 = (d1 + k) % 26;
		ed2 = (d2 + k) % 26;
		ed3 = (d3 + k) % 26;
		ed4 = (d4 + k) % 26;

		// Compute encrypeted letters
		int e1, e2, e3, e4;
		e1 = 'a' + ed1;
		e2 = 'a' + ed2;
		e3 = 'a' + ed3;
		e4 = 'a' + ed4;

		// Write out the encrypted message
		fputs("Encrypted message is: ", stdout);
		fputc(e1, stdout);
		fputc(e2, stdout);
		fputc(e3, stdout);
		fputc(e4, stdout);
		fputs("\n", stdout);

		return 0;
	}

I might be moving a bit too fast, but these are high pressure days!

Let's step back and slow down with:

	#include "si204.h"

	int main() {
		int sum;
		int k;
		sum = 0;

		k = readnum(stdin);
		while (k >= 0) {
			sum = sum + k;
			k = readnum(stdin);
		}
	writenum(sum, stdout);
	fputs("\n", stdout);
	}

and some K&R for some good (slow) measure:

	#include <stdio.h>

	/* print Fahrenheit-Celsius table
	 * for fahr = 0, 20, ..., 300 */

	int main() 
	{
		int fahr, celsius;
		int lower, upper, step;

		lower = 0; /* lower limit of temperature table */
		upper = 300; /* upper limit */
		step = 20; /* step size */

		fahr = lower;
		while (fahr <= upper) {
			celsius = 5 * (fahr-32) / 9;
			printf("%d\t%d\n", fahr, celsius);
			fahr = fahr + step;
		}

		return 0;
	}

Should one always `return 0`? What do people say on the internet? Let's see:

> Is it a good practise? Yes and No. Not all functions should return a value. And quite a few in the standard library even, do not return any value. Hence their return type is void.

so says `Aniket Inge` with a decently up voted answer on Stackoverflow.

Some say the `main signature` requires it. But that doesn't seem so to me since some of these programs work without any return. I'm sure we could go on and on and start discussing ANSI C or whatnot, but that's what Google is really for.

Might be wise to note at this point that throughout I was switching between `gcc` and `cc` or `clang` as it were. I don't know enough to comment in any detail and I don't understand pros or cons for each. Gut feeling? `gcc` is related to Stallman and maybe I want to avoid it? Or maybe I'm getting too close to the vox populi...as it were.

Just a minor note if you're using `<math.h>`; compile with `-lm` which is to say "link math library, please".

Onward and upward with MORE EXAMPLES OF C:

	#include "si204.h"

	int main() {

		for (int i = 0; i < 25; ++i) {
			fputc('*', stdout);
		}
		fputs(" E L E P H A N T S  N E E D  S P A C E ", stdout);

		for (int i = 0; i < 25; ++i) {
			fputc('*', stdout);
		}
		fputs("\n", stdout);
	}	

Just to remind one how easy it is to layout output.

C does much more when files get in on the mix. According to SI204:

	#include "si204.h"

	int main() {
		cstring filename;
		fputs("Enter the name of the file: ", stdout);
		readstring(filename, stdin);

		stream filein = fopen(filename, "r");

		if (filein == 0) {
			fputs("ERROR: No such file?\n", stdout);
			return 1;
		}

		cstring firstword;
		readstring(firstword, filein);

		fclose(filein);

		fputs("First word is ", stdout);
		fputs(firstword, stdout);
		fputs(".\n", stdout);
		
		return 0;
	}

at some point we'd want to print to `stderr`.

but what is `stderr` anyway? Well, dear reader, it is part of the "standard streams" which are input and output channels.

if you want to see `stderr` on the terminal just direct it that way via:

	2>&1

This all has to do with redirection and it's probably best to reference the authoritative text on the matter (i.e., "The Unix Programming Environment".)

Which goes something like this in my edition:

>7.1 Low-level I/O

>	The lowest level of I/O is a direct entry into the operating system. Your program read and writes files in chunks of any convenient size. The kernel buffers your data into chunks that match the peripheral devices, and schedules operations on the devices to optimize their performance over all users.

> *File descriptors*
>	All input and output is done by reading and writing files, because all peripheral devices, even your terminal, are files in the file system. This means that single interface handles all communication between a program and peripheral devices. 
>	In the most general case, before reading or writing a file, it is necessary to inform the system of your intent to do so, a process called *opening* the file. If you are going to write a file, it may also be necessary to create it. The system checks your right to do so (Does the file exist? Do you have permissions to access it?), and if all is well, returns a non-negative integer called a *file descriptor*. Whenever I/O is to be done on the file, the file descriptor is used instead of the name to identify the file. All information about an open file is maintained by the system; your program refers to the file only by the file descriptor. A FILE pointer as discussed in Chapter 6 points to a structure that contains, among other things, the file descriptor; the macro `fileno(fp)` defined in `<stdio.h>` returns the file descriptor.

There's lots to take in here, but it goes on:

>	There are special arrangements to make the terminal input and output convenient. When it is started by the shell, a program inherits three open files, with the file descriptors 0, 1, and 2, called the standard input, the  standard output, and the standard error. All of these are by default connected to the terminal, so if a program only reads file descriptor 0 and writes file descriptors 1 and 2, it can do I/O without having to open files. If the program opens any other files, the will have file descriptors 3, 4, etc.

>	If I/O is redirected to or from files or pipes, the shell changes the default assignments for file descriptors 0 and 1 from the terminal to the named files. Normally file descriptor 2 remains attached to the terminal, so error messages can go there (AHA!). Shell incantations such as `2>filename` and `2&>1` will cause rearrangements of the defaults, but the file assignments are changed by the shell, not by the program. (The program itself can rearrange these further if it wishes, but this is rare.)

and there you have it, right from the horse's mouth...so to speak.

Wheel it back into SI204 since we got off track there:

(aside: hey, you! Yeah, you...when using `ed` (as you should be) command `n` will show current line). God bless;

Is this right? O, constant reader<span>&mdash;</span>are you still there?

	#include <stdio.h>

	#define IN 1
	#define OUT 0

	int main() {

		int c, state;
		state = OUT;

		while((c = getchar()) != EOF) {
			if (c == '\n' || c == '\t' || c == ' ') {
				state = OUT;
				putchar('\n');
			}
			else {
				state = IN;
				putchar('#');
			}
		}
		return 0;
	}

What does it do? 

In the immortal words of John Hammond:
>Let me show you...

	I got a pain you will never know
	#
	###
	#
	####
	###
	####
	#####
	####

See the pattern? 

I think I will end Part I here. Much more to come as we learn C and beyond!
