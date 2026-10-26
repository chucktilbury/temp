# Creating a Complete Software System with AI

*Cwhip: a native compiler with demonstration software.*

**Working title,** agreed 2026-10-26. On the cover, `cedit`, `cdbg`, `cmacros`, `cparsegen`, and `cvm` are the demonstration software. In the chapters, call them confidence tests.

# Chapter templates

**Written:** 2026-10-25  
**Updated:** 2026-10-26  
**Status:** Draft outline for the author to edit by hand. Each chapter below is a template, not manuscript prose.  
**Compiler pin:** `cwhip` `abfbf83` (2026-10-25, SDK 0.0.2, ABI `numeric-v2`). Standalone apps: `cedit` `59394f4`, `cdbg` `e7980b2`.  
**Ignores:** `notes/book-outline.md`. Do not merge the two outlines unless you choose to.

This file is AI-assisted planning. `notes/publisher-notes.md` records that the Pragmatic Bookshelf proposal template asks for an author-written proposal and sample chapter without AI assistance. Do not submit this file, or a light rewording of it, as that proposal.

## How to use these templates

Edit a chapter in place until its sections are the headings you want in the manuscript. Then copy that chapter into `text/` as its own file. Leave the other chapters here until you are ready for them.

The book opens with an introduction, then two main parts, then five short confidence-test chapters.

- The **introduction**, before Part I, tells the reader what Cwhip is, how the book is arranged, and what the confidence tests are for.
- **Part I** is the development record: design, implementation, tests, and the AI workflow that produced a toolchain.
- **Part II** is the system a reader uses: the language rules, the SDK, and one worked program.
- **Part III** is short. `cedit`, `cdbg`, `cmacros`, `cparsegen`, and `cvm` are confidence tests. They exist to show that the compiler is not a toy and has the features real software requires. Each chapter outlines how to use the tool, names what is missing, and gives one short example of adding a missing piece with an AI.

Part I may name files, rejected attempts, and tests. Part II states the rule a programmer needs and points at the language reference for the grammar. The same decision can appear in both halves. In Part I it is a choice that was made and held. In Part II it is a rule of the language. Part III treats the public tools as evidence. It does not turn them into product manuals.

The audience is intermediate and advanced developers who are looking for sharper tools. Assume they can read C, C++, and Python. Teach the Cwhip choice and its consequence. Do not open with what a compiler is.

`cparsegen` is still moving; do not promise its final syntax or generated examples. `cwhip-workspace` is retired. Shop Boss is not part of this project.

## Audit notes for drafting

Use these sources as the current contract:

- `doc/LANGUAGE-REFERENCE.md` and `doc/GRAMMAR.md` for the language.
- `doc/NUMERIC-MIGRATION.md` for the 0.0.2 / `numeric-v2` break.
- `doc/DIAGNOSTICS.md`, `doc/TESTING.md`, `tests/README.md` for errors and tests.
- `doc/PACKAGES.md`, `doc/INSTALLATION.md`, `doc/CMAKE-SDK.md`, `doc/GETTING_STARTED.md` for the SDK.
- `doc/STDLIB.md`, `doc/GTK.md`, `doc/CALLBACKS.md`, `doc/DEBUGGING.md` for library, GUI, native calls, and debug info.
- `doc/EXAMPLES.md` and `examples/` for programs that the test suite already runs.

Treat these as history, not as the current language:

- `doc/CWHIP-LANGUAGE-NOTES.md` says its numeric discussion is historical, then "Agreed language semantics" still requires `int` as signed 64-bit, `unsigned` as unsigned 64-bit, and `float` as IEEE double. Later sections still say the compiler uses LLVM's C++ APIs. The implementation does neither. `int`, `uint`, and `float` are 32-bit aliases for `int32`, `uint32`, and `fp32`. `unsigned` is reserved and rejected. The driver emits textual LLVM IR and invokes host `clang`. There is no `find_package(LLVM)`.
- `/home/chuck/Src/cwhip-work/syntax-cleanup.md` describes an older tree (`05e93ae`) in which `else (expr)` was rejected and dict literals used braces. Current source accepts `else (condition)`, rejects `else if`, and uses bracket dictionaries, with `[:]` for an empty dict.
- `/home/chuck/Src/cwhip-work/numeric-type-system-specification.md` is a design specification. Check any rule you quote against `LANGUAGE-REFERENCE.md` and `NUMERIC-MIGRATION.md` before it enters a chapter.

Facts that were true in this tree on the pin above:

