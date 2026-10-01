- Feature Name: `weak-linkage`
- Start Date: 2026-09-28
- RFC PR: [rust-lang/rfcs#0000](https://github.com/rust-lang/rfcs/pull/0000)
- Rust Issue: [rust-lang/rust#0000](https://github.com/rust-lang/rust/issues/0000)

## Summary
[summary]: #summary

This RFC aims to address shortcomings in Rust's FFI interoperability – specifically relating to [weak linkage](https://en.wikipedia.org/wiki/Weak_symbol) – by replacing the hack implementation currently present (although perma-unstable) in the language with a more direct analogue.
This RFC is not a replacement for the high-level feature "Externally Implementable Items", and aims to target low-level, primarily FFI-related use-cases.
Please note that when "weak linkage" is used here, it refers to "weak definitions" (`weak` in LLVM) rather than "weak references" (`extern_weak` in LLVM).

## Motivation
[motivation]: #motivation

As Rust matures and is used more and more (especially in embedded contexts or ports of / bindings to C libraries), it is increasingly important that FFI parity for low-level platform features is provided for users of the language.

<!-- Any changes to Rust should focus on solving a problem that users of Rust are having. -->
<!-- This section should explain this problem in detail, including necessary background. -->

<!-- It should also contain several specific use cases where this feature can help a user, and explain how it helps. -->
<!-- This can then be used to guide the design of the feature. -->

<!-- This section is one of the most important sections of any RFC, and can be lengthy. -->

## Guide-level explanation
[guide-level-explanation]: #guide-level-explanation

```log
warning: use of `#[linkage = "weak"]` is deprecated
  --> <source>:04:15
   |
04 |		#[linkage = "weak"]
   |					^^^^^^
   |
   = help: consider using `#[weak]` and unwrapping the type to its pointee instead
   = warning: this was previously accepted by the compiler but is being phased out; it will become a hard error in a future release!
```

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

An attribute – `#[weak]` – shall be defined, which may be applied to the following language items:

- Statics and functions in `extern` blocks; and
- Statics and functions which are `extern` and marked with `#[unsafe(no_mangle)]`.

Any item which has the `#[weak]` attribute should cause the following:

- For ELF output, the symbol should be marked `STB_WEAK`;
- For Mach-O output, the symbol should be marked `N_WEAK_DEF`; and
- For COFF/PE output, the symbol should be marked `IMAGE_SYM_CLASS_WEAK_EXTERNAL`.

When using the LLVM backend, the above can be achieved via the `weak` linkage type.

## Drawbacks
[drawbacks]: #drawbacks

There are some issues with `rustc` causing errors when LTO is enabled and weak symbols are present.<br>
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
- Should the attribute be considered `unsafe`; and
- Implementation details for compiler backends other than LLVM.

## Future possibilities
[future-possibilities]: #future-possibilities

A related but out-of-scope feature is "weak references" (`extern_weak` in LLVM), which allows that a symbol may not exist (being treated as "null", should it not exist).<br>
It is more than likely that "weak references" require a more substantial language implementation, such as an FFI type, rather than a mere function attribute (see [this issue comment](https://github.com/rust-lang/rust/issues/29603#issuecomment-966900393)).<br>
Name bikeshedding between "weak references" and "weak definitions" will be needed in future.
