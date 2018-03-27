Test Cases
===============================

ID is used to display. Also to link with test suite.
Type CONFORMANCE needs to be provided.

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
            <!-- Step 1. Request the user to upload the UBL invoice -->
            <interact desc="UBL invoice upload" with="User">
                <request desc="Upload the UBL invoice to validate" with="User" contentType="BASE64">$file_content</request>
            </interact>
            <!-- Step 2. Validate the uploaded invoice  -->
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

Metadata
~~~~~~~~

The ``metadata`` element is basically the same as the one defined for the test suite [TODO]. Its purpose is to provide basic information 
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

Namespaces
~~~~~~~~~~

The ``namespaces`` optional element is used to define one or more expression languages that are used in test case constructs that support them. This needs to
be done when expressions are used that should not be processed using the default XPath 1.0 language. A detailed discussion on GITB expressions as well 
as where and how you can use them is provided in [TODO].

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
            <!-- Assignment using the default XPath 1.0 -->
            <assign to="$result">concat(substring('result1', 1 div number(boolean($var == 1))), substring('result2', 1 div number(not(boolean($var == 1)))))</assign>
            <!-- Assignment using JavaScript -->
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
attribute, this needs to refer to an appropriate type from the GITB type system (see [TODO]). Given that in this case we are referring to a file 
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
                <!-- Variable $fileContent is loaded in another step -->
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
and use, the ``preliminary`` element is a UserInteraction element (see [TODO]). The difference is that the interaction takes place before the 
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
            <!-- The provided file can be referenced as $sutConfigFile -->
        </steps>
    </testcase>

.. note::
    **GITB software support:** Only ``request`` interactions are currently supported.

Actors
~~~~~~

The ``actors`` element is where the test case defines the actors involved in its steps and, importantly, their role. It contains
one or more ``actor`` children with the following structure:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @id, yes, The actor's unique (within the specification) ID. This must match an actor ID specified in the test suite.
    @name, yes, The name to display for the actor. This can differ from the ID to display an actor name specific to the test case.
    @role, yes, The actor's role in the test case. This is "SUT" if the actor is the focus of the test case, "SIMULATED" if the actor is simulated by the test bed, or "MONITOR" if the actor is present for monitoring purposes.
    endpoint, no, An optional sequence of configuration endpoints if the actor is simulated.

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
  properties from the handler that will be mapped to the SUT's corresponding endpoint (discussed in [TODO]).
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
            <!-- receiver's address can be referenced by $sender{receiver}{deliveryAddress} -->
        </steps>
    </testcase>

.. note::
    **GITB software support:** Currently the actor ID and name must be identical. Providing a different name will result in the test case not 
    being considered for the actor's conformance statements. Note in addition that the "MONITOR" value for the actor ``role`` is currently
    not supported.

Variables
~~~~~~~~~

Steps
~~~~~

Scriptlets
~~~~~~~~~~

Types
-----

SCHEMA is basically OBJECT with a schemaLocation.

Expressions
-----------
