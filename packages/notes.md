---
title: Packages Specification
layout: default
---
 
This specification attempts to assimilate most of the points on
CommmonJS/Packages/1.1.

Node's module loader gives priority to its own built-ins over the
contents and configuration of packages.  This specification clarifies
that the package's name space should come first so that packages do not
unexpectedly break when engines change their built-ins and packages are
not subject to name-space collisions with engines unless they really
need to load those modules.

This specification encourages engines like Node to add a system for
capability based package linkage, particularly for its own API's, so
that packages are encouraged to explicitly request API's from the
engines they support.  The intent is to make it possible for a package
registry to automatically construct a reliable compatibility table which
can be enforced by engines.

Some things that are not used in practice have been removed.

Some requirements have been relaxed, with the intent that individual
package managers will enforce stronger requirements based on their own
design.  Particularly, some package managers will require "name" and
"version" to be provided, while others will only need "location".

Consistent hashing has been defined.

Support for "mappings" has been specified and a translation function for
NPM's "dependencies" has been provided and mandated.  It's my hope that
both properties will be supported, even by NPM eventually, since they
both have merits.

This specification hopes to encourage NPM to eventually relax the name
"node_modules" to "packages", and provides a migration path.

Requirements necessary for browser based loaders, particularly
deterministic package and module locations, have been addressed.

