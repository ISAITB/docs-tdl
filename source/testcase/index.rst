Test Cases
===============================

Overview
--------

Test cases are the means by which a specific testing scenario is implemented. One or more test cases form the content of a test suite.
The following example represents a complete, simple test case for the validation of a invoice that is uploaded by a user.

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
            <artifact type="schema" encoding="UTF-8" name="UBL_Invoice_Schema_File">UBL_invoice_validation/artifacts/UBL/maindoc/UBL-Invoice-2.1.xsd</artifact>
            <artifact type="schema" encoding="UTF-8" name="BII_CORE_Invoice_Schematron_File">UBL_invoice_validation/artifacts/BII/BII_CORE/BIICORE-UBL-T10-V1.0.xsl</artifact>
        </imports>
        <variables>
            <var name="file_content" type="object"/>
        </variables>
        <actors>
            <gitb:actor id="User" name="User" role="SUT"/>
        </actors>
        <steps>
            <!-- 
                Step 1. Request the user to upload the UBL invoice 
            -->
            <interact desc="UBL invoice upload" with="User">
                <request desc="Upload the UBL invoice to validate" with="User" contentType="BASE64">$file_content</request>
            </interact>
            <!-- 
                Step 2. Validate the uploaded invoice  
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
    </testcase>

The following table provides an overview of the attributes and child elements that a ``testcase`` may have. A more detailed discussion per case 
follows in the subsequent sections.

**Attributes and child elements**

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
    scriptlets, no, An optional named sequence of test steps which can be used within the test case multiple times.

Elements
--------

We will now see how a test case breaks down into its individual sections and discuss the purpose of each.

.. _test-case-metadata:

Metadata
~~~~~~~~

The ``metadata`` element is basically the same as the one defined for the test suite :ref:`test-suite-metadata`. Its purpose is to provide basic information 
about the test case to help users understand its purpose.

**Attributes and child elements**

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

.. note::
    **GITB software support:** Contrary to a test suite's ``name``, the ``name`` of the test case is recorded but not otherwise used. Matching of test cases
    and display use the test case's ``id`` attribute. Regarding the test case ``type`` this has to currently be set to "CONFORMANCE" as the 
    "INTEROPERABILITY" type is not supported. Moreover, even if "CONFORMANCE" is considered the default, it currently must be explicitly set to avoid 
    errors during test suite upload. Finally, the ``version``, ``authors``, ``published`` and ``lastModified`` values are recorded but never used or displayed.

.. _test-case-namespaces:

Namespaces
~~~~~~~~~~

The ``namespaces`` optional element is used to define one or more expression languages that are used in test case constructs that support them. This needs to
be done when expressions are used that should not be processed using the default XPath 1.0 language. A detailed discussion on GITB expressions as well 
as where and how you can use them is provided in :ref:`test-case-expressions`.

**Attributes and child elements**

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    ns, yes, One or more elements to define the expression languages used in the test case. Each `ns` element must specify a ``prefix`` attribute to define the namespace prefix.

The values specified in the ``prefix`` attributes define how each expression language is referenced within the test case. As an example consider a test case that needs to 
check a number variable ``var`` and set a ``result`` variable with "result1" if ``var`` is 1 or with "result2" otherwise. Simple if-else constructs like this are a good
example of a shortcoming of XPath 1.0 as they are not supported and need a non-intuitive workaround based on string concatenation. A simpler approach would be to use
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
                Assignment using the default XPath 1.0 
            -->
            <assign to="$result">concat(substring('result1', 1 div number(boolean($var == 1))), substring('result2', 1 div number(not(boolean($var == 1)))))</assign>
            <!-- 
                Assignment using JavaScript 
            -->
            <assign to="$result" lang="JavaScript">if ($var == 1) { return 'result1' } else { return 'result2' }</assign>
        </steps>
    </testcase>

Needless to say the assignment using JavaScript is much more natural and easy to understand. However use of alternative expression languages, and their definition
through the ``namespaces`` element, is tricky because we need to know exactly how the target test bed refers to the language (i.e. "JavaScript" in our case) to 
correctly identify it. Furthermore it must be clear how the test bed will process the expression and how session context variables are looked up. In the above example
we assume that context variables (e.g. "$var") are looked up in exactly the same way as with XPath 1.0 expressions and that the entire expression will be evaluated
by first wrapping it in a function, the result of which is returned. These details need to first be defined unambiguously by the test bed and made known to its users.
Only then can we use them in a deterministic way in test cases.

