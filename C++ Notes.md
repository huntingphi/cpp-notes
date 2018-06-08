# C++ Notes

## The C++ Pre-processor

The pre-processor scans the source code for embedded instructions prior to compilation. These commands usually modify the code prior to compilation, although they can be used to modify the way the compiler generates code (using the ```#pragma``` instruction for example)

### Key questions

(CSC3022H June 2015)

1. What do we mean by macro expression and where does this fit into the C++ compilation process?
2. Given the following simple macro, show how to use this to evaluate the sum in an expression in your code can lead to logic errors: ```#define SQR(x) x*x```
3. What is the purpose of ```#ifdef``` statements in a header file
There is no answer in the course notes but there is an answer for ```#ifndef```:
> The purpose of the ```#ifndef``` (“if symbol not defined”) directive is to ensure that the header file is not included multiple times.

### Preprocessor Directives

The header file itself is inserted using an ```#include``` directive, which simply inserts the file into the current source file.

The ```#define``` directive allows us to define constants which can be subsequently be used throughout our code. In the case of this directive the C++ pre-processor scans the code looking for occurrences of the *macro* we have defined. When it finds one, it physically replaces it with the value of the defined macro.

For example:
```c++

#define MYNUMBER 22
if (variable == MYNUMBER)

```
expands to
``` c++

if(variable == 22)

```

This happens prior to compilation and is known as *macro expansion*.

Macros can accept arguments and refer to previously defined arguments.

For example:

``` c++

#define NEW_VALUE (VALUE + 22)

```
and with arguments:

```c++

#define MAX(a,b) ((a)>(b)?(a):(b))

```

The two arguments to the macro occur in brackets followed by the code to be inserted by the preprocessor after the arguments have been substituted:

```c++

if(MAX(i,j)>3){
    return MAX(i++,j++);
}

```

expands to

```c++

if((i)>(j)?(i):(j))>3){
    return (i++)>(j++)?(i++):(j++));
}

```

> Note how the macro allows us to simplify the code, whilst avoiding the expense of a function call. There is however a trade-off between code size and speed.
Note the extensive use of parentheses. Logic and syntax errors can easily creep in if the macros aren't protected correctly. See the following examples:

```c++

#define SUM(a,b) a+b
if (SUM(2,3)*5 >5)

```

expands to

```c++

if (2+3*5>5) //Not (2+3)*5

```

A define directive can also span multiple lines by ending the line with a '\':

```c++

#define LONGDEF(a) if ( (a)>3 || (a) < 7) { \
    //body \
    }\
    else{ \
    //body \
    }

}

```
By convention the name of the macro is capitalized to distinguish it from a variable or function call. If a constant value or string is used at various places in code a macro should be defined and used instead.

### Pre-processor string operations

In addition to macro expansion, the C++ pre-processor has the following string operations:

Stringizing - Taking an identifier and turning it into a string of characters using the macro operator #. I.e ```#define func(x) #x``` will replace x with "x".

Token pasting - The creation of a new identifier from two supplied tokens using the macro operator ##: