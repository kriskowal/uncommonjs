
Assertions
==========

An assertion object must conform to the following specification.
*Note: the ``"assert"`` module identifier is conventionally reserved
for an assertion module.*

1.  The assertion object must have an ``AssertionError``
    constructor function.

    1.  All instances of ``AssertionError`` must be instances of
        ``Error``.

    1.  An ``AssertionError`` must be instantiated with an options
        object describing the failure.

        1.  Assertion options must have a ``message`` string.

        1.  Assertion options must have an ``actual`` value.

        1.  Assertion options must have an ``expected`` value, or
            string describing the expected value or range of values.

        1.  Assertion options may have an ``operator`` string, that
            may represent any JavaScript operator or function name
            used to test whether the actual value met the expectation.

1.  In every assertion function, given its criterion for passing or
    failing,

    1.  If ``this`` has a ``pass`` function property and the criterion
        is met, the assertion function must call ``pass`` with the
        given message.

    1.  If ``this`` has a ``fail`` function property and the criterion
        is not met, the assertion function must call ``fail`` with an
        ``AssertionError`` object with the corresponding ``message``,
        ``actual`` value, ``expected`` value description, and
        ``operator`` if applicable.

    1.  If ``this`` does not have a ``fail`` function property and the
        criteron is not met, the assertion function must throw an
        ``AssertionError`` object with the corresponding ``message``,
        ``actual`` value, ``expected`` value description, and
        ``operator`` if applicable.

1.  The assertion object must have a function
    ``ok(guard, message_opt)``.

    1.  The criterion of ``ok`` is ``!!guard``.

    1.  The corresponding ``operator`` for an ``AssertionError`` is
        ``"=="``.

    1.  The corresponding ``expected`` value for an ``AssertionError``
        is ``true``.

1.  The assertion object must have a function
    ``equal(actual, expected, message_opt)``

    1.  The criterion of ``equal`` is that ``actual == expected``.

    1.  The corresponding ``operator`` for an ``AssertionError`` is
        ``"=="``.

1.  The assertion object must have a function
    ``notEqual(actual, unexpected, message_opt)``

    1.  The criterion of ``notEqual`` is that ``actual != expected``

    1.  The corresponding ``operator`` for an ``AssertionError`` is
        ``"!="``.

1.  The assertion object must have a function
    ``deepEqual(actual, expected, message_opt)``

    1.  The corresponding ``operator`` for an ``AssertionError`` is
        ``"deepEqual"``.

    1.  The criterion for ``deepEqual`` is that:

        1.  If ``expected == actual``, they are deeply equal.

        1.  If the expected value is a ``Date``, they are deeply equal
            only if the actual value is a ``Date`` representing the
            same time.

        1.  If the ``typeof`` ``actual`` or ``expected`` is not
            ``"object"``, they are deeply equal if ``expected ==
            actual``.

        1.  Otherwise, for all objects, including arrays, they are
            deeply equal only if they have the same number of owned,
            enumerable properties, the same property names (although
            not necessarily in the same order), and the respective
            values for each key are deeply equal.

1.  The assertion object must have a function
    ``notDeepEqual(actual, unexpected, message_opt)``

    1.  The corresponding ``operator`` for an ``AssertionError`` is
        ``"notDeepEqual"``.

    1.  The criterion for ``notDeepEqual`` is that the actual and
        expected values do not meet the criterion of ``deepEqual``.

1.  The assertion object must have a function
    ``strictEqual(actual, expected, message_opt)``

    1.  The criterion of ``strictEqual`` is that ``expected ===
        actual``.

    1.  The corresponding ``operator`` for an ``AssertionError`` is
        ``"==="``.

1.  The assertion object must have a function
    ``notStrictEqual(actual, expected, message_opt)``

    1.  The criterion of ``notStrictEqual`` is that ``expected !==
        actual``.

    1.  The corresponding ``operator`` for an ``AssertionError`` is
        ``"!=="``.