- Frontend is C++17. Runtime is C: `src/RuntimeGc.c`, `src/RuntimeOS.c`, `src/RuntimeProcess.c`, `src/RuntimeProcessAsync.c`, `src/RuntimeStdlib.c`, `src/RuntimeThreads.c`.
- Pipeline in `src/main.cpp`: command line, path resolution, source load, lex, parse, modules, semantic analysis, textual IR, then `clang`. `--check-only` stops before code generation. `-c` emits an object. `-g` emits DWARF. `--emit-llvm` keeps the IR. `-t` traces `scanner`, `parser`, `ast`, or `symbols`.
- Object files carry the sidecar header `CWHIP-LINK 2 numeric-v2`.
- Keywords are case-insensitive. Identifiers are case-sensitive. `;` is an unknown character. Statements end at a newline. Indentation is not syntax.
- A program has exactly one top-level `start`. Modules have none. There are no free functions, closures, or user-defined operators. Methods return with `return`; `start` does not.
- `print` does not add a newline. Lists and dicts are tagged and heterogeneous. A list element extracted to the wrong numeric width fails. Callables are spelled `R<P…>`, for example `int64<int64>`.
- Standard packages under `stdlib/`: `gtk`, `math`, `networking`, `process`, `random`, `sourceview`, `sql`, `sqlite`, `synchronization`, `system`, `terminal`, `time`. `String` lives in `builtin/String.cw`.
- `cedit`, `cdbg`, `cmacros`, `cparsegen`, and `cvm` are confidence tests and standalone SDK consumers. They are not targets in the compiler tree. `doc/GETTING_STARTED.md` matches that split for the editor and debugger. `cwhip-workspace` is retired and is not a build path. Shop Boss is not part of this project.
- DWARF maps lines and some locals. It does not present class fields as source-level members. `cdbg` is a local GDB/MI prototype; its value inspector needs Python-enabled GDB. Step Into can enter runtime C checks.
- The install prefix defaults to `$HOME/opt/cwhip`. The supported distribution is the whole prefix, not `bin/cwhip` alone. `cwhip-pkg` manages pins and locks; it does not build the application. Host `clang`, Python 3.11+, CMake, and SQLite development files stay outside the prefix. Native Windows is not a target.

## Introduction

This chapter sits before Part I. It is the reader's map. Chapter 1 carries the longer account of why the language exists.

### Introduction. What you are holding

**Outcome.** The reader can say what Cwhip is, how the three parts of the book differ, and why the public tools are in the book.

**Reader already knows.** C, C++, Python, and the feel of a compiler that gets in the way.

**Sections**

1. The language and the toolchain
2. How this book is arranged
3. Confidence tests
4. The version in your hands

**Must stay true.** The title is *Creating a Complete Software System with AI*. The subtitle is *Cwhip: a native compiler with demonstration software.* Cwhip is a statically typed class language. The name is the coachwhip snake: a small, fast compiler. The frontend is C++17. The runtime is C. The driver emits textual LLVM IR and invokes host `clang`. A program has one `start`. Methods replace free functions. Numeric widths are explicit. This edition matches the pin in the header: SDK 0.0.2, ABI `numeric-v2`.

The book is an introduction, then Part I (how the toolchain was built), Part II (the language a reader compiles today), and Part III (five short confidence tests). `cedit`, `cdbg`, `cmacros`, `cparsegen`, and `cvm` are those tests. Refer to them that way. Their job is to show that the compiler can carry real programs. `cwhip-workspace` is retired. Shop Boss is not part of this project. Do not open the introduction by defining what a compiler is.

**Draw from.** `README.md` opening and "Implemented language highlights." `doc/GETTING_STARTED.md` "What you will install." The audit notes in this file.

**Author fills in**

- One paragraph, in your voice, on why a reader should trust a language they have not seen before. Leave the longer history for Chapter 1.
- Whether the reader installs the SDK before Part I, or only when Part II begins.
- The sentence you will actually use for "confidence test," so the later chapters repeat your wording.

**Draft**

_The language and the toolchain._

_How this book is arranged._

_Confidence tests._

_The version in your hands._

## Part I — Building the toolchain

### Chapter 1. The tool you wished you had

**Outcome.** The reader can state the dissatisfaction that justifies a new language, and the few constraints that were fixed before implementation started.

**Reader already knows.** C, C++, Python, and the feel of a compiler that gets in the way.

**Sections**

1. What current tools make expensive
2. What this project refused to become
3. The constraints fixed before the first compiler
4. What the reader will be able to build by the end of the book

**Must stay true.** Cwhip is a statically typed class language with a C++17 frontend, a C runtime, textual LLVM IR, and host `clang`. The name is the coachwhip snake: a small, fast compiler. The early constraints to cover, in the author's own order, are one `start`, methods rather than free functions, explicit numeric widths, and a runtime written in C.

**Draw from.** `README.md` opening and "Implemented language highlights." The vision section of `doc/CWHIP-LANGUAGE-NOTES.md` for the C / C++ / Python mix, with the later numeric rules left for Chapter 5.

**Author fills in**

- The specific failures in existing tools that started the project.
- Which constraints were decided in conversation, and which arrived only after code existed.
- One paragraph on what this book will not try to be: a memoir without a compiler, or a manual without a record of how the compiler was built.

**Draft**

_What current tools make expensive._

_What this project refused to become._

_The constraints fixed before the first compiler._

_What the reader will be able to build._

### Chapter 2. Deciding the language in public

**Outcome.** The reader can see a language decision as a written rule with a consequence, and can tell a design note from the implementation.

**Sections**

1. Writing the rule down before the code
2. A decision that survived: newline statements and the rejected semicolon
3. A decision that was replaced: the old numeric model
4. How a design note goes stale

