# dbj++nanolib&trade; 
&copy; 2019 by dbj.org

Very opinionated and very small ISO C++ library. Testing, concepts, mechanisms. 

dbj nano lib, is not portable. It is developed on Windows 10 PRO, using Visual Studio and Windows SDK, the latest.

## The Purpose & Motivation
However small they might be, C++ programs should always promote good design, good implementation and high degree of reusability.

Also, good coding solutions should not "get in the way". They should be small and transparent and comfortable to use, beside being resilient.

## Modern C++

I am actively developing in C++ since 1992. C++ is a powerful beast. One is never sure is it tamed. Never turn your back to C++. The weeds of over engineering will very quickly stop your project and drag it to the bottom of the sea-of-redundant code.

Part of the required self discipline is to constantly develop and use mature core concepts one has proven to work.  

#### But, why not just using `std::` lib? 

I do indeed. But my key concepts are not over there. I belong to the users of C++ and inevitably the std:: lib. Part of my responsibilities is to make sure organizations are adhering to rigour and method. Not just in Architecture (my primary role), but in developing , testing and delivering, what is architected.

But in doing so I know people are good and creative, thus I do not want to impose a lot of discipline, rigour and method. Thus my coding solutions tend to be more of an agreement than complex code. 

Modern, fancy keyword is "contract". `dbj++ Nanolib` is primarily an tiny universal contract, I might advise you to impose on your projects.

### Top Level Library Organization

Diagram of the top level dependencies with some descriptions.

