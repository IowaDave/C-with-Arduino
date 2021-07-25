# C-with-Arduino
How to use straight C code with Arduino IDE

Arduino IDE version 1.x anticipates code written in C++. It will compile code written in plain C, also. 

Most of C++ uses the same syntax as C, so it might look like you are writing C in the Arduino IDE. Yet, the compiler will treat it as C++.

You must tell the compiler to treat your code as C. Do this by putting the C code into a separate "tab" (Arduino IDE's terminology for a code module), naming the tab with the ".c" suffix. Example tab name: "my_module.c".

```
// my_module.c

int my_variable = 3;

int my_function() {
  return 7;
}
```

Some of the variables or functions in a code module might be intended for use by code written in other modules. The C++ way to do that is to create a header file with the same name as that of the code module, but ending with ".h". In this example, such a header would be named "my_module.h".

```
// my_module.h

// use 'extern' to expose variable declarations to code in other modules
extern int my_variable;

// use prototypes to expose functions to code in other modules
int my_function();
```
Alas, the C++ approach illustrated above will fail when the code module is named ".c".

### Use .cpp modules

The easiest way to avoid the problems described below is to write code in C++.

* The main ".ino" sketch file will be compiled as C++ because that's how the IDE works.
* Append the suffix ".cpp" to the tab names for other code modules in the project.

Seriously. Changing the name of the code module shown above, to "my_module.cpp", is all it would take to make it work.

The rest of this note addresses problems that I've encountered when using C modules in Arduino IDE projects.

### Don't try to use C++ objects in a C module. 

This explains why ```Serial.println("Hello");``` will throw an error when used in a C module.  The Serial.xxx functions belong to a C++ object named Serial, as defined in the Arduino.h library. 

The compiler will complain that it cannot locate the object in the context of your ".c" module, even when you explicitly include the library that defines the object. 

For example, Serial is defined in the "Arduino.h" library. However, placing ```#include "Arduino.h"``` in a ".c" module will not enable code in that module to use Serial functions.

### Special syntax for exposing external variables and functions

Problem: you want to expose externally a variable or a function, defined in a C module, for use in the main sketch. The C way to do this is different from the C++ way.

#### Question: what does it mean to "define" a variable or a function?

The Official Answer: it means to allocate memory to the variable or to the function.

My answer: a variable becomes *defined* at the first place in the code where a value gets associated with that variable; for a function it means the place where the function's code is written out between {curly brackets}.  

Examples:

This *declares* an integer variable but does not *define* it: 

```int my_variable; // not a definition because no value gets assigned here```

The following *defines* that variable:

```my_variable = 3; // allocate memory to contain the value, 3```

Variables have to be declared before they can be defined. YOu can do both at the same time. The following example both declares and then defines a variable with an initial value

```int my_variable = 3;```

Likewise, for functions, the following example is a prototype

``` void my_function(); // no curly brackets here```

Whereas, the example below defines a function:

``` 
void my_function() {
       // containing some code inside curly brackets
    }
```
### OK, back to the main topic

Expose variables defined in ".cpp" modules by putting an "exern" declaration in the header (".h") file for that module. Likewise, for a function, the header file contains a prototype declaration.  This won't work for variables and functions defined in a ".c" module.

There are four other ways to make it work.  We are grateful to an Arduino Forum contributor named "merlin13" for explaining some of the following solutions. See: [https://forum.arduino.cc/t/how-to-use-additional-c-files-in-a-sketch/41402/2](https://forum.arduino.cc/t/how-to-use-additional-c-files-in-a-sketch/41402/2).

#### Did I mention using .cpp modules?

Just change the name of the ".c" module to ".cpp", for example "my_function.cpp", and presto! function prototypes and "extern" go back to work.

#### Tell the Linker explicitly about each C variable or function

When the code module name ends with ".c", the ```#include``` directive does not work as it does for ".cpp" code modules sharing the same hame. Don't use it. In lieu -- and in place -- of using ```#include "my_module.h"``` in the main sketch, say this about each thing you want to link from "my_module.c".

```
extern "C" int my_variable;
extern "C" int my_function();
```

#### Do it the really clumsy way

(I never got this to work, but some self-disclosed experts swear by it.)

```
#if defined(__cplusplus)
extern "C"
{
#endif
//     put here the things to be exposed externally
#if defined(__cplusplus)
}
#endif
```

#### Or, my idea, a special C_Includes header

This, for the main sketch
```
// place this line in the main sketch
#include "c_externals.h"
```

and these, in a module (tab) named "c_externals.h"
```
extern "C" int my_variable;
extern "C" int my_function();
```

It worked for me. I appreciate two things about this approach:

1. the tidiness of keeping all those funky ```extern "C"``` lines out of sight in a header of their own, 
2. then being able to include that in the normal (c++ style) way.

To be honest, I wrote this note to remind me of reasons to avoid using straight C code in an Arduino IDE project. The IDE is a C++ environment. I should simply remember that and write code compatible with C++. Most of which looks and works exactly like C. Where C++ differs from C, choose to embrace the difference and turn it to my good.