**Must stay true.** Current lexical rules are in the language reference: `#`, `//`, and `/* */` comments; `;` is an unknown character. The historical notes still describe `int` as i64, `unsigned` as a type, and `float` as double. SDK 0.0.2 replaced that model. See the audit notes.

**Draw from.** `doc/LANGUAGE-REFERENCE.md` "Lexical structure." `doc/CWHIP-LANGUAGE-NOTES.md` status banner and "Agreed language semantics." `doc/NUMERIC-MIGRATION.md` opening. `syntax-cleanup.md` only as an example of a note that implementation later overtook.

**Author fills in**

- Two or three decisions you argued about and then kept.
- One decision you reversed, with the program or test that forced the reversal.
- How you want the reader to treat AI-written design notes: useful record, not a specification.

**Draft**

_Writing the rule down before the code._

_A decision that survived._

_A decision that was replaced._

_How a design note goes stale._

### Chapter 3. Using AI on a compiler

**Outcome.** The reader has a repeatable loop for specifying a change, generating an attempt, and accepting it only when the compiler's own tests agree.

**Sections**

1. What was given to the model
2. What review kept
3. What the test suite was allowed to veto
4. Where the model could not see the design

**Must stay true.** Do not invent a percentage of lines written by a model. The verifiable method in this tree is: a change lands with diagnostics, fixtures under `tests/functional/`, and a `tests/cases/*.cmake` case. `doc/DIAGNOSTICS.md` is the text contract: `tool: severity[CODE]: explanation`, diagnostics on stderr, success on stdout.

**Draw from.** `doc/DIAGNOSTICS.md`. `tests/README.md` "Adding an integration test." One real merged change you are willing to show in public, with its tests.

**Author fills in**

- The prompt and review habit you actually used. Name the tools.
- A change the model got structurally right and semantically wrong.
- A change you rejected even though it compiled.
- What you will not claim about authorship when a publisher asks.

**Draft**

_What was given to the model._

_What review kept._

_What the test suite was allowed to veto._

_Where the model could not see the design._

### Chapter 4. The compiler, phase by phase

**Outcome.** The reader can follow one source file from `cwhip` on the command line to a native executable, and can say which phase owns which failure.

**Sections**

1. The driver
2. Tokens and the parser
3. Names, types, and inheritance
4. Textual IR and the call to `clang`
5. Reading a diagnostic back to a phase

**Must stay true.** The phase files are:

| Phase | Files |
| --- | --- |
| Driver | `src/main.cpp`, `src/CommandLine.cpp` |
| Paths and source | `src/PathResolution.cpp`, `src/SourceLoader.cpp` |
| Lex | `src/Lexer.cpp` |
| Parse | `src/Parser.cpp`, `src/ParserClass.cpp` |
| Modules and packages | `src/ModuleRegistry.cpp`, `src/PackageRegistry.cpp`, `src/PackageIntegrity.cpp` |
| Semantics | `src/SemanticAnalyzer.cpp`, `src/SemanticExpressions.cpp`, `src/SemanticStatements.cpp`, `src/SemanticInheritance.cpp` |
| IR | `src/CodeGenerator.cpp`, `src/CodeGeneratorStatements.cpp`, `src/CodeGeneratorNumeric.cpp`, `src/CodeGeneratorLayout.cpp`, `src/CodeGeneratorExceptions.cpp`, `src/CodeGeneratorCallbacks.cpp`, `src/CodeGeneratorDispatch.cpp`, `src/CodeGeneratorDebug.cpp`, `src/CodeGeneratorPrint.cpp`, `src/CodeGeneratorModule.cpp` |
| Diagnostics | `src/Diagnostic.cpp` |

The parser limits recursive nesting and expression depth to 128. That limit is a diagnostic, not a crash. `--check-only` accepts source and does not take `-c`, `-o`, `-L`, or `-l`. Link objects must begin with `CWHIP-LINK 2 numeric-v2`.

**Draw from.** `src/main.cpp` from option handling through the clang invocation. `doc/LANGUAGE-REFERENCE.md` opening paragraphs on the pipeline and the nesting limit. `doc/cwhip.1` for the switches you show.

**Author fills in**

- One small program traced through every phase, including the IR you are willing to print.
- Which phase was hardest to keep under AI edits, and why.
- The reason textual IR plus `clang` was chosen over linking the LLVM C++ API. The historical notes claim the API; the build does not use it. Tell that as a decision, not as a contradiction the reader must resolve.

**Draft**

_The driver._

_Tokens and the parser._

_Names, types, and inheritance._

_Textual IR and the call to clang._

_Reading a diagnostic back to a phase._

### Chapter 5. The runtime and the numeric break

**Outcome.** The reader understands what the generated program still needs from C, and why 0.0.2 was allowed to break source and ABI together.

**Sections**

1. What the runtime owns
2. Memory, threads, and failure
3. The fixed-width numeric contract
4. Rebuilding the world that sat on the old types

