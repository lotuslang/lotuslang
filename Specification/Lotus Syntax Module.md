# Lotus Syntax Module

*11th of May, 2019*

Last updated : *11th of May, 2019*

**Editors :**

- Blokyk

**Authors :**

- Blokyk

---

## Abstract

This document describes the basic structure and syntax of Lotus source code. It defines, in details, the tokenizing, parsing, and translation of Lotus.

## Overview

Lotus is an open-source language created to facilitate the work of web designers and web developpers. This document describes its syntax as well as the way it should be interpreted by a compiler to conform to the Lotus specification.

## Table Of Contents

1. **[Introduction](#1.-introduction)**
   
   - [x] [Visual and Writing Conventions](1.1-visual-and-writing-conventions)

2. **[Tokenizing and Parsing Lotus](2.-tokenizing-and-parsing-lotus)**
   
   - [x] [Definitions](2.1-definitions)
   - [x] [Token types](2.2-token-types)
   - [ ] [Tokens Railroad Diagrams](2.3-tokens-railroad-diagrams)
   - [ ] [Tokenization Errors](2.4-tokenization-errors)
   - [ ] [Parse Tree Model](2.5-parse-tree-model)
   - [ ] [Parse Tree Node Railroad Diagrams](2.6-parse-tree-node-railroad-diagrams)
   - [ ] [Parsing Errors](2.7-parsing-errors)

3. **[Tokenization](3.-tokenization)**
   
   - [ ] [Preprocessing the input stream](3.1-preprocessing-the-input-stream)
   - [ ] [Consume a token](3.2-consume-a-token)
   - [ ] [Consume a string token](3.3-consume-a-string-token)
   - [ ] [Consume a number token](3.4-consume-a-number-token)
   - [ ] [Consume an escaped code point](3.5-consume-an-escaped-code-point)
   - [ ] [Consume a name](3.6-consume-a-name)
   - [ ] [Consume an ident-like token](3.7-consume-an-ident-like-token)
   - [ ] [Consume an identifier token](3.8-consume-an-identifier-token)
   - [ ] [Check if a code point would start an identifier](3.9-check-if-a-code-point-would-start-an-identifier)
   - [ ] [Check if two code points are a valid escape](3.10-check-if-two-code-points-are-a-valid-escape)
   - [ ] [Check if two code points would start a number](3.11-check-if-two-code-points-would-start-a-number)

4. **[Parsing](4.-parsing)**
   
   - [ ] [Consume a token](4.1-consume-a-token)
   - [ ] [Consume a simple block](4.2-consume-a-simple-block)
   - [ ] [Consume an assignement](4.3-consume-an-assignement)
   - [ ] [Consume an operation](4.4-consume-an-operation)
   - [ ] [Consume a function call](4.5-consume-a-function-call)
   - [ ] [Consume a function declaration](4.6-consume-a-function-declaration)
   - [ ] [Consume an if clause](4.7-consume-an-if-clause)
   - [ ] [Consume an else clause](4.8-consume-an-else-clause)
   - [ ] [Consume a for loop declaration](4.9-consume-a-for-loop-declaration)
   - [ ] [Consume a foreach loop declaration](4.10-consume-a-foreach-loop-declaration)
   - [ ] [Consume a while loop declaration](4.11-consume-a-while-loop-declaration)
   - [ ] [Consume a do-while loop declaration](4.12-consume-a-do-while-loop-declaration)
   - [ ] [Consume a class declaration](4.13-consume-a-class-declaration)
   - [ ] [Consume an interface declaration](4.14-consume-an-interface-declaration)
   - [ ] [Consume an enum declaration](4.15-consume-an-enum-declaration)
   - [ ] [Consume an import directive](4.16-consume-an-import-directive)
   - [ ] [Consume a from directive](4.17-consume-a-from-directive)
   - [ ] [Consume a namespace declaration](4.18-consume-a-namespace-declaration)
   - [ ] [Consume an extends directive](4.19-consume-an-extends-directive)

## 1. Introduction

This document describes the syntax of Lotus source code.

It also defines algorithms to convert text into a list of tokens, and then transform it into an abstract syntax tree, that can be analysed, verified, and used for interpretation or translation purposes, for example to HTML, CSS, and JavaScript as does [LotusJS](https://github.com/lotuslang/lotusjs).

### 1.1 Visual and Writing Conventions

**Notes :** Notes and informative sections are presented as a quote block and starts with the word "Note:".

> Note: This is an example of an informative section

**Examples :** Examples in this document are presented as a quote block and starts with the word "Example".

> Example : This is an example of an example section

**Short Code Segment :** Short code segments, such as token names, are presented in `<code>` tags.

`This is a short code segment`

**Multi-line Code Segment :** Long/Multi-line code segments, such as informal code examples, are presented in `<pre>` tags.

```
This is
A multiline 
Code segment
```

**Informal section :** Informal sections are preceded by the words "*This section is informative*" written in *italics*.

**Definition :** Words definitions are presented as h6 headers for the word(s) being defined, and the definition is presented in *italics*

## 2. Tokenizing and Parsing Lotus

Compilers and interpreters must use the parsing rules described in this specification to generate an *Abstract Syntax Tree* (AST) that can then be used to analyze, optimize or translate Lotus source code. 

### 2.1 Definitions

This section defines terms used throughout this 

###### code point

*A [Unicode code point](http://unicode.org/glossary/#code_point). A Unicode character with a value between 0 (hexadecimal) and 10FFFF (hexadecimal).*

###### input stream

*A Unicode stream (or list) of [code points](code-point).*

###### consume the next code point

*If the [input stream](input-stream) is treated as a stack, is analogous to poping a [code point](code-point) from the [input stream](input-stream) and returning the resulting [code point](code-point). If the [input stream](input-stream) is treated as an indexed list, is analogous to incrementing the list's index.*

###### next input code point

*The first [code point](code-point) in the [input stream](input-stream) that has not been [consumed](consume-the-next-code-point) yet.*

###### current input code point

*The last [code point](code-point) to have been [consumed](consume-the-next-code-point).*

###### reconsume the current code point

*Push the [current input code point](current-input-code-point) on top of the [input stream](input-stream), so that next time you will need to [consume the next code point](consume-the-next-code-point), the [current input code point](current-input-code) will be [consumed](consume-the-next-code-point) instead.*

###### EOF code point

*A conceptual [code point](code-point) used to represent the end of the [input stream](input-stream). If the [input stream](input-stream) is empty, the [next code point](next-code-point) is always an [EOF code point](eof-code-point).*

###### digit

*A [code point](code-point) between U+0030 DIGIT ZERO (0) and U+0039 DIGIT NINE (9).*

###### hex digit

*A [digit](digit), or a [code point](code-point) between U+0041 LATIN CAPITAL LETTER A (A) and U+0046 LATIN CAPITAL LETTER F (F), or a [code point](https://www.w3.org/TR/css-syntax-3/#code-point "code point") between U+0061 LATIN SMALL LETTER A (a) and U+0066 LATIN SMALL LETTER F (f).*

###### number

*A sequence of [digit](digit), optionally followed by a single full stop (i.e. U+002E FULLSTOP `.`) and a sequence of digit, and/or optionally followed by a U+0045 LATIN CAPITAL LETTER E (`E`) or U+0065 LATIN SMALL LETTER E (`e`), a U+002B PLUS SIGN (`+`) or a U+002D HYPHEN MINUS (`-`), and a sequence of [digit](digit).*

###### uppercase letter

*A [code point](code-point) between U+0041 LATIN CAPITAL LETTER A (A) and U+005A LATIN CAPITAL LETTER Z (Z).*

###### lowercase letter

*A [code point](code-point) between U+0061 LATIN SMALL LETTER A (a) and U+007A LATIN SMALL LETTER Z (z).*

###### letter

*An [uppercase letter](uppercase-letter) or a [lowercase letter](lowercase-letter).*

###### non-ASCII code point

*A [code point](code-point) with a value equal or greater than U+0080 <control>.*

###### name code point

*A  [letter](letter), a [digit](digit), a [non-ASCII code point](non-ascii-code-point), a U+002D HYPHEN MINUS (-), or a U+005F LOW LINE (_).*

###### newline

*A [code point](code-point) with a value of U+000A LINE FEED (represented as the escape sequence '\n' in most programming language).*

###### whitespace

*A [newline](newline), a U+D800 CHARACTER TABULATION, or a U+0020 SPACE.*

###### maximum allowed code point

*The [code point](code-point) with the value U+10FFF.*

###### input token stream

*The stream of tokens produced by the tokenizer.*

###### current input token

*The last token from the [input token stream](input-token-stream) to have been [consumed](consume-a-token).*

###### next input token

*The first token in the [input token stream](input-token-stream) that has not been [consumed](consume-a-token) yet.*

###### EOF token

*A conceptual token used to represent the end of the [input token stream](input-token-stream). If the [input token stream](input-token-stream) is empty, the next token is always an [EOF token](eof-token).*

###### reconsume the current input token

*Push the [current input token](current-input-token) on top of the [input token stream](input-token-stream), so that next time you will need to [consume the next token](consume-a-token), the [current input token](current-input-token) will be [consumed](consume-a-token) instead.*

### 2.2 Token types

Implementations should use the algorithms described in [section 4](tokenization), or act as they  use them (i.e. the resulting token stream should be identical, no matter the underlying implementation). The [tokenization](tokenization) phase should output a stream of zero or more of the following tokens.

###### `<{-token>`

*A token representing an opening curly brace (`{`). It doesn't hold any specific value.*

###### `<}-token>`

*A token representing a closing curly brace (`}`). It doesn't hold any specific value.*

###### `<(-token>`

*A token representing an opening/left parenthesis (`(`). It dosn't hold any specific value.*

###### `<)-token>`

*A token representing a closing/right parenthesis (`)`). It doesn't hold any specific value.*

###### `<[-token>`

*A token representing an opening/left square bracket (`[`). It doesn't hold any specific value.*

###### `<]-token>`

*A token representing a closing/right square bracket (`]`). It doesn't hold any specific value.*

###### `<point-token>`

*A token representing a point (`.`). It doesn't hold any specific value.*

###### `<comma-token>`

*A token representing a comma (`,`). It doesn't hold any specific value.*

###### `<colon-token>`

*A token representing a colon (`:`). It doesn't hold any specific value.*

###### `<semicolon-token>`

*A token representing a semicolon (`;`). It doesn't hold any specific value.*

###### `<SLC-token>`

*A token representing the start of a single line comment (i.e. U+0023 NUMBER SIGN (`#`)). It doesn't hold any specific value.*

###### `<MLCD-token>`

*A token representing a multi-line comment delimitor (i.e. U+0023 NUMBER SIGN U+0023 NUMBER SIGN U+0023 NUMBER SIGN (`###`)). It doesn't hold any specific value.*

###### `<DCO-token>`

*A token representing a opening delimiter of an embedded documentation (`=begin`). It doesn't hold any specific value.*

###### `<DCC-token>`

*A token representing a closing delimiter of an embedded documentation (`=end`). It doesn't hold any specific value.*

###### `<logical-OR-token>`

*A token representing a logical OR operator (i.e. U+007C VERTICAL LINE U+007C VERTICAL LINE (`||`)). It doesn't hold no specific value.*

###### `<logical-AND-token>`

*A token representing a logical AND operator (i.e. U+0026 AMPERSAND U+0026 AMPERSAND (`&&`)). It doesn't hold any specific value.*

###### `<logical-XOR-token>`

*A token representing a logical XOR operator (i.e. U+005E CIRCUMFLEX ACCENT (`^`)). It doesn't hold any specific value.*

###### `<number-token>`

*A token representing a [number](number). It holds a value composed of one or more [code points](code-point), and a numeric value.*

###### `<ident-token>`

*A token representing an [identifier](identifier). It holds a value composed of one or more [code points](code-point).*

###### `<string-token>`

*A token representing a string (i.e. a sequence of [code points](code-point), delimited by a delimiting code point, either U+0022 QUOTATION MARK (`"`) or U+0027 APOSTOPHE (`'`)). It holds a value composed of zero or more code points*

###### `<assign-token>`

*A token representing a single equals sign (i.e. U+003D EQUALS SIGN `=`) [code point](code-point). It doesn't hold any specific value.*

###### `<plus-sign-token>`

*A token representing a single plus sign (i.e. U+002B PLUS SIGN `+`) [code point](code-point). It doesn't hold any specific value.*

###### `<minus-sign-token>`

*A token representing a single minus sign (i.e. U+002D HYPHEn MINUS `-`) [code point](code-point). It doesn't hold any specific value.*

###### `<multiply-sign-token>`

*A token respresenting a single multiply sign (i.e. U+002A ASTERISK `*`) [code point](code-point). It doesn't hold any specific value.*

###### `<divide-sign-token>`

*A token representing a single divide sign (i.e. U+002F SOLIDUS `/`) [code point](code-point). It doesn't hold any specific value.*

### 2.3 Tokens Railroad Diagrams

### 2.4 Tokenization Errors

### 2.5 Parse Tree Model

### 2.6 Parse Tree Node Railroad Diagrams

### 2.7 Parsing Errors

## 3. Tokenization

*This section is informal*

Tokenization is a process by which an algorithm will convert a sequence of [code points](code-point) to a list of tokens. 

A token is an object representing zero or more [code points](code-point) and optionally metadata associated with it. 

> Example: A [`<number-token>`](number-token) represents a number.
> 
> It holds a sequence of [code points](code-point), as well as the numeric value of this sequence.
> 
> The sequence of [code points](code-point) is its reprensetation, and the numeric value is its metadata

The following algorithms describes how to transform a stream of [code points](code-point), designated as the [input stream](input-stream), into a stream of tokens. 

Implementations should act as if they used those algorithms (i.e. the resulting token stream should be contain the same information as if it implemented the following algorithms. See [the Lotus tokenizer reference implementation]() for more information).

### 3.1 Preprocessing the input stream

Before sending the [input stream](input-stream) to the tokenizer, it needs to be processed as following :

- Replace any U+0000 NULL [code point](code-point) with U+FFFD REPLACEMENT CHARACTER

- Replace any U+000D CARRIAGE RETURN [code point](code-point), U+000C FORM FEED [code point](code point), or pair of U+000D CARRIAGE RETURN U+000A LINE FEED [code points](code-point) by a single U+000A LINE FEED [code point](code-point).

### 3.2 Consume a token

This section describes ***how to consume a token*** from a stream of [code points](code-point).

[Consume the next code point](consume-the-next-code-point).

If it is a **whitespace** :

   Consume as much [whitespace](whitespace) as possible and return nothing.

If it is a **U+0022 QUOTATION MARK (`"`)** :

   [Consume a string token](consume-a-string-token) with the ending delimiter set to U+0022 QUOTATION MARK (`"`). Return the resulting [`<string-token>`](string-token).

### 3.3 Consume a string token

### 3.4 Consume a number token

### 3.5 Consume an escaped code point

### 3.6 Consume a name

### 3.7 Consume an ident-like token

### 3.8 Consume an identifier token

### 3.9 Check if a code point would start an identifier

### 3.10 Check if two code points are a valid escape

### 3.11 Check if two code points would start a number

## 4. Parsing

### 4.1 Consume a token

### 4.2 Consume a simple block

### 4.3 Consume an assignement

### 4.4 Consume an operation

### 4.5 Consume a function call

### 4.6 Consume a function declaration

### 4.7 Consume an if clause

### 4.8 Consume an else clause

### 4.9 Consume a for loop declaration

### 4.10 Consume a foreach loop declaration

### 4.11 Consume a while loop declaration

### 4.12 Consume a do-while loop declaration

### 4.13 Consume a class declaration

### 4.14 Consume an interface declaration

### 4.15 Consume an enum declaration

### 4.16 Consume an import directive

### 4.17 Consume a from directive

### 4.18 Consume a namespace declaration

### 4.19 Consume an extends directive