![Current dbj++nanolib](https://yuml.me/0b59568c.jpg)

Please keep in mind however, the prefix "nano" (aka tiny). In future sequence of releases dbj++ Nano lib will grow. Hopefully not that match.

### How to use

`dbj++nanolib` is C++17, Windows only library. It uses always the latest Visual Studio reincarnation. This is header only library.

As per diagram above, one can include `dbj++nanolib.h` for the core functionality.

Or just `dbj++tu.h` for the testing framework, or just `dbj++valstat` to start using `valstat` and two valstat traits from there, to achieve consistent but simple, returns consuming.

**Slight detour: Subset of UML**

Whatever you do, always plan and think about it beforehand. Use simple diagrams before deciding your design is complete and you can freely proceed onto the implementation.

Designs and diagrams are invaluable. First for explaining to yourself your solution, and second to make others understand your solution, before you or them, loose the patience `¯\_(ツ)_/¯`

## What's inside

I see no much point of generating documentation from code, if code is available and properly commented. I tend to write a lot of comments.

Please just dive in and enjoy. At last this is made for you, not for someone needing a book to understand how to use the modern C++ library.

## what is 'dbj++nano play'

That is a twin project. We use it to test the quality of implementation and usability of the nanolib.

Newcomers to `dbj++nanolib` should definitely start from that, as the entry point. It will contain more and more test to show how are the things from here to be used.

Next I will just list and quickly describe, in random order, important things in the Nanolib. As per your demand this documentation will be more detailed as `dbj++nanolib` spirals through time.

Do not forget: code should be simple and headers manageable.

## dbj++nanolib.h

The library core header. Namespace is:  `dbj::nanolib`.

#### configuration

`#define DBJ_TERMINATE_ON_BAD_ALLOC` in order yor project, does not to throw `bad_alloc`, on heap memory exhausted. Instead, default termination will be called.
```cpp
  (std::set_new_handler([]{terminate();}););
```
For safe testing experience, `#define DBJ_SYNC_WITH_STDIO` in case of mixing `stdio.h` and `iostreams`.
```cpp
			ios_base::sync_with_stdio(true);
```
We use `iostreams`, only when testing. `dbj++tu.h` contains a tiny testing framework.

#### Unavoidable Macros

- macros are named starting with `DBJ...` or `_DBJ...`
 - `DBJ_REPEAT(N)` is interesting, there is no "do something N times" C++ keyword
 - the namespace is `dbj::nanolib`
  - most peculiar little thingy in there is perhaps `v_buffer`, because we do not want to use `std::string` as buffer, in core libraries (see Appendix A)
   - we use exclusively `std::vector<char>` as the char buffer in dbj++nanolib
   - `v_buffer` is actually a trait made to provide the required functionality for handling the `v_buffer::buffer_type`
  - Another peculiarity worth mentioning is we never use `printf`/`fprintf` naked but through  two variadic macros
    - DBJ_FPRINTF(...)
    - DBJ_PRINT(...)
    - Primarily because debug build versions do contain checking the returns of `std::fprintf`. If you have  been (and I was) ever beaten by not checking `stdio.h` functions return values, you will understand.  Invaluable even if it works only once for you.

## dbj++tu.h

Fully functional Testing framework. namespace `dbj::tu`

It works as one might expect C++ Testing Frameworks to work. You register the "Test Unit". It gets collected at compile time and executed at runtime.

Advisable to start understanding it, is to look at the bottom of the `dbj++valstat` header where one test resides. This is where you want to put your Visual Studio break point and hit F5, while in the [nano play project](https://github.com/dbj-systems/dbj--nanoplay).
```cpp
TU_REGISTER(
	[] {
    // the rest of the code
  });
```

Macro `DBJ_TX` is indeed using `iostreams`. That is ok in the context of testing apps. Makes for simpler and shorter testing macro. 

Lastly, to run the test units collected, on has to call, somewhere from main():

`dbj::tu::catalog.execute()`

As expected for every decent testing framework, the coloured console output pops-up, and every TU is timed too.

![tu in action](docs/dbj_tu_in_action.jpg)

# Appendix A
## Key concepts

This is the advice, I am advising you to respect.

### No sub-classing
- inheritance in C++ is used for
  - [Sub-Typing](https://en.wikipedia.org/wiki/Subtyping)
  - Sub-Classing
    - Sub-Classing is evil
- [Polymorphism does not require inheritance](https://en.wikipedia.org/wiki/Composition_over_inheritance)
  - Sub-classing inside the c++ std lib is very rare

### Optimise the std::string usage

- `std::string` is very versatile, but it is **not** made to be used as char buffer. 
  - it makes for one complex type and  large memory structure, too complex to act as a simple char buffer
  - example: `std::string` operator '+' is notoriously slow, thus people try and use the `append()` method, complicating the code.
  - it is more elegant and faster to do `sprintf` into the buffer, instead of using `std::string` append
  ```cpp
  // using std::string to compose a new string
  // need to use append() to avoid '+'
  std::string new_val(  reader->kv_map_[key].data()  );
  new_val.append("\n").append(value);

  // instead dbj nano lib offers an elegant replacement
  	buffer_type new_val = buffer::format( "%s%c%s",
		reader->kv_map_[key].data() , '\n', value
	);
  ```
  Unless you are developing large and complex text processing, be very modest with `std::string`.
### No throwing and catching

First, if your executable, is intended to operate under continuous uptime requirement, it should never fail and never exhibit undefined behavior. Thus, the error handling is of utmost importance. It has to be very resilient, simple, explicit and unforgiving. Some advice.

- throwing and catching do create slow applications, with many hidden paths of execution
- the raising of the error and handling the error raised, should be tightly coupled and at the same place in the source code. Make it easy to understand what happens if error happens.
  - if you need to throw an exception from a constructor, your design is probably wrong.
  - if your constructor is complex and might be source of operational error's consider factory functions or friends, whatever you fancy most.
  - however silly that looks to a C++ developer consider removing tricky destructors into static or friend functions to be called by some external deterministic mechanism, similar to the lock unlock pattern.

### Avoid system error
- `std::system_error` is old, over-engineered design and somewhat poor implementation
- `std::system_error` was not designed to be universal error mechanism for C++ `std::` lib. Neither it was ever adopted or implemented, as such.
- it is not a replacement but consider using a subset: `std::errc` with `std::generic_category()`
- also, win32 errors are well implemented by MSVC and `std::system_category()`

### Avoid c++ streams
- C++ streams is old, complex design too. 
- using `iostreams` creates large and slow applications
- if you need to do a lot of console output consider using the [fmt library](https://github.com/fmtlib/fmt).
     - Although, I am personally puzzled how is `fmt` better than `cstdio` functions

### Standards
- use ISO C++, which is C++17 as of 2019 Q4.
- use ISO C++ std lib as much as possible
- Do not lose your C experience. 
  - C is simple and makes you deliver things, instead of re-inventing things before delivering things. 
   - take same time to learn about [modern ISO C](https://gforge.inria.fr/frs/?group_id=6881). It has advanced. A lot.

The final advice:

**This is "nano" lib. Before adding anything to it, think.**

## Contact

In case you hotly dislike or just like my work, or anything in between, for what you might find interesting in here, please do write me:

[dbj@dbj.org](mailto:dbj@dbj.org)

## License

### CC BY-SA 4.0 -- https://creativecommons.org/licenses/by-sa/4.0/ 

---
[![dbjdbj github avatar](https://github.com/dbjdbj.png)
 &copy; 2019](https://dbj.netlify.com)


