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