**Must stay true.** Runtime sources are the `src/Runtime*.c` files listed in the audit. Integer widths are `int8` through `int64` and `uint8` through `uint64`. Floating widths are `fp32` and `fp64`. Unsuffixed integers take the smallest fitting signed width; `u`/`U` takes the smallest unsigned width. A floating literal rounds to `fp32` unless it carries `f64`. `fp64(0.1)` widens an already rounded `fp32` value; `0.1f64` rounds directly. Lists preserve the exact tag, so `[1][0] as int64` fails because `1` is `int8`. Counts and indices in the bundled library stayed `int64`. Clock values stayed `uint64`. Math stayed `fp64`. FP8, FP16, and BF16 were deferred. Exceptions use C `setjmp` / `longjmp` from generated code.

**Draw from.** `doc/NUMERIC-MIGRATION.md`. `doc/LANGUAGE-REFERENCE.md` numeric sections. `include/cwhip/Stdlib.h` for `CWHIP_SDK_ABI` and `CWHIP_SDK_VERSION`. `numeric-type-system-specification.md` where it still matches the reference.

**Author fills in**

- Why the first numeric model was attractive.
- The bug or the ABI fact that made i64-`int` and double-`float` the wrong contract.
- How painful the rebuild of `cedit`, `cdbg`, and the standard library actually was.

**Draft**

_What the runtime owns._

_Memory, threads, and failure._

_The fixed-width numeric contract._

_Rebuilding the world that sat on the old types._

### Chapter 6. Tests are the specification

**Outcome.** The reader can add a behavior to this kind of project by writing the fixture and the expected result first, then accepting the implementation only when that case passes.

**Sections**

1. A positive program and its exact output
2. A negative program and its diagnostic
3. Sanitizers, the garbage collector, and GUI tests
4. Keeping examples and documentation inside the suite

**Must stay true.** Integration cases are `tests/cases/*.cmake` plus `tests/functional/positive/` or `tests/functional/negative/`. Success compares exact stdout, including the final newline. Rejection matches `CASE_EXPECTED_DIAGNOSTIC`. `CWHIP_SANITIZE` instruments the compiler, the C runtime, and native tests, and the instrumented compiler forwards sanitizer flags into generated programs. Leak detection is off for generated programs because the managed heap retains objects until process exit. Address and undefined-behavior checks stay fatal. GTK tests run when the host has GTK 4.10+, GtkSourceView 5, Xvfb, and `dbus-daemon`. The compiler command itself does not require GTK.

**Draw from.** `tests/README.md`. `doc/TESTING.md`. One positive case and one negative case, copied short enough to read on a page. `tests/test_doc_programs.py` and `doc/EXAMPLES.md` for the documentation programs.

**Author fills in**

- A bug that existed until a fixture named it.
- A case you refuse to weaken in order to accept a generated patch.
- How much of the suite you run before trusting an AI edit.

**Draft**

_A positive program and its exact output._

_A negative program and its diagnostic._

_Sanitizers, the garbage collector, and GUI tests._

_Keeping examples and documentation inside the suite._

### Chapter 7. From a compiler to a system

**Outcome.** The reader can explain why the editor, debugger, and package tool live outside the compiler repository, and what "install the SDK" actually installs.

**Sections**

1. The prefix is the product
2. Packages, locks, and the decision not to reach the network from `cwhip`
3. Confidence tests as SDK consumers
4. What Part III will show

**Must stay true.** Default prefix is `$HOME/opt/cwhip`. Staged layout is `bin/`, `lib/`, `include/`, `share/`. `cwhip` compiles installed source and does not fetch packages. `cwhip-pkg` does `init`, `add`, `install`, `doctor`, and `new`. Manifest schema 1 uses exact `=VERSION` pins, not ranges. `cwhip-template` is what `cwhip-pkg new` downloads. `cedit`, `cdbg`, `cmacros`, `cparsegen`, and `cvm` are confidence tests that live in their own repositories. `cedit` is a GTK editor, not a language server. `cdbg` is a local debugger. `cmacros` is a UTF-8 macro processor and does not invoke a shell. `cparsegen` emits parsers for Cwhip or C99 and is still a prototype. `cvm` is a stack machine. Use, gaps, and one AI-sized addition for each tool belong in Part III. `cwhip-workspace` is retired. Shop Boss is not part of this project. Binary installers are not the documented path.

**Draw from.** `doc/GETTING_STARTED.md` "What you will install." `doc/INSTALLATION.md`. `doc/PACKAGES.md`. `doc/CMAKE-SDK.md`. The standalone `README.md` files in `cedit`, `cdbg`, `cmacros`, `cparsegen`, and `cvm`.

**Author fills in**

- When you decided the compiler tree should stop owning `cedit` and `cdbg`.
- Which confidence test first proved the SDK boundary.
- The sentence that points the reader at Part III without previewing every gap.

**Draft**

_The prefix is the product._

_Packages, locks, and the compiler that does not use the network._

_Confidence tests as SDK consumers._

_What Part III will show._

### Chapter 8. What the process can and cannot carry

**Outcome.** The reader leaves Part I with a judgment: which kinds of systems this AI-plus-tests loop can build, and which kinds it should leave alone.

**Sections**

