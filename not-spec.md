# The not-RISC-V Specification

The RISC-V specifications are unique in a handful of ways, one of which is the
concept wherein language within the specifications is designated as "to be
ignored" by the authors.  As this frequently happens via communication methods
other than the official specification relases it can be a bit tricky to keep
track of everything, so here's my attempt to enumerate it all.

## User ISA Manual

### ISA String Encoding

* The `zicntr`, and `zihpm` extensions should be ignored.  See
  [Andrew's post](https://groups.google.com/a/groups.riscv.org/g/sw-dev/c/QKjQhChrq9Q/m/7gqdkctgAgAJ)
  for more details, specifically
  ````
  > Option 2: who cares about these details
  >
  > Make the CSRs always available, even when zicntr or zihpm are not
  > specified in the march string. Expose the test macros based on the
  > availability of the extension (via march string).
  >
  > Pros: No compatibility issue.
  > Cons: Unexpected behavior (CSRs are available although the compiler
  > knows better); potentially harmful (compiler errors are expected to
  > prevent unsupported code from being generated);

  The RISC-V architects have long been clear that this is the only answer that we
  think is sensible. (It almost seems like you are going to these mailing lists
  because you didn’t like the answer we provided in other forums…)

  We only approved these specific backwards-incompatible changes on the basis
  that they will, in fact, “just work” in practice. We wouldn’t have done so for
  extensions that compilers actually need to target.
  ````
