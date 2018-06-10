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

A define directive can also span multiple lines by ending the line with a '\\':

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

std::string &&s = std::string("hello"); // create rvalue reference to string object

```

Here s refers to the temporary string object holding the string literal “hello”.

These r-value references are used with the std::move() function to transfer data from an object that is about to go out of scope (the r-value reference) to one that will persist, and thus we can avoid unnecessary temporary object creation and data copying.

### Classes and RAII

#### The Default Constructor

> "The purpose of a constructor is to provide a means to perform any necessary initialisation on the object, such as the allocation of memory or assigning default values to member variables" -Course Notes

If a constructor isn't provided for the class then one is automatically generated by the compiler. The default constructor performs a number of housekeeping tasks but doesn't assign values to variables.
An initialiser list should always be used over using the constructor body where possible.

Note that while a constructor need not be public, this limits class usability.

Any number of constructors can be defined provided they have unique argument lists.

Given the Matrix class, and the two constructors below,

```c++

Matrix(){
    std::cout<<"Default constructor called"<<std::endl;
    rows=cols=0;
    space = NULL;
}

Matrix(int r, int c):rows(r),cols(c){
    std::cout<<"Paramterised constructor called"<<std::endl;
}

```

The first constructor will be called when we instantiate an object with no parameters, and the second when we instantiate it with values:

```c++

Matrix foo_matrix; //prints "Default constructor called"
Matrix bar_matrix(2,2); //prints "Paramterised constructor called"
Matrix* matrix_ptr = new Matrix(2,2); //prints "Paramterised constructor called"

```

A constructor is also used to initialise data members which cannot be initialised in a normal manner, such as ```const``` and reference variables that must be initialised when the object is created.

If we declare an array of objects, the default constructor will be invoked for each one. It is possible for each array element to invoke a different constructor but this will only work with arrays declared on the stack. (Apparently, although I've tried it with arrays declared with the ```new``` keyword and it works fine )

Using an array initialisation list:

```c++

Matrix foo_matrix; //prints "Default constructor called"
Matrix bar_matrix(2,2); //prints "Paramterised constructor called"
Matrix* matrix_ptr = new Matrix(2,2); //prints "Paramterised constructor called"
Matrix matrices[] = {Matrix(2,2),Matrix()}; //prints "Paramterised constructor called \n "Default constructor called""

```

**Note that if any other constructor is defined, a default constructor must be defined or an error will be thrown at compile time**

#### Destructor

The destructor ensures that an class is properly eliminated when the object goes out of scope or is destroyed by the ```delete``` instruction. If it is not explicitly defined the compiler will generate one for housekeeping. A destructor must be defined for a constructor that dynamically allocates memory for that memory to be deallocated correctly when the object is destroyed.

A destructor must satisfy the following rules:

1. It must not have a return type or argument list - it therefore also can't be overloaded
2. It must be in the public section of the class, take the same name as the class name, and be prefixed with a '~'

#### Copy Constructor

An object can also be initialised to the same as an existing instance via the copy constructor.
> "C++ generates a default copy constructor which does a field by field copy from the source object to the (new) target object. Naturally, this copy is “shallow” in the sense that only field values are copied: any memory which a pointer field may point to will not be duplicated. If our class contains member variables that point to a dynamically allocated block of memory, we must write our own copy constructor to ensure that a “deep” copy occurs" - Course Notes

Note the following subtley when calling a copy constructor defined for the above mentioned Matrix class:

```c++

Matrix(const Matrix& other) : row(other.row), col(other.col){
    std::cout<<"Copy constructor called"<<std::endl;
}

Matrix matrix1();

Matrix newMatrix_1(matrix1); // prints "Copy constructor called"
Matrix newMatrix_2 = matrix1; //prints "Copy constructor called"
newMatrix_2 = matrix1; //Doesn't print anything as it calls the default copy assignment operator

```

**The = operator invokes the copy constructor only in the context of the declation**


#### Move Constructor

Member functions often return a copy of an object. (Because C++ uses pass by value: it copies the actual parameter into the formal parameter, it returns that copy) This "return by value" causes the class copy constructor to be invoked, building a new class object with the contents of the temporary variable returned by the function.

For example, consider the Person class, that has an object of a Car class as an attribute:

```c++

#include <iostream>

