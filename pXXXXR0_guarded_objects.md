# pXXXXR0

guarded objects - express the combination of a class and its locking mechanism

Draft for Proposal, 6 September 2024

### Authors:

 - Jan Wilmans
 
 ### Audience:
 
  - Library Evolution Working Group (Design & Target)
  - Library Working Group (Wording and Consistency)

### Project:
  - ISO JTC1/SC22/WG21: Programming Language C++
  
### Current version:
  - https://github.com/janwilmans/proposals/blob/master/pXXXXR0_guarded_objects.md

### Reply To: 
  * Jan Wilmans <janwilmans@gmail.com>

# Abstract

In multithreaded programming locking mechanisms are used to prevent concurrent access to data. Common practice to create a locking mechansim, lets say an std::mutex along side the data it is protecting.
However, the relationship is only implied and expressed in code only by 'doing it right' in all places. This proposal intents to provide a way to clearly express this relationship are make it impossible to access data without locking its associated guarding mechanism.

# Revision History

## Revision 0

Initial version

## Motivation



# Design Considerations

## Synopsis

An example of what the result could look like:

```
std::guared<std::string> guarded_string;
std::guared<std::string, user_defined_lock> guarded_string; // user_defined_lock has a .lock()/.unlock() interface

```


## Overview

The goal of this proposal is ...
1) to express the relationship 
2) to make it impossible to access T without locking its guarding mechanism



## Consistency

discuss locking interface

# Bikeshedding

This section is for naming, conventions and pinning down details to make it suitable for the standard.


# Acknowledgements

- this proposal is based on the the work of Ansel Sermersheim and his https://github.com/copperspice/cs_libguarded library


# References

- https://github.com/copperspice/cs_libguarded






