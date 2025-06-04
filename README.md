<div style="display: flex; gap: 0px;">
<img src="./logo-light.png" alt="logo-light" width=49%>
<img src="./logo-dark.png" alt="logo-dark" width=49%>
</div>

> The limits of my language are the limits of my word.
> --- Ludwig Wittgenstein

# Lu Specification

## About
Lu is a language designed to be used in microcontrollers and stuff like that
where it is more important the control than the security. It looks like C
because I love C but I want to be a bit closer to the device and have some
abstractions.

## disclaimer
As you can see nowadays lu is just a bunch of rules and ideas. I plan to have
a beta version in September 2025.

## functions
A function is on the form of `type name ( arg1, arg2, arg3,...) { body }`.
name have to follow `[a-zA-Z_][a-zA-Z_0-9]*` format. The space between name
and parenthesis is not neded. The arguments are in the form `type name [=
default]`. The `...` points is to say that more arguements can be provided. If
a unknown number of arguemts can be provided use arrays or some data structure
that fits better. The function body have its own scope. Main function have to
return int so the type can be not provided.

```c
none say_hello(str name) {
    printf("Hello, " + name);
}
```

## types

- `byte` -> 1 byte (== raw1)
- `rawN` -> N bytes
- `ptr` -> mem address
- `none` -> as c void
- `word` -> cpu word size variable ( I think its a bad idea because of portability)
- `int` -> signed non decimal num
- `uint` -> unsigned int

- struct -> one or more variables packed
- enum -> enum

- `da` -> dynamic array
- `hm` -> hash map
- `ll` -> double linked list

- `str` -> da of chars

There is no floating point type. Use bigger numbers

```c
byte a = 0b10;
raw1 b = 0xFF;
ptr c = &a; // address of a
int d = -1023;
uint e = 1023;

da<int> f = [a, b, e]; ??? just use slices
hm<int> g = {"house":1, "me":10};
ll<str> h = {"hello", "world"};

struct i{
    int a = 10; // default value on initialization
    zero byte b; // zero init field by default
    byte c;
}

ptr j = &i;
i.a == j.a;
```

## comments
Just as int C I think.

## Keywords

### export
If a function is declared as export it can be used from other files thatinclude
the file in which it is declared. Otherwise it is only visible for thefiles in
the same file and its name can be used in other non-global functionfrom other
files.

```c
export none foo() {};
```

### zero
Set var-size bytes to zero starting at var-address.
```c
raw a[10];
zero a;
zero raw b[10];
```

### defer
Store the statement in a LIFO queue and execute it when the end of the scope
in which the defer is set is reached.
```c
{ <- start of the scope
    defer print("Hello");
    zero ptr a;
    alloc(a, 10)?;
    defer free(a);
    ...
    <- free(a) is called
    <- print("Hello") is called
} <- end of the scope

```

### self

```c
struct A {
    str name;
    str get_name(self a) = { a.name };
    none set_name(self a, str name) = {self.name = name; };
}
```

if a function inside a struct has a self parameter, outside the declaration
    this argument must not be provided, instead it automatically is set to the
    struct (or to a pinter to the struct).

```c
A a;
a.foo(10) ---> foo(a, 10)
```

### todo, panik, unreachable, abort
Those functions terminate the program execution with a diagnostic of where it
is called and a message from the user.
```c
panik("Some error");
// $ path/file:line: PANIK: Some error
```

## Some strange behaviour
if a {,} pair ends with a statement without ; it returns its value.

## Loops

- while(condition) {}: Execute statement until condition is false
- for(variable :: iterator) {}: Execute statement once per element in iterator
- for(*c like*) {}: Execute statement as c for loop
- do {} while(condition); : Execute statement once and until condition is false
- loop {}: Execute until a manual break.

### range

`for (i :: S ... E .. T)){}` evaluates the range from start S to end E with
step T. `.. T` can be ignored, the default step is `1`.

## Loop labels
In the case where two o more loops are nested, it would be needed to break
other different than the last loop. It can be solved using loop labels. also
work for continue.

```c
f: for (){
    for(){
        break f;
    }
}
```

## STR
`TODO: just a array, dynaically allocated if defined as dynamic string or
something like that`

str is a pointer to a str-struct at the offset there data starts. It works as a
dynamic array. The data ends with a null terminated character to be compatible
with C. The str struct has some methods related to strings.
```c
struct str_struct {
        uint capacity;
        uint count;
        char data;
    };

type str = str_struct + offset(str_struct.data);
```


## Iterators

`TODO: I think it is not a good idea`

Every struct with a `T current` field and a `T2 next(self)` funtion can be
used as an iterator. `current` can be of any type and have to store the needed
info to calculate next value. `next` have to return the next element and
update `current` until `null` is returned.

```c
struct array<T> {
    zero int current;
    T *next(self) = { current++ }
    zero int size, capacity;
    zero data;
}

array A;
for (a :: A) {
    do something
}

```


## External modules

### import
import a lu file. All export functions and definitions are avaliable.

### extern
say the compiler that a function or definition with a given schema would exist.

### link
Link the program with a runtime library

```c
link glibc;
extern printf(const char*, ...);
```

#### Given libc prototypes
There is a section that can be included that provide external definitions for
libc functions and automatically link with libc. Note that it can only be used
in environments where libc is avaliable.

```c
import libc.stdio;
```

## Operators
### <> Diamond operator
Diamond operator: `<>` is used to swap the values of two variables of the same
type or compatible ones.
```c
int a = 10;
int b = 5;
a <> b; // a is 5 and b is 10
```

## Thread-safe blocks
To think about. I think its a good idea but thread_safe is a name that I dont
like.
```c
none shared_method(){
    thread_safe{
        some_var+=1;
    }
}
```

## Slices
A pointer and a size. Array are slices. A slice that does not point to the
start of an array represents a part of the array.
- Syntaxis: (ptr)\[offset, elems]. ptr can be another slice.

```c
int[10] a;
int[] b = a[1,8]; // b is constant size, determined at compile time
```

The size of a slice is constant by default. If it is set as dynamic it can
grow, at the cost that is allocated in the heap. It would work as an dynamic array.

```c
int[-] a; // set as dynamic
a[100] = 1; // now a has size > 100
```
I have 3 options, `int[<some char>]`, `int[dynamic]`, `dynamic int[]` I have to
think about what I like more.

## VA ARGS

I think it would be possible to pass vaargs as an slice of pointer, so it can
be passed recursively as an array but if accessing a slice position it automatically
dereference it. It can be a generic-slice or something like that.

```c
int @a [] = {1, 'a', "hello", 0x92A};
```

`@x` is the address of variable `x`, but it can only be read by value. It can be
cast explicity. It can be used in function as `func(ptr @a)`, so a is a pointer
in the way that it can be read as `int i = a`. Idk if this is useful

