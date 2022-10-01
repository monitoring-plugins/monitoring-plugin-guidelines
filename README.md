# Abstract
With the emergence of NetSaint/Nagios at the latest, this system and their successors/clones
have relied on a loose group of programs called "Monitoring Plugins" to do the lower level
task of actually determining the state of particular entity or conduct measurements of certain
values.

This document shall help users and especially developers of those programs as a basis
on how they should be implemented, how they should work and how they should behave.
It encourages the standardization of libraries, Monitoring Plugins and Monitoring Systems,
to reduce the cognitive load on users, administrators and developers, if they work with
different implementations.

These guidelines aim to be mostly as general as possible and not to assume anticipate a special
implementation detail, e.g. the programming language, the install mechanism or the monitoring
system which executes the Monitoring Plugin.

# Language
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
"OPTIONAL" in this document are to be interpreted as described in
BCP 14 [RFC2119] [RFC8174] when, and only when, they appear in all
capitals, as shown here.

# Terminology

## Monitoring Plugin
Is an executable on a _normal_ computer system (meaning something like a commonly occurring system with an operating system
like something bases on Linux, FreeBSD, Windows or something similar)

## Monitoring System
Is a software which, for the scope of this document, executes a *Monitoring Plugin*


# The Monitoring Plugin Interface

## The basic Monitoring Plugin usage
A Monitoring System executes a Monitoring Plugin. The Monitoring Plugin MAY accept parameters in
the form of command line arguments, environment variables or a configuration file (the location of which
MAY in turn be given on the command line or via environment variable).
The Monitoring Plugin then proceeds to execute it's duty and returns the result to the Monitoring System.
Part of the process of returning the result is the termination of the execution of the Monitoring Plugin itself.

## Input Parameters for a Monitoring Plugin

## Ouput of a Monitoring Plugin
The output of a Monitoring Plugin consists of two parts on the first level, the *Exit Code* and
output in textual form on _stdout_.

### Exit Code
The *Monitoring Plugin* MUST make use of the *Exit Code* as a method to communicate a result to
the *Monitoring System*. Since the *Exit Code* is more or less standardized over different systems
as a number with a size of or greater than 256 bit, the following mapping is used:

| *Exit Code* (numerical) | Meaning (short) | Meaning (extended) |
| --- | --- | --- |
| 0 | OK | The execution of the *Monitoring Plugin* proceeded as planned and whatever it test appeared to function properly and the measured values are with their respective thresholds |
| 1 | WARNING | The execution of the *Monitoring Plugin* proceeded as planned and whatever it test appeared to *not* function properly or the measured values are *not* with their respective thresholds. The problem(s) do(es) *not* seem exceptionally grave though and do(es) *not* require immediate attention |
| 2 | CRITICAL | The execution of the *Monitoring Plugin* proceeded as planned and whatever it test appeared to *not* function properly or the measured values are *not* with their respective thresholds. The problem(s) *do(es)* seem exceptionally grave though and *do(es)* require immediate attention |
| 3 | UNKNOWN | The execution of the *Monitoring Plugin* *did not* proceed as planned. The reasons might be manifold, e.g. missing permissions, missing libraries, no available network connection to the destination, etc.. In summary: The *Monitoring Plugin* could *not* determine the state of whatever it should have been checking and can therefore make no reliable statement about it. |
| 4-31 | reserved for future use |

### Textual Output
The original purpose of the output on _stdout_ was to provide human readable information for the user of the *Monitoring System*,
a way for the *Monitoring Plugin* to communicate further details on what happened.
This purpose still exists, but was expanded with the, so called, *perfdata* (performance data) to allow the machine readable
communication of measured values for further processing in the *Monitoring System*, e.g. for the creation of diagrams.

Therefore the further explanation is split into *human readable output* and *perfdata*.

#### Human readable output
This part of the output should give an user information about the state of the test and, in the case of problems, ideally hint what
the origin of the problem might be or what the symptoms are. If the test relies on numeric values, this might be displayed to
give an user more information about the specific problem.
It might consist of one or more lines of printable symbols.

Examples:
```
Remaining space on filesystem "/" is OK

Sensor temperature is within thresholds

Available Memory is too low

Sensore temperature exceeds thresholds
```
are OK, but
```
Remaining space on filesystem "/" is OK ( 62GiB / 128GiB )

Sensor temperature is within thresholds ( 42°C )

Available Memory is too low ( 126MiB / 32GiB )

Sensor temperature exceeds thresholds ( 78°C > 70°C )
```
are better.

Although no strict guidelines for creating this part of the output can really be given, a developer should
keep a potential user in mind. It might, for example, be OK to put the output in a single line if there are
only one or two items of a similar type (think: multiple file systems, multiple sensors, etc.) are present,
but not if there 10 or 100, although this might present a total valid use case.
If there are several different items exists in the output of the *Monitoring Plugin*, furthermore called *partial results*,
they probably SHOULD be given their own line in the output.

#### Performance data
In addition to the human readable part the output can contain machine readable measurement values. These data points
are separated from the human readable part by the "|" symbol which is in effect until the end of the line.
The performance data then MUST consist of space separated single values, these MUST have the following format:

`'label'=value[UOM][;warn[;crit[;min[;max]]]]`

with the following definitions:
 1. _label_ must consist of at least on non-space character, but can otherwise contain any printable characters except for the equals sign ("=") or single quotes ("'").
 If it contains spaces, it must be surrounded by single quotes
 2. _value_ is a numerical value
