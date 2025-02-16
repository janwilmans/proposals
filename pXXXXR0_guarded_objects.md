# Versions

* Latest version: https://github.com/janwilmans/proposals/blob/master/p3497_guarded_objects.md
* 2024 Papers: https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/
* pre-hagenberg version: https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p3497r0.html
* presentated at hagenberg (13-02-2025): https://github.com/janwilmans/proposals/blob/29d878bf2f2029bf41e27c6313e0928d09072e02/p3497_guarded_objects.md (p3497R2)

## The proposal was seem by SG1 in Hagenberg 2025, both Jan and Anthony were present:

SG1 Hagenberg 2025 Notes on P3497R0 Guarded Objects:

- Jan explains the proposal
- Anthony: the proposal is looks a lot like an early version of synchronized_value<T> from the concurrency TS2. Having a locked-state object was discussed and the current ::apply() solution was eventually preferred. Having a handle object was deemed easier to use incorrectly because you have to 'manually' construct the right scope for it.
- The proposal does not bring new insights
- SG1: feel free to create a new proposal that takes synchronized_value<T> from the concurrency TS2, with added implementation experience it might be passable to move into the standard on its own.
- The way to get this done would be to have one of the 3 major compilers implement it as 'experimental' and gain experience with it.
