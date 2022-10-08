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
A _Monitoring Plugin_ MUST expect input parameters as arguments during execution, if any are needed/expected at all. It MAY accept these parameters
given as _environment variables_ and it MAY accept them in a configuration file (with a default path or a path given via arguments or _environment variables_).

In general positional arguments are strongly discouraged.

Some arguments MUST have this predetermined meaning, if they are used:
| Argument (long) | Argument (short version, optional) | Argument | Meaning | optional | can be given multiple times |
| --help | -h | None | "Triggers the help functionality of the _Monitoring Plugin_, showing the individual parameters and their meaning, examples for usage of the _Monitoring Plugin_ and general remarks about the how and why of the _Monitoring Plugin_. SHOULD overwrite all other options, meaning, they are ignored if `--help` is given. The _Monitoring Plugin_ SHOULD exit with state UNKNOWN (3). | no | -- (makes no difference) |
| --version | -V | None | Shows the version of the _Monitoring Plugin_ to allow users to report errors better and therefore help them and the developers. The _Monitoring Plugin_ SHOULD exit with state UNKNOWN (3). | no | -- (makes no difference) |
| --timeout | -t | Integer (meaning seconds) or a time duration string | Sets a limit for the time which a _Monitoring Plugin_ is given to execute. This is there to enforce the abortion of the test and improve the reaction time of the _Monitoring System_ (e.g. in bad network conditions it might be helpful to abort the test prematurely and inform the user about that, than trying forever to do something which won't succeed. Or if soft real time constraints are present, a result might be worthless altogether after some time). A sane default is probably 30 seconds, although this depends heavily on the scenario and should be given a thought during development. If the execution is terminated by this timeout, it should exit with state UNKNOWN (3) and (if possible) give some helpful output in which stage of the execution the timeout occurred. | no | no |
| --hostname | -H | String, meaning either a DNS name, an IPv4 or an IPv6 address of the targeted system | If the _Monitoring Plugin_ targets ONE other system on the network, this option should be used to tell it which one. If the _Monitoring Plugin_ does it's test just locally or the logic does not apply to it, this option is, of course, optional. | yes | yes |
| --verbose | -v | None | Increases the verbosity of the output, thereby breaking the suggested rules about a short and concise output. Can be used to debug the _Monitoring Plugin_ and should there expose internals, intermediate results and so on. | yes | yes |
| --detail | | None | Increases the level of detail in the output, thereby giving the user more information about the result of the test and helping with determining errors and problems or just satisfy some curiosity. SHOULD NOT be used to debug the _Monitoring Plugin_, so there is no need to expose internals. | yes | yes |
| --exit-ok | | The _Monitoring Plugin_ exits unconditionally with OK (0). Mostly useful for the purpose of packaging and testing plugins, but might be used to always ignore errors (e.g. to just collect data). | yes | no |

### Examples
For the execution with `--help`:
```
$ my_check_plugin --help
```
the output might look like this:
```
my_check_plugin version 3.1.4
Licensed under the AGPLv1.
Repository: git.example.com/jdoe/my_check_plugin

This plugin just says hello. It fails if you don't give it a name.

Usage:
 my_check_plugin --name NAME [--greeting GREETING]

Options:
 --help
   this help
 --version
   Shows the version of the plugin
 --name NAME
   if given, uses NAME as a name to greet.
 --greeting GREETING
   if given, uses GREETING instead of Hello.

Examples:
$ my_check_plugin --name Jane
Hello Jane

$ my_check_plugin --greeting Ciao --name Alice
Ciao Alice
```
This imaginary _Monitoring Plugin_ tries to be really helpful here,
displays the version, the license and the upstream repository with the help
(although not necessary), has a short description about the purpose,
lists the options in an easily readable way and even gives some examples.

For the execution with `--version`
```
$ my_check_plugin --version
```
the output might be a bit shorter:
```
my_check_plugin version 3.1.4
```
or even:
```
3.1.4
```
where both show the necessary information.


## Output of a Monitoring Plugin
The output of a Monitoring Plugin consists of two parts on the first level, the *Exit Code* and
output in textual form on _stdout_.

### Exit Code
The *Monitoring Plugin* MUST make use of the *Exit Code* as a method to communicate a result to
the *Monitoring System*. Since the *Exit Code* is more or less standardized over different systems
as an integer number with a width of or greater than 8bit, the following mapping is used:

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
but not if there 10 or 100, although this might present a valid use case.
If there are several different items exists in the output of the *Monitoring Plugin*, furthermore called *partial results*,
they probably SHOULD be given their own line in the output.

#### Performance data
In addition to the human readable part the output can contain machine readable measurement values. These data points
are separated from the human readable part by the "|" symbol which is in effect until the end of the line.
The performance data then MUST consist of space separated single values, these MUST have the following format:

`'label'=value[UOM][;warn[;crit[;min[;max]]]]`

with the following definitions:

 1. _label_ must consist of at least on non-space character, but can otherwise contain any printable characters except for the equals sign (`=`) or single quotes (`'`).
 If it contains spaces, it must be surrounded by single quotes
 2. _value_ is a numerical value, might be either an integer or a floating point number. Using floating point numbers if the value is really discreet SHOULD be avoided. Also the
 representation of a floating point number SHOULD NOT use the "scientific notation" (e.g. `6.02e23` or `-3e-45`), since some systems might not be able to parse them correctly.
 Also values with a base other then 10 SHOULD be avoided (see below for more information on `Byte` values).
 3. _UOM_ is the _Unit of measurement_ (e.g. "B" for _Bytes_, "s" for seconds) which gives more context to the _Monitoring System_. The following constraints MUST be applied:
    1. An _UOM_ of `%` MUST be used for percentage values
    2. An _UOM_ of `c` MUST be used for continuous counters (commonly used for the sum of bytes transmitted on an interface)

 The following recommendations SHOULD be applied:
    1. The _UOM_ for `Byte` values is `B` and although many systems do understand units like
    `KB`,`KiB`, `MB`, `GB`, `TB` they SHOULD be avoided, at the least to avoid the ugly hassle about
    people misinterpreting the *base10* values as *base2* values and the other way round.
    This is also a prime example where floating point number SHOULD NOT be used, since there are
    obviously only integer numbers included.
    2. The _UOM_ for time is `s`, meaning seconds, SI-Prefixes (e.g. `ms` for milli seconds) are allowed if
    necessary or useful, but be aware, that many systems may not understand `μs` for micro seconds and expect
    `us` instead.
    3. In general, SI units and SI prefixes SHOULD be used as _UOM_ if applicable, but the _Monitoring System_
    may not understand them correctly (mostly in uncommon cases), in that cases appropriate workarounds
    MAY be applied on the side of the _Monitoring Plugin_, but it would be nice make the developer
    of the _Monitoring System_ aware of the problem.

 4. _warn_ and _crit_ are the threshold values for this measurement, which may have been given by the user as input, may be hardcoded in the _Monitoring Plugin_
 or may be retrieved from a file or a device or somewhere else during the execution of the tests. The unit used MUST be the same as for _value_.
 These values are not simple numbers, but _range_ expressions.
 5. _min_ and _max_ are the minimal respectively the maximal value the _value_ could possibly be. The unit is the same as for _value_.
 These values can be omitted, if the _value_ is a percentage value, since _min_ and _max_ are always `0` and `100` in this case.

## Range expressions

In many cases thresholds for metrics mark a certain range of values where the values is considered to be good or bad if it is inside or outside.
While for significant number of metrics a upper (e.g. load on unixoid systems) or lower (e.g. effective throughput, free
space in memory or storage) border might suffice, for some it does not, for example a temperature value from a temperature
sensor should be within certain range (let's say 10℃ and 45℃).

Regarding input parameters this might be handled with with options like `--critical-upper-temperature` and `--critical-lower-temperature`,
this presents a problem with the performance data, if only scalar values could be used.
To resolve this situation the _Range expression_ format was introduced, with the following definition:

`[@][start:][end]`
where:
 1. `start` <= `end`
 2. If `start` == 0, then it can be omitted.
 3. If `end` is omitted, it has the "value" of positive infinity.
 4. Negative infinity can be specified with `~`.
 5. If the prefix `@` is NOT given, the value exceeds the threshold if it is OUTSIDE of the range between `start` and `end` (including the endpoints).
 6. If the prefix `@` IS given, the value exceeds the threshold if it is INSIDE the range between `start` and `end` (including the endpoints).
 7. Contrary to the short definition above, an empty _Range expression_ is not a valid one, at least either `start` or `end` must be provided.

### Examples

| Range definition | Exceeds threshold if x...|
| 10 | < 0 or > 10, (outside the range of {0 .. 10}) |
| 10: | < 10, (outside {10 .. ∞}) |
| ~:10 | > 10, (outside the range of {-∞ .. 10}) |
| 10:20 | < 10 or > 20, (outside the range of {10 .. 20}) |
| @10:20 | ≥ 10 and ≤ 20, (inside the range of {10 .. 20}) |