.. note::
    **GITB software support:** Expression languages other than the default XPath 1.0 are not supported. As such the ``namespaces`` element is currently ignored.

.. _test-case-imports:

Imports
~~~~~~~

The ``imports`` element allows the use of arbitrary resources that are present in the test suite. This can be very useful when a test case needs to send messages
based on a template or load a binary file that is needed as input by a messaging, processing or validation handler (e.g. a certificate).

**Attributes and child elements**

The ``imports`` element defines one or more ``artifact`` children with attributes as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @name, yes, The name with which this artifact will be associated to the test session context for subsequent lookups.
    @type, yes, The type as which the artifact needs to be loaded.
    @encoding, no, In case the artifact is to be treated as text, this is the character encoding to apply when reading its bytes (default is "UTF-8").

The text value of the ``artifact`` element is the path within the test suite from which the relevant resource will be loaded. Regarding the ``type``
attribute, this needs to refer to an appropriate type from the GITB type system (see :ref:`test-case-types`). Given that in this case we are referring to a file 
being loaded, the types that can be used are:

* "binary": Load the artifact as a set of bytes without additional processing.
* "object": Load the artifact as a XML Document Object Model. In this case it is best to also explicitly provide the ``encoding``.
* "schema": Load the artifact as a XML Schema. As in the "object" case it is best to explicitly provide the ``encoding``.

Regarding the path to the resource, this considers as the root the folder of the test suite, named using the test suite name. As an example consider the
following test case fragment where a XML schema is loaded and set in the session context as a variable of type "schema" that is named "ublSchema". The
path specified suggests that this test case is part of a test suite with name "UBL_invoice_validation" (the path root) and the file itself is named 
"UBL-Invoice-2.1.xsd" and exists in a folder within the test suite archive named "resources".

.. code-block:: xml

    <testcase>
        <imports>
            <artifact type="schema" encoding="UTF-8" name="ublSchema">UBL_invoice_validation/resources/UBL-Invoice-2.1.xsd</artifact>
        </imports>
        <steps>
            <verify handler="XSDValidator" desc="Validate invoice against UBL 2.1 Invoice Schema">
                <!-- 
                    Variable $fileContent is loaded in another step 
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

Preliminary
~~~~~~~~~~~

The ``preliminary`` element allows the test case to interact with the user before the test session begins. The purpose here is to allow the
user to provide preliminary input or be informed of certain actions that need to take place before the test session starts. In terms of structure 
and use, the ``preliminary`` element is a ``UserInteraction`` element (see :ref:`tdl-step-interact`). The difference is that the interaction takes place before the 
test session starts.

The following example shows a test case that before starting prompts the user to start their server and upload a configuration file.

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

.. note::
    **GITB software support:** Only ``request`` interactions are currently supported.

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
    @name~ yes~ The name to display for the actor. This can differ from the ID to display an actor name specific to the test case.
    @role~ yes~ The actor's role in the test case. This is "SUT" if the actor is the focus of the test case, "SIMULATED" if the actor is simulated by the test bed, or "MONITOR" if the actor is present for monitoring purposes.
    endpoint~ no~ An optional sequence of configuration endpoints if the actor is simulated.

The ``endpoint`` elements require a bit more explanation to understand their use. A specification may foresee actors that are all valid
selections for conformance statements. Imagine a specification that defines "sender" and "receiver" actors that can both be the SUTs 
depending on the selected actor to conform to. As such, a test suite focusing on the sender will include test cases with the 
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

* **Dynamically** by the simulated actor's handler. Using this approach, the test bed, while in its initiation phase, will request configuration
  properties from the handler that will be mapped to the SUT's corresponding endpoint (see :ref:`test-suite-actors-endpoints-simulated`).
