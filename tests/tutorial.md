---
title: Unit Testing Tutorial
layout: default
---
 

### A typical test, failing at the first failed assertion

Having arranged for "assert" and "test" to be modules in my package
for assertions and running tests, I create a test function and run it
if the module is executed as the main module.

    var assert = require("assert");

    function test() {
        assert.ok(false, "it worked"); // it doesn't
        // but we don't get here
        assert.equal(1, 1, "they're equal");
        // and they would have been, but they
        // didn't get a chance.
    }

    if (require.main === module)
        require("test").run(test);

Depending on the default logger, this might look like:

    FAIL it worked
    0 passes, 1 failure, 0 errors


### A test that logs passes and fails instead of stopping at the first failed assertion

    exports['test foo'] = function (assert) {
        assert.ok(false, "it worked"); // it doesn't
        // and we log it, moving on...
        assert.equal(1, 1, "they're equal");
        // and they are, and we log that too
    };

    if (require.main === module)
        require("test").run(exports);

Again, depending on the logger, it might look like:

    FAIL it worked
    PASS they're equal
    0 passes, 2 failures, 0 errors


### Composing test suites with modules.

I can now create a test suite as a module by requiring the foo test as
part of mine.

    exports['test foo'] = require("./foo");
    exports['test bar'] = require("./bar");

    if (require.main === module)
        require("test").run(exports);

Since the ``"foo"`` test module also checks whether it's the main
module before executing, we will only invoke the test runner once.


### An asynchronous test using a completion function

Relying on the accuracy of ``setTimeout``, we'll test the accuracy of
``inABit(callback)``.  We'll accept the assertion module as a first
argument and call the ``done`` function to end the test.

    var tolerance = 100; // miliseconds
    exports['test in a bit'] = function (assert, done) {
        var called = false;
        inABit(function () {
            called = true;
        });
        setTimeout(function () {
            assert.ok(called, "called one second later");
            done();
        }, 1000 + tolerance);
    };


### An asynchronous test using a promise

    var Q = require("q");

    var inABit = function () {
        var deferred = Q.defer();
        setTimeout(deferred.resolve, 1000);
        return deferred.promise;
    }

    var tolerance = 100; // miliseconds

    exports['test a promise'] = function () {
        var start = +new Date();
        return Q.when(inABit(), function () {
            var stop = +new Date();
            var duration = stop - start;
            assert.ok(duration < 1000 + tolerance, 'within tolerance');
        });
    };


### A set of tests with custom assertion functions

    var assert = require("assert");

    exports.Assert = function () {
        var self = Object.create(assert);
        self.inRange = function (actual, min, max, message) {
            var ok = actual >= min && actual <= max;
            if (ok) {
                self.pass(message);
            } else {
                self.fail({
                    message: message,
                    actual: actual,
                    expected: "between " + min + " and " + max,
                    operator: "inRange"
                });
            }
        };
        return self;
    };

    var tolerance = 100;
    exports['test a promise'] = function () {
        var inABit = require("in-a-bit").inABit;
        var start = +new Date();
        return Q.when(inABit(), function () {
            var stop = +new Date();
            var duration = stop - start;
            var min = 1000 - tolerance;
            var max = 1000 + tolerance;
            assert.inRange(duration, min, max, 'within tolerance');
        });
    };


### Running a test with a custom logger

    var Logger = function () {
        var self = {
            passes: 0,
            failures: 0,
            errors: 0,
            pass: function (message) {
                console.log('PASS', message);
                root.passes++;
            },
            fail: function (assertion) {
                console.error(assertion);
                root.failures++;
            },
            error: function (exception) {
                console.error(exception);
                root.errors++;
            },
            section: function () {
                return self;
            },
            report: function () {
                console.log(
                    self.passes, 'passes',
                    self.failures, 'failures',
                    self.errors, 'errors'
                );
            }
        };
        return self;
    };

    if (require.main === module) {
        var log = Logger();
        var allSpecs = require("./all");
        require("test").run(allSpecs, Logger())
        log.report();
    }


### Running a module as a test.

    exports['test foo'] = require.bind(null, './foo-spec');

    if (require.main === module)
        require("test").run(exports);