1.  The assertion object must have a function
    ``error(callback, Error_opt, message_opt)``
    *Note: in the CommonJS specification, this was named ``throws``,
    however this name was not ergonomic before ECMAScript 5.
    Assertion objects may provide ``throws`` for backward
    compatibility with that specification provided that they note that
    it is deprecated.*

    1.  The criterion for ``error`` is

        1.  If ``Error`` is provided, calling ``callback`` must throw
            an error that is an instance of the given ``Error``.

        1.  If ``Error`` is not provided, calling ``callback`` must
            throw an exception.

    1.  The corresponding ``operator`` for an ``AssertionError`` is
        ``"throws"``.

    1.  The corresponding ``actual`` value for an ``AssertionError``
        is ``undefined`` if callback does not throw an error.

    1.  The corresponding ``actual`` value for an ``AssertionError``
        is the exception thrown if the callback does throw an error.

    1.  The corresponding ``expected`` value for an ``AssertionError``
        is the global ``Error`` if the ``Error`` argument is undefined.

    1.  The corresponding ``expected`` value for an ``AssertionError``
        is the given ``Error`` argument if defined.

1.  Custom assertion objects may provide additional assertion
    functions. *Note: to use an alternate assertion object for logging,
    a test object must specify an ``Assert`` constructor property*


Tests
=====

1.  A test function,

    1.  A test function may accept an assertion object as its first
        argument, indicating that the test may log results rather than
        fail at the first exception.

    1.  A test function of ``length`` 2 must accept a completion
        function, indicating that the test will end when the
        test calls the completion function.

    1.  If a test function does not have ``length`` 2, it may return a
        promise to complete.

        1.  The completion value may be undefined.

1.  A test object,

    1.  May have an ``Assert`` constructor function for providing
        custom assertions in the context of contained tests.

    1.  All owned, enumerable properties of a test object that have
        names that start with ``"test"`` must be test objects or
        functions.


Runner
======

*Note: by convention, the ``"test"`` module in a package is reserved
for a test runner.*

1.  A test runner object must have a function
    ``run(tests, log_opt)``

    1.  If ``log`` is undefined, ``run`` must construct a
        default logger object.

    1.  If ``tests`` has an ``Assert`` constructor function property,
        ``run`` must use this function to create assertion objects.

    1.  For each enumerable property of ``tests`` that starts with
        ``"test"``, ``run`` must execute the corresponding test,
        proceeding to the next test only when the previous ends.

        1.  If the corresponding value is a function,

            1.  **Prepare:** The test runner must create a logger
                object by calling
                ``log.section()``.

            1.  The test runner must create an assertion object and
                connect its ``pass`` and ``fail`` function properties
                to the ``pass`` and ``fail`` properties of the
                constructed logger.

            1.  **Run:** The test runner must call the test
                function with an assertion object and a completion
                function.

            1.  **End:** If a test function does not return a
                promise, the test ends upon returning.

            1.  If the test function returns a promise, the test ends
                when the promise is resolved.
                *Note: resolution is either fulfillment or rejection.*

            1.  If the ``length`` of the test function is ``2``, the
                test must eventually call the completion function and
                must not return a promise.

                1.  The test ends when the test calls the completion
                    function.

            1.  **Fail:** If the test function throws an
                exception, or if the promise returned by the test
                function is rejected,

                1.  If that exception (or reason for rejection) is an
                    instance of the ``AssertionError`` in the
                    assertion object, the runner must call the
                    ``fail`` function of ``log`` with the exception.

                1.  Otherwise, the runner must call the ``error``
                    function of the ``log`` with the given exception
                    or reason for rejection.

            1.  **Pass:** If a test ends without throwing an
                exception or rejecting the returned promise, the
                runner must call the ``pass`` function of the ``log``
                object with the key of the test function property.

        1.  If the corresponding value is an object,

            1.  ``run`` must create a new logger by calling
                ``log.section()``.

            1.  ``run`` must call itself with the test and logger
                object.

    1.  ``run`` must return a promise.

        1.  The promise must be fulfilled when the last contained test
            ends.

        1.  The last contained test of a test object with no
            properties is already ended.

Log
===

1.  A logger object must have a ``pass`` function property.

1.  A logger object must have a ``fail`` function property.

1.  A logger object must have an ``error`` function property.

1.  A logger object must have a ``section`` function property that,
    when called, returns a new logger object.

