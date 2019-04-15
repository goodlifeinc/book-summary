#JVM

The Java Virtual Machine is an **abstract computing machine**. Like a real computing machine, it has an instruction set and manipulates various memory areas at run time.

The Java Virtual Machine **knows nothing of the Java programming language**, only of a particular binary format, the class file format. A class file contains Java Virtual Machine instructions (or bytecodes) and a symbol table, as well as other ancillary information

Compiled code to be executed by the Java Virtual Machine is represented **using a hardware- and operating system-independent binary format**, typically (but not necessarily) stored in a file, known as the class file format.

Java Virtual Machine operates on two kinds of types: **primitive types and reference types.**

The JVM expects that nearly all type checking is done prior to run time, typically by a compiler, and does not have to be done by the JVM itself.

The Java Virtual Machine contains explicit support for objects. An object is either a dynamically allocated class instance or an array.