1. Work the loop fits
2. Work that needs a design the model does not hold
3. Documentation drift as a project risk
4. The handoff to the language itself

**Must stay true.** Use Chapter 2's stale notes as the concrete drift example. Use the numeric migration as a change that tests and an ABI header could hold. Do not claim the loop replaces design. Part II begins from the language a reader can compile today, at the pin in the header. Part III then uses the five public tools as confidence tests.

**Draw from.** The audit notes in this file. `doc/DEBUGGING.md` for one limit inside the toolchain itself: DWARF line and local information without full class-member layouts.

**Author fills in**

- Your own rule for when to stop asking a model and write the design yourself.
- Anything in the toolchain you will describe as deliberately unfinished.
- The sentence that ends Part I and opens Part II.

**Draft**

_Work the loop fits._

_Work that needs a design the model does not hold._

_Documentation drift as a project risk._

_The handoff to the language itself._

## Part II — Using Cwhip

### Chapter 9. The shape of a program

**Outcome.** The reader can write, compile, and run a complete program made of imports, classes, methods, and one `start`.

**Sections**

1. A file, a class, and `start`
2. Names, comments, and the end of a statement
3. Imports and modules
4. Compile and run

**Must stay true.** Source suffix is `.cw`. Exactly one `start` across the compilation. `return` is for methods. Keywords fold case; identifiers do not. `String` is the builtin class; `strg` is the primitive keyword; `string` is an ordinary identifier. `list` and `dict` are keywords; `array` and `map` are not. Build with the installed `cwhip`, or `./bin/cwhip` from a compiler checkout. The commands in `README.md` are the ones to show.

**Draw from.** `doc/LANGUAGE-REFERENCE.md` "Program structure and entry point." `doc/FIRST-PROGRAM.md` if you want the editor-led path. `doc/GRAMMAR.md` for production names you mention, not for a grammar chapter.

**Author fills in**

- The first program of the book. Keep it small enough to reappear in later chapters.
- Whether the reader is expected to build the compiler or only install the SDK. Pick one primary path and put the other in a note.

**Draft**

_A file, a class, and start._

_Names, comments, and the end of a statement._

_Imports and modules._

_Compile and run._

### Chapter 10. Types and values

**Outcome.** The reader can choose a numeric width on purpose, format a value, and move a number through a list without losing its tag.

**Sections**

1. Booleans, widths, and aliases
2. Literals and conversions
3. Strings and format
4. Lists, dicts, and ranges

**Must stay true.** Aliases: `int` is `int32`, `uint` is `uint32`, `float` is `fp32`. `unsigned`, `float32`, and `float64` are not aliases. Integer casts reduce modulo the destination width. Text parsing does not. Floating-to-integer conversion truncates, then checks range. `format("value {}", value)` returns `strg`. `print` writes what you give it, including newlines you put in the string. `range<T>` stores inclusive endpoints. `for (i in first..last)` walks either direction. `else (condition)` is the chain; `else if` is not legal. Dict literals use brackets; the empty dict is `[:]`.

**Draw from.** `doc/LANGUAGE-REFERENCE.md` lexical structure, numeric types, strings, collections, switches, and ranges. `doc/NUMERIC-MIGRATION.md` "Source and API replacements." `doc/EXAMPLES.md` switches and ranges. `examples/range_slices.cw`, `examples/range_snapshots.cw`, `examples/range_windows.cw`, `examples/switch_delivery.cw`.

**Author fills in**

- One arithmetic example where the smallest-width literal changes the result, and the repair.
- How much of the format grammar belongs in the chapter, and what you will leave in the reference.

**Draft**

_Booleans, widths, and aliases._

_Literals and conversions._

_Strings and format._

_Lists, dicts, and ranges._

### Chapter 11. Control, failure, and checks

**Outcome.** The reader can write the control flow the language actually has, raise and catch a failure, and read a compiler diagnostic.

**Sections**

1. Conditions, switches, and loops
2. Exceptions
3. The checks the compiler refuses to skip
4. Runtime checks the compiler cannot prove

**Must stay true.** Conditions are parenthesized `bool` expressions. `switch` has constant alternatives, inclusive ranges, optional `default`, and explicit `next`, with no implicit fallthrough. Exceptions are part of the implemented language; `examples/normal_exception.cw`, `examples/unhandled_exception.cw`, `examples/reraise_exception.cw`, and `examples/virtual_exception.cw` are the runnable set. Diagnostics follow `tool: severity[CODE]: explanation`. A warning alone is not a failed compile.

**Draw from.** `doc/LANGUAGE-REFERENCE.md` control-flow and exception sections. `doc/DIAGNOSTICS.md` text contract. `examples/err.cw`.

**Author fills in**

- The error-handling style you want readers to copy.
- A diagnostic you will walk through character by character.

**Draft**

_Conditions, switches, and loops._

_Exceptions._

_The checks the compiler refuses to skip._

_Runtime checks the compiler cannot prove._

### Chapter 12. Classes, methods, and native code

**Outcome.** The reader can design a small class hierarchy, call a method, and bind one native function without pretending the language has free functions.

**Sections**

