# pXXXXR0

guarded objects - make the relationship of class and its locking mechanism explicitly expressable

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

In multithreaded programming locking mechanisms are used to prevent concurrent access to data. Common practice is to create a locking mechansim, lets say an std::mutex along side the data it is protecting.
However, the relationship is only implied and expressed in code only by naming variables and/or 'doing it right' in all places. This proposal intents to provide a way to clearly express the relationship and make it impossible to access the data without locking its associated guarding mechanism.

# Revision History

## Revision 0

Initial version

## Motivation

## Encapsulation of locking logic

The `guarded` type encapsulates the locking mechanism, ensuring that locks are acquired and released properly when accessing the protected data. This reduces the chances of errors such as forgetting to unlock a mutex or accidentally leaving a shared resource in an inconsistent state due to race conditions.

## Simplified API

With a type that directly ties the data and its locking mechanism, the API for interacting with the protected data becomes clearer. Users don’t have to worry about manually locking and unlocking. Instead, the locking and unlocking can be handled internally within the type. This simplifies the API and reduces the cognitive load.

## Strong ownership semantics / RAII guarantees

By returning a handle to the locked object (see futher down for examples) we make the ownership and the access pattern explicit. 
The locking mechanism is based on RAII (Resource Acquisition Is Initialization), the type can automatically handle acquiring and releasing the lock in a scoped manner. This provides automatic and exception-safe locking.

## Separation of concerns

The guarded type eliminates the need to embed thread-safety directly into the design of a class. Such implementations are pessimizing single-threaded use and have locking overhead on every API call. For demonstration purposes, here is a naive implementation of a 'guarding' class still having the locking overhead on every API call:

```
// Naive example without strong ownership on the API and still having locking overhead

#include <mutex>
#include <string>
#include <iostream>

template<typename T>
class naive_guarded {
private:
    T data;
    mutable std::mutex mtx;

public:
    template<typename Func>
    auto with_lock(Func&& func) {
        std::lock_guard<std::mutex> lock(mtx);
        return func(data);  // Pass the data to the provided function.
    }
};

int main() {
    naive_guarded<std::string> naive_guarded_string;
    
    // Modify the string safely.
    naive_guarded_string.with_lock([](std::string& str) {     // locking overhead
        str = "Hello, World!";
    });
    
    // Read the string safely.
    naive_guarded_string.with_lock([](const std::string& str) { // locking overhead
        std::cout << str << '\n';
    });

    return 0;
}
```

[compiler explorer link](https://cppcoach.godbolt.org/z/4Evees8nd)

## Prevents accidental access without locking

By requiring access to the data through the guarded type, you make it harder (or impossible) to accidentally access the data without properly locking it first. Without such a type, a programmer might forget to lock the corresponding mutex, leading to potential data races and undefined behavior.

## Easier to reason about the code / better readability

When locking is tightly coupled with the data it protects, it becomes easier to reason about the behavior of the code. A guarded<std::string> makes it clear that access to the std::string is controlled and synchronized. 
There is also no question about what the lock is protecting, this makes the code more understandble and maintainable. The guarded type abstracts away the details of how the concurrency mechanisms are implemented. Users of the type do not need to worry about whether the data is protected by a mutex, spinlock, or some other concurrency primitive; they just know the data access is synchronized properly.

# Design considerations

## Synopsis

An example of what the result could look like:

```
#include <string>
#include <iostream>

#include "https://raw.githubusercontent.com/copperspice/cs_libguarded/master/src/cs_plain_guarded.h"

int main() {
    libguarded::plain_guarded<std::string> guarded_string;
    
    auto accessor = guarded_string.lock();

    // Modify the string safely.
    *accessor = "Hello, World!";
    
    // Read the string safely.
    std::cout << *accessor << '\n';
    return 0;
}
```

[compiler explorer link](https://cppcoach.godbolt.org/z/PTv1MG4oq)

Alternatively:

```
std::guared<std::string> guarded_string;
std::guared_lock<std::string, std::mutex> guarded_string; // user_defined_lock has a .lock()/.unlock() interface
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
- ([thread.req.lockable])[https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/n4950.pdf] Cpp17BasicLockable requirements 
