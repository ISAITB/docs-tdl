.. index:: Test cases
.. _test-case:

Test Cases
===============================

Overview
--------

Test cases are the means by which a specific testing scenario is implemented. One or more test cases form the content of a test suite.
The following example represents a complete, simple test case for the validation of an invoice that is uploaded by a user.

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <testcase id="UBL_invoice_validation_test_1" xmlns="http://www.gitb.com/tdl/v1/" xmlns:gitb="http://www.gitb.com/core/v1/" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.gitb.com/tdl/v1/ ../../gitb_tdl.xsd">
        <metadata>
            <gitb:name>UBL_invoice_validation_test_1</gitb:name>
            <gitb:type>CONFORMANCE</gitb:type>
            <gitb:version>1.0</gitb:version>
            <gitb:description>Test case to verify the correctness of a UBL invoice. The invoice is provided manually through user upload.</gitb:description>
        </metadata>
        <imports>
            <artifact type="schema" encoding="UTF-8" name="UBL_Invoice_Schema_File">artifacts/UBL/maindoc/UBL-Invoice-2.1.xsd</artifact>
            <artifact type="object" encoding="UTF-8" name="BII_CORE_Invoice_Schematron_File">artifacts/BII/BII_CORE/BIICORE-UBL-T10-V1.0.xsl</artifact>
        </imports>
        <variables>
            <var name="file_content" type="object"/>
        </variables>
        <actors>
            <gitb:actor id="User" name="User" role="SUT"/>
        </actors>
        <steps>
            <!-- 
                Step 1. Request the user to upload the UBL invoice.
            -->
            <interact desc="UBL invoice upload" with="User">
                <request desc="Upload the UBL invoice to validate" with="User" contentType="BASE64">$file_content</request>
            </interact>
            <!-- 
                Step 2. Validate the uploaded invoice.
            -->
            <verify handler="XSDValidator" desc="Validate invoice against UBL 2.1 Invoice Schema">
                <input name="xmldocument">$file_content</input>
                <input name="xsddocument" source="$UBL_Invoice_Schema_File"/>
            </verify>
            <verify handler="SchematronValidator" desc="Validate invoice against BII2 CORE restrictions for Invoice Transaction">
                <input name="xmldocument">$file_content</input>
                <input name="schematron" source="$BII_CORE_Invoice_Schematron_File"/>
            </verify>
        </steps>
        <output>
            <failure>
                <default>"The test session resulted in a failure. Please check the validation reports and apply required corrections."</default>
            </failure>
        </output>
    </testcase>

The following table provides an overview of the attributes and child elements that a ``testcase`` may have. A more detailed discussion per case 
follows in the subsequent sections.

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @id, yes, A string to uniquely identify the test case by. This is referenced in the test suite XML.
    metadata, yes, A block containing the metadata used to describe the test case.
    namespaces, no, An optional set of namespaces to define the expression languages used in the test case.
    imports, no, An optional set of imports used to load additional resources from the test suite.
    preliminary, no, An optional set of user interaction steps to display before the test session starts.
    variables, no, An optional set of variables that are used in the test case.
    actors, yes, The set of actors that this test case refers to.
    steps, yes, The sequence of steps that this test case foresees.
    output, no, Definition of an output message to display for the overall test session.
    scriptlets, no, Optional named groups of test steps which can be used within the test case.

Elements
--------

We will now see how a test case breaks down into its individual sections and discuss the purpose of each.

.. index:: Metadata (Test cases)
.. _test-case-metadata:

Metadata
~~~~~~~~

The ``metadata`` element is basically the same as the one defined for the test suite :ref:`test-suite-metadata`. Its purpose is to provide basic information 
about the test case to help users understand its purpose. Its structure is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    name, yes, The name of the test case that is used to identify it to users.
    type, no, Either "CONFORMANCE" (the default) or "INTEROPERABILITY". "INTEROPERABILITY" is used when more than one actor are defined as SUTs.
    version, yes, A string that indicates the test case's version.
    authors, no, A string to indicate the test case's authors.
    description, no, A string to provide a user-friendly description of the test case that is displayed to users.
    published, no, A string acting as an indication of the test case's publishing time.
    lastModified, no, A string acting as an indication of the last modification time for the test case.
    documentation, no, Rich text content that provides further information on the current test case.

