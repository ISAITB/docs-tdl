To make test cases available to the Test Bed they must be wrapped as a **test suite**. A test suite descriptor is needed for this
which in our case will be defined as follows:

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <testsuite id="helloWorld" xmlns="http://www.gitb.com/tdl/v1/" xmlns:gitb="http://www.gitb.com/core/v1/" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
        <metadata>
            <gitb:name>Hello world</gitb:name>
            <gitb:description>Example acting as a hello world for the GITB TDL.</gitb:description>
            <gitb:version>1.0</gitb:version>
        </metadata>
        <!-- 
            Define the actors involved in the test suite. The test cases referencing an actor with the role of SUT (System Under Test)
            must be passed to complete conformance testing.

            Depending on options selected during the test suite's upload, the actor definitions will be created or updated in the target specification.
        -->
        <actors>
            <gitb:actor id="User">
                <gitb:name>User</gitb:name>
                <gitb:desc>The user that will be requested to perform tasks during tests.</gitb:desc>
            </gitb:actor>
        </actors>
        <!-- 
            The list of included test cases (referenced by ID). The ordering of the test cases here defines their execution
            order when running the test suite.
        -->
        <testcase id="testCase1"/>
    </testsuite>

The test suite's **root element** serves a similar purpose as the one of the test case:

.. code-block:: xml

    <testsuite id="helloWorld" xmlns="http://www.gitb.com/tdl/v1/" xmlns:gitb="http://www.gitb.com/core/v1/" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
        ...
    </testsuite>

The element specifies the namespaces we need, as well as the ``id`` attribute with the test suite's identifier. The identifier serves
to uniquely identify the test suite in the Test Bed, and is used to match an existing test suite when making an update. Next is the
``metadata`` element that is defined as follows:

.. code-block:: xml

    <metadata>
        <gitb:name>Hello world</gitb:name>
        <gitb:description>Example acting as a hello world for the GITB TDL.</gitb:description>
        <gitb:version>1.0</gitb:version>
    </metadata>

As in the similarly named test case element, this is used to specify a name, description and version that are used when presenting the
test suite in the Test Bed. Next up is the ``actors`` element:

.. code-block:: xml

    <actors>
        <gitb:actor id="User">
            <gitb:name>User</gitb:name>
            <gitb:desc>The user that will be requested to perform tasks during tests.</gitb:desc>
        </gitb:actor>
    </actors>

This element is needed to define the specification actors referenced by the test suite's test cases. Besides the ``id`` attribute
used by test cases to refer to actors, each actor defines its name and description used for presentation on the Test Bed. The
final point to cover is the ``testcase`` element:

.. code-block:: xml

    <testcase id="testCase1"/>

This is used to refer to our test case, based on the test case's ``id`` attribute, so that it is included in the test suite.
Although in our case we have a single ``testcase`` element, this is in fact **repeated** for as many different test cases we want
to include.
