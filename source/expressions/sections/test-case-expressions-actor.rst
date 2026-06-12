Configuration relating to specific actors is defined by means of endpoints and parameters. These can be declared in the following ways:

* **Externally:** Actors may be defined fully in the Test Bed. In this case the test suite simply references actors by their ID (see :ref:`test-suite-deploying`).
* **In the test suite:** Actors can be fully defined in a test suite, listing their endpoints and parameters (see :ref:`test-suite-actors`).
* **During the test session initiation:** Simulated actors participating in messaging transactions with SUTs have their messaging handlers 
  called during test suite initiation at which time they can return endpoints and parameters (see :ref:`test-suite-actors-endpoints-simulated`).
* **In the test case:** Simulated actors participating in messaging transactions with SUTs can define endpoints and parameters as fixed values 
  in the test case (unless already defined during the initiation step - see previous point) (see :ref:`test-case-actors`).

As discussed in :ref:`test-suite-actors-endpoints-simulated`, the latter two cases result in the configured parameters being set for the SUT actor 
by matching the SUTs endpoint name against the endpoint name of the simulated actor's configuration (defined during initiation or statically in the test case).

The resulting configuration from the above sources for all actors are recorded in the test session context as variables. Each actor's configuration
results in a ``map`` being created named using the actor's ID. Configuration for the SUT actor pertinent to simulated actors is stored as additional 
``map`` variables under the SUT actor's ``map``. Each such ``map`` is named using the ID of the corresponding simulated actor. Once in place, these
configuration variables can be referenced and manipulated in exactly the same way as regular variables.

The following example illustrates the creation of session context variables for different types of actor configuration. Consider a test suite defined as
follows:

.. code-block:: xml

    <testsuite>
        <actors>
            <gitb:actor id="Sender">
                <gitb:name>Sender</gitb:name>
                <gitb:endpoint name="info">
                    <gitb:config name="dataVersion" kind="SIMPLE"/>
                </gitb:endpoint>
            </gitb:actor>
            <gitb:actor id="Receiver">
                <gitb:name>Receiver</gitb:name>
            </gitb:actor>
        </actors>
        <testcase id="config_test_1"/>
    </testsuite>

And the "config_test_1" test case is defined as follows:

.. code-block:: xml

    <testcase id="config_test_1">
        <actors>
            <gitb:actor id="Sender" name="Sender" role="SUT"/>
            <gitb:actor id="Receiver" name="Receiver" role="SIMULATED">
                <gitb:endpoint name="info">
                    <gitb:config name="address" kind="SIMPLE">AN_ADDRESS</gitb:config>
                </gitb:endpoint>
            </gitb:actor>
        </actors>
        <variables>
            <var name="temp" type="string"/>
        </variables>
        <steps>
            <!-- 
                Lookup the "dataVersion" property configured for the "Sender". 
            -->
            <assign to="temp">$Sender{dataVersion}</assign>
            <!-- 
                Lookup the "address" property configured by the simulated "Receiver" for the "Sender". 
                This is statically defined here but could also be received from the "Receiver" messaging
                handler as part of the test session's initiation phase.
            -->
            <assign to="temp">$Sender{Receiver}{address}</assign>
        </steps>
    </testcase>

.. index:: SESSION
.. index:: sessionId
.. index:: testCaseId
.. index:: testEngineVersion
.. _test-case-expressions-session-metadata:

Accessing test session metadata
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The test engine automatically makes available **metadata** to your test cases relevant to the **ongoing test session**. This is
information that does not necessarily need to be used but could prove useful for :ref:`logging purposes<tdl-step-log>` or as input to
:ref:`external test service<handlers>` calls.

Test session metadata are included in a special purpose ``map`` named ``SESSION`` that is automatically included in the test
session's context. This map contains child items for each metadata item of type ``string``, specifically:

* ``sessionId``: The unique identifier assigned to the test session.
* ``testCaseId``: The identifier of the test session's test case.
* ``testEngineVersion``: The version of the test engine, matching also the version of the GITB TDL.