1. Fields, constructors, and destructors
2. Inheritance and `super`
3. Methods and callbacks
4. `from "symbol"` and inline C

**Must stay true.** No free functions or user-defined operators. Base constructors use `super Base(args)`. Virtual bases are supported and have an initializer order the reference specifies; quote that order from the reference rather than from memory. Enum members are immutable `int64` constants, not a new type. Callbacks use the `R<P…>` spelling and the ABI in `doc/CALLBACKS.md`. Native bindings are explicit symbols. Inline C exists; show a short example from `examples/trivial_inline.cw` only if you also show the boundary around it.

**Draw from.** `doc/LANGUAGE-REFERENCE.md` classes, inheritance, enums, and native sections. `doc/CALLBACKS.md`. `examples/ctor_dtor.cw`, `examples/virtual_base.cw`, `examples/trivial_inline.cw`, `examples/cast_scalar.cw`, `examples/collection_cast.cw`.

**Author fills in**

- Why methods-only was worth the inconvenience.
- One native binding from the standard library, small enough to read, with the C symbol it calls.

**Draft**

_Fields, constructors, and destructors._

_Inheritance and super._

_Methods and callbacks._

_Native symbols and inline C._

### Chapter 13. Packages and projects

**Outcome.** The reader can create a project, pin a dependency, lock it, and build against an installed SDK without a compiler checkout.

**Sections**

1. The manifest and the lock
2. Search order and versions
3. `cwhip-pkg` without the network in `cwhip`
4. CMake from outside the compiler tree

**Must stay true.** Schema 1. Direct dependencies only. `version = "=1.2.3"` form. The lock records the commit that later installs use. `cwhip-pkg doctor` is read-only. `cwhip-pkg new` can take `--ref` or an offline `--source`. External projects use the relocatable CMake SDK. SDK selection is `CWHIP_SDK`, then `sdk.toml`, then CMake discovery. Python 3.11+ is required to read that configuration.

**Draw from.** `doc/PACKAGES.md`. `doc/INSTALLATION.md` package-manager and module-search sections. `doc/CMAKE-SDK.md`. `examples/package_workflow.cw`. `doc/GETTING_STARTED.md` "Use packages."

**Author fills in**

- The smallest public or local package you will have the reader add.
- What happens in the book if that package is not published yet. Prefer a fixture you control.

**Draft**

_The manifest and the lock._

_Search order and versions._

_The package tool and the compiler._

_CMake from outside the compiler tree._

### Chapter 14. One working program

**Outcome.** The reader builds one complete program that uses the language rules from the previous chapters and produces an output the book can check.

**Sections**

1. The problem the program solves
2. The types and the data
3. The classes
4. Build, run, and the expected result

**Must stay true.** Pick one program and stay with it. Candidates already in the tree:

- `examples/network.cw` with `examples/xor.csv`: a two-layer network, deterministic weights, explicit `\n` in prints, success line `result learned`.
- `examples/gtk.cw`: a short GTK 4 program, optional on hosts without GTK.

Leave `cedit`, `cdbg`, `cmacros`, `cparsegen`, and `cvm` to Part III. A sentence may point there. This chapter does not walk those programs.

**Draw from.** The chosen example, its expected output in the test suite, and `doc/EXAMPLES.md`. For a GUI choice, `doc/GTK.md` and `stdlib/gtk/`.

**Author fills in**

- Which candidate is the book's program. The network example is self-contained and already has a known successful run. The GTK example shows the application side and depends on the host.
- The output you will print, taken from a run of the pinned compiler, not from memory.

**Draft**

_The problem the program solves._

_The types and the data._

_The classes._

_Build, run, and the expected result._

### Chapter 15. Seeing a running program

**Outcome.** The reader can compile with `-g`, step in GDB or LLDB, and use `cdbg` with a clear idea of what the debugger can and cannot show.

**Sections**

1. Line information and locals
2. What a stop looks like in GDB
3. The `cdbg` session
4. The values DWARF does not reconstruct

**Must stay true.** `-g` / `--debug` writes DWARF for the executable and the IR. Stepping can enter runtime C before a Cwhip method. Distinct instructions on one source line are not a lost step. Class fields are not necessarily source-level members. `cdbg` talks GDB/MI, can compile a `.cw` file with debug information or load an executable, and needs Python-enabled GDB with `gdb.MICommand` for its friendly inspector. Raw native locals remain available without that inspector. This chapter is about debugging a program the reader wrote. Chapter 7 records why `cdbg` lives outside the compiler repository. Chapter 18 is the confidence test: how to use `cdbg`, what it lacks, and one AI-sized addition.

**Draw from.** `doc/DEBUGGING.md`. `cdbg/CDBG.md` and `cdbg/README.md`. A tiny `-g` program from `tests/functional/positive/positive_debug_info.cw` or from the Chapter 14 program.

**Author fills in**

- A transcript of one real stop: command, file, line, and one local.
- Whether `cdbg` screenshots belong in the book. If the UI is still moving, prefer a command transcript.

**Draft**

_Line information and locals._

_What a stop looks like in GDB._

_The cdbg session._

_The values DWARF does not reconstruct._

