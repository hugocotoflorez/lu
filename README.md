# Lu Specification

## About
Lu is a language designed to be used in microcontrollers and stuff like that
where it is more important the control than the security. It looks like C
because I love C but I want to be a bit closer to the device and have some
abstractions.

## disclaimer
As you can see nowadays lu is just a bunch of rules and ideas. I plan to have
a beta version in September 2025.

## Using libc
It can. (todo: specification)

## types

- `byte` -> 1 byte (== raw1)
- `rawN` -> N bytes
- `addr` -> mem address
- `none` -> as c void
- `word` -> cpu word size variable
- `int` -> signed non decimal num
- `uint` -> unsigned int

- `type` -> a type

- `da` -> dynamic array
- `hm` -> hash map
- `ll` -> double linked list

- `str` -> text string


## Keywords
### global / export
If a function is declared as global / export (I have to decide the name) it can
be used from other files that include the file in which it is declared.
Otherwise it is only visible for the files in the same file and its name can be
used in other non-global function from other files.

```c
global none foo() {};
export none foo() {};
```

### zero
Set var-size bytes to zero starting at var-address.
```c
raw a[10];
zero a;
raw b[10] = zero;
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

## Some strange behaviour
if a {,} pair ends with a statement without ; it returns its value.

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
str is a pointer to a str-struct at the offset there data starts. It works as a dynamic array. The data ends with a null
terminated character to be compatible with C. The str struct has some methods related to strings.
```c
struct str_struct {
        uint capacity;
        uint length;
        char data;
    };

type str = str_struct + offset(str_struct.data);
```