* **Statically** by defining the endpoint and one or more of its parameters in the test case.

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
    **GITB software support:** Currently the actor ID and name must be identical. Providing a different name will result in the test case not 
    being considered for the actor's conformance statements. Note in addition that the "MONITOR" value for the actor ``role`` is currently
    not supported.

.. _test-case-variables:

Variables
~~~~~~~~~

The ``variables`` element can be defined to create one or more variables that will be used during the test case's execution. It contains one 
or more ``var`` elements, one per variable, with the following structure:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @name, yes, The name of the variable. It is with this name that the variable can be referenced.
    @type, yes, The type of the variable. One of the GITB data types can be used.
    value, no, One or more values for the variable. More than one value are applicable in case of a ``map`` or ``list`` type.

Variables can be used to record arbitrary information for the duration of the test session. These can be fixed values defined along with the
variable's definition or dynamically produced values resulting from test steps. The way to reference variables is defined based on the expression
language in place. Using the default XPath 1.0 expression language a variable named ``myVar`` is referenced as ``$myVar``. More information on 
expressions to reference variable values is provided in :ref:`test-case-expressions`.

The following example shows use of two variables, one to store a user-uploaded file and another to store a part of it, extracted via XPath:

.. code-block:: xml

    <testcase>
        <imports>
            <artifact type="schema" encoding="UTF-8" name="schemaFile">actors/artifacts/UBL/maindoc/UBL-Invoice-2.1.xsd</artifact>
        </imports>
        <variables>
            <var name="fileContent" type="object"/>
            <var name="targetElement" type="object"/>
        </variables>
        <steps>
            <!-- 
                Store the uploaded result in the fileContent variable 
            -->
            <interact desc="UBL invoice upload" with="User">
                <request desc="Upload the UBL invoice to validate" with="User" contentType="BASE64">$fileContent</request>
            </interact>
            <!-- 
                Extract a part of it and store in the targetElement variable 
            -->
            <assign to="$targetElement" source="$fileContent">/*[local-name() = 'testcase']/*[local-name() = 'steps']</assign>
            <!-- 
                Pass the targetElement for validation 
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

.. note::
    **GITB software support:** Currently values for variables are defined in a fixed manner. Using expressions to set variable values
    is not currently supported.

.. _test-case-steps:

Steps
~~~~~

The ``steps`` element is where the test case's testing logic is implemented. It consists of a sequence of test steps that use a GITB 
TDL step construct per test step. The available test steps that can be defined are:

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
    :ref:`tdl-step-repuntil`, Repeat a set of steps until a condition is met
    :ref:`tdl-step-foreach`, Execute a set of steps a fixed number of times
    :ref:`tdl-step-flow`, Execute a set of steps concurrently
    :ref:`tdl-step-exit`, Immediately terminate the test session
    :ref:`tdl-step-assign`, Assign variable values to processing results
    :ref:`tdl-step-group`, Display a set of steps as a logical group
    :ref:`tdl-step-verify`, Validate content
    :ref:`tdl-step-call`, Call a scriptlet
    :ref:`tdl-step-interact`, Trigger an interaction with the user

.. _test-case-scriptlets:

Scriptlets
~~~~~~~~~~

The ``scriptlets`` element is meant to define reusable blocks of steps that can be called during the test case's execution. 
They are similar to function blocks in programming languages considering that:

* They receive one or more inputs and produce one or more outputs.
* They define a nested context for their own variables and processing that is isolated from the test session context.
* They can access variables from the parent test session context.

The main benefit of using scriptlets is really one of test step organisation to avoid copying (and then maintaining) sequences
of steps that might be executed multiple times and at different locations in the test case. Each scriptlet is defined in a
``scriptlet`` element with the following structure:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @id, yes, The ID of the scriptlet used to refer to it in ``call`` steps.
    metadata, no, Optional scriptlet metadata. Structurally this matches exactly the ``metadata`` for the test case (see :ref:`test-case-metadata`).
    namespaces, no, Optional namespaces for contained expressions. Structurally this matches exactly the ``namespaces`` for the test case (see :ref:`test-case-namespaces`).
    imports, no, Optional artifacts to import for the scriptlet. Structurally this matches exactly the ``imports`` for the test case (see :ref:`test-case-imports`).
    params, no, An optional set of parameters that the scriptlet will expect from a ``call`` step.
    variables, no, An optional set of local variables. Structurally this matches exactly the ``variables`` for the test case (see :ref:`test-case-variables`).
    steps, yes, The sequence of steps to be executed in this scriptlet. Can contain any supported test steps (see :ref:`test-case-steps`).
    output, yes, One or more output values resulting from the scriptlet's execution.

The ``params`` and ``variables`` for the scriptlet define their individual elements as ``var`` elements. These follow the same structure and 
logic as the test cases variables (see :ref:`test-case-variables`).

A scriptlet is called using its ``id`` in a ``call`` step (see :ref:`tdl-step-call`). As part of this ``call`` the test case needs to pass as inputs any parameters that
the scriptlet expects. Concerning the scriptlet's output, each defined ``output`` value will be evaluated once the scriptlet completes and the overall result
will be returned as a ``map`` named using the ``id`` used in the relevant ``call`` step that triggered it's execution. Regarding output values:

* If the ``call`` does not specify named ``output`` then all the scriptlet's outputs will be returned.
* If the ``call`` does specify named ``output`` only these will be returned.

The following example shows a ``scriptlet`` defined that will validate a file passed as an input parameter. Once completed it will also
return a string result matching a value provided by the user. Calling the scriptlet occurs for two different files an results in the 
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
                        Validated the file passed as a parameter.
                    -->
                    <verify handler="XSDValidator" desc="Validate content">
                        <input name="xmldocument">$docToValidate</input>
                        <!-- 
                            The schemaFile variable is retrieved from the global session context.
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
                    This output is not specified in the call so its ignored.
                -->
                <output name="anotherOutputMessage" type="string">$anotherOutputMessage</output>
            </scriptlet>
        </scriptlets>
    </testcase>

.. note::
    **GITB software support:** Currently the GITB software requires that scriptlets have at least one variable
    and one parameter. In addition, declared outputs must also exist as declared variables.

.. _test-case-types:

Types
-----

The GITB specification foresees a type system that is used to better manage ongoing testing state and facilitate calculations.
The supported types can be split in three main categories:

* **Primitive types**: Simple values.
* **Object types**: XML documents or other complex structures.
* **Container types**: Structures that can contain primitive and object types or embedded container types. 

The following table lists the available types. Note that the type name is the value that needs to be used when referring to a given type
in GITB TDL:

.. csv-table::
    :stub-columns: 1
    :header: "Type name", "Type category", "Description"

    ``string``, Primitive, Used for text values
    ``number``, Primitive, Used for numeric values (integers and floating point numbers)
    ``boolean``, Primitive, Boolean values (true/false)
    ``binary``, Primitive, A byte buffer used typically to represent file content
    ``object``, Object, An XML document recorded as a Document Object Model (DOM)
    ``schema``, Object, Identical to a ``object`` but with an additional schemaLocation property
    ``map``, Container, A map of key-value pairs (similar to a ``java.util.Map``).
    ``list``, Container, A list of values (similar to a ``java.util.List``)

Map variables
~~~~~~~~~~~~~

Variables of type ``map`` can contain values or any type, with even different types contained in the same ``map``. In fact a ``map``
may also contain embedded ``map`` variables at any level of depth. They keys of a ``map`` are however always strings.

List variables
~~~~~~~~~~~~~~

Variables of type ``list`` can contain an arbitrary sequence of elements. A key difference however when comparing to a ``map`` is that
the elements contained in a ``list`` are of a single type. This type is defined when the ``list`` is declared in the :ref:`test-case-variables` section.

.. _test-case-types-type-convesions:

Type convesions
~~~~~~~~~~~~~~~

It is often the case when executing a test case that we need to convert a session context variable of one type to another. An an example
consider a ``string`` variable containing XML content that we want to validate using a validator that only expects ``binary`` or ``object``
input. Conversions between types can be done explicitly using the ``assign`` step and the ``type`` attribute:

.. code-block:: xml

    <assign to="$toVariable" source="$fromVariable" type="object"/>

In this example, the ``fromVariable`` is converted to the ``toVariable`` as an ``object``. Explicit conversions like this can however pollute
the testing logic and can in most cases happen implicitly. An implicit conversion takes place when we attempt to use a source variable of a 
given type as a variable of another type. The supported conversions and specific conversion assumptions between types are provided in the 
following table:

.. csv-table::
    :stub-columns: 1
    :header: "", ``string``, ``number``, ``boolean``, ``binary``, ``object``, ``schema``, ``map``, ``list``

    ``string``, Yes, Yes - parsed as double (e.g. "10" or "5.03"), Yes - parsed as "true" or "false" (ignoring case), Yes - result is the string's bytes, Yes - considered as serialised XML, Yes - same as with ``object``, No, No
    ``number``, Yes - result as string (e.g. "10"), Yes, Yes - result is true if 1, Yes - bytes of ``string`` representation, No, No, No, No
    ``boolean``, Yes - result as string (e.g. "true"), Yes - result is 1 if "true" otherwise 0, Yes, Yes - bytes of ``string`` representation, No, No, No, No
    ``binary``, Yes - result is the string representation of the bytes, No, No, Yes, Yes - result is the XML representation of the bytes, Yes - same as with ``object``, No, No
    ``object``, Yes - result is the serialised XML, No, No, Yes - result is the XML's bytes, Yes, Yes, No, No
    ``schema``, Yes - result is the serialised XSD, No, No, Yes - result is the XSD's bytes, Yes, Yes, No, No
    ``map``, No, No, No, No, No, No, Yes, No
    ``list``, No, No, No, No, No, No, Yes - result is a map of the elements with keys their zero-based list index, Yes

.. note::
    **GITB software support:** Implicit conversions between types occurs when the expression referencing the source variable
    is a pure variable reference. For example given a ``binary`` variable ``myVariable``, referencing ``$myVariable`` when 
    assigning to a ``string`` works whereas but ``concat('My value is ', $myVariable)`` does not. This is because the second
    case is a full XPath expression whereas the first case only identifies the variable.

.. _test-case-expressions:

Expressions
-----------

Expressions are used in GITB TDL to perform arbitrary operations on context variables and to provide more control over the input and
output of specific steps. The approach to interpret and execute expressions is pluggable, meaning that a test bed implementation can
support any number of expression languages that a test case can then refer to. As we previously saw this is achieved by means of the
declarations in the ``namespaces`` element (see :ref:`test-case-namespaces`) where a prefix is used to identify each language used. 
The default expression language assumed by GITB TDL is XPath 1.0 given that processing XML constructs is one of the more frequent needs 
when conformance testing for content specifications. However, the use of XPath does not restrict us to using XML documents as variables; 
XPath provides sufficient expressiveness to define any kind of operation.

The following ``assign`` operations illustrate some interesting examples:

.. code-block:: xml

    <!-- 
        From an object variable fileContent (i.e. an XML document), extract part matching 
        /testcase/steps into another object variable named targetElement.
    -->
    <assign to="$targetElement" source="$fileContent">/*[local-name() = 'testcase']/*[local-name() = 'steps']</assign>
    <!-- 
        Assign to a number variable named result the result of adding 1 to another number 
        variable named counter.
    -->
    <assign to="$result">'$counter + 1'</assign>
    <!-- 
        Create a custom XML fragment from a string variable named value and assign it to 
        the tempXml string variable
    -->
    <assign to="$tempXml">concat('<temp>', $value, '</temp>')</assign>
    <!--
        Assign to the boolean result variable the result of checking that a string variable 
        named input has at least 10 characters (expression wrapped in CDATA block to use '>' 
        character without escaping it
    -->
    <assign to="$result"><![CDATA[string-length($input) >= 10]]></assign>
    <!-- 
        Assign to string variable result the value "result1" if the number var is 1, or 
        "result2" otherwise
    -->
    <assign to="$result">concat(substring('result1', 1 div number(boolean($var == 1))), substring('result2', 1 div number(not(boolean($var == 1)))))</assign>

As you can see the expressions you can use are limited only to the functions available in XPath 1.0. Using these there is typically always 
a way to express what is needed, potentially by first wrapping one or more values in a custom XML wrapper and then using XPath functions
on that:

.. code-block:: xml

    <!-- 
        Store custom content as a string in the string tempStr 
    -->
    <assign to="$tempStr"><![CDATA[concat('<toc>', $toc{tocEntries}, '</toc>')]]></assign>
    <!-- 
        Convert tempStr to an object tempXml 
    -->
    <assign to="$tempXml">$tempStr</assign>
    <!-- 
        Use object tempXml to evaluate an XPath expression and store the result in a boolean result 
    -->
    <assign to="$result" source="$tempXml">contains(/toc/text(), 'file.xml')</assign>

In the above example, we are using the value contained in a ``map`` variable named "toc" to construct a temporary ``string`` with 
XML content. We then assign this to an ``object`` variable named "tempXml" to convert it into an XML document (i.e. a variable
of type ``object``). We can then use the "tempXml" variable for any XPath manipulation that requires a source document.

Escaping XML in expressions
~~~~~~~~~~~~~~~~~~~~~~~~~~~

You need to always keep in mind that when you are writing an expression you are doing so within an XML document. This means that
special characters such as ``<`` and ``>`` need to be handled. There are two ways to handle this, matching how you would do this 
in any XML document.

**Approach 1: Escape entities**

Special XML characters can always be escaped using their corresponding entities. In the following example we use the ``&gt;``
entity to escape the ``>`` character:

.. code-block:: xml

    <assign to="$result">string-length($input) &gt;= 10</assign>

A similar approach can be taken to replace any other special character such as ``"`` (replaced by ``&quot;``) or ``'``
(replaced by ``&apos;``).

**Approach 2: CDATA block**

Using XML escape entities can result in expressions that are hard to read. In addition, they are insufficient when the text we
are using in the expression is unknown and may give unexpected results. In these case you can use a ``CDATA`` block:

.. code-block:: xml

    <assign to="$result"><![CDATA[string-length($input) >= 10]]></assign>

Referring to variables
~~~~~~~~~~~~~~~~~~~~~~

As you have seen in the previous examples, referring to variables is a very common use case in GITB TDL expressions. Variable
references are done using as ``$VARIABLE``, i.e. using the ``$`` character, followed by the variable's name. Furthermore, when
an expression consists only of a variable reference without other XPath elements it is refered to as a **pure variable reference**.
As we have discussed in :ref:`test-case-types-type-convesions`, pure variable references are important when we need to convert 
variables from one type to another in ``assign`` steps.

Map elements
++++++++++++

Variables of type ``map`` represent special cases as they can contain additional variables and even additional maps. Referring to 
``map`` variables themselves is done as with any other variable (i.e. ``$myMap``), but referring to entries in the ``map`` are done
by specifying the key in curly brackets (i.e. ``$myMap{myKey}``). In the case a ``map`` contains a nested ``map`` its inner values
can be referenced by appending to the expression an additional set of curly braces with the inner map's key. There is no limit to 
the nesting that is possible in a ``map``. In addition note that when specifying the keys you may also use variable references
if the key is to be determined dynamically.

Assigning values to a ``map`` variable is achieved using the ``assign`` step. Using this we can either assign values to the ``map``
itself (i.e. point it to another ``map``) or one of its keys. Moreover, assignment to a ``map`` key may be done to an existing 
entry or by defining a new one. Note that when assigning a new ``map`` entry you also need to specify the ``type`` of this entry.
The following examples illustrate ``assign`` steps that showcases the possible assignment and reference scenarios:

.. code-block:: xml

    <!-- 
        Assign map "myMap" to another map named "anotherMap". 
    -->
    <assign to="$myMap">$anotherMap</assign>
    <!-- 
        Assign the entry of "myMap" named "myKey" to the string "A value". 
    -->
    <assign to="$myMap{myKey}" type="string">"A value"</assign>
    <!-- 
        Assign the entry of "myMap" named "myOtherKey" to an entry of "anotherMap" 
        named "anotherKey" as a string. 
    -->
    <assign to="$myMap{myOtherKey}" type="string">$anotherMap{anotherKey}</assign>
    <!-- 
        Assign value "key1" to the string variable "k1". 
        Then use the "k1" variable to pick the target entry of "myMap" 
        to set as the string "A value".
    -->
    <assign to="$k1">"key1"</assign>
    <assign to="$myMap{$k1}" type="string">"A value"</assign>
    <!-- 
        Assign value "key1" to the string variable "k1". 
        Assign value "key2" to the string variable "k2". 
        Then use the "k1" variable to pick the target entry of "myMap" to set as a string matching 
        the entry of "anotherMap" for the key matching the value of "k2".
    -->
    <assign to="$k1">"key1"</assign>
    <assign to="$k2">"key2"</assign>
    <assign to="$myMap{$k1}" type="string">$anotherMap{$k2}</assign>

List variable elements
++++++++++++++++++++++

Variables of type ``list`` contain a sequence of elements of a specific type. Given that their type is defined when they are declared
(see :ref:`test-case-variables`) you don't need to specify the ``type`` attribute when assigning values to them (in ``assign`` steps). Referencing a ``list``
element is done using its index which is 0-based. Adding values to a ``list`` is achieved using the ``assign`` step and can either
target an existing list element (identified by its index) or be appended to the list. The following examples illustrate how you can reference
and assign list values:

.. code-block:: xml

    <!-- 
        Assign list "myList" to another list named "anotherList". 
    -->
    <assign to="$myList">$anotherList</assign>
    <!--
        Append an item to "myList" (declared as a list of strings). 
    -->
    <assign to="$myList" append="true">"Value 1"</assign>
    <!-- 
        Append another item to "myList". 
    -->
    <assign to="$myList" append="true">"Value 2"</assign>
    <!-- 
        Replace the first item of "myList" with "Value 3". 
    -->
    <assign to="$myList{0}">"Value 3"</assign>
    <!-- 
        Assign to the number variables "index1" and "index2" values 1 and 2 respectivelly.
        Replace the item matching "index1" of "myList" with the item matching "index2" of "anotherList".
    -->
    <assign to="$index1">"1"</assign>
    <assign to="$index2">"2"</assign>
    <assign to="$myList{$index1}">$anotherList{$index2}</assign>

Referring to actor configuration parameters
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Configuration relating to specific actors is defined by means of endpoints and parameters. These can be declared in the following ways:

* **Externally:** Actors may be defined fully in the test bed. In this case the test suite simply references actors by their ID (see :ref:`test-suite-deploying`).
* **In the test suite:** Actors can be fully defined in a test suite, listing their endpoints and parameters (see :ref:`test-suite-actors`).
* **During the test session initiation:** Simulated actors participating in messaging transactions with SUTs have their messaging handlers 
  called during test suite initiation at which time they can return endpoints and parameters.
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
            <assign to="$temp">$Sender{dataVersion}</assign>
            <!-- 
                Lookup the "address" property configured by the simulated "Receiver" for the "Sender". 
                This is statically defined here but could also be received from the "Receiver" messaging
                handler as part of the test session's initiation phase.
            -->
            <assign to="$temp">$Sender{Receiver}{address}</assign>
        </steps>
    </testcase>

Where can expressions be used?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The following table provides an overview of the places where expressions can be used:

.. csv-table::
    :widths: 30, 70
    :header: "Use", "Description"

    Step :ref:`tdl-step-interact`, Used in the values displayed to (``instruct``) or requested from (``request``) users.
    Step :ref:`tdl-step-send`, Used to determine ``input`` values.
    Step :ref:`tdl-step-receive`, Used to determine ``input`` values.
    Step :ref:`tdl-step-listen`, Used to determine ``input`` values.
    Step :ref:`tdl-step-process`, Used to determine ``input`` values.
    Step :ref:`tdl-step-if`, Used to define and evaluate the if condition (``cond``).
    Step :ref:`tdl-step-while`, Used to define and evaluate the loop condition (``cond``).
    Step :ref:`tdl-step-repuntil`, Used to define and evaluate the repeat condition (``cond``).
    Step :ref:`tdl-step-assign`, Used as the expression to apply. Also a pure variable reference is used in the ``to`` and ``source`` elements.
    Step :ref:`tdl-step-verify`, Used to determine ``input`` values.
    Step :ref:`tdl-step-call`, Used to determine ``input`` values.