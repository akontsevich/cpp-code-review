# Best practices C++ code review recommendations
Possible C++ code review checklist.

## 1. The Rule of The Three

If a class implements one of the following 3 methods, then the class should implement all 3 of them - 

* Destructor 
* Copy constructor
* Copy assignment operator

**More:** http://www.geeksforgeeks.org/rule-of-three-in-cpp/

## 2. Do not use #define and macros unless you have to

Prefers `inline` for functions, `const` for variables and 'enum' for alias. For string, `const string` or `const char * const` instead of `#define`. 

Same for macros: they make readability, expandability, flexibility, maintenance, debugging significantly harder.

**More:** http://voidsid.blogspot.com/2007/04/prefer-const-and-inline-to-define.html

## 3. Iteration over STL containers 

For generic code, use range based `for loop` instead of the classical for loop. It is sorter and safer. For all other case, refrain yourself from using classical `for loop`. Instead use - 

```
for(const auto& element : elements)
```

**More:** https://stackoverflow.com/questions/36992260/comparing-different-types-of-c-for-loops

**Example: https://github.com/JustinSteventon/ct/pull/44/commits/c11bd165d9a168c2795fea43cf93264e8b15ba22

## 4. Try to use `const` member functions

All the member functions should be declared as `const` if the function don't modify any value of the object. Otherwise, it will be tough to work with `const reference to object / const object / const pointer to object` (function parameter for most of the case) as non-const member functions cant be called with `const reference to object / const object / const pointer to object`.

**More:** https://turbofuture.com/misc/C-Const-member-function-explained

## 5. Use smart pointers instead of new/delete
It is much safer and have better coding style to use smart pointers from C++11/14. Managing heap allocated objects manually is hard and error prone, with the common result that code leaks memory and is hard to maintain. Smart pointers heavily simplifies this by assigning stack-based memory ownership to heap allocations, more generally called resource acquisition is initialization(RAII). Smart pointer stores a pointer to a dynamically allocated object, and deletes it upon destruction which guarantees that the object pointed to will get deleted when the current scope disappears.

For pure C++ classes suggest  to use [std::unique_ptr](https://en.cppreference.com/w/cpp/memory/unique_ptr) and [std::shared_ptr](https://en.cppreference.com/w/cpp/memory/shared_ptr). And [QScopedPointer](https://doc.qt.io/qt-6/qscopedpointer.html) and [QSharedPointer](https://doc.qt.io/qt-6/qsharedpointer.html) for `QObject` classes.

## 6. Always use initializer list for user defined type for better performance

Initializing the user defined typed members of a class should be done using initializer list as it calls the copy constructor of that user defined type(no constructor is called). If we dont use initializer list and do the initializng in the constructor then first the default constructor of the user defined type will be called and then assignment operator overloaded function will be called. This will cause performance degradation.

**Remember:** initializer list order should be the same as declaration list order.

**Example:** https://github.com/swomack/cpp-tricks/blob/master/Initializer%20List/Initializer%20List/Initializer%20List/Source.cpp

## 7. Decompose big functions and long lines wrap
Function logic should be precepted in one view without scrolling: this improves readability and code review and maintenance. So big functions which do not fit in one screen should be decomposed. Same for long lines: should be wrapped in 80-90 characters to percept the logic without scrolling. Also word-wrapping is helpful on code review where we my accept original and modified sources in 2 columns (compare tool or PR review on Github) without scrolling as well.

## 8. Doxygen
Classes and member functions should be documented even for 1 person or small team and especially for an open source project where a lot of people may modify the code and everybody and even author when some time passes should understand or recall the classes and methods meanings. I suggest to do it with Doxygen or QDoc to automatically generate HTML/PDF documentation on the code.

## 9. Sources names
Sources names should correspond to the class name within the source. And only 1 class should be declared in such source. Source should represent 1 logical unit (class).

## 10. Unique code style across multiple developers
Install and setup `clang-format` tool for automatic and unified sources formatting in unified style for all developers. If specify `clang-format` config file in Qt Creator settings, it will automatically reformat all edited sources on every save on the fly - this saves a lot of time - not need in manual formatting. Config also should be committed to the main tree, attached to local and GIthub CI or even to a git trigger.

## 11. Automated code reviews

### Traditional Static Analysis Tools
Traditional tools focus on code style and common errors. They analyze the code without executing it, identifying issues such as:

  - **Syntax errors:** Basic mistakes that prevent code from compiling.
  - **Code smells:** Patterns that may indicate deeper problems, such as duplicated code or overly complex methods.
  - **Style violations:** Non-compliance with coding standards, which can affect readability and maintainability.

### Examples of popular static analysis tools include:

  - **Cppcheck:** A tool that detects bugs and undefined behavior in C++ code.
  - **Clang-Tidy:** A part of the LLVM project, it provides a rich set of checks and can automatically fix some issues.

You could enable and setup them in Qt Creator and/or in automated Github CI so you can be always sure one won't approves a PR which smells to main tree. Config files should be committed to the main tree, attached to local and GIthub CI or even to a git trigger.

## 12. Unit tests
Cover up to 80-90% of the code/classes by unit tests (Google Tests or Qt Test). Could be integrated into CI.

## 13. Address Sanitizer
AddressSanitizer (ASan) support could be integrated into Linux projects. You can enable ASan for both MSBuild-based Linux projects and CMake projects. It works on remote Linux systems and on Windows Subsystem for Linux (WSL).

ASan is a runtime memory error detector for C/C++ that catches the following errors:
- Use after free (dangling pointer reference)
- Heap buffer overflow
- Stack buffer overflow
- Use after return
- Use after scope
- Initialization order bugs

When ASan detects an error, it stops execution immediately. If you run an ASan-enabled program in the debugger, you see a message that describes the type of error, the memory address, and the location in the source file where the error occurred. You can also view the full ASan output (including where the corrupted memory was allocated/deallocated) in the Debug pane of the output window.

## 14.  Continuous integration (CI)
Organize CI for example with docker and embed it into Your Github repo. This will allow:
- automated builds (debug, release)
- automated static code analysis
- automated tests runs
- organize automated PR check in Github (cpp checks, format checks, tests checks, etc).
- make prebuilt packages (exe, deb, rpm, AppImage, apk, etc) and publish the on every release.
