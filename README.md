
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

This blurb might be out of date. Go to [this page]() for the latest information and further information about how we approach translation







## How we translate

We will release information about our process once we think it produces good output. Other people may otherwise blindly follow the instructions and fill the internet with poor translations.
But in short, we use Claude and Codex along with software that helps validate the quality.

One particularly useful software is [tracehash](https://github.com/henriksson-lab/tracehash), which helps instrument original + translated code for efficient bug tracking and bit-wise reproducibility.
You can point your LLM to it and it will figure out how to use it


## FAQ?


**Why not just write new software instead of translating?**
1. We could just ask our LLM to write the software we need but it would then just go out and copy it from somewhere, without us knowing the origin. This means unclear copyright, and nobody gets attributed for their contribution
2. Maintaining a competing software is more work, adds code to maintain, and steals citations. It's a lose-lose scenario
3. Written software has been tested, and LLMs tend to produce rather shoddy software. It still cannot compete with hand-crafted code from an engineer. By translating, we can compare our software to a reference to ensure correctness

**Why use LLMs to translate?**
Software like [c2rust](https://github.com/immunant/c2rust) can convert C to Rust 100% faithfully, but at a serious cost: It doesn't look like Rust at all! This causes several problems:
1. Heavy use of "unsafe" means that the memory safety guarantees of Rust are disabled
2. The Rust compiler will not recognize the code. It is designed for "code that looks like Rust", and if it does not, the generated binary will be slower than otherwise
3. Ideally we want to promote Rust as the future language of bioinformatics! And to smoothen the transition, we ideally offer code that is as similar to the original as possible, to provide familiarity with its structure. LLMs are needed for idiomatic rewrites

That said, we are looking into ways of reducing LLM usage, as an algorithm can be proven to generate faithful results, while LLMs not really


