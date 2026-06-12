Our hello world test will ask the user for her name and then use it to display a greeting.

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <testcase id="testCase1" xmlns="http://www.gitb.com/tdl/v1/" xmlns:gitb="http://www.gitb.com/core/v1/">
        <metadata>
            <gitb:name>Test case 1: Hello world</gitb:name>
            <gitb:description>Test case that asks for the user's name and responds by saying hello.</gitb:description>
            <gitb:version>1.0</gitb:version>
        </metadata>
        <actors>
            <!-- Define which test suite actor is the System Under Test (SUT). -->
            <gitb:actor id="User" role="SUT"/>
        </actors>
        <steps>
            <!-- Present a popup requesting the user's name as a required input. -->
            <interact id="input" desc="Tell me your name">
                <request name="name" desc="What's your name?" required="true"/>
            </interact>
            <!-- Present a second popup (but hide it on the test execution diagram) to respond. -->
            <interact hidden="true">
                <!-- Construct the greeting message based on the user's input. -->
                <instruct desc="Greeting">"Hello " || $input{name} || "!"</instruct>
            </interact>
        </steps>
        <!-- Show a user-friendly message once the test session completes successfully. -->
        <output>
            <success>
                <default>"Test session completed successfully."</default>
            </success>
        </output>     
    </testcase>    

Let's break this down to understand each part. The first point to cover is the test case **root element**:

.. code-block:: xml

    <testcase id="testCase1" xmlns="http://www.gitb.com/tdl/v1/" xmlns:gitb="http://www.gitb.com/core/v1/">
        ...
    </testcase>

This defines the namespaces needed for the GITB TDL, but more interestingly the ``id`` attribute with the test case's
identifier. This will be used in the :ref:`test suite <helloWorld_testSuite>` to include the test case. Next up is
the ``metadata`` section:

.. code-block:: xml

    <metadata>
        <gitb:name>Test case 1: Hello world</gitb:name>
        <gitb:description>Test case that asks for the user's name and responds by saying hello.</gitb:description>
        <gitb:version>1.0</gitb:version>
    </metadata>

This section does not affect the test's execution but defines a name, version number and description that will be used when presenting
the test case. Following this is the ``actors`` element:

.. code-block:: xml

    <actors>
        <!-- Define which test suite actor is the System Under Test (SUT). -->
        <gitb:actor id="User" role="SUT"/>
    </actors>

Every test case needs this element to at least reference one actor (to be defined in the :ref:`test suite <helloWorld_testSuite>`),
as the ``SUT`` (System Under Test). When testing on the Test Bed you are doing so to prove you pass the tests for a given
specification actor. The Test Bed identifies these tests by collecting the test cases referring to the actor as their ``SUT``.

The following ``steps`` element is the core of the test case. It lists test steps as child elements, specifying the actions to take place.
In our case these are two :ref:`interact steps <tdl-step-interact>` for user interactions:

.. code-block:: xml

    <steps>
        <!-- Present a popup requesting the user's name as a required input. -->
        <interact id="input" desc="Tell me your name">
            <request name="name" desc="What's your name?" required="true"/>
        </interact>
        <!-- Present a second popup (but hide it on the test execution diagram) to respond. -->
        <interact hidden="true">
            <!-- Construct the greeting message based on the user's input. -->
            <instruct desc="Greeting">"Hello " || $input{name} || "!"</instruct>
        </interact>
    </steps>

The first ``interact`` step shows a popup to the user asking her to provide her name (a required input). Once completed
and the name is provided, the second ``interact`` step is used to display another popup with a greeting message
based on the provided name. Notice in this second case how we set the step as ``hidden`` to not display it on the 
execution diagram. In addition, the greeting message is built as an :ref:`expression <test-case-expressions>`, using string
concatenation.

The final ``output`` element is used to present a user-friendly message once the test completes:

.. code-block:: xml

    <output>
        <success>
            <default>"Test session completed successfully."</default>
        </success>
    </output> 

This section runs after the test session has completed. It is not strictly required but in practice all test cases should include
such a section with user-friendly success and failure messages as an overview of the result.