### Chapter 16. Where the language stops

**Outcome.** The reader can choose Cwhip for a program it fits, and can name the neighboring tool when the program needs arrays, a GPU, or a cluster.

**Sections**

1. Programs the language is for
2. Numeric work that wants packed arrays and a native kernel
3. Processes, machines, and deployment
4. What to keep stable

**Must stay true.** Cwhip lists are tagged and heterogeneous. They are the wrong container for a large matrix. The language can hold a buffer or a native handle and call a kernel compiled elsewhere; that is the same split Python uses when NumPy or PyTorch does the arithmetic. A cluster runtime, a load balancer, and a deploy-time pipeline are outside the language. The SDK ABI pin is `numeric-v2`. Mixing prefixes, or mixing objects built before 0.0.2, is a rebuild, not a compatibility mode. The book should end on the toolchain the reader can run, at a named version.

**Draw from.** `doc/NUMERIC-MIGRATION.md` deferred items. `include/cwhip/Stdlib.h`. Chapter 14's program as the positive example of fit. No new compiler theory.

**Author fills in**

- The largest real program you are willing to say has been written in Cwhip.
- The next language or SDK change you will not promise in this edition.
- The version, commit, and example outputs the printed book is bound to.

**Draft**

_Programs the language is for._

_Numeric work that wants packed arrays and a native kernel._

_Processes, machines, and deployment._

_What to keep stable._

## Part III — Confidence tests

These chapters are short, and they share one shape.

1. How to use the tool.
2. What is missing.
3. One short example of adding a missing piece with an AI.

Call `cedit`, `cdbg`, `cmacros`, `cparsegen`, and `cvm` confidence tests. They show that the compiler can carry real software. The third section is a proposed change. It is not a feature already in the repository. Show the prompt, the patch shape, and the test that would reject a wrong result. Do not invent a share of the tool written by a model. `cwhip-workspace` is retired. Shop Boss is not part of this project.

### Chapter 17. cedit as a confidence test

**Outcome.** The reader can start the editor, name a gap that is still in the current tree, and follow one AI-sized change that would close part of that gap.

**Sections**

1. How to use `cedit`
2. What is missing
3. Adding one missing piece with an AI

**Must stay true.** `cedit` is a GTK 4 and GtkSourceView editor written in Cwhip. It is a standalone consumer of SDK 0.0.2 / `numeric-v2`, and a proof of concept rather than a production IDE. There is no language server. Unsaved edits stay in memory. The current tree also defers build commands, plugins, Git integration, project trust, persistent recent history, an extension runtime, and a configurable toolbar. The AI example proposes one of those gaps. It does not describe the gap as already implemented. Prefer a change the headless suite can check.

**Draw from.** `cedit/README.md`. `cedit/CEDIT.md`, especially "Deliberately deferred." `cedit/doc/BUILD.md`.

**Author fills in**

- The session you will show: open a file, edit, save.
- Which deferred gap is the AI example. Unsaved-buffer recovery is small enough to explain. A language server is not.
- The prompt you would actually give, and the test that would veto a bad patch.

**Draft**

_How to use cedit._

_What is missing._

_Adding one missing piece with an AI._

### Chapter 18. cdbg as a confidence test

**Outcome.** The reader can run a local debug session, name a limit of the prototype, and follow one AI-sized change inside `cdbg`.

**Sections**

1. How to use `cdbg`
2. What is missing
3. Adding one missing piece with an AI

**Must stay true.** `cdbg` is a local GTK debugger that talks GDB/MI. It can compile a `.cw` file with `-g` or load an executable. The friendly inspector needs Python-enabled GDB with `gdb.MICommand`. Launch is local: no remote debugging, attach, DAP, or LLDB. The stack view is capped at 31 frames. The output PTY does not supply stdin. Watches use a read-only subset of GDB expressions. Class fields display as **members unavailable** because the compiler emits forward declarations, not member layouts. Gutter-click breakpoint toggles are future work. Chapter 15 teaches the reader to debug their own program. This chapter is the confidence test. An AI example that repairs class-member DWARF is a compiler change; keep that with Chapter 15 unless you deliberately move it here. Prefer a change inside `cdbg`, such as the gutter toggle, that the component suite can check.

**Draw from.** `cdbg/README.md`. `cdbg/CDBG.md` "Boundaries" and "Cwhip values and runtime values." `cwhip/doc/DEBUGGING.md`.

**Author fills in**

- One real session: file, breakpoint, and one local.
- The missing piece the AI example adds.
- Whether screenshots belong here. If the UI is still moving, prefer a command transcript.

**Draft**

_How to use cdbg._

_What is missing._

_Adding one missing piece with an AI._

### Chapter 19. cmacros as a confidence test

**Outcome.** The reader can expand one macro file, name a boundary of the processor, and follow one AI-sized change that the example runner can check.

**Sections**

1. How to use `cmacros`
2. What is missing
3. Adding one missing piece with an AI