Through this map, these metadata elements can be accessed as any other information recorded in the test session context, and in
any scenario where TDL expressions can be used. The following example illustrates logging of metadata, as well as
using it as input to a `remote validator <https://www.itb.ec.europa.eu/docs/services/latest/validation/index.html>`_ via
the :ref:`verify<tdl-step-verify>` step:

.. code-block:: xml

    <log>'Launched session '||$SESSION{sessionId}||' (TDL version: '||$SESSION{testEngineVersion}||')'</log>

    <verify id="validateData" handler="$DOMAIN{validatorAddress}">
        <input name="testCase">$SESSION{testCaseId}</input>
    </verify>

.. _test-case-expressions-step-results:

Checking the result of test steps
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

During the course of a test session you may want to check the result of previous steps. This could be used to determine the flow of execution,
adapt processing, :ref:`display custom messages to users<tdl-step-interact>`, or determine the :ref:`overall output message<test-case-output>` of the test session.

For this purpose you have three special variables maintained in the test session context:

* **STEP_SUCCESS** to check whether specific steps have succeeded.
* **TEST_SUCCESS** to check the overall test session result.
* **STEP_STATUS** to check specific steps' status.

.. note::
    **Stopping on failures:** Although its possible to use these variables in combination with :ref:`if<tdl-step-if>` and :ref:`exit<tdl-step-exit>` steps to conditionally stop test sessions,
    it is not the simplest way to do so. If this is your need it is much simpler to use a step's (or sequence of steps) :ref:`stopOnError<tdl-steps-common-stoponerror>` flag.

.. index:: STEP_SUCCESS
.. _test-case-expressions-step-success:

Checking whether a specific step succeeded
++++++++++++++++++++++++++++++++++++++++++

A typical scenario where you would want to check the result of a specific step would be to avoid showing 
an information popup to the user in case a previous step failed (e.g. via a :ref:`tdl-step-interact` step). Whether or not a step has
succeeded is recorded in a special purpose ``map`` named **STEP_SUCCESS** that contains one key per step identifier mapping to a ``boolean``
value. This value is initially set to "false" and, if the step completes successfully, is set to "true".

The following example illustrates use of this feature to conditionally present a message if a ``receive`` (see :ref:`tdl-step-receive`) step has succeeded:

.. code-block:: xml
    :emphasize-lines: 7

    <!-- Receive a request in step "dataReceive". -->
    <receive id="dataReceive" desc="Receive data" from="Actor2" to="Actor1" handler="HttpMessagingV2">...</receive>
    <!-- Check the step result before showing an information message. -->
    <if desc="Check success">
        <cond>$STEP_SUCCESS{dataReceive}</cond>
        <then>
            <interact desc="Show success message">
                <instruct desc="Messaging was completed successfully!"/>
            </interact>
        </then>
    </if>

In case the step you want to check is contained within a :ref:`scriptlet<scriptlets>`, the step's identifier is prefixed by the identifier from the 
scriptlet's :ref:`call step<tdl-step-call>`. For more information on this specific case :ref:`check here<test-case-expressions-step-status-scriptlets>`.

.. note::
    For ``verify`` steps (see :ref:`tdl-step-verify`) the step ID is directly set in the test session context with a ``boolean`` flag to match the validation result. The
    **STEP_SUCCESS** ``map`` makes this possible for any other step as well (including ``verify`` steps).

.. index:: TEST_SUCCESS
.. _test-case-expressions-test-success:

Checking the current result of a test session
+++++++++++++++++++++++++++++++++++++++++++++

As a complement to the **STEP_SUCCESS** variable, used to check a specific step's result, the **TEST_SUCCESS** variable can be used to check the overall result of the test 
session at any given point. This variable is similarly a ``boolean`` flag that you can check in any constructs that support expressions. Revisiting the example of 
conditionally displaying a user interaction popup, we could use the **TEST_SUCCESS** variable as follows:

.. code-block:: xml
    :emphasize-lines: 3

    <!-- Check the step result before showing an information message. -->
    <if desc="Check success">
        <cond>$TEST_SUCCESS</cond>
        <then>
            <interact desc="Show success message">
                <instruct desc="Messaging was completed successfully!"/>
            </interact>
        </then>
    </if>

