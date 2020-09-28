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
  * https://github.com/janwilmans/proposals

### Reply To: 
  * Jan Wilmans <janwilmans@gmail.com>

# Abstract

Add reserving constructors to containers to allow the user to express size requirements at construction.

# Revision History

## Revision 0

Initial version

## Motivation

Standard library containers have an initial memory allocation strategy, common practice seems to be to do no heap allocation until elements are add that need it. 
The current practice is that containers have to be constructed first and a then have space reserved by called the .reserve() method. Also some container do not have a reserve method.

* todo, explore std::string 
* todo, explore std::vector<T> 
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

## Overview

## Consistency

discuss consistency over containers and stack/heap allocation.

# Bikeshedding

This section is for naming, conventions and pinning down details to make it suitable for the standard.

## Alternatives

```
std::string s(std::reserve_capacity(1000));
std::vector v(std::reserve_capacity(1000));

```