**Must stay true.** `cmacros` is a standalone UTF-8 macro processor written in Cwhip. It reads one source file, expands directives in memory, and writes the result to standard output or `-o`. It does not invoke a shell, a compiler, or another subprocess. `.define`, `.import`, `.include`, `.if`, `.else`, `.error`, and `.message` are the directive surface. Forward references converge across passes. `doc/USAGE.md` places compiler integration, GUI support, binary distribution packages, and icons out of scope for this milestone. `doc/DIAGNOSTICS.md` records the processing contract, including a 16 MiB source limit, import nesting of 128, expansion nesting of 128, and 256 passes before a non-convergence error. The AI example is one checked behavior inside the processor. It is not compiler integration, and it is not a feature the repository already contains. The original passing examples are the comparison the chapter should use.

**Draw from.** `cmacros/README.md`. `cmacros/doc/USAGE.md`. `cmacros/doc/LANGUAGE.md`. `cmacros/doc/DIAGNOSTICS.md`. `cmacros/doc/EXAMPLES.md` and `examples/run_examples.py`.

**Author fills in**

- One input file and the output the runner expects.
- Which missing behavior is small enough for the AI example. Keep the documented limits as the contract unless you are changing that contract on purpose.
- The example you will refuse to weaken in order to accept the generated patch.

**Draft**

_How to use cmacros._

_What is missing._

_Adding one missing piece with an AI._

### Chapter 20. cparsegen as a confidence test

**Outcome.** The reader can generate a parser from one grammar, name a backend the tool does not yet emit, and follow one AI-sized slice of that work.

**Sections**

1. How to use `cparsegen`
2. What is missing
3. Adding one missing piece with an AI

**Must stay true.** `cparsegen` is a parser generator written in Cwhip. A shared grammar model feeds Cwhip, standalone C99, and DOT emitters, with generated AST and visitor APIs. C++ and Python emitters are not implemented. `doc/ARCHITECTURE.md` separates the implemented pipeline from proposal-only features. The tool is still moving. This chapter describes the tested behavior at the pin you choose. It does not promise a final grammar, the C++ backend, or the Python backend as finished book material. The AI example is a slice the existing tests can accept or reject: one diagnostic, one grammar feature, or the start of one missing emitter. It is a proposed change, not a merged one.

**Draw from.** `cparsegen/README.md`. `cparsegen/doc/USAGE.md`. `cparsegen/doc/INPUT-OUTPUT.md`. `cparsegen/doc/ARCHITECTURE.md` "Pipeline" and "Milestone and open decisions." `cparsegen/doc/EXAMPLES.md`.

**Author fills in**

- One grammar small enough to print, and the generated result you will show.
- Which missing piece the AI example takes on. A whole new language backend is larger than this chapter.
- The sentence that tells the reader the syntax may still change after the book is printed.

**Draft**

_How to use cparsegen._

_What is missing._

_Adding one missing piece with an AI._

### Chapter 21. cvm as a confidence test

**Outcome.** The reader can assemble and run one v1 program, name what the virtual machine does not yet include, and follow one AI-sized addition.

**Sections**

1. How to use `cvm`
2. What is missing
3. Adding one missing piece with an AI

**Must stay true.** `cvm` is a stack virtual machine written in Cwhip. The shared bytecode package is `CVM` 1.0.0. `cvm-as` assembles v1 text. Its grammar is generated by `cparsegen` at build time, so a build needs `cparsegen` on `PATH`. After installation, `cvm-run` executes a validated container through public SDK APIs and does not need `cparsegen`. A successful run prints `result: <signed-i32>`. The guest result is not the process exit status. A disassembler and a SmallC backend are not included. Structural validation and runtime checks are distinct, and neither is a security boundary. The short AI example is the disassembler, or a smaller checked slice of it. The SmallC backend is a named absence. This chapter does not build it. The example is proposed work, checked by the `cvm` tests, and it is not already in the tree.

**Draw from.** `cvm/README.md`. `cvm/doc/ASSEMBLER.md`. `cvm/doc/INTERPRETER.md`. `cvm/doc/BYTECODE-ASSEMBLY.md` for the opcode facts you quote. `cvm/doc/BUILDING.md` for the `cparsegen` build dependency.

**Author fills in**

- One assembly program and the `result:` line from a run at the pin.
- How much of a disassembler the AI example is allowed to cover.
- What you will say about the missing SmallC backend so a reader does not expect it in this edition.

**Draft**

_How to use cvm._

_What is missing._

_Adding one missing piece with an AI._

## Front and back matter, when you want them

These are not chapters. Add them after the chapter templates have been edited.

- **Preface.** Who the book is for, the SDK version it matches, and how the introduction, the two main parts, and the confidence tests differ.
- **A note on the tools.** Host packages: CMake, Clang, Python 3.11+, SQLite, and optional GTK 4 / GtkSourceView 5 / GDB. Point at `doc/GETTING_STARTED.md` rather than copying the install chapter into the book.
- **Sample chapter for a proposal.** Choose Chapter 4 or Chapter 10 after you have rewritten it in your own voice. Chapter 4 shows the process. Chapter 10 shows the product.
- **Companion repository.** Scripts that configure the pinned SDK and run the Chapter 14 program, with the expected output checked the same way `tests/cases/` checks a fixture.