Using this variable provides an efficient shorthand in place of separately checking each step's outcome. In addition, it is a useful
abstraction given that it allows you to ignore steps that were skipped.

.. index:: STEP_STATUS

.. _test-case-expressions-step-status:

Checking the status of a specific step
++++++++++++++++++++++++++++++++++++++

Complementing the **STEP_SUCCESS** and **TEST_SUCCESS** variables you may also make use of the **STEP_STATUS** variable that records the
specific status of each step. This can be used for more advanced checks that **STEP_SUCCESS** cannot cover, especially when it is interesting
to know if a step was successful, skipped or failed. A typical case where this is important is when we want to calculate an :ref:`output message<test-case-output>`
explaining the result of a test session. Using **STEP_SUCCESS** for this is not ideal as we need to be able to distinguish failed steps from
steps that were skipped.

The **STEP_STATUS** variable is recorded as a ``map`` that maintains for each step (identified by its ID) the latest applicable status. The status
is recorded as a ``string`` that takes the following values:

    * "COMPLETED", for steps having succeeded.
    * "WARNING", for steps having succeeded with warnings.
    * "ERROR", for steps having failed.
    * "SKIPPED", for steps that were skipped.
    * Empty, for steps that are currently pending.

Note that in case an unknown step ID is looked up the result is an empty string, similar to the case of pending steps. In case the step you want to check 
is contained within a :ref:`scriptlet<scriptlets>`, the step's identifier is prefixed by the identifier from the scriptlet's :ref:`call step<tdl-step-call>`.
For more information on this specific case :ref:`check here<test-case-expressions-step-status-scriptlets>`.

The following example illustrates use of the **STEP_STATUS** variable to determine a test session's output message by checking specific steps for failures:

.. code-block:: xml

    <output>
        <success>
            <default>"The test session completed successfully."</default>
        </success>
        <failure>
            <case>
                <cond>$STEP_STATUS{verifyType} = 'ERROR'</cond>
                <message>"The provided type was invalid."</message>
            </case>
            <case>
                <cond>$STEP_STATUS{verifyContent} = 'ERROR'</cond>
                <message>"The provided content was invalid."</message>
            </case>
            <default>"The test session resulted in a failure. Please check the validation reports and apply required corrections."</default>
        </failure>
    </output>

.. _test-case-expressions-step-status-scriptlets:

Status checks for scriptlet steps
+++++++++++++++++++++++++++++++++

When you need to refer to steps within :ref:`scriptlets<scriptlets>`, the steps' identifiers need to be prefixed with the identifier of the scriptlet's
:ref:`call step<tdl-step-call>` followed by an underscore. This is done to make sure we can distinguish a scriptlet's steps from other scriptlets and
from the top-level test case, as well as across different calls of the same scriptlet. In case scriptlets are in turn used within scriptlets, each call step's identifier is
additionally prefixed to produce the final one. Nonetheless, the identifier used for the step status lookup is always considered relative to the current
scope.

To illustrate how this works, consider the following example. We define a scriptlet that contains a :ref:`verify step<tdl-step-verify>` with 
identifier "validateData". 

.. code-block:: xml

    <scriptlet>
        ...
        <verify id="validateData" handler="$DOMAIN{validatorAddress}">
            <input name="data">$data</input>
        </verify>
        ...
    </scriptlet>

Let's now use this scriptlet within a test case and call it twice via two separate :ref:`call steps<tdl-step-call>`. In this case if we
want to check the status of the ``verify`` step, we will prefix its identifier with the relevant ``call`` steps' identifiers.

.. code-block:: xml

    <testcase>
        ...
        <call id="call1" path="scriptlets/scriptlet.xml"/>
        <call id="call2" path="scriptlets/scriptlet.xml"/>
        ...
        <!-- Log steps' status -->
        <log>$STEP_STATUS{call1_validateData}</log>
        <log>$STEP_STATUS{call2_validateData}</log>
        <!-- Applies also to STEP_SUCCESS -->
        <log>$STEP_SUCCESS{call1_validateData}</log>
        <log>$STEP_SUCCESS{call2_validateData}</log>
        ...
    </testcase>

