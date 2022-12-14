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
* `H` is a valid single-letter extension, it specifies the hypervisor
  extension.  See [Greg's post](https://github.com/riscv/riscv-isa-manual/issues/781#issuecomment-1103284237)
  for more details, specifically
  ````
  As mentioned in #781 (comment):

  For Priv-related extensions the naming convention being established for add-ons
  to the Priv spec is "Sxzzz" where 'S' represents all Priv-related extensions,
  'x' represents a category (e.g. 'v' for virt-mem extensions, 'm' for
  machine-level extensions, 'h' for hypervisor extensions, etc.), and 'zzz' is a
  multi-letter name for the individual extension. But the H extension itself
  corresponds to misa.H and has a single-letter 'H' name (like the other
  single-letter extensions that have a corresponding misa letter).

  The extension naming chapter in the ISA manuals remains to be updated
  accordingly.
  ````
* The profile specification says that profile names can substitute for
  extensions, but [Krste's post](https://lists.riscv.org/g/sig-toolchains/message/486)
  clearly says they can substitute for base ISAs as well (sligtly reformatted
  for readibility):
  ````
  > On Nov 3, 2022, at 12:13 PM, Palmer Dabbelt <palmer@dabbelt.com> wrote:
  > On Thu, 03 Nov 2022 11:53:45 PDT (-0700), krste@sifive.com wrote:
  >> The profile strings specify a base ISA and a set of extensions.
  >>
  >> i.e., should not be treated just as extensions if that makes a
  >> difference in toolchain handling.
  >
  > The proposed syntax in GCC is to accept things like `-march=rva22s64zkr`,
  > which places the profile name where base ISAs usually go.  That's forbidden
  > by the ISA string rules and we say -march takes an ISA string, so this
  > risks running into compatibility issues later if more base ISAs are added
  > that somehow conflict with this scheme we just made up.
  >
  > The language that was just added to the spec says "there will be no
  > standard ISA extension with a name that matches the profile naming
  > convention", which means we'd need something like `-march=rv64irva22s64zkr`
  > in order to ensure we avoid those possible issues.  I don't really care all
  > that much about the syntax, just trying to make sure we avoid shooting
  > ourselves in the foot here.

  Please try to be pragmatic and not pedantic here.

  Folks want to write one string RVA22U64<+extensions>

  If they can’t do that, fix the tool chain rules,

  Krste
  ````

### Zihintpause

The `Zihintpause` extension defines a `PAUSE` instruction, which directly
contradicts the generic hint description by disallowing any architectural state
to change.  This means the PC doesn't change, thus resulting in an infinite
loop.  There's no way to use an instruction like this, so software is just
assuming that `PAUSE` instruction increments the PC like a normal instruction.

## Privileged ISA Manual

### WARL Fields

The ISA spec allows for RISC-V implementations that handle illegal values to
WARL fields by returning a different value on every read, including implicit
reads via instructions that depend on the CSR value and explicit CSR access
instructions.  That's not generally something that software can tolerate.
I can't find anything that explicitly states this as being problematic, but, [Allan's
post](https://groups.google.com/a/groups.riscv.org/g/isa-dev/c/vhNMytS1tMk/m/8PRcnyvlBwAJ)
at least hints at some of the issues

````
Just re-read the question again - and I don't think either of us answered it.

Effectively, the illegal value is never written, because if it were, then the
HW would be required to actually use that value until it was read back.

Microarchitecturally, an implementation could write the illegal value, but any
use of it, (including reading it), must never see the illegal value.

So the legal->illegal mapping could be implemented on the input or the output
of the CSR.

A scan chain might be able to see the difference, but nothing else should.
````

## SBI Specification

### Non-Retentive Suspend

The specification provides no guarantees that it is possible to wake up from
non-retentive suspend, see [Anup's
post](https://github.com/riscv-non-isa/riscv-sbi-doc/issues/98) for more
information:
````
Whether the timer interrupt will fire during non-retentive suspend or not is a platform/implementation-specific detail which needs to be discovered from DT/ACPI (similar to what ARM has done). The SBI specification only defines mechanism to enter suspend states.

I had sent out a Linux patch to add HART DT property for this. Maybe we can move in that direction?
(Refer, https://lore.kernel.org/all/20220727114302.302201-1-apatel@ventanamicro.com/)
````
