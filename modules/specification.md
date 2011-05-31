
This specification addresses how modules should be written in order to
be interoperable among a class of module systems that can be both client
and server side, secure or insecure, implemented today or supported by
future systems with syntax extensions.  These modules are offered
privacy of their top scope, facility for importing singleton objects
from other modules, and exporting their own API.  By implication, this
specification defines the minimum features that a module system must
provide in order to support interoperable modules.


Require
-------

1.  ``require`` is a Function
    1.  The ``require`` function accepts a module identifier.
    1.  ``require`` returns the exported API of the foreign module.
    1.  If there is a dependency cycle, the foreign module may not have
        finished executing at the time it is required by one of its
        transitive dependencies; in this case, the object returned by
        ``require`` must contain at least the exports that the foreign
        module has prepared before the call to require that led to the
        current module's execution.
    1.  If the requested module cannot be returned, ``require`` must
        throw an error.
    1.  The ``require`` function may have a ``main`` property.
        1.  This attribute, when feasible, should be read-only, don't
            delete.
    1.  The ``main`` property must either be undefined or be identical
        to the ``module`` object in the context of one loaded module.
    1.  The ``require`` function may have a ``paths`` attribute, that is
        a prioritized Array of path Strings, from high to low, of paths
        to top-level module directories.
        1.  The ``paths`` property must not exist in "sandbox" (a
            secured module system).
    1.  The ``paths`` attribute must be referentially identical in all
        modules.
        1.  Replacing the ``paths`` object with an alternate object may
            have no effect.
        1.  If the ``paths`` attribute exists, in-place modification of
            the contents of ``paths`` must be reflected by corresponding
            module search behavior.
        1.  If the ``paths`` attribute exists, it may not be an
            exhaustive list of search paths, as the loader may
            internally look in other locations before or after the
            mentioned paths.
        1.  If the ``paths`` attribute exists, it is the loader's
            prorogative to resolve, normalize, or canonicalize the paths
            provided.

Module Context
--------------

1.  In a module, there is a free variable ``require``, which conforms to
    the above definiton.
1.  In a module, there is a free variable called ``exports``, that is an
    object that the module may add its API to as it executes.
    1.  modules must use the ``exports`` object as the only means of
        exporting.
    1.  In a module, there must be a free variable ``module``, that is
        an Object.
    1.  The ``module`` object must have a ``id`` property that is the
        top-level ``id`` of the module.  The ``id`` property must be
        such that  ``require(module.id)`` will return the exports object
        from which the  ``module.id`` originated. (That is to say
        module.id can be passed to another module, and requiring that
        must return the original module). When feasible this property
        should be read-only, don't delete.
    1.  The ``module`` object may have a ``uri`` String that is the
        fully-qualified URI to the resource from which the module was
        created.  The ``uri`` property must not exist in a sandbox.


Module Identifiers
------------------

1.  A module identifier is a String of "terms" delimited by forward
    slashes.
1.  A term must be a camelCase identifier, ``.``, or ``..``.
1.  Module identifiers may not have file-name extensions like ``.js``.
1.  Module identifiers may be "relative" or "top-level".  A module
    identifier is "relative" if the first term is ``.`` or ``..``.
1.  Top-level identifiers are resolved off the conceptual module name
    space root.
1.  Relative identifiers are resolved relative to the identifier of the
    module in which ``require`` is written and called.


Unspecified
-----------

This specification leaves the following important points of
interoperability unspecified:

1.  Whether modules are stored with a database, file system, or factory
    functions, or are interchangeable with link libraries.
1.  Whether a PATH is supported by the module loader for resolving
    module identifiers.

