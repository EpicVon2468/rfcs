- Feature Name: `weak-linkage`
- Start Date: 2026-09-28
- RFC PR: [rust-lang/rfcs#0000](https://github.com/rust-lang/rfcs/pull/0000)
- Rust Issue: [rust-lang/rust#0000](https://github.com/rust-lang/rust/issues/0000)

## Summary
[summary]: #summary

This RFC aims to address shortcomings in Rust's implementation (or lack thereof) of the concept of [weak linkage](https://en.wikipedia.org/wiki/Weak_symbol) by introducing a direct analogue to weak linkage in the form of an attribute.
Please note that this RFC pertains to "weak definitions" (`weak` in LLVM) rather than "weak references" (`extern_weak` in LLVM); When "weak linkage" is used here, it refers to "weak definitions" rather than "weak references".
<!-- TODO: Can the above note be made more concise?  Should it be put somewhere else? -->

## Motivation
[motivation]: #motivation

As Rust is increasingly used for embedded & very low-level development, being able to easily interface with ABI & FFI is crucial to reduce friction of use.
One of these FFI concepts which Rust currently falls short is weak linkage.
A feature does currently exist to address linkage in Rust, but it is mostly for internal `std` use, and suffers from design issues from trying to circumvent invariant violations from weak references & weak definitions.
<!-- TODO: Reword a little here, make it easier to segue into talking about the shortcomings of `#[linkage = "weak"]` -->

<!-- Any changes to Rust should focus on solving a problem that users of Rust are having. -->
<!-- This section should explain this problem in detail, including necessary background. -->

<!-- It should also contain several specific use cases where this feature can help a user, and explain how it helps. -->
<!-- This can then be used to guide the design of the feature. -->

<!-- This section is one of the most important sections of any RFC, and can be lengthy. -->

## Guide-level explanation
[guide-level-explanation]: #guide-level-explanation

<!-- Explain the proposal as if it was already included in the language and you were teaching it to another Rust programmer. That generally means: -->

<!-- - Introducing new named concepts. -->
<!-- - Explaining the feature largely in terms of examples. -->
<!-- - Explaining how Rust programmers should *think* about the feature, and how it should impact the way they use Rust. It should explain the impact as concretely as possible. -->
<!-- - If applicable, provide sample error messages, deprecation warnings, or migration guidance. -->
<!-- - If applicable, describe the differences between teaching this to existing Rust programmers and new Rust programmers. -->
<!-- - Discuss how this impacts the ability to read, understand, and maintain Rust code. Code is read and modified far more often than written; will the proposed feature make code easier to maintain? -->

<!-- For implementation-oriented RFCs (e.g. for compiler internals), this section should focus on how compiler contributors should think about the change, and give examples of its concrete impact. For policy RFCs, this section should provide an example-driven introduction to the policy, and explain its impact in concrete terms. -->

## Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

<!-- This is the technical portion of the RFC. Explain the design in sufficient detail that: -->

<!-- - Its interaction with other features is clear. -->
<!-- - It is reasonably clear how the feature would be implemented. -->
<!-- - Corner cases are dissected by example. -->

<!-- The section should return to the examples given in the previous section, and explain more fully how the detailed proposal makes those examples work. -->

## Drawbacks
[drawbacks]: #drawbacks

There appears to be a lack of consistency between platforms in stability & functionality of weak linkage, and there are some issues with `rustc` causing errors when LTO is enabled and weak symbols are present.

See also: <https://github.com/rust-lang/rust/issues/29603#issuecomment-5869051585>

## Rationale and alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

<!-- - Why is this design the best in the space of possible designs? -->
<!-- - What other designs have been considered and what is the rationale for not choosing them? -->
<!-- - What is the impact of not doing this? -->
<!-- - If this is a language proposal, could this be done in a library or macro instead? Does the proposed change make Rust code easier or harder to read, understand, and maintain? -->

## Prior art
[prior-art]: #prior-art

A previous attempt at linkage control in Rust:
- <https://github.com/rust-lang/rust/issues/29603>

GCC & Clang's `weak` attribute for C:
- <https://gcc.gnu.org/onlinedocs/gcc/Common-Attributes.html#index-weak>
- <https://clang.llvm.org/docs/AttributeReference.html#weak>

LLVM IR's `weak` linkage type:
- <https://llvm.org/docs/LangRef.html#linkage-types:~:text=weak>

## Unresolved questions
[unresolved-questions]: #unresolved-questions

- Attribute name bikeshedding;
- Should the attribute be considered `unsafe`, as `rustc` cannot verify the symbol which ends up being resolved at link-time matches the function prototype or type that is expected;
- Implementation details for non-ELF-based platforms (apparently [Windows has issues](https://github.com/rust-lang/rust/issues/29603#issuecomment-5866715657));

## Future possibilities
[future-possibilities]: #future-possibilities

A related but out-of-scope feature is "weak references" (`extern_weak` in LLVM), which allows that a symbol may not exist (being treated as "null", should it not exist).
There have been suggestions pertaining to this idea in the `#[linkage]` tracking issue (<https://github.com/rust-lang/rust/issues/29603>).
It is more than likely that "weak references" require a more substantial language implementation, such as an FFI type, rather than a mere function attribute (see [this issue comment](https://github.com/rust-lang/rust/issues/29603#issuecomment-966900393)).
It is likely that name bikeshedding will be needed between these two features in future.