.. index:: documentation (test case)

The ``documentation`` element complements the test case's ``description`` by allowing the author to include extended rich text documentation as HTML. The structure of this element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    import, no, A reference to a separate file within the test suite archive that defines the documentation content.
    encoding, no, In case an ``import`` reference is defined this can be used to specify the file's encoding. If not provided ``UTF-8`` is considered.

Using the above attributes to specify a reference to a separate file is not mandatory. The documentation's content can also be provided as the element's text content,
typically enclosed within a CDATA section if this includes HTML elements (in which case the ``import`` and ``encoding`` attributes are omitted).

This documentation can provide further information on the context of the test case, diagrams or reference information that are useful to understand how it is to be completed or its purpose within the
overall specification. The content supplied supports several HTML features:

    * Structure elements (e.g. headings, text blocks, lists).
    * In-line styling.
    * Tables.
    * Links.
    * Images.

The simplest way to provide such information is to enclose the HTML content in a CDATA section to ensure the XML remains well-formed. The
following sample provides an example of this approach:

.. code-block:: xml

    <testcase id="TS1-TC1" xmlns="http://www.gitb.com/tdl/v1/" xmlns:gitb="http://www.gitb.com/core/v1/">
        <metadata>
            <gitb:name>TS1-TC1</gitb:name>
            <gitb:version>1.0</gitb:version>
            <gitb:description>A short description of the test case to offer a short summary of its purpose.</gitb:description>
            <gitb:documentation><![CDATA[
                <p>Extended documentation for test case <b>TS1-TC1</b></p>
                <p>This is an example to support the <a href="https://www.itb.ec.europa.eu/docs/tdl/latest">GITB TDL docs</a>.</p>
            ]]></gitb:documentation>
        </metadata>    
        ...
    </testcase>

Note that documentation such as this is also supported for:

    * The overall :ref:`test suite<test-suite-metadata>`.
    * Individual :ref:`test case steps<tdl-steps-common-documentation>`.

.. note::
    **GITB software support:** The test case ``type`` must currently be set to "CONFORMANCE" (the default value) as the
    "INTEROPERABILITY" type is not supported. Finally, the ``version``, ``authors``, ``published`` and ``lastModified`` values are recorded but never used or displayed.

.. index:: Namespaces (Test cases)
.. _test-case-namespaces:

Namespaces
~~~~~~~~~~

The ``namespaces`` optional element is used to define one or more expression languages that are used in test case constructs that support them. This needs to
be done when expressions are used that should not be processed using the default XPath 1.0 language. A detailed discussion on GITB expressions as well 
as where and how you can use them is provided in :ref:`test-case-expressions`. Each referred expression language is defined in a ``namespace`` element with
the following structure:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    ns, yes, One or more elements to define the expression languages used in the test case. Each ``ns`` element must specify a ``prefix`` attribute to define the namespace prefix.

The values specified in the ``prefix`` attributes define how each expression language is referenced within the test case. As an example consider a test case that needs to 
check a number variable ``var`` and set a ``result`` variable with "result1" if ``var`` is 1 or with "result2" otherwise. Simple if-else constructs like this are a good
example of a shortcoming of XPath 1.0 as they are not natively supported and need a non-intuitive workaround based on string concatenation. A simpler approach would be to use
a scripting language more adapted to this kind of operation such as JavaScript. The following test case illustrates this assignment using both the default XPath 1.0 and
JavaScript:

.. code-block:: xml

    <testcase>
        <namespaces>
            <ns prefix="JavaScript"/>
        </namespaces>
        <variables>
            <var name="var" type="number"/>
            <var name="result" type="string"/>
        </variables>
        <steps>
            <!-- 
                Assignment using the default XPath 1.0.
            -->
            <assign to="$result">concat(substring('result1', 1 div number(boolean($var = 1))), substring('result2', 1 div number(not(boolean($var = 1)))))</assign>
            <!-- 
                Assignment using JavaScript.
            -->
            <assign to="$result" lang="JavaScript">if ($var == 1) { return 'result1' } else { return 'result2' }</assign>
        </steps>
    </testcase>

