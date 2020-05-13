---
title: 'COBOL: A Brief Introduction'
layout: post
---
<style>
#levels {
display: block;
margin-left: auto;
margin-right: auto;
}
</style>

## Common Business-Oriented Language. 
_Suggested listening_
<br>
[_Fanfare for the Common Man_](https://en.wikipedia.org/wiki/Fanfare_for_the_Common_Man)
<br>
<audio
controls
src="https://wiki.boringtranquility.io/assets/audio/Fanfare.mp3">
    Your browser does not support the
    <code>audio</code> element.
</audio>

COBOL was born in the crucible of the Cold War. As Nixon engaged in the [Kitchen Debate](https://en.wikipedia.org/wiki/Kitchen_Debate) with Khrushchev memories of the International Geophysical Year were fading fast. Rival satellites had been propelled toward the heavens, spun their course into atmospheric vapor, and now rest forever in collective memory. The Cold War was in full swing.

A marriage of national security and industry resulted in plans. And meetings. Business met government and one steering committee turned into another. Eggheads meeting overdecorated military brass meeting political actors and back and forth and so on in that fashion until an executive committee was formed and more meetings took place. A "Conference on Data Systems Languages" was gathered. Grace Hooper was there, yes. Lending her superb mind and talents in an advisory capacity. More meetings and talk; so drunk with talk these American minds gathered. In the end<span>&mdash;</span>after all that talk<span>&mdash;</span>[COBOL 60](https://en.wikipedia.org/wiki/COBOL#COBOL_60).

COBOL's adoption rate slowly grew and by the early 1970s it was the most widely used programming language in the United States. Busy were many a programmer bringing seemingly endless ideas to fruition. Some ideas ripened into useful programs and some were merely forgotten. But, though programs appear to die they still manage to rise up.

COBOL continues to power portions of the Individual Master File, the IRS system used to process tax returns. [According to the Government Accountability Office](https://www.gao.gov/assets/680/677454.p) myriad other government agencies continue to write and maintain COBOL code. Efforts are underway to "modernize" some of these aging systems, but it is more than likely that COBOL will continue to power critical governmental and non-governmental systems alike for the foreseeable future.
<hr>
### Getting Started

_Suggested listening_
<br>
[_Saku_](https://en.wikipedia.org/wiki/Susumu_Yokota)
<br>
<audio
controls
src="https://wiki.boringtranquility.io/assets/audio/Saku.mp3">
    Your browser does not support the
    <code>audio</code> element.
</audio>


_This document assumes a bit of programming knowledge and familiarity with GNU tools_.

Luckily for us, getting started with COBOL is relatively painless. GnuCOBOL is a free implementation of the language which one is welcome to study, share, and modify. GnuCOBOL offers latitude regarding use of syntax but we'll be following (as best we can, at least) conventions found in IBM documentation. 

Debian based distributions of Linux should have access to the GnuCOBOL package. Otherwise, follow the [installation instructions](https://sourceforge.net/p/open-cobol/wiki/Install%20Guide/) on the wiki.

### Boarding the plane...

COBOL source programs are grouped into the following four divisions:

*    Identification division
*    Environment division
*    Data division
*    Procedure division

Our program is simple. We're taking data from one file and generating a report that will count and add the presidency number from a sorted listed in our data file. The Presidents aren't out of order and Grover Cleveland's nonconsecutive terms are marked.

The Identification division contains some program meta data:

```COBOL
* This is a comment
* Here's some boring, dull stuff about our program.
PROGRAM-ID. Presidents
AUTHOR. rjpc
DATE-WRITTEN. May 6th, 2020.
```
Followed thereby with a bit about a program's working environment. We'll call this the PRE-FLIGHT AREA:

```COBOL
** PRE-FLIGHT AREA
** *********************
ENVIRONMENT DIVISION.
INPUT-OUTPUT SECTION.
...
```
The Environment Division contains two sections, one of which is the Configuration Section:

>The Configuration Section is optional. When specified, it can describe the computer on which the source program is compiled and the computer on which the object program is executed. 

Which is followed by the Input-Output Section:

>The Input-Output Section defines each file, identifies its external storage medium, assigns the file to one or more input/output devices, and specifies information needed for transmission of data between the external medium and the program.

Nestled under INPUT-OUTPUT but still within the PRE-FLIGHT AREA is:

```COBOL
FILE-CONTROL. 

* SELECT chooses a file for use with our program;
* ASSIGN associates our program's name with an external file
* aptly called PresidentReport

SELECT PRESIDENT-REPORT ASSIGN TO "PresidentReport"

* You can organize your files as 
* sequential, line-sequential, indexed, or relative.
* Using LINE SEQUENTIAL reads the file's records 
* in the order in which they were written
	ORGANIZATION IS LINE SEQUENTIAL.

* We can select another file:
SELECT PRESIDENTS ASSIGN TO 'presidents.dat'
* And how we'll read it:
	ORGANIZATION IS LINE SEQUENTIAL.
```

Still within PRE-FLIGHT:

```COBOL
* Structure of our program's data:
DATA DIVISION.
FILE SECTION.

* FD: file-name that will be referred to later in program
FD PRESIDENT-REPORT.
01 PRINTLINE PIC X(44).
* PICTURE or PIC specifies general characteristics 
* and editing requirements of a data item i.e., variable 
*
* PIX X - Alphanumeric or any character, (including binary).
* PIC 9 - Numeric; numbers 0-9, but no letters.
* PIC A - Alpha (A-Z, a-z, and space only).

* COBOL uses a hierarchical structure.
* Structure of presidents.dat file
FD PRESIDENTS.
01 PRESIDENT
   05 FIRSTNAME PIC X(25) VALUE SPACES.
* "FILLER" is optional; a matter of style
   05 FILLER PIC X(15) VALUE SPACES.
   05 LASTNAME PIC X(25) VALUE SPACES.
88 EOF VALUE "N".
```

It's important to remember that data described in the FILE SECTION should be exact. Know your data, they say. 

COBOL can handle JSON and XML nowadays too, by the way. But that's a topic for another post in another timeline.

Let's go over a few things before moving on. You might have noticed COBOL levels: <mark>01</mark> or <mark>05</mark>.

> These levels tell the COBOL compiler how to associate, or group, fields in the record.  Level 01 is a special case, and is reserved for the record level;  the 01 level is the name of the record. Levels from 02 to 49 are all "equal" (level 2 is no more significant than level 3), but there is a hierarchy to the structure.  Any field listed in a lower level (higher number) is subordinate to a field or group in a higher level (lower number).<sup><a href="#sup1">[1]</a></sup>

Sometimes you'll see levels incremented by 3 or 5. It's programmer preference. I hear some organizations have standards regarding such matters.Seems like a good idea.

The figure below illustrates levels:

<img id="levels" src="https://www.ibm.com/support/knowledgecenter/SS6SG3_4.2.0/com.ibm.entcobol.doc_4.2/PGandLR/images/igyl1015.gif"/>

66, 77, 88 are special level values:

* 66 - Assigns an alternate name to an existing field. Avoid it.
* 77 - Denotes an elementary item. Just use 01.
* 88 - Used as a conditional. You will see and use this plenty.

We use 88 for our End of File check. We'll discuss that a bit later.

We'll want to interact with our data. We do so within WORKING-STORAGE section, as below:

```COBOL
WORKING-STORAGE SECTION.
* More variables or data items
01 PAGEHEADING.
	05 FILLER PIC X(25) VALUE "Presidents of U.S.".
01 HEADERS PIC X(36) VALUE "#     FIRST  LAST".
01 PRESIDENCY PIC 9(2).
01 PRESIDENT-DETAIL-LINE.
	05 PRINT-PRESIDENCY PIC 9(2).
	05 FILLER PIC X(4) VALUE SPACES.
	05 PRINT-FIRSTNAME PIC X(25).
	05 FILLER PIC XXX VALUE SPACES.
	05 PRINT-LASTNAME PIC X(25).
01 REPORT-FOOTING PIC X(13) VALUE "END OF REPORT".
01 LINECOUNT PIC 99 VALUE ZERO.
*** END PRE-FLIGHT
*
```

PRE-FLIGHT checks done, cabin doors closed, whiskey safely in hand. Take a sip. Listen to the ice cubes clink against each other and the confines of the highball glass itself.

```COBOL
* Let's rock! 
PROCEDURE DIVISION.
* READ FROM:
OPEN INPUT PRESIDENTS
* WRITE TO:
OPEN OUTPUT PRESIDENTS-REPORT
```
Let's read a bit about the OPEN statement:

> The phrases INPUT, OUTPUT, I-O...specify the mode to be used for opening [a] file. At least one of the phrases INPUT, OUTPUT, I-O, must be specified with the OPEN keyword. The INPUT, OUTPUT, I-O phrases can appear in any order. <sup><a href="#sup2">[2]</a></sup>

The true bulk of our program reads:

```COBOL
* TAKE OFF!
PERFORM PRINT-PAGE-HEADING
PERFORM UNTIL EOF
READ PRESIDENTS
   AT END 
	   WRITE PRINT-LINE FROM REPORT-FOOTING 
	   AFTER ADVANCING 2 LINES
	   CLOSE PRESIDENTS
	   CLOSE PRESIDENTS-REPORT
	   SET EOF TO TRUE
   NOT AT END 
	PERFORM PRINT-REPORT-BODY
END-READ
END-PERFORM
STOP RUN.
```
Let's review the above:

```COBOL
PERFORM PRINT-PAGE-HEADING
```
You might have heard of [GOTO](https://en.wikipedia.org/wiki/Goto) statements and the resulting spaghetti code that more often than not results in their use. PERFORM is a way to lessen such messes. We're returned control after the PERFORM action is done.

PRINT-PAGE-HEADING is known as a Paragraph in COBOL. Check it out:

```COBOL
PRINT-PAGE-HEADING.
WRITE PRINT-LINE FROM HEADERS AFTER ADVANCING 2 LINES
```
Looks much like a function body, no? In any case, PERFORM calls the above "function".

```COBOL
PERFORM UNTIL EOF
READ PRESIDENTS
```
Recall our EOF check from earlier. We're saying we want to PERFORM this action until we've set EOF to true.

```COBOL
AT END 
* Print report's footing
   WRITE PRINT-LINE FROM REPORT-FOOTING 
   AFTER ADVANCING 2 LINES
   CLOSE PRESIDENTS
   CLOSE PRESIDENTS-REPORT
* Stop reading 
   SET EOF TO TRUE
NOT AT END
* If we haven't set EOF, let's write the report
   PERFORM PRINT-REPORT-BODY
 END-READ
END-PERFORM
STOP RUN.
```
Lots going on here. The most important aspect of this code is the SET EOF TO TRUE after we close all our files. There are many ways to do this in COBOL, but I found this appears the most readable to me.

Compile the code with:

```bash
$ cobc -x presidents.cob
```
Run it and see that the PresidentReport file should be populated with some useful information. Make sure the file exists before running the program.

In the next post we'll go over COBOL arrays which are known as tables. We'll also play around with some interactivity and jump into network programming.
<hr>
<sup id="sup1">[1]</sup>http://3480-3590-data-conversion.com/article-reading-cobol-layouts-1.html

<sup id="sup2">[2]</sup>https://www.ibm.com/support/knowledgecenter/SS6SG3_4.2.0/com.ibm.entcobol.doc_4.2/PGandLR/ref/rlpsopen.htm
