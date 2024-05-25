---
title: Programming Language
---

The kernel is written in the C programming language [\[c-language\]](#c-language){.citation}. More precisely, the kernel is typically compiled with `gcc` [\[gcc\]](#gcc){.citation} under `-std=gnu11` [\[gcc-c-dialect-options\]](#gcc-c-dialect-options){.citation}: the GNU dialect of ISO C11. `clang` [\[clang\]](#clang){.citation} is also supported, see docs on `Building Linux with Clang/LLVM <kbuild_llvm>`{.interpreted-text role="ref"}.

This dialect contains many extensions to the language [\[gnu-extensions\]](#gnu-extensions){.citation}, and many of them are used within the kernel as a matter of course.

# Attributes

One of the common extensions used throughout the kernel are attributes [\[gcc-attribute-syntax\]](#gcc-attribute-syntax){.citation}. Attributes allow to introduce implementation-defined semantics to language entities (like variables, functions or types) without having to make significant syntactic changes to the language (e.g. adding a new keyword) [\[n2049\]](#n2049){.citation}.

In some cases, attributes are optional (i.e. a compiler not supporting them should still produce proper code, even if it is slower or does not perform as many compile-time checks/diagnostics).

The kernel defines pseudo-keywords (e.g. `__pure`) instead of using directly the GNU attribute syntax (e.g. `__attribute__((__pure__))`) in order to feature detect which ones can be used and/or to shorten the code.

Please refer to `include/linux/compiler_attributes.h` for more information.

# Rust

The kernel has experimental support for the Rust programming language [\[rust-language\]](#rust-language){.citation} under `CONFIG_RUST`. It is compiled with `rustc` [\[rustc\]](#rustc){.citation} under `--edition=2021` [\[rust-editions\]](#rust-editions){.citation}. Editions are a way to introduce small changes to the language that are not backwards compatible.

On top of that, some unstable features [\[rust-unstable-features\]](#rust-unstable-features){.citation} are used in the kernel. Unstable features may change in the future, thus it is an important goal to reach a point where only stable features are used.

Please refer to Documentation/rust/index.rst for more information.

::: {#citations}

[c-language]{#c-language .citation-label}

:   <http://www.open-std.org/jtc1/sc22/wg14/www/standards>

[clang]{#clang .citation-label}

:   <https://clang.llvm.org>

[gcc]{#gcc .citation-label}

:   <https://gcc.gnu.org>

[gcc-attribute-syntax]{#gcc-attribute-syntax .citation-label}

:   <https://gcc.gnu.org/onlinedocs/gcc/Attribute-Syntax.html>

[gcc-c-dialect-options]{#gcc-c-dialect-options .citation-label}

:   <https://gcc.gnu.org/onlinedocs/gcc/C-Dialect-Options.html>

[gnu-extensions]{#gnu-extensions .citation-label}

:   <https://gcc.gnu.org/onlinedocs/gcc/C-Extensions.html>

[n2049]{#n2049 .citation-label}

:   <http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2049.pdf>

[rust-editions]{#rust-editions .citation-label}

:   <https://doc.rust-lang.org/edition-guide/editions/>

[rust-language]{#rust-language .citation-label}

:   <https://www.rust-lang.org>

[rust-unstable-features]{#rust-unstable-features .citation-label}

:   <https://github.com/Rust-for-Linux/linux/issues/2>

[rustc]{#rustc .citation-label}

:   <https://doc.rust-lang.org/rustc/>
:::
