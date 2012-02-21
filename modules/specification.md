
This specification establishes a convention for creating a style of
reusable JavaScript modules and systems that will load and link those
modules.

*   Modules are singletons.
*   Modules have an implicitly isolated lexical scope.
*   Loading may be eager in asynchronous loaders.
*   Execution must be lazy.
*   With some care, modules may have cyclic dependencies.
*   Systems of modules may be isolated but still share the same context.
*   Modules may be used both in browsers and servers.


Guarantees Made by Module Writers
=================================

No requirement in this section implies that a module system must enforce
that requirement.  These are merely requirements that a module must
satisfy in order to function in any module system.

1.  A module must be encoded in UTF-8.
1.  A module must only assume that its scope contains ``require``,
    ``exports``, and ``module`` in addition to the globals provided by
    JavaScript.

    -   *Depending on any other values limits portability to engines
        that do not provide those values and will cause reference errors
        on those systems*

1.  A module must consider that every object in its scope may be
    immutable.

    -   *Depending on the mutability of an object precludes the use of
        the module in a system that shares its global scope among
        mutually suspcious programs, like between mashups in a secured
        JavaScript context.*

    -   *It may be acceptable to upgrade an object in scope (shim) if it
        does not provide the needed, specified behavior, but this will
        limit portability to secure contexts with legacy
        implementations.*

1.  A module must only depend on the specified behavior of values in
    scope.

    -   *Depending on any extension to the specified behavior limits
        portability.*

1.  All calls to ``require`` must be given a single string literal as
    the argument, or the value of ``require.main`` if it is defined.

    -   *If the argument to ``require`` is not a string, the dependency
        cannot be discovered and asynchronously loaded before executing
        the module.  This would limit portability to systems that read
        the text of a module to discover dependencies and load them
        asynchronously before execution, which is necessary for
        portability to browsers and other systems that only support
        asynchronous loading.*

    -   *``require.main`` is guaranteed to already have been loaded, so
        it is a reasonable exception to the rule that all dependencies
        must be discoverable without executing the module.*

1.  All calls to ``require`` must be by the name ``require``.

    -   *If ``require`` takes on a different name, the dependency will
        not be discoverable without executing the module.*

1.  All calls to ``require`` must be given a module identifier as the
    argument.

    -   *Some module loaders may accept values that are not module
        identifiers.  Depending on this behavior inside a module limits
        the portability of the module, particularly to systems that use
        packages to isolate the module identifier name space for a set
        of modules.*


Guarantees Made by Module Interpreters
======================================

1.  The top scope of a module must not be shared with any other module.

    -   *All ``var`` declarations in a module will only be accessible
        within that module and will not collide with declarations in
        other modules*

1.  A ``require`` function must exist in a function's lexical scope.

    1.  The ``require`` function must accept a module identifier as its
        first and only argument.

    1.  ``require`` must return the same value as ``module.exports`` in
        the identified module, or must throw an exception while trying.

    1.  ``require`` may have a ``main`` property.

        1.  The ``require.main`` property may be read-only and
            non-configurable.

        1.  The ``require.main`` property must be the same value as
            ``module`` in the lexical scope of the first module that
            began executing in the current system of modules.

        1.  The ``require.main`` property must be the same value in
            every module.

    1.  ``require`` must have a ``resolve`` function.

        1.  ``resolve`` accepts a module identifier as its first and only
            argument

        1.  ``resolve`` must return the resolved module identifier
            corresponding to the given module identifier relative to
            this module's resolved module identifier.

1.  An ``exports`` object must exist in a function's lexical scope.

    1.  the ``exports`` object must initially be the same value as
        ``module.exports``.

1.  A ``module`` object must exist in a function's lexical scope.

    1.  The ``module`` object must have an ``id`` property.

        1.  The ``module.id`` property must be a module identifier such
            that ``require(module.id)`` must return the
            ``module.exports`` value when called in this or any module
            in the same system of modules.

        1.  The ``module.id`` property may be read-only and
            non-configurable.

    1.  The ``module`` object must have an ``exports`` property.

        1.  The ``module.exports`` property must initially be an empty
            object.

        1.  The ``module.exports`` property must be writable and
            configurable.

    1.  The ``module`` object may have a ``location`` absolute URL.

    1.  The ``module`` object may have a ``directory`` absolute URL.

        1.  The ``directory`` must be the directory containing the
            ``location``.


Module Identifiers
==================

1.  A module identifier is a string of "terms" delimited by forward
    slashes.

1.  A term is either:

    1.  any combination of lower-case letters, numbers, and hyphens,
    1.  "``.``", or
    1.  "``..``"

1.  Module identifiers should not have file-name extensions like
    ``.js``.

1.  Module identifiers may be "relative" or "resolved".  A module
    identifier is "relative" if the first term is ``.`` or ``..``.

1.  Top-level identifiers are resolved relative to ``""``.

1.  The ``require`` function in each module resolves relative
    identifiers from the corresponding ``module.id``.

1.  To resolve any path of module identifiers,

    1.  An array of terms must be initialized to an empty array.

    1.  For each module identifier in the path of identifiers,

        1.  Pop off the last term in the array, provided one exists.

        1.  For each term in a module identifier in order,

            1.  Take no action if the term is ``"."``.

            1.  Pop a term off the end of the array if the term is
                ``".."``.

            1.  Push the term on the end of the array otherwise.

    1.  The array of terms must be joined with forward slashes, ``"/"``
        to construct the resulting "resolved" identifier.


Unspecified
===========

This specification leaves the following important points of
interoperability unspecified:

1.  Whether modules are stored with a database, file system, or factory
    functions, or are interchangeable with link libraries.

1.  Whether ``require`` accepts values that are not module identifiers
    as specified here.

