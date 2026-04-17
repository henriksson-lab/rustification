
## Intro

Our lab has since 2026 started using LLMs. It wasn't an easy step - there's no lack of ethical concerns here, and we take pride in our programming skills.
But the world moves on and it's evolve-or-die: We can't stop the progress. We can only try to move in the least bad direction.

We are delivering increasingly complex bioinformatics pipelines and we are a small number of people, with limited monetary supply and time, against an ecosystem that keeps
asking more out of us. Our use of LLMs was born out of necessity to keep up, fixing the long list of trivial problems, so we can focus our attention on
the really hard ones. Below is a blurb that all our crates will have, while they also point here for the latest one and further info.


## Below is a blurb that we should add to all our crates; it is the latest version
## This is an LLM-mediated faithful (hopefully) translation, not the original code! 

Most users should probably first see if the existing original code works for them, unless they have reason otherwise. The original source
may have newer features and it has had more love in terms of fixing bugs. In fact, we aim to replicate bugs if they are present, for the
sake of reproducibility! (but then we might have added a few more in the process)

There are however cases when you might prefer this Rust version. We generally agree with [this manifesto](https://rewrites.bio/) but more specifically:
* We have had many issues with ensuring that our software works using existing containers (Docker, PodMan, Singularity). One size does not fit all and it eats our resources trying to keep up with every way of delivering software
* Common package managers do not work well. It was great when we had a few Linux distributions with stable procedures, but now there are just too many ecosystems (Homebrew, Conda). Conda has an NP-complete resolver which does not scale. Homebrew is only so-stable. And our dependencies in Python still break. These can no longer be considered professional serious options. Meanwhile, Cargo enables multiple versions of packages to be available, even within the same program(!)
* The future is the web. We deploy software in the web browser, and until now that has meant Javascript. This is a language where even the == operator is broken. Typescript is one step up, but a game changer is the ability to compile Rust code into webassembly, enabling performance and sharing of code with the backend. Translating code to Rust enables new ways of deployment and running code in the browser has especial benefits for science - researchers do not have deep pockets to run servers, so pushing compute to the user enables deployment that otherwise would be impossible
* Old CLI-based utilities are bad for the environment(!). A large amount of compute resources are spent creating and communicating via small files, which we can bypass by using code as libraries. Even better, we can avoid frequent reloading of databases by hoisting this stage, with up to 100x speedups in some cases. Less compute means faster compute and less electricity wasted
* LLM-mediated translations may actually be safer to use than the original code. This article shows that [running the same code on different operating systems can give somewhat different answers](https://doi.org/10.1038/nbt.3820). This is a gap that Rust+Cargo can reduce. Typesafe interfaces also reduce coding mistakes and error handling, as opposed to typical command-line scripting

But:

* **This approach should still be considered experimental**. The LLM technology is immature and has sharp corners. But there are opportunities to reap, and the genie is not going back into the bottle. This translation is as much aimed to learn how to improve the technology and get feedback on the results.
* Translations are not endorsed by the original authors unless otherwise noted. **Do not send bug reports to the original developers**. Use our Github issues page instead.
* **Do not trust the benchmarks on this page**. They are used to help evaluate the translation. If you want improved performance, you generally have to use this code as a library, and use the additional tricks it offers. We generally accept performance losses in order to reduce our dependency issues
* **Check the original Github pages for information about the package**. This README is kept sparse on purpose. It is not meant to be the primary source of information
* **If you are the author of the original code and wish to move to Rust, you can obtain ownership of this repository and crate**. Until then, our commitment is to offer an as-faithful-as-possible translation of a snapshot of your code. If we find serious bugs, we will report them to you. Otherwise we will just replicate them, to ensure comparability across studies that claim to use package XYZ v.666. Think of this like a fancy Ubuntu .deb-package of your software - that is how we treat it

This blurb might be out of date. Go to [this page](https://github.com/henriksson-lab/rustification) for the latest information and further information about how we approach translation







## How we translate / tools

We will release information about our process once we think it produces good output. Other people may otherwise blindly follow the instructions and fill the internet with poor translations.
But in short, we use Claude and Codex along with software that helps validate the quality.


We are designing software to aid faithful and efficient translation. You can point your LLM to them and it will figure out how to use them
* [tracehash](https://github.com/henriksson-lab/tracehash): Helps instrument original + translated code for efficient bug tracking and bit-wise reproducibility. Hash based to be fast; mainly used for quick testing
* [deep-code-comparator](https://github.com/henriksson-lab/deep-code-comparator): Designed to obtain realistic test data for isolated functions, and enable one-shot translation of code from the bottom and up; but it is also useful for rapid optimization of key functions
* [code-complexity-comparator](https://github.com/henriksson-lab/code-complexity-comparator): Pinpoints unfaithful translation in code by comparing complexity and content of functions, using static analysis.


## FAQ?


### Why not just write new software instead of translating?

1. We could just ask our LLM to write the software we need but it would then just go out and copy it from somewhere, without us knowing the origin. This means unclear copyright, and nobody gets attributed for their contribution
2. Maintaining a competing software is more work, adds code to maintain, and steals citations. It's a lose-lose scenario
3. Written software has been tested, and LLMs tend to produce rather shoddy software. It still cannot compete with hand-crafted code from an engineer. By translating, we can compare our software to a reference to ensure correctness

### Why use LLMs to translate?

Software like [c2rust](https://github.com/immunant/c2rust) can convert C to Rust 100% faithfully, but at a serious cost: It doesn't look like Rust at all! This causes several problems:
1. Heavy use of "unsafe" means that the memory safety guarantees of Rust are disabled
2. The Rust compiler will not recognize the code. It is designed for "code that looks like Rust", and if it does not, the generated binary will be slower than otherwise
3. Ideally we want to promote Rust as the future language of bioinformatics! And to smoothen the transition, we ideally offer code that is as similar to the original as possible, to provide familiarity with its structure. LLMs are needed for idiomatic rewrites

That said, we are looking into ways of reducing LLM usage, as an algorithm can be proven to generate faithful results, while LLMs not really

### Can we speed up software by translating it to Rust?

Don't assume this; it can be much faster if you use the final software as a library, but don't expect a basic CLI replacement to be faster. While Rust is faster than R, python etc, it can be hard to beat well-written C++ code that already includes SIMD or assembly code.
But you can get a ton of speed improvement if your tool reads a database and then applies it to input data. If you call the crate as a library (not CLI!), and apply it to a lot of input data, you
can reduce the initial loading cost by a ton.

### Can you help me translate X?**

You can put it as a request in this thread and I will see what I have time for. **If you are an author of a commonly used tool and want to move to Rust, I am happy to help!**


## Floating point is difficult

One challenge in translating code to Rust is that floating point (decimal) math is really hard to translate well. **If you care about p-values, this is important.**

Everyone learn in school that (ab)c = a(bc), but this is not true in computers for floating point. Orders matter. This even worse for addition of decimal numbers.
"YOLO" is a very bad approach for translating any code that deals with numbers as once you are off, you will have to search the whole code base for the reordering error.
This is hard for a human, and it is hard for an LLM. Verbatim translation for day 1 is the only serious solution (and use tracehash when you undoubtedly will still fail).

This has implications for SIMD, which can really help speed up computations. Instead of (((ab)c)d), a CPU can e.g. compute (abcd) in one swoop. But now the order was changed!
There are thus potential tradeoffs to be made when enabling SIMD vs faithful reproduction of the original. **SIMD is typically offered as an optional crate feature because of this.**
If you enable this feature, beware that you might get somewhat different results.

**Bioinformaticians, please don't use p-value cutoffs if you can!!!** If you include genes at p=0.05, then a gene that got 0.051 once, and then 0.049 after SIMD,
this is a huge difference for the results (exaggerated, differences are smaller, but you get the point). If you run multiple steps with cutoffs everywhere, it is better if you
can instead weigh by p-value. This is much more robust to numeric differences. i.e., if you want to compute P(x and y), then do this as P(x)P(y).
This is better than *if P(x<0.05) then check P(y)* -- both more a numerical and statistical standpoint. The future will love you.


## User interfaces are hard to translate

There are many libraries for graphical user interfaces, some only working on certain operating systems. Faithful reproduction of GUIs is usually not a good idea. Ideally a modern
design should also cater to different monitor sizes, and enable adaptive font sizes to aid those with poor eye sight, or needing special readers for aid. We translate GUIs
very losely and more work can be done in this domain.

Testing GUIs is also hard. LLMs only work well if they are allowed to write A LOT of tests, and we also lack the equivalent of a "byte per byte" comparison. GUIs are not fun
to translate.

Pragmatically, we have picked the Rust Iced library as the default. It results in the least code but is not perfect. Default translations don't look good. Instead,
the best trick so far is to include screenshots of the original and let the LLM do some adjustments. Input of GUI translation is most welcome!



