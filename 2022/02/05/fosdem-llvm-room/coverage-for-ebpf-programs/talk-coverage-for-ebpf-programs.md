theme: Work, 1
autoscale: true
build-lists: true
slidenumbers: true
slidecount: true
slide-transition: fade(0.3)
footer: [@leodido](https://twitter.com/leodido)

## [fit] [bpfcov](https://github.com/elastic/bpfcov)

<br>

# [fit] **Coverage**
# [fit] for **eBPF** programs

![original](assets/img/fosdem_bottom_right_transparent.png)

<br>

## **Leonardo Di Donato** - 05 Feb 2022 @ [FOSDEM 22 - LLVM devroom](https://fosdem.org/2022/schedule/event/llvm_ebpf/)

[.hide-footer: true]
[.slidenumbers: false]

^ Hello everybody!

^ Today, I'm gonna present y'all a tool I've been working on during the past 2 months.

^ Its name is bpfcov.

^ And it's meant to let you obtain source-based code coverage of your eBPF applications actually running in the Linux kernel.

^ Clicking here on the title of the slide you can take a look at the source code of bpfcov on GitHub.

---

![left filtered](assets/img/myself.jpg)

# whoami
<br>

### **Leonardo Di Donato**

### Open Source Software Engineer
### Falco Maintainer
### Senior eBPF Engineer @ Elastic Security

[![inline 20%](assets/img/elastic.png)](https://github.com/elastic/ebpf) [![inline 90%](assets/img/falco-logo-only.png)](https://github.com/falcosecurity/falco) ![inline 25%](assets/img/ebpf-logo.png)

## @**leodido** [![inline fit](assets/img/twitter.png)](https://twitter.com/leodido) [![inline fit](assets/img/github.png)](https://github.com/leodido)

[.hide-footer: true]
[.slidenumbers: false]

^ My name is Leonardo Di Donato, but people usually shorten my name.
So feel free to just call me Leo.

^ What do I love to do?

^ I love to mess with the Linux kernel, I love to fight with BPF VM in it, I love to build open-source tools, libraries. Things like Falco, the de-facto runtime security solution of the CNCF. You can find many of my projects around my GitHub.

^ I also love to share what I learn along the way, so here's why I came up with this talk!

^ In the meantime, you can find me on Twitter, where I tweeted these slides, with the handle leodido.

^ Feel free to follow me, drop me a line, ask questions about eBPF, kernel, Falco, security, whatever! No problem at all.

---

![right](assets/img/html1.png)

# [fit] why?

* Lot of **eBPF** for tracing and security applications out there
* Lot of **developers** approaching eBPF
* No simple way for them to get **coverage** for their **eBPF code running in the Linux kernel**
* **Test** eBPF programs via `BPF_PROG_TEST_RUN`, but not all program types are supported
* Which path my eBPF code took while running in the kernel? Which **code regions or branches** got evaluated and to what?
* General **lack of tooling** in the **eBPF ecosystem**

[.slidenumbers: false]

^ I bet we all have heard so much about eBPF in recent years.

^ Every day we wake up we hear about a new project or application using some eBPF black magic underneath.

^ Data shows how eBPF is fastly becoming the first choice for implementing tracing and security applications.

^ And in one simple twist of fate, a few months ago, I joined Elastic as a Senior eBPF Engineer. In fact, at Elastic we are writing a ton of eBPF programs (with a ton of developers) to supercharge our Security solution.

^ But the problem is that the eBPF ecosystem lacks tooling to make developers' life easier. There are no tools helping developers to clearly understand which path their code took while running in the Linux kernel. Which code regions or - better - code branches are uncovered, and maybe why.

^ I'm sure everyone here is already familiar with code coverage. It generally shows you which lines of code execute. Also, code coverage is usually applied to tests to discover which line gets run and which is not.

^ But even testing the eBPF programs is a pain, let me say it, given that `BPF_PROG_TEST_RUN` does not support all the types of eBPF programs living in the Linux kernel.

---

# Goal 🎯

Gather **source-based code coverage** for our **eBPF applications**.

eBPF is:

* usually written in C
* compiled via Clang to BPF ELF `.o` files
  * [LLVM BPF target]()
* loaded through the `bpf()` syscall
* executed by the eBPF Virtual Machine in the Linux kernel

^ That's why I sat down and wrote bpfcov: a tool to gather source-based coverage info for our eBPF programs running in the Linux kernel.
Whether they are getting loaded through `BPF_PROG_TEST_RUN` or by other ordinary means.

^ Until today, there was no simple way to visualize how the flow of your eBPF program running in the kernel was. Hopefully, there is one starting today.

^ eBPF is usually written in C. That code is then compiled to its specific instruction set thanks to Clang and the underlying LLVM BPF target which outputs a BPF ELF.

^ So, what makes it so different from other C programs? Couldn't we obtain coverage for eBPF programs just the way we do it for normal C programs?

^ Long story short, nope!

^ We can't mainly because the eBPF programs are verified, loaded, and executed in a constrained virtual machine in the Linux kernel that doesn't allow the common LLVM instrumentation needed for code coverage.

^ But, during this talk, I will show you the patches we need to apply to the LLVM IR of our eBPF programs to make it happen!

---

# What's source-based coverage?

TODO? CON IMG DI QUELL'ALTRO TALK?

^ Line-level granularity is sometimes too coarse... Source-based code coverage is more precise. It even precisely counts things like short-circuited conditionals.

^ LLVM's existing coverage tools (llvm-profdata and llvm-cov) generate coverage summaries with very fined-grained code regions, helping us find graps in the code.

^ We wanna coverage reports that do not show only an approximation of what code actually executed. That's why we shoot for source-base code

^ Counter arithmetics and expressions

^ Knows the AST

---

# Source-based code coverage[^1] for C programs

[.column]
```c
#include <stdio.h>
#include <stdint.h>

void ciao()
{
    printf("ciao\n");
}

void foo()
{
    printf("foo\n");
}

int main(int argc, char **argv)
{
    if (argc > 1)
    {
        foo();
        for (int i = 0; i < 22; i++) {
            ciao();
        }
    }
    printf("main\n");
}
```

[.column]
```console
$ clang \
  -fprofile-instr-generate \
  -fcoverage-mapping \
  hello.c \
  -o hello


$ ./hello yay


$ llvm-profdata merge \
  -sparse default.profraw \
  -o hello.profdata


$ llvm-cov show \
  --show-line-counts-or-regions \
  --show-branches=count \
  --show-regions \
  -instr-profile=hello.profdata \
  hello
```

[^1]: for more details visit the [LLVM docs](https://clang.llvm.org/docs/SourceBasedCodeCoverage.html)

[.hide-footer: true]
[.slidenumbers: false]

^ To explain to you how I did bpfcov, I'll try to go over the steps I did to understand what source-based code coverage is, and how LLVM implements it.

^ I started from a dummy C program.

^ So, let's start with it to get an overview of the steps commonly needed to obtain source-based code coverage.

^ First, compile this simple C code with Clang adding the `-fprofile-instr-generate` and `-fcoverage-mapping` flags

^ Now, just execute your binary

^ When the process exits, the runtime instrumented in the binary by the previous flags outputs a profraw file

^ At this point, all is left to you to do is use LLVM's existing coverage tools (llvm-profdata and llvm-cov) to generate coverage summaries with very fined-grained code regions

---

# [fit] Source-based coverage

![left fit](assets/img/hello-srccov.png)

- **Efficient** and **accurate**
- Works with the existing LLVM coverage tools
- Highlights **exact regions** of code (**line:col** to **line:col**) that were skipped or executed
- Counts how many times a condition (**branches**) was taken or not (see lines 16 and 23)
- Tells us what was the **execution path** through the code

[.hide-footer: true]
[.slidenumbers: false]

^ And this is the awesome result we get! You may notice it is very accurate.

^ Looking at it we instantly know that the `if` conditional evaluated to true, and that the `for` cycle inside it iterated 22 times.

^ This kind of code coverage accounts for the finest details, telling us even what was the execution path through the code...

^ This dummy example can't show it but source-based code coverage unveils also things like short-circuited conditionals, expanses macros, and more.

^ This is definitely what I wanted for eBPF programs too!

---

# -fprofile-instr-generate

### Instruments the program functions to collect execution counts

![inline](assets/img/profile-instr-generate.png)

^ So, let's see what the `-fprofile-instr-generate` actually does:
- it creates one `__profc_*` global array per function
  - the size of the array is the number of the counters for that function, see the yellow boxes
  - in fact, we can see that it created a total of 3 `__profc_*` global variables, one for each function we have there
  - you may also have noticed that `__profc_main` has 3 counters, one for each region of the `main` function:
    - the entry of the function, the `if` body, and the `for` body
- such a flag also creates one `__profd_*` global variable for each function
  - these are structs containing various info, like an ID (the ones in red boxes), a reference to the corresponding `__profc_*` array, the address of the instrumented function (and more), which are later used by LLVM to bind the encoded coverage info to the counters
- furthermore, it also creates a private constant `__llvm_prf_nm` containing the names of the functions that are being instrumented

---

![inline](assets/img/profile-instr-generate2.png)

^ Finally, such a flag, also patches the instructions of every function.

^ Look, these are the functions' instructions in LLVM intermediate representation...

^ As you can see in orange, it increments the counters in the right spots, letting LLVM take care of the global state of the registers

^ Notice that this flag also defines global functions like `__llvm_profile_init` and others...

^ Their goal is to initialize the profile at runtime and flush it out at the exit of the process in a profraw file.

^ Among the other things, we will need to strip these functions to have valid eBPF ELF files that we can load in the BPF VM in the Linux kernel... We'll see later.

---

# -fcoverage-mapping
## Generate coverage mappings

![inline](assets/img/coverage-mapping.png)

^ Now, the `-fcoverage-mapping flag.

^ First of all, let me say that this flag does not apply any patch to the program instructions!

^ But it creates one `__covrec_*` global constants for each function in your program

^ They encode the coverage regions, branches, and so on

^ Those are structs that - among other things - most notably contain:
- a LEB128 string, encoding various info about the coverage regions (the ones in purple)
- the ID of the function, the one annotated with a red rectangle
  - it is the same ID contained by `__profd_*` variables, do you remember?
  - its hex representation is also in the name of the `__covrec_*` variable

^ It also creates the `__llvm_coverage_mapping` constant, a struct made of two parts:
- the header, that contains things like the coverage mapping format version
- the data, which contains the file names of the source files

^ If you wanna know all the details, I've put some links at the end of this deck. Now, it's time to move on...

---

# Demystifying the profraw format

1. header
2. data (`__profd_*` variables)
3. counters (`__profc_*` variables)
4. names (`__llvm_prf_nm` constant)

[.build-lists: false]

^ We've seen that when the `hello` binary exits, a `default.profraw` file is created

^ This happens because LLVM injected a runtime that detects the process dying and when it does happen, it outputs such a file

^ We can't do this for eBPF programs, because they are not userspace processes, so we're gonna figure out something...

^ It is necessary to take a look at what the profraw files contain because they are paramount for the success of bpfcov

^ Other than making us able to output something already supported by the LLVM existing coverage tools, it will give us a sense of what we need to support in the BPF ELF for giving coverage support to eBPF programs

^ So, the profraw format is made of 4 parts: a header, and then the data, the counters, and the names part

^ Good!

---

# [fit] Demystifying the profraw header

^ In case you're like me and love to read the code, in the resources slide I put some links you may find interesting...

^ Thank me later!

---

# How / PLAN

^ Why is LLVM pass the correct approach to this in my opinion?

^ Before creating a pass to patch the profiling and coverage instrumentation that LLVM put in my C eBPF programs, I tried out many other approaches...

^ Like patching the Linux kernel, for this goal...

^ Or like loading the BPF ELF, obtaining the function and line information with BPF APIs, and then patching the same BPF ELF after the instructions of any line by incrementing a counter in an eBPF map... It was very clumbersome and very unstable because I had to keep track of the global state of registers. No way.

^ This is when I realized I had to let the compiler do its job...

All of them are clumbersome

Patching? Instructions? Global registries...

^ profraw image

---

# Steps / Usage

---

# Custom eBPF sections
# eBPF globals
# profc, etc. (IR)
# libBPFCov.so
# bpfcov run
# bpfcov gen
# bpfcov cov

---

# Demo

Who wanna read LLVM IR for eBPF with me? 😎

---

# Results

---

# Resources

- The [Coverage Mapping](https://llvm.org/docs/CoverageMappingFormat.html) format
- [Dissecting the coverage mapping sample](https://llvm.org/docs/CoverageMappingFormat.html#dissecting-the-sample)
- The encoding of the coverage mapping values: [LEB128](https://en.wikipedia.org/wiki/LEB128)
- [Demystifying the profraw format](https://leodido.com/demystifying-profraw-format)

^ Here there are the resources I believe you may find useful to go even deeper into this topic
^ Enojoy them!

[.build-lists: false]

---

![fit](assets/img/wyvern.png)

# [fit] Thanks!
### Questions?

![inline fit](assets/img/FOSDEM_logo.png)

[.column]
* [twitter.com/leodido](https://twitter.com/leodido)
* [github.com/leodido](https://github.com/leodido)
* [github.com/elastic/bpfcov](https://github.com/elastic/bpfcov)

[.column]
![inline](assets/img/handwaving.png)

^ We mae it to the end!

^ Thanks everyone for being here and to bear with me!

^ I hope you enjoyed my approach on using the LLVM powers to create a

^ See you soon, let’s hope in person!

^ In the meantime, follow me on Twitter, and let’s keep in touch!

[.build-lists: false]
[.hide-footer: true]
[.slidenumbers: false]