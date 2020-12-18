---
RFC: XXXX
Author: Roni Choudhury
Created: 2020-06-29
Created: 2020-12-09
Last-Modified: 2020-12-09
---

# RFC XXXX: Configuration Handling

One of the core features of our vision for Tinker Engine is the ability to
accept a kind of "meta-configuration" from which concrete configurations are
generated, used to drive individual protocol runs, with results aggregated
together in some way that enables post-hoc data analysis. This RFC lays out some
ideas for some of the core features such a handler will have, along with an
overall architecture for configuration hanlding in Tinker Engine.

## Proposals

This RFC will discuss a few concrete proposals. One is the overall architcture
by which the Tinker Engine runtime receives, processes, and submits
configuration data to a protocol. Another is the method by which a configuration
file can express variation in its parameters. A third is a fallback mechanism
allowing programmatic configuration handling when the declarative methods are
not sufficient for some use case.

### Overall Architecture

Currently, invoking the `tinker` CLI requires specifying a configuration file,
but currently Tinker does nothing with this file. The new behavior will be for
Tinker to load the specified file and parse it as YAML (bearing in mind that
valid JSON is also valid YAML). If the file cannot be opened, or if the contents
do not parse properly, Tinker will exit with an error condition.

The YAML content describes a "meta-configuration" (see below) that requires
processing to turn it into a sequence of configurations, one per experimental
run. This sequence of configurations will be used to construct one protocol per
configuration, running each to complete one trial of the experimental protocol.

This will be accomplished by sending each configuration dictionary to the
protocol initializer (so that client logic can dictate how the configuration is
used to set the protocol up), and then also as an argument to the protocol's
`run()` method (allowing client logic to make use of the configuration data to
direct the protocol runtime).

The design of the particular experiment will be captured in the variation of
configuration values, as expressed via *declarative meta-configuration* or
*programmatic configuration*.

### Declarative Meta-Configuration

In many cases, the user of Tinker Engine will want to run a protocol multiple
times, changing some set of parameters in order to observe the effect those
changes have on the results of the run. In this way, Tinker Engine supports
computational experiments through the use of the scientific method.

The concept of "some set of parameters" is intentionally abstract: this may mean
anything from selecting a set of numeric values for a protocol setting to take
on, to automatically marching linearly through a range of floating point values,
to swapping out entire algorithmic implementations to compare different
approaches. These types of variations will be expressible through specific
meta-configuration syntax within the configuration file schema. Some examples
follow.

#### Combinatorial variation

The most straightforward idea is to provide lists of values for each varying
paramater to assume, and then to produce a stream of configurations visiting
every ordered tuple of such values in their collective product space. For
instance, the following hypothetical snippet of a configuration:

```yaml
a: 0
b:
  values: [1, 2, 3]
c:
  values: [4, 5]
```

would result in the following sequence of six ordered triples for `(a, b, c)`:

```
(0, 1, 4)
(0, 1, 5)
(0, 2, 4)
(0, 2, 5)
(0, 3, 4)
(0, 3, 5)
```

Note that `a = 0` in all six tuples, while `b` and `c` vary combinatorially as
you would expect.

#### Lockstep variation

Combinatorial variation provides "multiplicative" effects, but we may want a
certain kind of "additive" effect as well. The idea here is to vary a set of
parameters as a group. For example:

```yaml
a: 0
b:
  lockstep: foo
  value: [1, 2, 3]
c:
  lockstep: foo
  value: [4, 5, 6]
```

yielding the following `(a, b, c)` ordered triples:

```
(0, 1, 4)
(0, 2, 5)
(0, 3, 6)
```

Note that this requires specifying which "lockstep group" each parameter is a
part of (identified by a string, in this case `foo`). Note also that `a = 0`
once more, as it is specified as a plain value.

#### Ranges

We may want to support different types of range operators. The `value` field in
the foregoing examples takes a YAML list as its argument, explicitly listing the
values to take on. However, something like the following:

```yaml
a: 0
b:
  value:
    sequence:
      start: 1
      stop: 3
      step: 1
c:
  value:
    sequence:
      start: 4
      stop: 6
```

would be the equivalent of the combinatorial example above (with the `step`
parameter defaulting to `1`, as in the case of `c`).

Other ranges can be implemented as needed.

#### SMQTK algorithms

Tinker Engine includes first-class support for SMQTK. A special
meta-configuration option will enable SMQTK algorithms to be used as swappable
values during configuration processing. For example:

```yaml
foo:
  value:
    - smqtk: MyAlgorithm.FooBar
      config:
        foo: 1
        bar: 2
    - smqtk: MyAlgorithm.FooBaz
      config:
        foo: 1
        bar: 2
```

This will cause the `foo` value to take on a SMQTK algorithm instance as
directed by the arguments. The runtime will first check to see that
`MyAlgorithm.FooBar` and `MyAlgorithm.FooBaz` are existing, usable SMQTK
algorithms as part of the validation phase.

In general, directives such as `smqtk` and `values` (and others that will be
defined in future efforts as needed) can be embedded within other values in the
configuration file. For instance, if a SMQTK algorithm requires another SMQTK
algorithm as an initialization parameter, it can be specified the same way as
the preceding examples at the appropriate place:

```yaml
foo:
  smqtk: MyAlgorithm.SomeAlgo
  config:
    foo: 1
    bar:
      smqtk: MyAlgorithm.OtherAlgo
      config:
        baz: 3
```

This example would construct an instance of `OtherAlgo`, then use that algorithm
object to construct an instance of `SomeAlgo`, which would in turn be available
to the protocol.

#### Schema and validation

The meta-configuration schema will be formalized in such a way that the format
can be presented in documentation easily, and so that configurations can be
validated at runtime.

JSON Schema may be the right tool for this. With such a defined schema, Tinker
Engine could validate a configuration file before any further processing is
done, detecting "syntax" errors and other deficiencies, guiding users to a
valid, sensical configuration that can then be submitted for processing and
execution.

There are tools that can convert a JSON schema specification into documentation.
This will ease the documentation burden, both on the developers and users.

#### Simple configuration

As a special case, the meta-configuration can forgo any of these parameter
varying options to produce a single configuration with all values statically
set.

### Programmatic Configuration

As a fallback, the user can forgo using a declarative configuration at all and
instead define a Python generator function that yields one or more simple
configuration values itself. This could be useful if the declarative mechanisms
are not expressive enough to achieve some result.

As a matter of software architecture, the declarative meta-configuration logic
will be implemented as a function of this type, that includes logic for parsing
and interpreting the meta-configuration file, yielding concrete implementations
as appropriate.

## Reference Implementation

Because this RFC is light on specific, concrete implementation direction, a
reference implementation using this document as a guide will be instrumental in
finalizing both the design and the (evolving) implementation thereof. To start,
a simple mechanism that only handles combinatorial variation will be created,
and incremental updates will add further features as needed by the team.
