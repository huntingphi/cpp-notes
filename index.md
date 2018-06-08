# C++ Notes

## The C++ Pre-processor

The pre-processor scans the source code for embedded instructions prior to compilation. These commands usually modify the code prior to compilation, although they can be used to modify the way the compiler generates code (using the ```#pragma``` instruction for example)

### Key questions

**(CSC3022H June 2015)**

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

Token pasting - The creation of a new identifier from two supplied tokens using the macro operator ##

## Pointers and references

### Key questions

**(CSC3022H June 2015)**

1. Explain how pointers and references differ in terms of accessing a variable.
2. Given a variable x, of type int, show how you would declare a variable to hold the address of x and how you could create a reference y, which refers to x.
3. Write C++ code to create an array of 10 char pointers. Also provide the code that will destroy this array
4. If you are given a function, ```void myfunc (float *x);```, show how you could set up a function pointer to refer to this function. How would you call the function via this pointer?
5. Explain how an  r - value reference differs from a normal (l - value) reference. Illustrate your answer with an example.

### Pointers

A *pointer* is simply the address of an item in memory, while a *pointer variable* is a variable that contains such an address.

The most common reason why we would need a pointer is that we may be expected to deal with a memory reference that isn't explicitly coded into our program.

> "Say, for example, that during code execution, the operating system allows our program access to a segment of shared memory. How will it communicate that location of the data? The code is static — it has been compiled, and the executable cannot change. However, if the program has been written correctly, the OS can return a *pointer* to the block of shared memory, and this value can then be used to gain access to that memory. In this example, one would make a system function call requesting access to the memory, and would then receive the pointer value (the starting address) which could be placed in a pointer variable for later use." - Course Notes

Note that it is possible to mix pointer and non-pointer types in a variable declaration:

```c++

int *row, *col, length //row and col are int*, length is an int

```

Consider the following: we have a block of memory which is referred to by a pointer ```ptr```.
How do we access the actual data to which the pointer refers? The **dereference operator \*** provides that function. So, if we know that a data item resides at address 0x100000 (i.e. ```ptr = 0x100000```) we can use ```*ptr``` to access the actual data item itself.

Thus, if we have a block of memory, which is by definition contiguous, we can increment of decrement the memory address to move onto the next location.

Consider the following piece of code:

```c++

float *fptr;
fptr = GetBlockOfFloats(1024);
cout << "Float at fptr is " << *fptr << "Float at position fptr + 20 is " << *(fptr+20) << endl;

```

All we have done here, is print out the value of the float at the address pointed to by fptr, which is the 1st float of 1024. We then **print out the float which is 20 steps higher in memory** i.e. the 21st float in the block. Note that **the address is updated in units of the appropriate size**: if we add one to the pointer, we move one float forward, not one byte or one word. If the pointer type was int, adding one would adjust the pointer in units of size int.

It is not necessary to initialise the pointer when it is declared. Not doing so will result in an undefined value. C++ defines the constant NULL — which is the address 0x0. If you create a pointer variable you should initialise it to NULL to reduce the chance of a pointer error later.

#### Multiple Indirection

> "In certain cases it is desirable to store the address of variable in memory, which itself contains an address. This is known as multiple indirection, and the indirection may continue for as many levels as required. Each new level requires that one add an additional * after the type declaration, thus for two levels of indirection."

#### The Pointer void*

C++ allows for a *generic pointer* denoted by:

```c++

void * ptr;


```

It has no specific type and therefore must be cast to the appropriate type before use.

#### Arrays and pointers

C++ considers the array name to be a pointer to the first element of the array. That is,

 ```c++

int arr[7] = {0,1,2,3,4,5,6};

std::cout<<*arr<<std::endl;

```

will print out ```0```.

Also note that any pointer in C++ can be dereferenced either by the ```*``` operator or the array index operator:

```c++

char* cptr = new char[2];
cptr[0] = 'b';
cptr[1] = 'a';
int* iptr = new int[2];
*iptr = 0;
*(iptr+1) = 10; //Equivalent to iptr[1];

std::cout<<cptr[1]<<std::endl; //prints a
std::cout<<*(cptr+1)<<std::endl; //Also prints a
std::cout<<*(iptr+1)<<std::endl; //prints 10
std::cout<<*iptr+1<<std::endl; //prints 1

```

Also note that the indirection operator has higher precedence than the + operator as shown above, hence ```*iptr+1``` results in ```(*iptr)+1```.

Multi-dimensional arrays are also stored as a sequence of consecutive items in memory.

Note that, given a 2D char array, because a pointer to a char is how C++ interprets a string, the following code:

```c++

char* cptr = new char[2];
cptr[0] = 'b';
cptr[1] = 'a';
std::cout<<cptr<<std::endl; //prints co

char carr[2][2];
    carr[0][0] = '1';
    carr[0][1] = '2';
    carr[1][0] = '3';
    carr[1][1] = '4'; //prints 1234\0

```

prints everything in the array, and will do the same for the nth array in a multidimensional array.

#### Dynamic Allocation

Arrays are often inadequate as it isn't possible to change their allocated memory space as the program runs. The solution to this is *dynamic memory allocation* via the ```new``` keyword. Memory acquired this way is allocated on the heap and will persist until freed up by the programmer (via the ```delete``` keyword which, for object types, invokes the destructor) or the program terminates. This is in contrast to auto (automatic) variables, created on the stack, that are destroyed as soon as they go out of scope.

#### Function Pointers

A function pointer is a variable that stores the address of a function that can later be called through that function pointer.

**Syntax**

