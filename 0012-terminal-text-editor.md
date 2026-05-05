- Feature Name: terminal_text_editor
- Start Date: 2026-04-09
- RFC PR: N/A
- Twizzler Issue: N/A

# Summary
[summary]: #summary
<!-- One paragraph explanation of the feature. -->

I propose we implement [`femto`](https://github.com/frapa/femto), a rust-based command line text editor, as the first test for future text editor implementations. This text editor has only 300 lines of code, basic functionality based on text editors such as pico and nano, and one dependency. The one dependency is [`termion`](https://github.com/redox-os/termion), specifically it's 1.5.6 version. `termion` uses version 0.2 of `libc` which will need to be mocked in Twizzler's patched version of the repository [`mlibc`](https://github.com/twizzler-operating-system/mlibc), specifically some `termios` and  `ioctl` support. Currently there is some work being done with porting the slightly more complex, but more feature-rich rust-based command line text editor, [`kibi`](https://github.com/ilai-deutel/kibi) in the fork found [here](https://github.com/twizzler-operating-system/kibi/tree/twizzler) being implemented in the [`dbittman-e1000-driver` branch](https://github.com/twizzler-operating-system/twizzler/tree/dbittman-e1000-driver).

# Motivation
[motivation]: #motivation
<!-- Why are we doing this? What use cases does it support? What is the expected outcome? -->

Twizzler is currently lacking any form of development tool to create an application inside the operating system. This means that any application that runs on Twizzler, must be created, edited, compiled, etc. outside of Twizzler. With the introduction of a text editor, we can begin to create and edit of text or code inside Twizzler, which can lead to future projects in compilation of code into applications and eventually having a full development cycle fully inside of Twizzler.

# Guide-level explanation
[guide-level-explanation]: #guide-level-explanation

<!-- Explain the proposal as if it was already included in the system and you were teaching it to another Twizzler programmer. That generally means:

- Introducing new named concepts.
- Explaining the feature largely in terms of examples.
- Explaining how Twizzler programmers should *think* about the feature, and how it should impact the way they use Twizzler. It should explain the impact as concretely as possible.
- If applicable, provide sample error messages, deprecation warnings, or migration guidance.
- If applicable, describe the differences between teaching this to existing Twizzler programmers and new Twizzler programmers. -->

`femto` is a command line text editor based on two popular command line text editors `pico` and `nano`. As such, it functions in a very similar way. Once the Twizzler Operating is started via QEMU, the editor can be opened with the following command:

```sh
femto
```

After executing the command, the editor should take over the screen, and empty buffer for the text is created and you should be able to write text inside it. When saving, the editor will ask you for a file name, and then will save the file in the directory you ran `femto` in.

You can also specify a file to be opened in the first argument of the program. For example, say you wanted to open a file named: `test.txt`.

```sh
femto test.txt
```

This will load the file, allowing you to edit previously created text files.

This is the basic funcitonality of the proposal. The proposal should otherwise have no affects on the operating system or programs within it.

Here are a few error messages a Twizzler programmer would receive when interacting with `femto`.

If more than one argument is provided after `femto`, the following error message will be displayed:

```plain
Error: too many arguments.
usage: femto [FILE]
```

If `termion`, the library `femto` depends on, is not completely implemented then running `femto` will cause a panic, providing the following error message:

```plain
thread 'main' (1) panicked at ...:

Unsupported terminal.: ...
```

# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

<!-- This is the technical portion of the RFC. Explain the design in sufficient detail that:

- Its interaction with other features is clear.
- It is reasonably clear how the feature would be implemented.
- Corner cases are dissected by example.

The section should return to the examples given in the previous section, and explain more fully how the detailed proposal makes those examples work. -->

This proposal should be relatively isolated and should not have any interaction with other features within the operating system. `femto` is just another command line activated application being added to the operating system that enables text editing within the operating system.

# Drawbacks
[drawbacks]: #drawbacks

<!-- Why should we *not* do this? -->

Although I believe implementing `femto` is a step in the right direction, there are many other alternatives that could potentially be better starting points for implementing a command line text editor into rust. For example, the current work being done on KiBi may be a better place to start than trying to implement `femto`.

# Rationale and alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

<!-- - Why is this design the best in the space of possible designs?
- What other designs have been considered and what is the rationale for not choosing them?
- What is the impact of not doing this? -->
I believe that `femto` is the best in the space of possible designs due to its small size and very few dependencies. If the goal is to get a working text editor in Twizzler, then `femto` is a beautiful solution to this. Obviously, as I've stated previosly, KiBi is a another great contender for an initial command line text editor, however, it is a bit more complex and depends a bit more on `libc` than `femto` does.

Going with this simpler option. however, we do leave ourselves with more work in the future when trying to upgrade the text editor to include more quality of life features such as syntax highlighting, config customization, and more keyboard shortcuts.

<details>
    <summary>Currently `femto` only requires the following features from `libc`:</summary>

    Four functions:
    * `ioctl`
    * `tcgetattr`
    * `tcsetattr`
    * `isatty`

    One struct:
    * `termios`

    Three constants:
    * `STDOUT_FILENO`
    * `TIOCGWINSZ`
    * `TCSANOW`

    One type:
    * `c_ushort`
</details>

<details>
    <summary>`kibi` requires the following `libc` features:</summary>

    Six functions:
    * `ioctl`
    * `tcgetattr`
    * `tcsetattr`
    * `sigemptyset`
    * `sigaction`
    * `cfmakeraw`

    Three structs:
    * `termios`
    * `sigaction`
    * `winsize`

    Nine constants:
    * `STDOUT_FILENO`
    * `STDIN_FILENO`
    * `TIOCGWINSZ`
    * `TCSANOW`
    * `TCSADRAIN`
    * `SA_SIGINFO`
    * `SIGWINCH` 
    * `VMIN`
    * `VTIME`

    Four types:
    * `c_int`
    * `c_void`
    * `sighandler_t`
    * `siginfo_t`
</details>

# Prior art
[prior-art]: #prior-art

<!-- Discuss prior art, both the good and the bad, in relation to this proposal.
A few examples of what this can include are:

- Does this feature exist in other operating systems and what experience have their community had?
- For community proposals: Is this done by some other community and what were their experiences with it?
- For other teams: What lessons can we learn from what other communities have done here?
- Papers: Are there any published papers or great posts that discuss this? If you have some relevant papers to refer to, this can serve as a more detailed theoretical background.

This section is intended to encourage you as an author to think about
the lessons from other systems, provide readers of your RFC with a
fuller picture.  If there is no prior art, that is fine - your ideas
are interesting to us whether they are brand new or if it is an
adaptation from other operating systems.

Note that while precedent set by other operating systems is some
motivation, it does not on its own motivate an RFC. -->

Command line text editors exist in almost every major operating system and are generally considered a positive addition.

In another experimental operating system, [Redox OS](https://github.com/redox-os/redox), they took the route of creating a clean-slate editor designed specifically for their OS, inspired by `vim`, called [Sodium](https://github.com/redox-os/sodium).

Something that seems to be exclusive to Twizzler, however, is an editor that deals with persistent objects rather than files. This could lead to some differences with how files are opened and edited. File paths would have to be substituted with object IDs and with persistent objects instead of files, saving might become unnescessary, which may cause issues when dealing with the expectation of being able to discard unsaved changes or undo changes unless a history log is taken into account.

# Unresolved questions
[unresolved-questions]: #unresolved-questions

<!-- - What parts of the design do you expect to resolve through the RFC process before this gets merged?
- What parts of the design do you expect to resolve through the implementation of this feature before stabilization?
- What related issues do you consider out of scope for this RFC that could be addressed in the future independently of the solution that comes out of this RFC? -->

By implementing and writing this RFC, I expect to answer the following questions that are currently unresolved:
* How are Twizzler syscalls implemented?
* How are Twizzler syscalls organized in the repository?
* How do I ensure libc calls are being correctly translated through mlibc and understood by the operating system?
* How do I manipulate the Pseudo Terminal (PTY) in the current Twizzler build?
* How do the different features of command line text editors work?

Issues that I would consider out of scope for this RFC:
* Implenting a compiler to compile code written in a command line text editor
* Extensions like `rust-analyzer` that highlight errors or warnings in written code objects
* Being able to transfer written text objects outside the operating system
* A GUI interface for the text editor


# Future possibilities
[future-possibilities]: #future-possibilities

<!-- Think about what the natural extension and evolution of your proposal
would be and how it would affect the operating system and project as a
whole in a holistic way. Try to use this section as a tool to more
fully consider all possible interactions with the project and system
in your proposal.  Also consider how this all fits into the roadmap
for the project and of the relevant sub-team.

This is also a good place to "dump ideas", if they are out of scope for the
RFC you are writing but otherwise related.

If you have tried and cannot think of any future possibilities,
you may simply state that you cannot think of anything.

Note that having something written down in the future-possibilities section
is not a reason to accept the current or a future RFC; such notes should be
in the section on motivation or rationale in this or subsequent RFCs.
The section merely provides additional information. -->

This feature will hopefully be the start of more terminal manipulation magic. Many utility functions, for example, `htop` also take over the terminal to show properties of the machine running the OS, so by beginning to implement these libc syscalls in Twizzler and understand how Twizzler manages the terminal, we allow for more porting capabilities.

Eventually, the ideas I listed in the "out of scope" section of my unresolved questions section, should also be added, so I refer you to that section for inspiration on where to take this feature if implemented.

Week 6:
Be able to open `femto` in Twizzler and open and write a text file/object.

Week 10:
Look into fleshing out `femto` or implementing `kibi`, see what more advanced editors need to work, patch one of these editors to utilize Twizzler's unique way of storing objects.
