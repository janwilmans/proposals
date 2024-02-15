# pXXXXR0
reserving_constructors - express an initial allocation size for standard containers
Draft Proposal, 28 September 2020

### Authors:
 * Jan Wilmans
 * Nevin Liber
 
 ### Audience:
  * Library Evolution Working Group (Design & Target)
  * Library Working Group (Wording and Consistency)

### Project:
  * ISO JTC1/SC22/WG21: Programming Language C++
  
### Current version:
  * https://github.com/janwilmans/proposals/blob/master/pXXXXR0_reserving_constructors.md

### Reply To: 
  * Jan Wilmans <janwilmans@gmail.com>

# Abstract

Add reserving constructors to containers to allow the user to express size requirements at construction.

# Revision History

## Revision 0

Initial version

## Motivation

Standard library containers have an initial memory allocation strategy, common practice seems to be, to do no allocation until elements are added that need it. 
The current practice is that containers have to be constructed first and then have space reserved by calling the .reserve() method. Not all containers have a reserve method;  std::map, std::set, std::deque, std::stack for example do not, while std::string, std::vector and std::unordered_map do have it.

The reserving constructor comes with new optimization oppertunities, for example `std::string s(std::reserve, 64);` could decide to create a bigger SOO (small-object optimization) buffer, avoiding extra allocations. This is particularly interesting for locality of small containers such as std::string or std::function<T> that currenly have no way to specify this. 
 
* todo, explore std::string, stack vs. heap allocation
* todo, explore std::stack
* todo, explore std::deque
* todo, explore std::function<T>, stack vs. heap allocation

# Design Considerations

## Synopsis

An example of what the result could look like:

```
struct reserve_t { constexpr explicit reserve_t() = default; };
inline constexpr reserve_t reserve;

std::string s(std::reserve, 1000);
std::vector<int> v(std::reserve, 1000);
```

The std::reserve syntax here is to disallow `std::string s({}, 1000);`

## Overview

The goal of this proposal is twofold, 
1) to be able to specific the capacity to reserve on container construction 
2) to give some control over small-object optimizations

To achieve the second goal the specified reserved capacity shouldn't be merely a suggestion.

## Consistency

discuss consistency over containers and stack/heap allocation.

# Bikeshedding

This section is for naming, conventions and pinning down details to make it suitable for the standard.

## Alternatives

an alternative syntax:

```
std::string s(std::reserve_capacity(1000));
std::vector v(std::reserve_capacity(1000));
```

alternative approach 1); if we write:

```
namespace nonstd
{
    auto reserve = [](std::size_t size){
        std::string result;
        result.reserve(size);
        return result;
    };
}

std::string str(nonstd::reserve(1000));
``` 
could the above be optimized to effectively do what we want in terms for memory capacity reservation? It certainly does nothing to achieve the second goal.

alternative approach 2)

```
std::string s(std::reserve_stack, 1000);
std::vector<int> v(std::reserve_heap, 1000);
```

This wording is not right since the C++ abstract machine does not define a stack, but the idea here is that 'std::reserve_stack' would create a 1000-byte small-object buffer and that `std::reserve_heap` would dispense with the small-object buffer entirely and do the heap allocation immediately. Problem with this approach is, the choise for 'no small-object buffer' is a compile time decision, so it probably needs to be a template argument.

# Acknowledgements

* this proposal came out of this brief twitter thread: https://twitter.com/janwilmans/status/1308175343824580608
* examples taken out of context from Mustafa Kemal GILOR's stackoverflow post

# References

* https://stackoverflow.com/questions/32738879/why-is-there-no-reserving-constructor-for-stdstring






