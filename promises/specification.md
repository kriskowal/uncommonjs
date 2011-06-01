
An asynchronous promise loosely represents the eventual result of a
function.  A promise is initially "unresolved" and may eventually become
"resolved".  A resolution can either be "fulfilled" with a value or
"rejected" with a reason, corresponding by analogy to synchronously
returned values and thrown exceptions respectively.  Once a promise has
become resolved, it cannot be resolved again, so a promise's fulfillment
value or rejection reason are guaranteed to be the same across multiple
observations.  Although the identify of the value or reason cannot
change, their properties may be mutable.


Thenable Promises
=================

1.  A promise is an object with a ``then`` function.
    1.  ``then`` accepts a ``fulfilled`` callback and a ``rejected``
        callback.
    1.  ``then`` may be called any number of times to register multiple
        observers.
    1.  The callbacks registered by any ``then`` call must not be called
        after callbacks registered in a later ``then`` call.
    1.  Callbacks should not be called before ``then`` returns.
    1.  ``fulfilled`` and ``rejected`` are both optional arguments.
    1.  If truthy, ``fulfilled`` must accept a value as its only
        argument.
        1.  The ``fulfilled`` callback must be called after the promise
            is fulfilled.
        1.  The ``fulfilled`` callback must not be called more than
            once.
        1.  The ``fulfilled`` callback must not be called if the
            ``rejected`` callback has been called.
    1.  If truthy, ``rejected`` must accept a rejection reason as its
        only argument.
        1.  The ``rejected`` callback must be called after the promise
            has been rejected.
        1.  The ``rejected`` callback must not be called more than once.
        1.  The ``rejected`` callback must not be called if the
            ``fulfilled`` callback has been called.
    1.  ``then`` must return a thenable promise.
        1.  The returned promise must be fulfilled with the value
            returned by either callback.
        1.  The returned promise must be rejected if either callback
            throws an exception.
            1.  The reason for the rejection must be the error thrown by
                the callback.


