It is also possible to define a test suite as a pure resource holder for commonly used files (scriptlets, imports and
documentation). Such test suites can omit the definition of :ref:`actors<test-suite-actors>` and
:ref:`test cases<test-suite-test-cases>`, including only a :ref:`metadata<test-suite-metadata>` block to provide, at least,
the test suite's name and version.

Test suites that do not include test cases are considered as **hidden** in the GITB software and are not visible to testers.
They are visible to administrators for their management and can only be used by other test suites to load common resources.