Now let's consider that we have an additional scriptlet that in turn calls the initial one that contains the ``verify`` step. Within this scriptlet
we refer to the ``verify`` step in a relative manner as follows:

.. code-block:: xml

    <scriptlet>
        ...
        <call id="validate" path="scriptlets/scriptlet.xml"/>
        <log>$STEP_STATUS{validate_validateData}</log>
        ...
    </scriptlet>

In the test case, to refer to the ``verify`` step we would use its identifier prefixed by the identifiers of all intermediate ``call`` steps:

.. code-block:: xml

    <testcase>
        ...
        <call id="call" path="scriptlets/otherScriptlet.xml"/>
        <!--
            The step identifier used for the lookup consists of:
            1. The call step id for the top level scriptlet call ("call").
            2. The call step id for the internal scriptlet call ("validate").
            3. The verify step id ("validateData").
        -->
        <log>$STEP_SUCCESS{call_validate_validateData}</log>
        ...
    </testcase>

.. index:: Templates
.. index:: asTemplate
.. _test-case-expressions-template-files:

Expressions and templates
~~~~~~~~~~~~~~~~~~~~~~~~~

Expression processing handles input expressions using the GITB expression processor. This processor is capable of detecting
variable references in expressions so that it can proceed with appropriate variable lookup and their replacement in 
the resulting output. In simple terms this means that if the input to an expression includes variable references, these will be replaced
if they match variables already present in the test session context at the time of the expression's evaluation. Using this feature,
text content (either imported as an artefact, received from a service call or constructed in the test case itself) can act as a template
that is instantiated with specific values when needed.

.. note::
    **Using templates to generate complex data:** The templating approach listed here as well as subsequent examples assume basic templating needs
    based on simple placeholder replacements. If your templates need to be more complex, for example including loops and conditional blocks, the advised 
    approach is to use the :ref:`TemplateProcessor<handlers-TemplateProcessor>` with `FreeMarker templates <https://freemarker.apache.org/>`_.

As an example consider the following scenario. A XML file is provided in the test suite as an artefact named "metadata-template.xml" that 
serves as a template for metadata responses to be provided by the Test Bed. This file contains variable references as follows:

.. code-block:: xml
    :emphasize-lines: 3,4

    <?xml version="1.0" encoding="UTF-8"?>
    <test:metadata xmlns:test="http://test.org/metadata/1.0/">
        <test:identifier>${identifier}</test:identifier>
        <test:address>${address}</test:address>
    </test:metadata>

The highlighted lines above show use of variables that are expected to be replaced during test execution. In the test case this
replacement occurs as follows:

.. code-block:: xml

    <testcase>
        <imports>
            <!--
                Import the metadata template.
            -->
            <artifact name="metadataTemplate">artifacts/metadata-template.xml</artifact>
        </imports>
        <steps>
            <!--
                Use a system-level custom property as the "address" placeholder.
            -->
            <assign to="address">$SYSTEM{endpointAddress}</assign>
            <!--
                Generate a UUID for the "identifier" placeholder.
            -->
            <process output="identifier" handler="TokenGenerator" operation="uuid"/>
            <!--
                Using the template here triggers the replacement of the placeholders based on the
                existing session context variables. Note here how we set "asTemplate" to true so
                that the assign step processes the result of its expression as a template before
                this is set to its target variable.
            -->
            <assign to="bodyToUse" asTemplate="true">$metadataTemplate</assign>
            <!--
                Send the metadata via HTTP POST to a configured registry service.
            -->
            <send desc="Send system metadata" handler="HttpMessagingV2">
                <input name="uri">$DOMAIN{registryAddress}</input>
                <input name="method">"POST"</input>
                <input name="body">$bodyToUse</input>
            </send>
        </steps>
    </testcase>

