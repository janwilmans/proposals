# pXXXXR0

guarded objects - make the relationship between objects and their locking mechanism explicitly expressable and hard to use incorrect

Draft for Proposal, 8 September 2024

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
However, the relationship is only implied and expressed in code only by naming variables and/or 'doing it right' in all places. This proposal intents to improved upon this by providing a way to clearly express the relationship and make it impossible to access the data without locking its associated guarding mechanism.

# Revision History

## Revision 0

Initial version

## Motivation

## Encapsulation of locking logic

The `guarded` type encapsulates the locking mechanism, ensuring that locks are acquired and released properly when accessing the protected data. 
By requiring access to the data through the guarded type, you make it harder (or impossible) to accidentally access the data without properly locking it first.

## Simplified API

With a type that directly ties the data and its locking mechanism, the API for interacting with the protected data becomes clearer. Users don’t have to worry about manually locking and unlocking. Instead, the locking and unlocking can be handled internally within the type. This simplifies the API and reduces the cognitive load.

## Strong ownership semantics / RAII guarantees

The only wat to access data is to call one of the locking functions, these functions an object (a unique_ptr-like object) that owns the lock.
It can be used in a familiar way, just like you would use a smart pointer.
The locking mechanism is based on RAII (Resource Acquisition Is Initialization), the type handles acquiring and releasing the lock in a scoped manner. This makes sure the lock is always released not matter how you leave the scope, returning of by an exception for example.

## Separation of concerns

The guarded type eliminates the need to embed thread-safety directly into the design of classes. Such implementations are pessimizing single-threaded use and have locking overhead on every API call. For demonstration purposes, here is a naive implementation of a 'guarding' class still having the locking overhead on every API call:

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

## Easier to reason about the code / better readability

When locking is tightly coupled with the data it protects, it becomes easier to reason about the behavior of the code. A guarded<std::string> makes it clear that access to the std::string is controlled and synchronized. 
There is also no question about what the lock is protecting, this makes the code more understandble and maintainable. The guarded type abstracts away the details of how the concurrency mechanisms are implemented. Users of the type do not need to worry about whether the data is protected by a mutex, spinlock, or some other concurrency primitive; they just know the data access is synchronized properly.

# Design considerations

## Overview

The goal of this proposal is to provide a type:

1) better readability and maintainability: by explicitly expressing the relationship between data and its guarding mechanism
2) better thread safely / hard to use incorrectly: by making it impossible to access the data without locking its guarding mechanism
3) make unlocking automatic and exception safe: by returning a RAII object that has ownership of the locked state

## Synopsis

An example of what the result could look like:

```
#include <string>
#include <iostream>

#include "https://raw.githubusercontent.com/copperspice/cs_libguarded/master/src/cs_plain_guarded.h"

int main() {
    libguarded::plain_guarded<std::string> guarded_string;
    
    auto accessor = guarded_string.lock();  // as long as 'accessor' remains in scope, the mechanism remains locked.

    // Modify the string safely.         
    *accessor = "Hello, World!";  
    
    // Read the string safely.
    std::cout << *accessor << '\n';

    return 0;
}  // accessor leaves scope, automatically unlocking
```

[compiler explorer link](https://cppcoach.godbolt.org/z/PTv1MG4oq)

Demonstration of clear separation of the class implementation and the sychronisation of the class.

```
#include <string>
#include <vector>
#include <iostream>

#include "https://raw.githubusercontent.com/copperspice/cs_libguarded/master/src/cs_plain_guarded.h"

struct person
{
    std::string name;
};

using contacts_t = std::vector<person>;

void print(const contacts_t& contacts)
{
    for (const auto& person: contacts)
    {
        std::cout << person.name << '\n';
    }
}

int main() 
{
    libguarded::plain_guarded<contacts_t> synchronized_contacts;  // specifiy only synchronized access to contacts_t is possible
    print(*synchronized_contacts.lock());                         // use of the lock
    return 0;
}
```


Discuss customizable [thread.req.lockable] interface with Cpp17BasicLockable requirements

```
std::guared<T>;
std::guared<T, L>;   // The type of second argument satisfies the [thread.req.lockable] interface
```

# Bikeshedding

This section is for naming, conventions and pinning down details to make it suitable for the standard.

As a suggestion: `std::guarded<T>` would express the intent.

# Acknowledgements

- this proposal is based on the the work of Ansel Sermersheim and his https://github.com/copperspice/cs_libguarded library

# References

- https://github.com/copperspice/cs_libguarded
- [[thread.req.lockable]](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/n4950.pdf) Cpp17BasicLockable requirements 