Needless to say the assignment using JavaScript is much more natural and easy to understand. However use of alternative expression languages, and their definition
through the ``namespaces`` element, is tricky because we need to know exactly how the target test bed refers to the language (i.e. "JavaScript" in our case) to 
correctly identify it. Furthermore it must be clear how the test bed will process the expression and how session context variables are looked up. In the above example
we assume that context variables (e.g. "$var") are looked up in exactly the same way as with XPath 1.0 expressions and that the entire expression will be evaluated
by first wrapping it in a function, the result of which is returned as the assignment output. Apart from actually supporting JavaScript for expressions, these 
additional details need to first be defined unambiguously by the test bed and made known to its users. Only then can we use them in a deterministic and portable manner.

.. note::
    **GITB software support:** Expression languages other than the default XPath 1.0 are not supported. As such the ``namespaces`` element is currently ignored.

.. index:: Imports (Test cases)
.. _test-case-imports:

Imports
~~~~~~~

The ``imports`` element allows the use of arbitrary resources that are present in the test suite. This can be very useful when a test case needs to send messages
based on a template or load a binary file that is needed as input by a messaging, processing or validation handler (e.g. a certificate). The ``imports`` element 
defines one or more ``artifact`` children with the following structure:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @name, yes, The name with which this artefact will be associated to the test session context for subsequent lookups.
    @type, yes, The type as which the artefact needs to be loaded.
    @encoding, no, In case the artefact is to be treated as text, this is the character encoding to apply when reading its bytes (default is "UTF-8").

The text value of the ``artifact`` element is the path within the test suite from which the relevant resource will be loaded. This path may be provided as a
fixed value or as a :ref:`variable reference<test-case-referring-to-variables>` to determine the imported resource dynamically. In case a variable reference
is provided this should be one of the following:

* A reference to a configuration value (i.e. a :ref:`domain<test-case-expressions-domain>`, :ref:`organisation<test-case-expressions-organisation>`, :ref:`system<test-case-expressions-system>` or :ref:`actor<test-case-expressions-actor>` parameter).
* A reference to a :ref:`variable<test-case-variables>` defined in the test case. In this case the value of the variable can even be adapted during the course of the test session resulting in
  different resources depending on the point at which the import is referenced.

Regarding the ``type`` attribute, this needs to refer to an appropriate type from the GITB type system (see :ref:`test-case-types`). Given that in this case we are referring to a file
being loaded, the types that can be used are:

* ``binary``: Load the artefact as a set of bytes without additional processing.
* ``object``: Load the artefact as a XML Document Object Model. In this case it is best to also explicitly provide the ``encoding`` to consider.
* ``schema``: Load the artefact as a XML Schema. As in the ``object`` case it is best to explicitly provide the ``encoding`` to consider.

Regarding the path to the resource this is the resource's path within the test suite archive (with or without the test suite name as a prefix). As an 
example consider the following test case fragment where a XML schema is loaded and set in the session context as a variable of type ``schema`` that is named "ublSchema". The
path specified suggests that the file is named "UBL-Invoice-2.1.xsd" and exists in a folder within the test suite archive named "resources". This example also includes
another input whose referenced resource is defined dynamically based on an external configuration parameter (at organisation level in this case).

.. code-block:: xml

    <testcase>
        <imports>
            <!--
                The "ublSchema" is loaded from a fixed resource within the test suite.
            -->
            <artifact type="schema" encoding="UTF-8" name="ublSchema">resources/UBL-Invoice-2.1.xsd</artifact>
            <!--
                The "organisationSpecificSchema" is loaded dynamically based on an organisation-level configuration property named "xsdToUseForOrganisation".
            -->
            <artifact type="schema" encoding="UTF-8" name="organisationSpecificSchema">$ORGANISATION{xsdToUseForOrganisation}</artifact>
        </imports>
        <steps>
            <verify handler="XSDValidator" desc="Validate invoice against UBL 2.1 Invoice Schema">
                <!-- 
                    Variable $fileContent is loaded in another step.
                -->
                <input name="xmldocument">$fileContent</input>
                <input name="xsddocument" source="$ublSchema"/>
            </verify>
        </steps>
    </testcase>

