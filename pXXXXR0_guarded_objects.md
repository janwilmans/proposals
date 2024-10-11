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

In multithreaded programming locking mechanisms are used to prevent concurrent access to data. Common practice is to create a locking mechansim, lets say an **std::mutex** along side the data it is protecting.
However, the relationship is only implied and expressed in code only by naming variables and/or 'doing it right' in all places. This proposal improves this by providing a way to clearly express the relationship and make it impossible to access the data without locking its associated guarding mechanism.

# Revision History

## Revision 0

Initial version

## Motivation

## Encapsulation of locking logic

The `guarded` type encapsulates the locking mechanism and its data, ensuring that lock is acquired and released properly when accessing the protected data. 
By requiring access to the data through the guarded type, you make it harder (or impossible) to accidentally access the data without properly locking it first.

## Simplified API

By combining the the data and its locking mechanism in one type they have the same lifetime and the API for interacting with the protected data becomes clearer. Users don’t have to worry about manually locking and unlocking. Instead, the locking and unlocking can be handled internally within the type. This simplifies the API and reduces the cognitive load.

## Strong ownership semantics / RAII guarantees

The only wat to access data is to call one of the locking functions, these functions either return an object (a unique_ptr-like object) that owns the lock, or allows you to pass in an operation that is exectured while holding the lock. It can be used in a familiar way, just like you would use a smart pointer, or when passing in the operation, not dealing with the lock directly at all.

The locking mechanism is based on RAII (Resource Acquisition Is Initialization), the type handles acquiring and releasing the lock in a scoped manner. This makes sure the lock is always released no matter how you leave the scope, returning of by an exception for example.

## Separation of concerns

The guarded type eliminates the need to embed thread-safety directly into the design of classes. Such implementations are pessimizing single-threaded use and have locking overhead on every API call. 
An example of a classic 'synchronized' queue:

```
#include <queue>
#include <mutex>
#include <condition_variable>
#include <chrono>

template <typename T>
class SynchronizedQueue
{
public:
    explicit SynchronizedQueue(size_t maxSize = 0) :
        m_maxSize(maxSize)
    {
    }

    bool Empty() const
    {
        std::unique_lock<std::mutex> lock(m_mtx);
        return Empty(lock);
    }

    bool Full() const
    {
        std::unique_lock<std::mutex> lock(m_mtx);
        return Full(lock);
    }

    size_t Size() const
    {
        std::unique_lock<std::mutex> lock(m_mtx);
        return m_q.size();
    }

    size_t MaxSize() const
    {
        return m_maxSize;
    }

    void WaitForNotFull() const
    {
        std::unique_lock<std::mutex> lock(m_mtx);
        m_cond.wait(lock, [&]() { return !Full(lock); });
    }

    template <typename Clock, typename Duration>
    bool WaitForNotFull(const std::chrono::time_point<Clock, Duration>& time) const
    {
        std::unique_lock<std::mutex> lock(m_mtx);
        return m_cond.wait_until(lock, time, [&]() { return !Full(lock); });
    }

    void WaitForNotEmpty() const
    {
        std::unique_lock<std::mutex> lock(m_mtx);
        m_cond.wait(lock, [&]() { return !Empty(lock); });
    }

    template <typename Clock, typename Duration>
    bool WaitForNotEmpty(const std::chrono::time_point<Clock, Duration>& time) const
    {
        std::unique_lock<std::mutex> lock(m_mtx);
        return m_cond.wait_until(lock, time, [&]() { return !Empty(lock); });
    }

    void Push(T t)
    {
        std::unique_lock<std::mutex> lock(m_mtx);
        m_cond.wait(lock, [&]() { return !Full(lock); });
        m_q.push(std::move(t));
        lock.unlock();
        m_cond.notify_one();
    }

    T Pop()
    {
        std::unique_lock<std::mutex> lock(m_mtx);
        m_cond.wait(lock, [&]() { return !Empty(lock); });
        T t(m_q.front());
        m_q.pop();
        lock.unlock();
        m_cond.notify_one();
        return t;
    }

private:
    bool Empty(std::unique_lock<std::mutex>&) const
    {
        return m_q.empty();
    }

    bool Full(std::unique_lock<std::mutex>&) const
    {
        return m_maxSize > 0 && m_q.size() == m_maxSize;
    }

    size_t m_maxSize;
    mutable std::mutex m_mtx;
    mutable std::condition_variable m_cond;
    std::queue<T> m_q;
};
```

Notice that the code above is fragile in the sense that every public method **must** acquire the lock and forgetting to do so is only cheked by reviewing.
Also, the private methods 'try' to make it harder to use incorrectly by requiring reference to a lock to be passed in, but not actually doing anything with that argument.

For demonstration purposes, here is a naive implementation of a 'guarding' class.

```
// Naive example how an operation could be passed in, to perform a set of operations while holding the lock

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

The example above is a demostration only 


[compiler explorer link](https://cppcoach.godbolt.org/z/4Evees8nd)

## Easier to reason about the code / better readability

When locking is tightly coupled with the data it protects, it becomes easier to reason about the behavior of the code. A guarded<std::string> makes it clear that access to the std::string is controlled and synchronized. 
There is also no question about what the lock is protecting, this makes the code more understandble and maintainable. The guarded type abstracts away the details of how the concurrency mechanisms are implemented. Users of the type do not need to worry about whether the data is protected by a mutex, spinlock, or some other concurrency primitive; they get a guarantee the data access is synchronized properly.

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
