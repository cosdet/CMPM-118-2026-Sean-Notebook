# Week 5
### What I Did

This week we discussed the Arrakis paper, I got some help from Surendra and Max on decyphering the errors I was getting with building the toolchain. Turns out I just needed to install `meson` and comment out a few lines. I also installed Tailscale on both my MacOS Laptop and the Lab desktop computer to allow for virtual SSH access. This essentially allows me to build the toolchain, make edits, etc. all from the comfort of my dorm! Although, I do like working in the lab with others, it makes me feel more productive. I checked in over the weekend to ensure the toolchain was still building correctly after installing `meson` and made a few more edits to the code after figuring out it was once again building Twizzler for the `aarch64` architechture.

### Troubles

Ran into some troubles with the commit I'm anchoring down on building for both `x86_64` and `aarch64` platforms, but got that sorted. Also had some trouble with `meson` not being an unlisted dependency in the commit.

### Goals And Aspirations

Make a pull request adding `meson` to `init.sh`. Start figuring out what's missing for `femto` on this new commit I've anchored down on.