From [CProgramming.com](https://www.cprogramming.com/tutorial/function-pointers.html):
> The syntax for declaring a function pointer might seem messy at first, but in most cases it's really quite straight-forward once you understand what's going on. Let's look at a simple example:

```c++

void (*foo)(int);

```

> In this example, ```foo``` is a pointer to a function taking one argument, an integer, and that returns void. It's as if you're declaring a function called "```*foo```", which takes an int and returns ```void```; now, if ```*foo``` is a function, then ```foo``` must be a pointer to a function. (Similarly, a declaration like ```int *x``` can be read as ```*x``` is an ```int```, so ```x``` must be a pointer to an ```int```.)
>The key to writing the declaration for a function pointer is that you're just writing out the declaration of a function but with (```*func_name```) where you'd normally just put ```func_name```.
>Sometimes people get confused when more stars are thrown in:

```c++

void *(*foo)(int *)

```

>Here, the key is to read inside-out; notice that the innermost element of the expression is ```*foo```, and that otherwise it looks like a normal function declaration. ```*foo``` should refer to a function that returns a ```void *``` and takes an ```int *```. Consequently, ```foo``` is a pointer to just such a function.

Functions are derereferenced by standard function invocation:

```c++

#include <stdio.h>
void my_int_func(int x)
{
    printf( "%d\n", x );
}


int main()
{
    void (*foo)(int);
    foo = &my_int_func;

    /* call my_int_func (note that you do not need to write (*foo)(2) ) */
    foo( 2 );
    /* but if you want to, you may */
    (*foo)( 2 );

    return 0;
}

```

> Note that function pointer syntax is flexible; it can either look like most other uses of pointers, with & and *, or you may omit that part of syntax. This is similar to how arrays are treated, where a bare array decays to a pointer, but you may also prefix the array with & to request its address.

#### Smart Pointers

Smart pointer: "Templated pointer management class which frees the user fro
m having to de-allocate pointers explicitly"

##### [Unique Pointers](http://en.cppreference.com/w/cpp/memory/unique_ptr)

```unique_ptr``` wraps a pointer in an automatic variable. Therefore the pointer is automatically deleted when it leaves the scope. It's just as efficient as normal pointers with zero overhead.

##### [Shared Pointers](http://en.cppreference.com/w/cpp/memory/shared_ptr)

\<Will be filled out as needed>

##### [Weak Pointers](http://en.cppreference.com/w/cpp/memory/weak_ptr)

\<Will be filled out as needed>

### References

When a function is called with arguements, the contents of each arguement are copied to each *formal parameter* (the variables listed in the arguement list of the function declaration). This is known as *call by value* and is how function arguements are processed.

An example:

```c++

void square(int x){
    x*=x;
}

int num = 2;
square(num);

std::cout<<num<<std::endl; //Prints 2

```

The value 2 is printed above as ```num``` is *passed by value* (a ```num``` is copied into ```int x``` when passed to the function) and therefore is unaffected by changes within the function.

To circumvent this we can pass ```num``` by reference:

```c++

void square(int& x){
    x*=x;
}

int num = 2;
square(num);

std::cout<<num<<std::endl; //Prints 4

```

By passing by reference, an alias to the actual parameter is stored in the formal parameters, therefore changes made in the function are made to the object at that aliased address and apply to ```num``` regardless of changes taking place within the function.

Reference arguments cannot usually accept constant values, which makes sense as you can't pass the address of the value of a literal, such as the number 5, as these aren't stored in memory:

>"In the C++ view of the world, a literal does not occupy any memory. A literal just exists.
>This view makes that there is no address for a pointer to refer to when it would point to a literal and for that reason, pointers to literals are forbidden.
>Const references are actually the exception here in that they allow apparent indirect access to a literal. What happens underneath is that the compiler creates a temporary object for the reference to refer to and that temporary object is initialized with the value of the reference.
>This creating of a temporary that the reference gets bound to is only allowed for const references (and gets used in other scenarios as well, not only for literals), because that is the only case where a reference/pointer to a temporary won't have undesired side-effects." -[StackExchange Answer](https://softwareengineering.stackexchange.com/questions/224501/why-are-pointers-to-literals-not-possible)

However if we use *constant reference*, declared as```const& type```we can circumvent this and pass in constant values. In this case, the compiler is notified that we will not try to modify the (constant) value we pass in and will thus allow the code to compile (it creates space for the constant, and connects the reference to this)

>"It is more efficient to pass arguments by reference, since a reference is essentially a pointer and an address has a small fixed size, whereas if you pass a class object or structure by value, the entire object is duplicated and passed into the function, which is both time consuming and wasteful of space" - Course Notes

References may also be defined for variables other than formal arguments of a function. The following limitations on references apply:

1. The reference must be initialised to an existing variable. That is, it can't be empty, undefined or null.
2. The reference cannot thereafter refer to any other variable. It is a fixed alias.

For example:

```c++

int N = 2, M;
int& myint = N;

```

>"Once this code sequence has been executed, myint and N are the same item. The statement ```myint = M``` will not change the reference (changing the reference is illegal) but will simply assign the value of M to both N and myint." - Course Notes

### R-Value references

An r-value reference is a reference to a temporary variable that will vanish when the scope ends.

```c++

string &&s = string(‘‘hello’’); // create rvalue reference to string object

```

Here s refers to the temporary string object holding the string literal “hello”.

These r-value references are used with the std::move() function to transfer data from an object that is about to go out of scope (the r-value reference) to one that will persist, and thus we can avoid unnecessary temporary object creation and data copying.