.. note::
    **Test module import:** The GITB TDL schema allows also for ``module`` elements to be defined for the import of test modules (validation, 
    messaging and processing handlers). This approach is no longer supported as it required the handler implementations to be bundled within 
    the test bed itself. The preferred and simpler approach now is to simply define the handler in the respective test step (e.g. the ``verify``
    step's ``handler`` attribute for validators) without previously importing it.

.. index:: Preliminary (Test cases)
.. _test-case-preliminary:

Preliminary
~~~~~~~~~~~

The ``preliminary`` element allows the test case to interact with the user before the test session begins. The purpose here is to allow the
user to provide preliminary input or be informed of certain actions that need to take place before the test session starts. In terms of structure 
and use, the ``preliminary`` element is a ``UserInteraction`` element (see :ref:`tdl-step-interact`). The difference is that the interaction takes place before the 
test session actually starts.

The following example shows a test case that prompts the user before starting to initialise their server and upload a configuration file.

.. code-block:: xml

    <testcase>
        <preliminary desc="Prepare your system" with="User">
            <instruct desc="Preparation instructions" with="User" type="string">"Make sure your system is up and running"</instruct>
            <request desc="Provide your configuration file" with="User" contentType="BASE64">$sutConfigFile</request>
        </preliminary>
        <variables>
            <var name="sutConfigFile" type="binary"/>
        </variables>
        <actors>
            <gitb:actor id="User" name="User" role="SUT"/>
        </actors>
        <steps>
            <!-- 
                The provided file can be referenced as $sutConfigFile 
            -->
        </steps>
    </testcase>

.. index:: Actors (Test cases)
.. _test-case-actors:

Actors
~~~~~~

The ``actors`` element is where the test case defines the actors involved in its steps and, importantly, their role. It contains
one or more ``actor`` children with the following structure:

.. csv-table::
    :stub-columns: 1
    :delim: ~
    :header: "Name", "Required?", "Description"

    @id~ yes~ The actor's unique (within the specification) ID. This must match an actor ID specified in the test suite.
    @name~ no~ The name to display for the actor. This can differ from the ID to display an actor name specific to the test case. Not specifying this will default to the name for actor provided in the test suite.
    @role~ yes~ The actor's role in the test case. This is "SUT" if the actor is the focus of the test case, "SIMULATED" if the actor is simulated by the test bed, or "MONITOR" if the actor is present for monitoring purposes.
    @displayOrder~ no~ A number indicating the relative positioning that needs to be respected when displaying the actor in test case's execution diagram. Setting this here overrides any corresponding setting at test suite level (see :ref:`test-suite-actors` for details).
    endpoint~ no~ An optional sequence of configuration endpoints if the actor is simulated.

The ``endpoint`` elements require a bit more explanation to understand their use. A specification may foresee actors that are all valid
selections for conformance statements. Imagine a specification that defines "sender" and "receiver" actors that can both be the SUTs 
depending on the actor a system selects to conform to. As such, a test suite focusing on the sender will include test cases with the 
sender as the SUT and the receiver as being simulated. Similarly, a test suite focusing on the receiver will define the 
receiver as SUT and the sender as simulated. In terms of configuration properties, the sender might need to define a "replyToAddress" to receive
replies, whereas the receiver simply needs to define his "deliveryAddress" which is where messages are expected. In terms of actor configuration 
in the test suite this would look like this:

.. code-block:: xml

    <testsuite>
        <gitb:actor id="sender">
            <gitb:name>Sender</gitb:name>
            <gitb:endpoint name="info">
                <gitb:config name="replyToAddress" desc="The address to return replies to" kind="SIMPLE"/>
            </gitb:endpoint>
        </gitb:actor>
        <gitb:actor id="receiver">
            <gitb:name>Receiver</gitb:name>
            <gitb:endpoint name="info">
                <gitb:config name="deliveryAddress" desc="The address to receive messages on" kind="SIMPLE"/>
            </gitb:endpoint>
        </gitb:actor>
    </testsuite>

Depending on the test case at hand, the user will be expected to provide the appropriate configuration parameters. For example, in a
conformance statement for the sender, the applicable test cases will be those defining the sender actor as the SUT, and the "replyToAddress"
parameter will need to be entered before starting the test. How is the "deliveryAddress" then provided for the simulated receiver actor? 
This can be achieved in two ways:

* **Dynamically** by the simulated actor's messaging handler. Using this approach, the test bed, while in its initiation phase, will request configuration
  properties from the handler that will be mapped to the SUT's corresponding endpoint (see :ref:`test-suite-actors-endpoints-simulated`).
* **Statically** by defining the endpoint and one or more of its parameters within the test case itself.

The second option is why we are able to configure ``endpoint`` elements as part of the test case. The values configured here will be used only 
if not already specified by the response of the simulated actor's handler. The below snippet illustrates this considering the sender as the SUT:

.. code-block:: xml

    <testcase>
        <gitb:actor id="sender" name="sender" role="SUT"/>
        <gitb:actor id="receiver" name="receiver" role="SIMULATED">
            <gitb:endpoint name="info">
                <gitb:config name="deliveryAddress">SIMULATED_ADDRESS</gitb:config>
            </gitb:endpoint>
        </gitb:actor>
        <steps>
            <!-- 
                receiver's address can be referenced by $sender{receiver}{deliveryAddress} 
            -->
        </steps>
    </testcase>

.. note::
    **GITB software support:** The "MONITOR" value for the actor ``role`` is currently not supported.

.. index:: Variables (Test cases)
.. _test-case-variables:

Variables
~~~~~~~~~

The ``variables`` element can be defined to create one or more variables that will be used during the test case's execution. It contains one 
or more ``var`` elements, one per variable, with the following structure:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @name, yes, The name of the variable. It is with this name that the variable can be referenced.
    @type, yes, The type of the variable. One of the GITB data types can be used (see :ref:`test-case-types`).
    value, no, One or more values for the variable. More than one values are applicable in case of a ``map`` or ``list`` type.

Variables can be used to record arbitrary information for the duration of the test session. These can be fixed values defined along with the
variable's definition or dynamically produced values resulting from test steps. The way to reference variables is defined based on the expression
language in place. Using the default XPath 1.0 expression language a variable named ``myVar`` is referenced as ``$myVar``. More information on 
expressions to reference variable values is provided in :ref:`test-case-expressions`.

Definition of variables using the ``variables`` element is **optional** given that test steps resulting in output will automatically create 
variables as needed to store the output in the test session context. Such steps include:

    * :ref:`Assign steps<tdl-step-assign>` that define new values or calculate expressions.
    * :ref:`User interaction steps<tdl-step-interact>` that request data from the user.
    * Messaging steps to record the output of a :ref:`send<tdl-step-send>` or a :ref:`receive<tdl-step-receive>`.
    * :ref:`Processing steps<tdl-step-process>` to record the output of the process.

The type of the automatically created variables in the above cases is inferred from the type of the relevant data or expression result. For example,
when assigning a string to a variable, this will automatically be set with a ``string`` type. Considering this, you would use the ``variables`` element
to predefine variables in the following cases:

    * To predefine all variables if you prefer this from the perspective of code organisation.
    * To explicitly set the type of variables in cases where the automatic determination is not suitable (e.g. force a ``string`` type for a numeric value).
    * To cover exceptional cases where automatic type determination is not possible.
    * To provide initial values to variables.

For examples of automatic variable definition refer to the corresponding steps as well as the documentation on :ref:`expressions<test-case-variables-from-expression-output>`.
Coming back to explicitly defined variables, the following example shows two such cases, one to store a user-uploaded file and another to store a part of it, 
extracted via XPath:

.. code-block:: xml

    <testcase>
        <imports>
            <artifact type="schema" encoding="UTF-8" name="schemaFile">testSuite/artifacts/UBL/maindoc/UBL-Invoice-2.1.xsd</artifact>
        </imports>
        <variables>
            <var name="fileContent" type="object"/>
            <var name="targetElement" type="object"/>
        </variables>
        <steps>
            <!-- 
                Store the uploaded result in the fileContent variable.
            -->
            <interact desc="UBL invoice upload" with="User">
                <request desc="Upload the UBL invoice to validate" with="User" contentType="BASE64">$fileContent</request>
            </interact>
            <!-- 
                Extract a part of it and store in the targetElement variable.
            -->
            <assign to="$targetElement" source="$fileContent">/*[local-name() = 'testcase']/*[local-name() = 'steps']</assign>
            <!-- 
                Pass the targetElement for validation.
            -->
            <verify handler="XSDValidator" desc="Validate content">
                <input name="xmldocument">$targetElement</input>
                <input name="xsddocument" source="$schemaFile"/>
            </verify>
        </steps>
    </testcase>

Setting a variable's initial value is achieved using the ``value`` element, with one or more being used in case of a ``map`` or ``list``
type. The following example illustrates setting values for different variable types:

.. code-block:: xml

    <testcase>
        <variables>
            <var name="aList" type="list[string]">
                <value>List value 1</value>
                <value>List value 2</value>
            </var>
            <var name="aMap" type="map">
                <value name="key1" type="string">Map value 1</value>
                <value name="key2" type="string">Map value 2</value>
            </var>
            <var name="aString" type="string">
                <value>A string value</value>
            </var>
        </variables>
    </testcase>

.. note::
    **List variables:** When a ``list`` is defined as a variable it also needs to specify its internal element type. To do this you
    need to specify the ``type`` attribute as ``list[INTERNAL_TYPE]``. For example a ``list`` of ``string`` elements is defined as
    ``<var name="myList" type="list[string]"/>``.

.. index:: Steps (Test cases)
.. index:: stopOnError (Test case)
.. _test-case-steps:

Steps
~~~~~

The ``steps`` element is where the test case's testing logic is implemented. It consists of a sequence of test steps realised by means
of a GITB TDL step construct. The structure of the element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @stopOnError, no, A boolean flag determining whether the test session should stop if any step fails (default is ``false``). See also :ref:`tdl-steps-common-stoponerror`.

The test case's steps are defined as children of the ``steps`` element. The available test steps that can be defined are:

.. csv-table::
    :widths: 30, 70
    :header: "Step name", "Description"

    :ref:`tdl-step-send`, Send a message to an actor
    :ref:`tdl-step-receive`, Receive a message from an actor
    :ref:`tdl-step-listen`, Listen for exchanged messages between actors
    :ref:`tdl-step-btxn`, Begin a messaging transaction
    :ref:`tdl-step-etxn`, End a messaging transaction
    :ref:`tdl-step-process`, Process a set of inputs to get an output
    :ref:`tdl-step-bptxn`, Begin a processing transaction
    :ref:`tdl-step-eptxn`, End a processing transaction
    :ref:`tdl-step-if`, Apply if-else logic to conditionally execute test steps
    :ref:`tdl-step-while`, Loop over a set of steps while a condition is true
    :ref:`tdl-step-repuntil`, Repeat a set of steps while a condition is true (executing at least once)
    :ref:`tdl-step-foreach`, Execute a set of steps a fixed number of times
    :ref:`tdl-step-flow`, Execute sets of steps concurrently
    :ref:`tdl-step-exit`, Immediately terminate the test session
    :ref:`tdl-step-assign`, Process an expression and assign its output to a variable
    :ref:`tdl-step-log`, Log a message in the test session log.
    :ref:`tdl-step-group`, Display a set of steps as a logical group
    :ref:`tdl-step-verify`, Validate content
    :ref:`tdl-step-call`, Call a scriptlet
    :ref:`tdl-step-interact`, Trigger an interaction with the user

.. index:: output
.. index:: success (output)
.. index:: failure (output)
.. index:: case (output)
.. index:: default (output)
.. _test-case-output:

Output
~~~~~~

The ``output`` element is an optional means of defining a final result message for a given test session. It is processed once all
steps have completed, checking the data in the test session's context to display specific success or failure messages. Using this
element allows extended feedback to be returned that may be important to summarise and contextualise the steps' results.

The ``output`` element supports both success and failure messages defined as different cases, each having a match condition, and an
overall default. The structure of the ``output`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    success, no, The set of output cases to apply in case the test completes with a success.
    failure, no, The set of output cases to apply in case the test completes with a failure.

The ``success`` and ``failure`` elements share a common structure to define their specific cases and overall defaults:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    case, no, Zero or more specific cases to apply depending on the provided match conditions.
    default, no, An optional default if no specific case was found to apply.

Finally, each ``case`` element shares a common structure as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    cond, yes, Defines a condition :ref:`expression<test-case-expressions>` expected to return a ``true`` or ``false`` value.
    message, yes, Defines an :ref:`expression<test-case-expressions>` expected to return the output message (as a string).

The ``output`` section is flexible as it doesn't require you to define both success and failure messages. In addition, you could
opt only to provide default messages without specific cases or only provide certain specific messages without generic defaults.
It is important to note that the condition expressions tolerate failures, meaning that if a condition cannot
be evaluated its relevant case will simply be skipped. In addition, if any error is raised when creating the text message itself, the output
message will be altogether skipped; under no circumstances will a test session fail due to the evaluation of its ``output`` section
(relevant warnings will however be included in the test session's log).

The following snippet illustrates how the ``output`` section could be leveraged to return user-friendly failure messages based on the executed
test steps:

.. code-block:: xml

    <testcase>
        <steps>
            ...
            <verify id="checkIntegrity" desc="Validate integrity">
                ...
            </verify>
            <verify id="checkSyntax" desc="Validate syntax">
                ...
            </verify>
            <verify id="checkContent" desc="Validate business rules">
                ...
            </verify>
            ...
        </steps>
        <output>
            <!-- We skip the "success" element as we only want failure messages. -->
            <failure>
                <case>
                    <cond>not($STEP_SUCCESS{checkIntegrity})</cond>
                    <message>"Please verify the integrity of your data and re-submit."</message>
                </case>
                <case>
                    <cond>not($STEP_SUCCESS{checkSyntax})</cond>
                    <message>"Please verify the syntax of your data and re-submit."</message>
                </case>
                <case>
                    <cond>not($STEP_SUCCESS{checkContent})</cond>
                    <message>"Please verify your data content and re-submit."</message>
                </case>
                <!-- The default will be applied if no specific case was found to match. -->
                <default>"Your data failed to be processed correctly. Please check the session log to determine the cause of the failure."</default>
            </failure>
        </output>
    </testcase>

.. note::
    **Messages are also expressions:** Output messages are themselves :ref:`expressions<test-case-expressions>` allowing dynamic
    output to be returned (e.g. concatenating text with session variable values). For a fixed message make sure to enclose the text
    in quotes.

.. index:: Scriptlets (Test cases)
.. _test-case-scriptlets:

Scriptlets
~~~~~~~~~~

The ``scriptlets`` element is meant to define reusable blocks of steps that can be called during the test case's execution. 
They are similar to function blocks in programming languages considering that:

* They receive inputs and produce outputs.
* They define a nested context for their own variables and processing that is isolated from the test session context.
* They can access variables from the parent test session context.

The main benefit of using scriptlets is really one of test step organisation to avoid copying (and then maintaining) sequences
of steps that might be executed multiple times and at different locations in the test case. Each scriptlet is defined in a
``scriptlet`` element with the following structure:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @id, yes, The ID of the scriptlet used to refer to it in ``call`` steps (see :ref:`tdl-step-call`).
    metadata, no, Optional scriptlet metadata. Structurally this matches exactly the ``metadata`` for the test case (see :ref:`test-case-metadata`).
    namespaces, no, Optional namespaces for contained expressions. Structurally this matches exactly the ``namespaces`` for the test case (see :ref:`test-case-namespaces`).
    imports, no, Optional artefacts to import for the scriptlet. Structurally this matches exactly the ``imports`` for the test case (see :ref:`test-case-imports`).
    params, no, An optional set of parameters that the scriptlet will expect from a ``call`` step  (see :ref:`tdl-step-call`).
    variables, no, An optional set of local variables. Structurally this matches exactly the ``variables`` for the test case (see :ref:`test-case-variables`).
    steps, yes, The sequence of steps to be executed in this scriptlet. Can contain any supported test steps (see :ref:`test-case-steps`).
    output, yes, One or more output values resulting from the scriptlet's execution.

The ``params`` and ``variables`` for the scriptlet define their individual elements as ``var`` elements. These follow the same structure and 
logic as the test cases variables (see :ref:`test-case-variables`).

A scriptlet is called using its ``id`` in a ``call`` step (see :ref:`tdl-step-call`). As part of this ``call`` the test case needs to pass as inputs any parameters that
the scriptlet expects. Concerning the scriptlet's output, each defined ``output`` value will be evaluated once the scriptlet completes and the overall result
will be returned as a ``map`` named using the ``id`` used in the relevant ``call`` step that triggered it's execution. Regarding output values:

* If the ``call`` does not specify named ``output`` elements then all the scriptlet's outputs will be returned.
* If the ``call`` does specify named ``output`` elements only these will be returned.

The following example shows a ``scriptlet`` that will validate a file passed as an input parameter. Once completed it will also
return a ``string`` result for a value provided by the user. Calling the scriptlet occurs for two different files and results in the 
display of their output values.

.. code-block:: xml

    <testcase>
        <steps>
            <!-- 
                Request two files to be uploaded.
            -->
            <interact desc="Upload files" with="User">
                <request desc="Upload the first file" with="User" contentType="BASE64">$fileContent1</request>
                <request desc="Upload the second file" with="User" contentType="BASE64">$fileContent2</request>
            </interact>
            <!-- 
                Call the scriptlet for the first file and store the result under variable "call1".
            -->
            <call id="call1" path="script1">
                <input name="docToValidate">$fileContent1</input>
                <output name="outputMessage"/>
            </call>
            <!-- 
                Call the scriptlet for the second file and store the result under variable "call2".
            -->
            <call id="call2" path="script1">
                <input name="docToValidate">$fileContent2</input>
                <output name="outputMessage"/>
            </call>
            <!-- 
                Display the results from both calls to the user.
            -->
            <interact desc="Scriptlet results" with="User">
                <instruct desc="Output one" with="User" type="string">$call1{outputMessage}</instruct>
                <instruct desc="Output two" with="User" type="string">$call2{outputMessage}</instruct>
            </interact>
        </steps>
        <scriptlets>
            <scriptlet id="script1">
                <!--
                    This parameter has to be provided when calling the scriptlet.
                -->
                <params>
                    <var name="docToValidate" type="object"/>
                </params>
                <!--
                    These variables are only locally visible.
                -->
                <variables>
                    <var name="userMessage" type="string"/>
                    <var name="outputMessage" type="string"/>
                    <var name="anotherOutputMessage" type="string">
                        <value>Another value</value>
                    </var>
                </variables>
                <steps>
                    <!-- 
                        Ask the user to enter a string value.
                    -->
                    <interact desc="Scriptlet call" with="User">
                        <request desc="Give me a value" with="User" type="string">$userMessage</request>
                    </interact>
                    <!-- 
                        Validate the file passed as a parameter.
                    -->
                    <verify handler="XSDValidator" desc="Validate content">
                        <input name="xmldocument">$docToValidate</input>
                        <!-- 
                            The schemaFile variable is retrieved from the global session context
                            (its declaration is ommitted from the example).
                        -->
                        <input name="xsddocument" source="$schemaFile"/>
                    </verify>
                    <assign to="$outputMessage">$userMessage</assign>
                </steps>
                <!-- 
                    This output is returned as it is specified in the call.
                -->
                <output name="outputMessage" type="string">$outputMessage</output>
                <!-- 
                    This output is not specified in the call and is thus ignored.
                -->
                <output name="anotherOutputMessage" type="string">$anotherOutputMessage</output>
            </scriptlet>
        </scriptlets>
    </testcase>

.. note::
    **GITB software support:** Currently the GITB software requires that scriptlets have at least one variable
    and one parameter. In addition, declared outputs must also exist as declared scriptlet variables.