The above example considers a template as a static resource that is bundled within the test suite archive and imported in the test case. As mentioned earlier,
templates don't need to be static files included in the test suite. Any text or text-based file that is recorded in the test session's context can also be used.
Examples of such cases could be (full listing :ref:`here<test-case-expressions-where>`):

    * Variables created, populated and/or modified during the course of the test session.
    * Values received from :ref:`processing steps<tdl-processing-steps>` or :ref:`messaging steps<tdl-messaging-steps>`.
    * External configuration properties (:ref:`domain parameters<test-case-expressions-domain>`, 
      :ref:`organisation properties<test-case-expressions-organisation>`, :ref:`system properties<test-case-expressions-system>`
      or :ref:`actor configuration<test-case-expressions-actor>`).

Processing of a variable as a template (i.e. checking it for placeholders and populating them from the context to produce the result) can take place wherever 
expressions are supported. This includes :ref:`assignments<tdl-step-assign>`, :ref:`providing inputs<handlers-inputs-outputs>`, :ref:`conditions<tdl-step-if>`
and receiving or providing information through :ref:`user interactions<tdl-step-interact>`. By default expressions don't perform template processing (i.e. the
expression's result is returned as-is). To make an expression treat its result as a template for placeholder replacement you need to specify the ``asTemplate``
attribute with a value of "true" as we saw in the earlier example. The following examples illustrate use of templates within various expressions:

.. code-block:: xml

    <testcase>
        <steps>
            <!-- Define a template text. -->
            <assign to="templateContent">'The value is ${placeholderValue}'</assign>

            <!-- Set the value to replace the placeholder. -->
            <assign to="placeholderValue">'REPLACED'</assign>

            <!-- Log the placeholder value where the log expression is itself considered as a template. -->
            <log asTemplate="true">'The placeholder value is ${placeholderValue}'</log>

            <!-- Process $templateContent as a template. -->
            <assign to="output1" source="$templateContent" asTemplate="true"/>
            <!-- Process $templateContent as-is. -->
            <assign to="output2" source="$templateContent"/>

            <interact desc="Resulting values">
                <!-- Displays "The value is REPLACED". -->
                <instruct desc="Output1">$output1</instruct>
                <!-- Displays "The value is ${placeholderValue}". -->
                <instruct desc="Output2">$templateContent</instruct>
                <!-- Displays "The value is REPLACED". -->
                <instruct desc="Output3" asTemplate="true">$templateContent</instruct>
                <!-- Displays "The value is ${placeholderValue}". -->
                <instruct desc="Output4">$output2</instruct>
                <!-- Displays "The value is REPLACED". -->
                <instruct desc="Output5" asTemplate="true">$output2</instruct>
            </interact>

            <!-- Input "body" is set using a template from an organisation property. -->
            <send desc="Call service" handler="HttpMessagingV2">
                <input name="uri">$SYSTEM{endpoint}</input>
                <input name="body" source="$ORGANISATION{template}" asTemplate="true"/>
            </send>
        </steps>
    </testcase>

Template processing in expressions is done over the result of the expression. This means that both the expression's content and ``source`` attribute are
considered to produce the result, which is then processed as a template if ``asTemplate`` is set to "true". This is illustrated in the following example:

.. code-block:: xml

    <testcase>
        <steps>
            <!-- Define a XML template (this would usually be imported or provided however). -->
            <assign to="templateXML"><![CDATA['<root><element>${placeholderValue}</element></root>']]></assign>

            <!-- Set the value to replace the placeholder. -->
            <assign to="placeholderValue">'REPLACED'</assign>

            <!-- 
               Process templateXML as a template and use also an XPath expression:
               1. The content from "templateXML" is considered as the source.
               2. Upon the content of "templateXML" we apply the provided XPath expression.
               3. The result of the XPath expression ("${placeholderValue}") is then further processed as a template.
            -->
            <assign to="output1" source="templateXML" asTemplate="true">//*[local-name() = 'element']/text()</assign>

            <interact desc="Resulting values">
                <!-- Displays "REPLACED". -->
                <instruct desc="Output1">$output1</instruct>
                <!-- Displays "<root><element>${placeholderValue}</element></root>". -->
                <instruct desc="Output2">$templateXML</instruct>
                <!-- Displays "<root><element>REPLACED</element></root>". -->
                <instruct desc="Output3" asTemplate="true">$templateXML</instruct>
                <!-- Displays "REPLACED". -->
                <instruct desc="Output4" asTemplate="true" source="$templateXML">//*[local-name() = 'element']/text()</instruct>
            </interact>
        </steps>
    </testcase>

