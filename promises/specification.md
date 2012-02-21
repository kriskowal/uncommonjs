---
title: Promises Specification
layout: default
---

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

Thenable promises are sufficent for modeling fulfillment and rejection
of promises within a single program.  If promises are needed to model
remote objects between cooperating event loops (like workers, processes,
and network clients, servers, and peers), they can either be extended
with the message passing "sendable promises" system, or assimilated by a
sendable promise manager system.  If a thenable promise is provided by a
system that is suspicious for safety or security reasons, it is
**necessary** to use a promise manager to provide the normative
guarantees of this specification and the stricter guarantees provided by
the promise manager specification.

1.  A promise is an object (*that may be a function*) with a ``then``
    function.
    1.  When using promises, the existence of a ``then`` function must
        be sufficient to distinguish a promise from any other value.
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
1.  A thenable promise may also be a sendable promise.


Thenable Promise Manager
========================

1.  A promise manager is an object with the following functions for
    creating and manipulating promises.
    1.  ``defer(annotation)``
        1.  Annotation is an optional string describing the operation
            being deferred, which may be used by debuggers to assist in
            the visualization of when promises are made and resolved.
        1.  Returns an object with ``promise``, ``resolve``, and
            ``reject`` properties.
        1.  ``promise`` is a promise (the "deferred promise").
        1.  ``resolve`` is a function that accepts a promise or a value.
            1.  Ignores all resolutions after the first resolution.
            1.  If the value is a promise, resolves the deferred promise
                with the resolved promise, such that any observations of
                the resolved promise are forwarded to the deferred
                promise.
            1.  If the value is not a promise, fulfills the deferred
                promise with the resolved value.
        1.  ``reject`` is a function that accepts a rejection reason
            1.  The rejection reason may be any value.
            1.  Resolves the deferred promise with a rejection promise
                for the given reason.
        1.  All functions of the deferred object may be called without
            being bound to the deferred object.
    1.  ``when(value, fulfilled, rejected)``
        1.  ``value`` may be a promise or any other value.
            1.  Non-promise values are equivalent to fulfilled promises
                with that value.
        1.  Arranges for ``fulfilled`` to be called
            1.  if ``fulfilled`` is truthy
            1.  with the fulfilled value as its argument
            1.  not before ``when`` returns
            1.  if and when the value is fulfilled
            1.  at most once
            1.  if ``rejected`` has not been called
        1.  Arranges for ``rejected`` to be called
            1.  if ``rejected`` is truthy
            1.  with the reason for rejection as its argument
            1.  not before ``when`` returns
            1.  if and when the promised value is rejected
            1.  at most once
            1.  if ``fulfilled`` has not been called
        1.  Must return a promise
            1.  that must be resolved if and when ``fulfilled`` or
                ``rejected`` return a value or promise, with that value
                or promise as the resolution.
            1.  that must be rejected if and when ``fulfilled`` or
                ``rejected`` trow an error, with the error as the reason
                for rejection.
    1.  ``ref(value)``
        1.  ``value`` may be a promise or any other value.
            1.  If ``value`` is a promise, returns that promise.
            1.  If ``value`` is not a promise, returns a promise that
                has already been fulfilled with the value.
    1.  ``reject(reason)``
        1.  Returns a promise that has been rejected for the given
            reason.
        1.  The reason may be any value.
    1.  ``isPromise(value)``
        1.  Returns a boolean value of whether the given value is a
            promise.
    1.  ``isResolved(value)``
        1.  Returns a boolean value of whether the given value is a
            resolved promise.
        1.  ``value`` may be any value.
    1.  ``isFulfilled(value)``
        1.  Returns a boolean value of whether the given value is a
            resolved and fulfilled promise.
        1.  ``value`` may be any value.
    1.  ``isRejected(value)``
        1.  Returns a boolean value of whether the given value is a
            resolved and rejected promise.
        1.  ``value`` may be any value.


Sendable Promises
=================

If a promise is a remote object or a proxy for a remote object, in
addition to being able to observe fulfillment and rejection, it is
useful to be able to pipeline "messages" to such promises so that the
remote promise can rapidly dispatch responses to those messages.  It is
also useful to have promises for remote objects and proxies for remote
objects that are not serializable, such as stateful functions or
functions that provide access to capabilities.

1.  A promise is an object (*that may be a function*) with a
    ``promiseSend`` function.
    1.  When using promises, the existence of a ``promiseSend`` function
        must be sufficient to distinguish a promise from any other
        value.
    1.  The ``promiseSend`` function must accept an "operator name" as
        its first argument.
    1.  Operator names are an extensible set of strings. The following
        operators are reserved:
        1.  ``"when"``, in which case the third argument must be a
            rejection callback.
            1.  A rejection callback must accept a rejection reason (any
                value) as its argument.
        1.  ``"get"`` in which case the third argument is a property
            name (string).
        1.  ``"put"`` in which case the third argument is a property
            name (string) and the fourth is a value for the new
            property.
        1.  ``"del"`` in which case the third argument is a property
            name.
        1.  ``"post"`` in which case the third argument is a property
            name and all subsequent arguments are variadic.
        1.  ``"isDef"``
    1.  The ``promiseSend`` function must accept a resolver function as
        its second argument.
        1.  The resolver function may eventually be called with a value
            or a promise as its argument.
    1.  The ``promiseSend`` function may receive variadic arguments.
    1.  The ``promiseSend`` function must return ``undefined``.
1.  A sendable promise may also be a thenable promise.

