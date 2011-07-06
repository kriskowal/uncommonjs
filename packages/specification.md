
1.  A package is a directory.
1.  A package must contain a ``package.json`` file.
1.  ``package.json`` must be valid JSON.
1.  Particular package registries or catalogs may impose a more strict
    requirements.
1.  Any properties of ``package.json`` not specified here must occur
    under an "overlays" mapping.


Informative Configuration
=========================

1.  In ``package.json``, the following apply to the properties of the
    root object.
    1.  "name" may be the name of the package.
    1.  "version" may be a string representing a semantic version
        <http://semver.org/>.
    1.  "description" may be a one-line description of the package.
    1.  "keywords" is an array of short words that may be used to search
        and find this package in a registry.
    1.  "author" may be a single person.
    1.  "maintainers" may be an array of people.
    1.  "contributors" may be an array of people.
    1.  "licenses" may be an array of licenses.  **This property is not
        legally binding and does not necessarily mean your package is
        licensed under the terms you define in this property**
        1.  The existence of multiple lincenses implies that a user may
            chose to be bound by any one of them, not that every user
            must be bound by all.
    1.  "bugs" may be an email address or a URL where bugs issues may be
        tracked.
    1.  "repositories" may be an array of repositories.
1.  A person can be represented as either a string or an object.
    *Note: this implies that the string must only contain information
    representable with an equivalent object.*
    1.  A person string must begin with a name.
    1.  A person string may then contain one email address in angle
        brackets (``<>``) delimited by a space.
    1.  A person string then may contain one web address in parentheses
        (``()``) delimited by a space.
    1.  A person object must contain a "name" property.
    1.  A person object may contain an "email" property with an email
        address.
    1.  A person object may contain a "web" property with a URL.
1.  A license may be represented either as a URL or an object.
    1.  A license object may have a "type" property, having a value from
        any of the parenthesized open source license abbreviations from
        <http://www.opensource.org/licenses/alphabetical>.
    1.  A license object may have a "url" property
1.  A repository must be represented as an object with "type" and
    "url" properties.
    1.  A repository "type" may be "git", "svn", "hg", or others
        based on their command name.
    1.  A repository may specify a "url" property.
    1.  A repository may spefify a "path" property if the root of
        the project is not the the root of the package.

Normative Configuration
=======================

1.  In ``package.json``, the following apply to the properties of the
    root object.
    1.  "main" may be provided and must be a URL relative to
        ``package.json`` to a JavaScript file, by its full name
        including its extension.
    1.  "overlays" may be provided and must be an object that maps overlay
        names to objects conforming to the same specification as the
        ``package.json``'s root object.
    1.  "directories" may be provided and must be an object that maps
        conventional directory names to configured directory names within
        the package.
        1.  "lib" may be configured
        1.  "packages" may be configured, and is a URL relative to the
            directory containing ``package.json`` where named package
            dependencies may be found.
    1.  "mappings" may be provided and must be an object that maps module
        identifiers to dependencies.
    1.  "registry" may be provided and must be a URL for a package registry.
        *What exists at that URL is beyond the scope of this specification*
    1.  "hash" may be provided and must be the consitent hash of the
        package.
    1.  "seed" may be provided and must be a string that is used to seed the
        consistent hash of the package.
    1.  "manifest" may be provided and must be a recursive directory listing
        of a package's contents in lexicographic order with paths
        represented as URL's relative to ``package.json``
1.  A package dependency may be represented by a string.
    1.  If the string contains "@", it is equivalent to an object with
        "name", "version", and "registry" properties.
        1.  ``@version`` where the name is implied by the mapping
            identifier and the registry is implied by the default
            registry location.
        1.  ``name@version`` where the registry is implied by the
            default registry location.
        1.  ``name@version@registry``
    1.  Otherwise, the dependency is equivalent to an object with a
        "location" property with the dependency as its value.
1.  A package dependency may be represented by an object.
    1.  A dependency may have a "location" property.  *In the absence of
        a location property, a module loader and a package manager may
        agree on a location computed based on the "packages" directory
        and the "name", "version", and "hash" of the package.*
    1.  A dependency may have a "name" property.
        1.  With a "name", a dependency may have a "version" property.
        1.  With a "name", a dependency may have a "registry" property.
    1.  A dependency may have a "capability" property.  The capability
        must be a URL for a specification of that capability, or an
        array of URLs for specifications for that capability.
        1.  Each specification must be sufficient alone to satisfy the
            dependency of the package.  *An array is supported in the
            case where there are multiple URLs of specifications that
            are effectively equivalent for the purpose of the package,
            when supporting package systems do not agree on which URL is
            the canonical version.*
    1.  A dependency may have a "zip" property that is a URL for a ZIP
        file containing the package.
    1.  A dependency may have a "tar" property that is a URL for a TAR
        or gzipped TAR file containing the package.
    1.  The "zip" and "tar" URLs are for the use of package managers and
        must not be used in place of "location" or "name" dependencies
        by module loaders, unless the module loader also serves as a
        package manager.  For compatibility with all module loaders, a
        "location" for a package must be provided or computable from
        some subset of "name", "version", "hash" and the
        ``directories.packages`` configuration.
    1.  A dependency may have an "overlays" property.
        1.  For each applicable overlay name, the mapped object must be
            copied over the dependency.
    1.  A dependency may have a "hash" property, corresponding to the
        consistent hash of the dependency package.