In terms of the placeholders used within templates, you are not limited to using simple variables. You can provide any valid variable reference expression, for example:

    * Variables of type ``map`` or ``list`` (e.g. ``<val>${myMap{myMapKey}}</val>``).
    * :ref:`Domain parameters<test-case-expressions-domain>` (e.g. ``<val>${DOMAIN{myParameter}}</val>``).
    * :ref:`Organisation properties<test-case-expressions-organisation>` (e.g. ``<val>${ORGANISATION{myParameter}}</val>``).
    * :ref:`System properties<test-case-expressions-system>` (e.g. ``<val>${SYSTEM{myParameter}}</val>``).
    * :ref:`Actor configuration<test-case-expressions-actor>` (e.g. ``<val>${myActor{aValue}}</val>``).

.. note::
    **Parameter placeholders in templates:** In the examples presented you may have noticed that a template parameter placeholder is contained
    within a ``${...}`` construct. Within this you are expected to provide a variable expression but **without** the leading ``$`` sign.

    For example, to use variable ``$map{key}`` in a template you would define the placeholder as ``${map{key}}``. Apart from the initial ``$`` sign
    in the expression you should still define any further ones as usual (e.g. if the map's key is also a variable reference such as ``${map{$dynamicKeyValue}}``.

.. _test-case-expressions-where:

Where can expressions be used?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The following table provides an overview of the places where expressions can be used:

.. csv-table::
    :widths: 30, 70
    :header: "Use", "Description"

    Section :ref:`test-case-imports`, Used for the paths to import artifacts from as an alternative to providing fixed path references.
    Section :ref:`test-case-steps`, Optionally a variable reference can be used to define the minimum test case log level.
    Section :ref:`test-case-output`, Used to evaluate message match conditions and produce the resulting message texts.
    Section :ref:`scriptlets_elements_output`, Used to evaluate scriptlet output values.
    Step :ref:`tdl-step-btxn`, Variable references can be used to set ``config`` element values.
    Step :ref:`tdl-step-bptxn`, Variable references can be used to set ``config`` element values.
    Step :ref:`tdl-step-send`, Used to determine ``input`` values. Variable references can be used to set ``config`` element values.
    Step :ref:`tdl-step-receive`, Used to determine ``input`` values. Variable references can be used to set ``config`` element values.
    Step :ref:`tdl-step-listen`, Used to determine ``input`` values. Variable references can be used to set ``config`` element values.
    Step :ref:`tdl-step-process`, Used to determine ``input`` values.
    Step :ref:`tdl-step-if`, Used to define and evaluate the if condition (``cond``).
    Step :ref:`tdl-step-while`, Used to define and evaluate the loop condition (``cond``).
    Step :ref:`tdl-step-repuntil`, Used to define and evaluate the repeat condition (``cond``).
    Step :ref:`tdl-step-foreach`, Variable references can be used to evaluate the loop boundaries (``start`` and ``end``).
    Step :ref:`tdl-step-assign`, Used as the expression to apply. Also a pure variable reference is used in the ``to`` and ``source`` elements.
    Step :ref:`tdl-step-log`, Used as the expression to apply when calculating the log output. Also a variable reference is used in the ``source`` element and optionally to set the log level.
    Step :ref:`tdl-step-verify`, Used to determine ``input`` values. Variable references can also be used to set ``config`` element values and the error level.
    Step :ref:`tdl-step-call`, Used to determine ``input`` values.
    Step :ref:`tdl-step-interact`, Used in the values displayed to (``instruct``) or requested from (``request``) users.