Any thenable promise can be trivially wrapped with a sendable promise
that implements all of the object-oriented operators by waiting for the
promise to be fulfilled and then performing the corresponding operation.
So, it is only necessary for a promise to implement promiseSend if it is
on the boundary between two processes.


Sendable Promise Manager
========================

A sendable promise manager extends the thenable promise manager system
with functions that can forward messages to a promise, returning
promises for the fulfilled object's response.  It also provides tools
for sending and handling messages, and for annotating an object that is
not serializable.

1.  A sendable promise manager creates and manipulates both sendable
    promises and thenable promises.
1.  A sendable promise manager must support the thenable promise manager
    specification.
1.  A sendable promise manager must support the following additional
    functions.
    1.  ``get``, ``put``, ``del``, ``post``, ``invoke`` are functions.
        1.  These functions must accept an object as their first
            argument.
            1.  The object may be a promise or any other value.
            1.  If object is not a promise, it is treated as a fulfilled
                promise with its value.
        1.  These functions must return a promise.
            1.  The promise must be rejected with the same reason if the
                object promise is rejected.
            1.  The promise must be rejected if the object promise is
                fulfilled with a type that does not support properties.
    1.  ``get(object, name)``
        1.  Must fulfill the returned promise with the named property of
            of the object if and when it is fulfilled.
    1.  ``put(object, name, value)``
        1.  Must fulfill the returned promise with ``undefined`` if and
            when the object is fulfilled and the named property has been
            assigned the value.
        1.  Must reject the returned promise if the object is fulfilled
            and setting the named property to value throws an exception.
    1.  ``del(object, name)``
        1.  Must fulfill the returned promise with ``undefined`` if and
            when the object is fulfilled and an attempt has been made to
            delete the named property .
        1.  Must reject the returned promise with an exception if the
            object is fulfilled and attempting to delete the named
            property throws an exception.
    1.  ``post(object, name, args)``
        1.  Must apply the ``args`` to the named function of object if
            and when it is fullfilled.
            1.  Must resolve the returned promise with the return value
                of that application.
            1.  Must reject the returned promise if that application
                throws an error, using the error as the reason for
                rejection.
    1.  ``invoke(object, name, ...args)``
        1.  Must apply the spread, variadic ``args`` to the named
            function of object if and when it is fullfilled.
            1.  Must resolve the returned promise with the return value
                of that application.
            1.  Must reject the returned promise if that application
                throws an error, using the error as the reason for
                rejection.
    1.  ``keys(object)``
        1.  Must fulfill the returned promise with an array of the owned
            property names of the object if and when it is fulfilled.
    1.  ``makePromise(handlers, fallback)``
        1.  Returns a sendable promise
        1.  Accepts a ``handlers`` object that maps message names to
            handler functions that return values or promises,
            particularly:
            1.  ``when(rejected)``
            1.  ``get(name)``
            1.  ``put(name)``
            1.  ``del(name)``
            1.  ``post(name, args)``
        1.  Accepts a ``fallback(operator, resolve, ...args)`` function
        1.  ``promiseSend(operator, resolve, ...args)`` calls the
            handler with the given operator name from handlers if one
            is truthy.
            1.  Calls the handler with the spreaded, variadic arguments
                only.
            1.  Calls resolve with the value returned by the handler.
        1.  ``promiseSend(operator, resolve, ...args)`` call the
            fallback function if property of the handlers mapping for
            the given operator name is falsy.
            1.  Calls fallback with the operator and the spreaded,
                variadic arguments only.
            1.  Calls resolve with the value returned by the fallback
                function.
    1.  ``send(operator, ...args)``
        1.  Constructs a deferred.
        1.  Calls ``promiseSend(operator, resolve, ...args)``
            1.  Where ``resolve`` is the resolve function of the
                deferred.
            1.  Not before ``send`` returns.
        1.  Returns the deferred promise.
    1.  ``ref(value)``
        1.  Must support ``ref`` as defined by the thenable promise
            manager specification.
        1.  Must handle the following messages (per ``makePromise``):
            1.  ``when(errback)``, ignores the errback and returns
                ``value``.
            1.  ``get(name)``, returns the named property of ``value``.
            1.  ``put(name, value)``, sets the named property to the
                given value of the ref value and returns ``undefined``.
            1.  ``del(name)``, deletes the named property of the
                resolved value and returns ``undefined``.
            1.  ``post(name, args)``, calls the named function of the
                value and returns the returned value.
        1.  Must reject all other messages with the reason ``"Promise
            does not handle OPERATOR"`` where ``OPERATOR`` is the
            message operator.
    1.  ``reject(reason)``
        1.  Must support ``reject`` as defined by the thenable promise
            manager specification.
        1.  Handles the ``"when"`` operator:
            1.  If the ``rejected`` callback is truthy, calls the
                ``rejected`` callback with ``reason`` and returns the
                value that the callback returns.
            1.  Otherwise, returns a rejected promise with ``reason``.
        1.  Handles all other messages by forwarding its own reason for
            rejection.
    1.  ``defer(annotation)``
        1.  Must support ``defer`` as defined by the thenable promise
            manager specification.
        1.  Must eventually forward all messages received to the ``ref``
            promise representing the resolution of its ``promise``.
    1.  ``def(value)``
        1.  Returns a promise.
        1.  ``value`` may be either a promise or any other value.
            1.  If ``value`` is not a promise, it is equivalent to a
                promise returned by ``ref(value)``.
        1.  Must handle the ``"isDef"`` operator by resolving to
            ``undefined``.
        1.  Must forward all other messages received by the returned
            promise to ``value``.
    1.  ``isDef(value)``
        1.  Returns a promise that the ``value`` is a promise wrapped by
            ``def``.