1.  Overlay names are strings that identify a specific implementation of
    this specification or a class of implementations such that
    configuration specific to those implementations may be copied over
    the object containing the overlays.


Linkage
=======

Each package has its own resolved module identifier name-space.  In the
parlance of the Modules specification, this means that each package is
an independent system of modules, linked by explicit configuration to
other packages.

1.  There must not be cyclic dependencies among packages.
1.  If a package has a "name" and the first term of a module identifier
    is that name, that module identifier is equivalent to the rest of
    the identifier, even if the rest of the identifier is ``""`` (empty
    string).
1.  The resolved module identifier ``""`` (empty string) corresponds to
    the main module if it exists.
1.  For each mapping, from longest to shortest, if the first terms of
    the mapping match the first terms of a module identifier
    1.  If there are no subsequent terms in the module identifier, the
        dependency must link the ``""`` (empty string) module from the
        dependency package.
    1.  If there are subsequent terms in the module identifier, the
        dependency must link the module from that package with the
        remaining terms.
1.  If no mapping corresponds to a module identifier, the package must
    link to a module in its own library.
1.  The library of a package when used in a browser consists of the
    contents of the ``lib`` directory of the package, where each module
    identifier corresponds to the lower-case and extensionless URL
    relative to the library directory for all files that have a ``.js``
    extension.
    1.  ``lib`` may be configured to an alternate location with the
        ``directories`` property of ``package.json``.
1.  The library of a package when used in any engine other than the
    browser may incorporate the contents of the ``overlays/OVERLAY/lib``
    directory before the ``lib`` directory, if it exists.
    1.  ``lib`` and ``overlays`` may be configured to alternate
        locations with the ``directories`` property of ``package.json``.
    1.  In each of these directories, any engine other than the browser
        may incoroporate modules with alternate extensions and loaders,
        searching for each extension before moving to the next
        directory.
1.  If a package requires a capability, a package management system can
    loosely link packages that provide a capability and packages that
    consume that capability, such that different packages can provide
    the same capability on different engines.  In particular, it would
    make sense for Node's own API's to be loaded as an explicitly
    requested capability.

A variety of arrangements are possible between package managers and
package-aware module loaders.

1.  The module loader may itself be the package manager, in which case
    it is repsonsible for downloading, executing, and linking the
    packages.  In such cases, the "name", "version", "registry", or
    "location" properties of dependencies might be used.
1.  The module loader and package manager may agree on locations where
    packages may be stored and found.  The module loader may elect to
    use the "location" of a package and compute missing locations based
    on "packages", "name", "version", and "hash", or some combination
    thereof.  The package manager would agree to drop packages in those
    locations.  Alternately, the package manager and module loader might
    agree on a database or file system location scheme based on some of
    those variables.


NPM
===

To maintain a semblance of compatibility with NPM 1.

1.  "dependencies" may be provided and must be an object that maps a
    module identifier and a package with the same name to a version of
    that package from the NPM registry.  These (name/id, version) pairs
    may be translated to mappings with the ``"http://npmjs.org"``
    registry implied.
1.  "directories" may configure the location of "packages".  If a
    "dependencies" property exists, the default "packages" location is
    "node_modules"


Hash
====

1.  The consistent hash for a package is a SHA256 hash in lower-case
    hexadecimal format.
1.  To compute the package's consistent hash
    1.  Digest the "seed" property of ``package.json`` if it exists.
    1.  Digest the "main" property of ``package.json`` if it exists.
    1.  For each mapping in lexicographic order based on the module
        identifier, digest the module identifier, location, name,
        version, registry, and hash, in that order, for each property
        that is defined.
    1.  For each file in a recursive traversal of the package contents
        in lexicographic order, except ``package.json``, digest the
        binary content.


Secure Loading
==============

For a package to be loaded with a secure loader:

1.  A package must provide a "hash".
1.  A package must provide a "manifest".

A secure loader:

1.  Must verify the consistent hash of the package before executing any
    code that links against it.
1.  Must only acknowledge the existence of paths in the manifest.
1.  Must refuse to load a package that lacks a "hash" or "manifest".

