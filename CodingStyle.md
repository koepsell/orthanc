

# Introduction #

The developers of Orthanc should follow these [C++ Programming Style Guidelines](http://geosoft.no/development/cppstyle.html), that are similar to the so-called ["BSD/Allman style"](https://en.wikipedia.org/wiki/Indent_style#Allman_style), with some adaptations that are described below.

A compliant Eclipse formatter [is available in the Orthanc distribution](https://code.google.com/p/orthanc/source/browse/Resources/EclipseCodingStyle.xml).

# Licensing #

Do not forget to include licensing information (GPLv3 with OpenSSL exception) in each `.cpp` and `.h`.

# Tabulations #

No tab characters. Replace 1 tab by 2 spaces.

# Strengthened Rules #

  * Rule 31: Use COLOR\_RED instead of Color::RED
  * Rule 34: Use the suffix .cpp
  * Rule 35: A single header file must contain a single public class
  * Rule 72: Use the Example 2 style (aka. Allman style, used by MSDN and Visual Studio):

```
while (!done)
{
  doSomething();
  done = moreToDo();
}
```

# Replaced Rules #

  * Rule 6: The names of the methods are camel-case to move the coding style closer to that of the .NET framework.
  * Rule 36:
    * One-liners are always ok in a `.h`,
    * High-performance code is also allowed but only with the `inline` keyword (the code being moved at the end of the header)
  * Rule 40: Use `#pragma once` in each header file ([Wikipedia](http://en.wikipedia.org/wiki/Pragma_once))
  * Rules 73 and 80: Use Visual Studio's default style that does not add two whitespaces in front of public, protected, private and case:

```
class SomeClass : public BaseClass
{
public:
  ...
protected:
  ...
private:
  ...
};
```

# Additional Rules #

  * Use C++ exceptions, avoid error codes.
  * Use [RAII](http://en.wikipedia.org/wiki/RAII) wherever possible.
  * No C-style casting, use `static_cast`, `reinterpret_cast`, `dynamic_cast` and `const_cast`.
  * Never use `using namespace` in header files (except inside `inline` implementations).
  * Complement to rule 20: `Finalize` is the complementary word to `Initialize`.
  * Minimize the number of `#include` in header files.
  * Never use `catch (...)`.
  * To ease unit testing, favor the [Java Beans](http://en.wikipedia.org/wiki/Java_beans) style:
    * Single constructor without argument,
    * Use getters/setters.