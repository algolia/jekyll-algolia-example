---
layout: post
title: Why develop our own Unicode Library? - The Algolia Blog
author:
  login: julien
  email: julien.lemoine@algolia.com
  display_name: julien
  first_name: Julien
  last_name: Lemoine
---

At one time or another, most developers come across bugs or problems with
Unicode (about 3,720,000 results on google for the request [unicode bug
developer][1] at the time
of this writing). Let me tell you about my experience in the last decade and
why we have now implemented our own unicode Library to produce exactly the
same result across devices/languages.

I first started to use Unicode in 2004 when I was developing a Text Mining
software specialized on information extraction. This software was fully
implemented in C++ and I used [IBM ICU library](http://www.google.com/url?q=ht
tp%3A%2F%2Fwww-01.ibm.com%2Fsoftware%2Fglobalization%2Ficu%2F&sa=D&sntz=1&usg=
AFQjCNHZa7RaPrYgI3i22oL77_ZBjvF4vw) to be Unicode compliant (all strings were
stored in UTF16). I also used some normalization functions of ICU based on
decomposition, but I did not notice any major problem at that time. I started
to understand the dark side of Unicode later when I used it in other languages
like Java, Python, and later in Objective-C. My first surprise was when I
understood that a simple isAlpha(unicodechar c) method can return different
results!

I started to look in details at the standard and downloaded UnicodeData.txt
(the file that contains most of the information about the standard, you can
grab the latest version
[here][2].

This file contains descriptions of all Unicode characters. Third column
represents "General Category" and is documented as:

## **General Categories**

The values in this field are abbreviations for the following. Some of the
values are normative, and some are informative. For more information, see the
Unicode Standard.

#### Normative Categories

  * **Lu**: Letter, Uppercase
  * **Ll**: Letter, Lowercase
  * **Lt**: Letter, Titlecase
  * **Mn**: Mark, Non-Spacing
  * **Mc**: Mark, Spacing Combining
  * **Me**: Mark, Enclosing
  * **Nd**: Number, Decimal Digit
  * **Nl**: Number, Letter
  * **No**: Number, Other
  * **Zs**: Separator, Space
  * **Zl**: Separator, Line
  * **Zp**: Separator, Paragraph
  * **Cc**: Other, Control
  * **Cf**: Other, Format
  * **Cs**: Other, Surrogate
  * **Co**: Other, Private Use
  * **Cn**: Other, Not Assigned (no characters in the file have this property)

#### Informative Categories

  * **Lm**: Letter, Modifier
  * **Lo**: Letter, Other
  * **Pc**: Punctuation, Connector
  * **Pd**: Punctuation, Dash
  * **Ps**: Punctuation, Open
  * **Pe**: Punctuation, Close
  * **Pi**: Punctuation, Initial quote (may behave like Ps or Pe depending on usage)
  * **Pf**: Punctuation, Final quote (may behave like Ps or Pe depending on usage)
  * **Po**: Punctuation, Other
  * **Sm**: Symbol, Math
  * **Sc**: Symbol, Currency
  * **Sk**: Symbol, Modifier
  * **So**: Symbol, Other

As you can see there is quite a lot of categories, some of them are very easy
to understand like "Lu" (Letter, uppercase) and "Ll" (Letter, lowercase) but
some of them are more complex like "Lo" (Letter, other)  and "No" (Number,
other), and this is exactly where the first problem begins.

Let's take the unicode character U+00BD(½) as an example. It is quite common
to describe spare parts and is defined as "No"... except that some unicode
libraries consider that this is not a number and return false to
isNumber(unicodeChar) method (e.g., Objective-C).

In fact the two most used methods, isAlpha(unicodeChar) and
isNumber(unicodeChar), are not directly defined by the Unicode standard and
are subject to interpretation.

The consequence is that results are not the same across devices/languages! In
our case this is a problem because our compiled index is portable, and we want
to have exactly the same results on different devices/languages.

However, this is not the only problem! Unicode normalization is also a tricky
topic. The Unicode standard defines a way to decompose characters (Characters
decomposition mapping), for example U+00E0(à) which is decomposed as U+0061(a)
+ U+0300( ̀). But most of the time you do not want a decomposition but a
normalization: get the most basic form of a string (lowercase without accents,
marks, …). This is key to be able to search and compare words. For example,
the normalization of the French word "Hétérogénéité" will be normalized as
"heterogeneite".

To compute this normalized form, most people compute the lowercase form of a
word (well defined by the Unicode standard), then compute the decomposed form
and finally remove all the diacritics. However, this is not enough.
Normalization can not always be reduced to just a matter of removing marks.
For example the standard German letter ß is widely used and
replaced/understood as "ss" (you can enter ß in your favorite web search
engine and you will discover that it also search for "ss"). The problem is
that there is no decomposition for "ß" in the Unicode standard because this
letter is not a letter with marks.

To solve that problem, we need to look in the [Character Fallback Substitution
table][3] that is not
part of most of Unicode library implementations. This substitution table
defines that "ß" can be replaced by "ss,". There are plenty of other examples;
For instance, 0153(œ) and 00E6(æ), letters of the French language, can be
replaced by "oe" and "ae".

At the end, this led us to implement our own Unicode library to ensure that
our isAlpha(unicodechar) and isNumber(unicodechar) methods have a unique
behavior on all devices/languages and to implement a normalize(unicodestring)
method that contains character fallback substitution table. By the way our
implementation of normalization is far more efficient because we implemented
it in one step instead of three (lowercase + decomposition + diacritics
removal).

I hope you found this post useful and gained a better understanding of the
Unicode standard and the limits of standard Unicode libraries. Feel free to
contribute comments or ask for precisions.


[1]: http://www.google.com/search?q=unicode+bug+developer
[2]: ftp://ftp.unicode.org/Public/UNIDATA/UnicodeData.txt)
[3]: http://unicode.org/repos/cldr-tmp/trunk/diff/supplemental/character_fallback_substitutions.html