using namespace std;
class Car{
public:
  Car(){
    std::cout<<"Default constructor called in Car"<<std::endl;
  }
  Car(Car&){
    std::cout<<"Copy constructor called in Car"<<std::endl;
  }
  Car(Car&&){
    std::cout<<"Move constructor called in Car"<<std::endl;
  }
};
class Person{
public:
  Car car;//Prints "Default constructor called in Car" This is because we have two Car attributes in Person
  Car car_1;//Prints "Default constructor called in Car"

  Person(){}
  Person(Car c):car(std::move(c)){}//Prints "Move constructor called in Car" /n "Default constructor called in Car"
  Person(Car c, Car c_1):car(std::move(c)), car_1(c_1){}//Prints "Move constructor called in Car" /n "Copy constructor called in Car"

  void setCar(Car c){//Prints "Copy constructor called in Car" bc passed by value

  }
  void setCarWithMove(Car c){}
};

int main()
{
    Person p;//Invokes Person Default constructor, and Car Default constructor for both Car attributes
    Car new_car; //Invokes car Default constructor
    Person p_1(new_car); //Invokes parametirsed constructor for Person, and move constructor for the first Car attribute and the Default constructor for the second
    Car new_car_1;
    Person p_2(new_car,new_car_1); //Invokes move constructor for the first Car attribute and the Default constructor for the second

    p.setCar(new_car_1);//Invokes copy constructor for

    return 0;
}

```

A move constructor was introduced to avoid the overhead of another object instantiation by *"moving"* the contents out of the temporary value and into the new receiving value.

The move constructor must:
1. Take an r-value reference as the argument
2. Must transfer the relevant resources from the source to the receiver
3. Must leave the source object in a state where it can be quickly and correctly destroyed

In the above example, we see how an object can either be moved or copied depending on what we pass to the constructor (note that ```std::move(<object>)``` takes the object and returns an r-value for it - **thats all**. It does no moving. Therefore by ```Car new_car(std::move(old_car)))``` we are invoking the move constructor.

Consider the case where we have a car generator which returns a car:

```c++

Car generateCar(){
  Car generated_car(/*some values to generate a car*/)
  return generated_car;
}

```

Here we have created a car object within the method, which is then passed to a constructor in order to copy/move the values over:

```c++

//In Person class, modifying the default constructor to now assign a value to car, using random_car as a intermediary variable just to be able to use the move constructor
Person(){
  Car random_car(Car::generateCar());
  car = random_car;
}

```

This results in 3 car objects, being made **inside the person constructor**. One is made and returned in ```generateCar()```, one to copy the return value of ```generateCar()``` to the random_car object (ie one is made when copying the generated car to the formal parameter of the copy constructor), and finally one when copying the generated car to the formal parameter of the copy assignment operator.

Now, given the following code:

```c++

//In Person class, modifying the default constructor to now assign a value to car, using random_car as a intermediary variable just to be able to use the move constructor
Person(){
  Car random_car(std::move(Car::generateCar()));
  car = std::move(random_car);
}

```

Instead of three objects, we only make one. in ```Car::generateCar()```, which is then returned. That same object is then passed to the move constructor of Car, which moves its resources over to ```random_car``` without creating a new object. Then, once again, that same object is then passed to the move assignment operator of Car, again not making a new object.

We see thus that two temporary objects could be avoided.

> "When the compiler sees a value return statement or the existence of a temporary variable as described above, it will look for a move constructor. If one doesn't exist it will use an expensive copy construction instead." - Course Notes

**Things to note:**
1. The r-value reference is not const - as its object will be changed (its values deleted)
2. In the vector class (and presumably other STL containers), the following code:

```c++

std::vector<int> v = {1,2,3,4}, w;
w = std::move(v);

```

"Moves" the data from v into w, leaving v empty.

>"No deep copy takes place: state and pointers to buffers etc are simply "moved" across!" - Course Notes

3. In the case of having a move constructor for person, we must call ```std::move(old_person.car)``` in the constructor, but for simple types such as an int for age, we have ```age = old_person.age; old_person.age =0``` and if we had a pointer called arr_ptr for, lets say an array of doubles, we would have ```arr_ptr = old_person.arr_ptr; old_person.arr_ptr =0```. So we copy and zero the old values, using ```<object_name>.<field_name>``` to access the values bc references.

#### Copy Assignment Operator

#### Move Assignment Operator
