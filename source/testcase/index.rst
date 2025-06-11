.. index:: Test cases
.. _test-case:

Test cases
==========

Overview
--------

Test cases are the means by which a specific testing scenario is implemented. One or more test cases form the content of a test suite.
The following example represents a complete, simple test case for the validation of an invoice that is uploaded by a user.

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <testcase id="testCase1" xmlns="http://www.gitb.com/tdl/v1/" xmlns:gitb="http://www.gitb.com/core/v1/">
        <metadata>
            <gitb:name>UBL invoice validation 1</gitb:name>
            <gitb:version>1.0</gitb:version>
            <gitb:description>Test case to verify the correctness of a UBL invoice. The invoice is provided manually through user upload.</gitb:description>
        </metadata>
        <imports>
            <artifact name="schema">artifacts/UBL/maindoc/UBL-Invoice-2.1.xsd</artifact>
            <artifact name="schematron">artifacts/BII/BII_CORE/BIICORE-UBL-T10-V1.0.xsl</artifact>
        </imports>
        <actors>
            <gitb:actor id="User" name="User" role="SUT"/>
        </actors>
        <steps>
            <!-- 
                Step 1. Request the user to upload the UBL invoice.
            -->
            <interact id="userData" desc="UBL invoice upload">
                <request name="invoice" desc="Upload the UBL invoice to validate" inputType="UPLOAD"/>
            </interact>
            <!-- 
                Step 2. Validate the uploaded invoice.
            -->
            <verify handler="XmlValidator" desc="Validate invoice">
                <input name="xml">$userData{invoice}</input>
                <input name="xsd">$schema</input>
                <input name="schematron">$schematron</input>
            </verify>            
        </steps>
        <output>
            <failure>
                <default>"The test session resulted in a failure. Please check the validation reports and apply required corrections."</default>
            </failure>
        </output>
    </testcase>

.. index:: id (Test case)
.. index:: supportsParallelExecution
.. index:: metadata (Test case)
.. index:: namespaces (Test case)
.. index:: imports (Test case)
.. index:: preliminary (Test case)
.. index:: variables (Test case)
.. index:: actors (Test case)
.. index:: steps (Test case)
.. index:: output (Test case)
.. index:: scriptlets (Test case)
.. index:: optional (Test case)
.. index:: disabled (Test case)

The following table provides an overview of the attributes and child elements that a ``testcase`` may have. A more detailed discussion per case 
follows in the subsequent sections.

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @disabled, no, A boolean flag indicating whether this test case is disabled (default is "false"). Disabled test cases cannot be executed and any existing test results don't count towards a conformance statement's status.
    @id, yes, A string to uniquely identify the test case by. This is referenced in the test suite XML.
    @optional, no, A boolean flag indicating whether this test case is optional (default is "false"). Optional test cases may be executed but their results don't count towards a conformance statement's status.
    @supportsParallelExecution, no, A boolean flag indicating whether this test case may be executed in parallel with other test cases for a given SUT (default is "true").
    actors, yes, The set of actors that this test case refers to.
    imports, no, An optional set of imports used to load additional resources from the test suite.
    metadata, yes, A block containing the metadata used to describe the test case.
    namespaces, no, An optional set of namespace declarations to define the namespace prefixes used in the test case's expressions.
    output, no, Definition of an output message to display for the overall test session.
    preliminary, no, An optional set of user interaction steps to display before the test session starts.
    scriptlets, no, Optional named groups of test steps which can be used within the test case.
    steps, yes, The sequence of steps that this test case foresees.
    variables, no, An optional set of variables that are used in the test case.

The ``id`` attribute is important in uniquely identifying the test case within a given test suite, and needs to be referenced by the 
:ref:`test suite<test-suite>` if it is to be considered. It is not presented to normal users, only administrators, and is used when 
:ref:`uploading a new version of a test suite<test-suite-deploying>` to determine whether the test case definition serves as an update to an existing 
test case.

The ``supportsParallelExecution`` attribute is important in determining how the test case is handled in batch background executions (i.e. not executions
that are interactively launched and followed by a tester). If this is set to "true", the default value considered if missing, the test case is assumed
to be able to correctly function while other test cases are being executed in parallel for the same SUT. This means that the design of the test case
caters for such concurrent sessions and is able to correctly map exchanged messages to sessions. This is not always possible to do, especially in 
scenarios where messaging is initiated from the SUT (not by the Test Bed) or are asynchronous in nature.

If the test case cannot correctly handle such concurrency, you need to set ``supportsParallelExecution`` to "true". Doing so instructs the test
engine to always execute the given test case in isolation. Any ongoing test session will first need to complete before the current test case is executed
and its own test session will itself need to complete before executing any other test cases. The order of execution of test cases when making such 
considerations is defined by their :ref:`declaration order<test-suite-test-cases>` in the :ref:`test suite<test-suite>`.

.. note::
    When ``supportsParallelExecution`` is set to "false", the test case's non-parallel execution is honoured only within the context of a single batch
    execution of a test suite. The flag becomes ineffective if the tester explicitly launches separate test sessions in parallel.

Elements
--------

We will now see how a test case breaks down into its individual sections and discuss the purpose of each.

.. index:: metadata (Test cases)
.. index:: name (Test case metadata)
.. index:: type (Test case metadata)
.. index:: version (Test case metadata)
.. index:: authors (Test case metadata)
.. index:: description (Test case metadata)
.. index:: published (Test case metadata)
.. index:: lastModified (Test case metadata)
.. index:: documentation (Test case metadata)
.. index:: update (Test case metadata)
.. index:: tags (Test case metadata)
.. _test-case-metadata:

Metadata
~~~~~~~~

The ``metadata`` element is basically the same as the one defined for the test suite :ref:`test-suite-metadata`. Its purpose is to provide basic information 
about the test case to help users understand its purpose. Its structure is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    authors, no, A string to indicate the test case's authors.
    description, no, A string to provide a user-friendly description of the test case that is displayed to users.
    documentation, no, Rich text content that provides further information on the current test case.
    lastModified, no, A string acting as an indication of the last modification time for the test case.
    name, yes, The name of the test case that is used to identify it to users.
    published, no, A string acting as an indication of the test case's publishing time.
    specification, no, Optional information regarding the test case's normative specification reference.
    tags, no, Optional tags used to record additional metadata for the test case and visually highlight its attributes.
    type, no, Either "CONFORMANCE" (the default) or "INTEROPERABILITY". "INTEROPERABILITY" is used when more than one actor are defined as SUTs.
    update, no, Instructions determining the default choices when an update of this test case is taking place.
    version, yes, A string that indicates the test case's version.

.. note::
    **GITB software support:** The test case ``type`` must currently be set to "CONFORMANCE" (the default value) as the
    "INTEROPERABILITY" type is not supported. Finally, the ``version``, ``authors``, ``published`` and ``lastModified`` values are recorded but never used or displayed.

.. index:: documentation (Test case)
.. index:: import (Test case documentation)
.. index:: from (Test case documentation)
.. index:: encoding (Test case documentation)
.. _test-case-metadata-documentation:

documentation
+++++++++++++

The ``documentation`` element complements the test case's ``description`` by allowing the author to include extended rich text documentation as HTML. The structure of this element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    import, no, A reference to a separate file within the test suite archive that defines the documentation content.
    from, no, The identifier of a test suite from which the ``import`` file will be loaded. If unspecified the current test suite is assumed.
    encoding, no, In case an ``import`` reference is defined this can be used to specify the file's encoding. If not provided ``UTF-8`` is considered.

Using the above attributes to specify a reference to a separate file is not mandatory. The documentation's content can also be provided as the element's text content,
typically enclosed within a CDATA section if this includes HTML elements (in which case the ``from``, ``import`` and ``encoding`` attributes are omitted).
When loading documentation from a separate file, it is also possible to lookup this file from another test suite. This is
done by specifying as the value of the ``from`` attribute the ``id`` of the target test suite. This is used to lookup the
target test suite as follows:

#. Look for the test suite in the same **specification** as the current test case.
#. If not found in the same specification, look for the test suite in the other specifications of the test case's **domain**.
   If across specifications multiple matching test suites are found, one of them will be arbitrarily picked. To avoid such
   a scenario it is obvious that you should ensure test suites used to load shared resources can be uniquely identified.

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
            <gitb:name>Test case 1</gitb:name>
            <gitb:version>1.0</gitb:version>
            <gitb:description>A short description of the test case to offer a short summary of its purpose.</gitb:description>
            <gitb:documentation><![CDATA[
                <p>Extended documentation for <b>Test case 1</b></p>
                <p>This is an example to support the <a href="https://www.itb.ec.europa.eu/docs/tdl/latest">GITB TDL docs</a>.</p>
            ]]></gitb:documentation>
        </metadata>    
        ...
    </testcase>

Note that documentation such as this is also supported for:

* The overall :ref:`test suite<test-suite-metadata>`.
* Individual :ref:`test case steps<tdl-steps-common-documentation>`.

.. index:: update (Test case)
.. index:: updateMetadata (Test case update)
.. index:: resetTestHistory (Test case update)
.. _test-case-metadata-update:

update
++++++

The ``update`` element allows the test suite's developer to prescribe what should happen when this test case is being uploaded and
an existing test case with the same identifier is found. Through this you can define if the test case's existing metadata 
(e.g. name, description and documentation) should be updated to match the definitions from the new archive. In addition, you can 
specify whether the testing history linked to the test case being updated should be reset. Note that these choices represent the
default selected options during the test suite upload, and can always be verified and replaced by the Test Bed's operator.

It could be interesting to use the ``update`` element if the test developer is not the one performing the test suite upload. Doing so,
avoids providing detailed instruction to operations staff, by already encoding the relevant choices within the test suite archive itself.

The structure of the ``update`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @resetTestHistory, no, A boolean value determining whether any previously executed test sessions for the test case being updated should be considered as obsolete (default is "false").
    @updateMetadata, no, A boolean value determining whether the existing test case's metadata should be updated based on the new archive (default is "false").

The following example shows how you can specify that the test case's metadata should be updated to reflect the new values in the archive
(see attribute ``updateMetadata``). Also we specify here that any existing test sessions should be considered obsolete, forcing users to 
re-execute their tests for the updated version (see attribute ``resetTestHistory``).

.. code-block:: xml
    :emphasize-lines: 6

    <testcase id="TS1-TC1" xmlns="http://www.gitb.com/tdl/v1/" xmlns:gitb="http://www.gitb.com/core/v1/">
        <metadata>
            <gitb:name>Test case 1</gitb:name>
            <gitb:version>1.0</gitb:version>
            <gitb:description>A short description of the test case to offer a short summary of its purpose.</gitb:description>
            <gitb:update updateMetadata="true" resetTestHistory="true"/>
        </metadata>    
        ...
    </testcase>

Relevant options to manage updates at test suite level are possible through a similar ``update`` element of the :ref:`test suite <test-suite-metadata-update>` definition.

.. index:: tags (Test case)
.. index:: tag (Test case)
.. index:: foreground (Test case tag)
.. index:: background (Test case tag)
.. index:: name (Test case tag)
.. _test-case-metadata-tags:

tags
++++

The ``tags`` element allows you to specify arbitrary additional metadata for the test case that may help better distinguish it for users. You
may also use the same tags across multiple test cases as a means of classifying them based on common traits. The meaning you provide to such
tags is fully up to you, but typical examples would be to highlight test cases linked to "security" or test cases focusing on "unhappy flow"
scenarios. Other than highlighting certain traits, tags have no impact on a test case's execution or result.

The structure of the ``tags`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    tag, yes, One or more elements representing the test case's tags.

Each ``tag`` element has the following structure:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @background, no, A string for the `hexadecimal code <https://en.wikipedia.org/wiki/Web_colors>`_ of the tag's background colour. If not specified a generic default colour will be selected by the Test Bed.
    @foreground, no, A string for the `hexadecimal code <https://en.wikipedia.org/wiki/Web_colors>`_ of the tag's foreground colour (its text). If not specified a generic default colour will be selected by the Test Bed.
    @name, yes, A string for the name of the tag to be displayed. Although no restrictions are applied this is expected to be concise.

The ``tag`` element can also have an optional text content. If this is provided it is considered as a explanation over the meaning of this tag and 
is treated depending on how the tag is viewed. If through the Test Bed's user interface this would be a tooltip, otherwise in a PDF report
this would be a description added in a legend section.

The following example illustrates the definition of two tags to be displayed for a test case. The first one highlights that it relates to
security features and is expressed as a white-on-red "security" label. The second one specifies a white-on-black tag to 
highlight that this is a new test case in "version 2.0".

.. code-block:: xml
    :emphasize-lines: 6-9

    <testcase id="TS1-TC1" xmlns="http://www.gitb.com/tdl/v1/" xmlns:gitb="http://www.gitb.com/core/v1/">
        <metadata>
            <gitb:name>Test case 1</gitb:name>
            <gitb:version>1.0</gitb:version>
            <gitb:description>A short description of the test case to offer a short summary of its purpose.</gitb:description>
            <gitb:tags>
                <gitb:tag foreground="#FFFFFF" background="#FF2E00" name="security">Test case related to security features.</gitb:tag>
                <gitb:tag foreground="#FFFFFF" background="#000000" name="version 2.0">New test case added in version 2.0.</gitb:tag>
            </gitb:tags>	            
        </metadata>    
        ...
    </testcase>

.. index:: specification (Test case)
.. index:: reference (Test case reference)
.. index:: description (Test case reference)
.. index:: link (Test case reference)
.. _test-case-metadata-specification:

specification
+++++++++++++

The ``specification`` element is an optional part of a test case's metadata, that allows you to record in a structured manner a normative specification
reference for the test case. Besides being present in the test case definition, this information will also be rendered appropriately in the test case's
on-screen display and in reports.

The structure of the ``specification`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    description, no, A text describing the referred specification.
    link, no, A link to allow navigation to the referred specification's online documentation.
    reference, no, The reference identifier or code.

All the above elements are optional, meaning that you can choose to provide any documentation you see fit for the specification. Depending on what is provided,
this information will be displayed accordingly, presenting for example the reference as a link if both are provided, or presenting only a link icon if only the
link is present.

The following example illustrates how this metadata could be used to identify the specification section relevant to the test case and point to its online
documentation.

.. code-block:: xml
    :emphasize-lines: 6-10

    <testcase id="TS1-TC1" xmlns="http://www.gitb.com/tdl/v1/" xmlns:gitb="http://www.gitb.com/core/v1/">
        <metadata>
            <gitb:name>Test case 1</gitb:name>
            <gitb:version>1.0</gitb:version>
            <gitb:description>A short description of the test case to offer a short summary of its purpose.</gitb:description>
            <gitb:specification>
                <gitb:reference>Section-1.2.A</gitb:reference>
                <gitb:description>Security requirements</gitb:description>
                <gitb:link>https://my.spec.wiki.org</gitb:link>
            </gitb:specification> 
        </metadata>    
        ...
    </testcase>

.. note::
    Similar specification reference information can also be added to :ref:`test suites<test-suite-metadata-specification>`.

.. index:: namespaces (Test case)
.. index:: ns (Test case namespaces)
.. index:: prefix (Test case namespaces)
.. _test-case-namespaces:

Namespaces
~~~~~~~~~~

The ``namespaces`` optional element is used to declare namespace mappings for use within the test case. The primary use cases of these namespaces is to allow
the definition of prefixes used in XML and XPath expressions. In principle they could be applied to any type of language or expression that has such a concept
(e.g. JSON-LD, Turtle) but currently their use is limited to XML.

Each namespace to declare is defined as a child ``ns`` element with the following structure:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @prefix, yes, The namespace prefix that will be used in expressions.

The value to which the ``prefix`` is mapped is provided as the ``ns`` element's text content.

Namespaces declared using this approach can be used in two cases:

* Within any GITB TDL step that supports :ref:`expressions<test-case-expressions>`.
* As the expression to apply for the :ref:`XPathValidator<handlers-XPathValidator>` embedded validation handler.

The following example illustrates how namespaces can be used for XML-based processing. The sample test case:

#. Requests an invoice from the user.
#. Extracts the invoice's type using namespaces in an :ref:`assign step<tdl-step-assign>` and then logs it.
#. Validates the invoice's type using namespaces with the :ref:`XPathValidator<handlers-XPathValidator>`.

.. code-block:: xml

    <testcase>
        <!-- 
            Declare the namespaces to use used in XPath expressions.
        -->
        <namespaces>
           <ns prefix="inv">urn:oasis:names:specification:ubl:schema:xsd:Invoice-2</ns>
           <ns prefix="cbc">urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2</ns>
        </namespaces>
        <steps>
            <!-- 
                Request the user to upload the invoice to validate.
            -->
            <interact id="input" desc="Upload invoice">
                <request desc="File upload" name="xml" inputType="UPLOAD"/>
            </interact>
            <!--
                Use an XPath expression to extract the invoice type as an XML element.
            -->
            <assign to="invoiceTypeElement" type="object" source="$input{xml}">/inv:Invoice/cbc:InvoiceTypeCode</assign>
            <!--
                Log the extracted element.
            -->
            <log>$invoiceTypeElement</log>
            <!--
                Use XPath to validate the invoice.
            -->
            <verify handler="XPathValidator" desc="Check invoice type">
                <input name="xml">$input{xml}</input>
                <input name="expression">"/inv:Invoice/cbc:InvoiceTypeCode/text() = '380'"</input>
            </verify>  
        </steps>
    </testcase>

.. index:: imports (Test case)
.. index:: artifact (Test case imports)
.. index:: name (Test case imports)
.. index:: type (Test case imports)
.. index:: encoding (Test case imports)
.. index:: from (Test case imports)
.. _test-case-imports:

Imports
~~~~~~~

The ``imports`` element allows the use of arbitrary resources from the same or another test suite. This can be very useful when a test case needs to send messages
based on a template or load a binary file that is needed as input by a messaging, processing or validation handler (e.g. a certificate). The ``imports`` element 
defines one or more ``artifact`` children with the following structure:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"
    :delim: |

    @encoding | No | In case the artefact is to be treated as text, this is the character encoding to apply when reading its bytes (default is "UTF-8").
    @from | No | The identifier of another test suite from which this resource will be loaded. If unspecified the current test suite is assumed.
    @name | Yes | The name with which this artefact will be associated to the test session context for subsequent lookups.
    @type | No | The type as which the artefact needs to be loaded (default is ``binary``).

The text value of the ``artifact`` element is the path within the test suite from which the relevant resource will be loaded. This path may be provided as a
fixed value or as a :ref:`variable reference<test-case-referring-to-variables>` to determine the imported resource dynamically. In case a variable reference
is provided this should be one of the following:

* A reference to a configuration value (i.e. a :ref:`domain<test-case-expressions-domain>`, :ref:`organisation<test-case-expressions-organisation>`, :ref:`system<test-case-expressions-system>` or :ref:`actor<test-case-expressions-actor>` parameter).
* A reference to a :ref:`variable<test-case-variables>` defined in the test case. In this case the value of the variable can even be adapted during the course of the test session resulting in
  different resources depending on the point at which the import is referenced.

Importing resources is not limited to the current test suite. Using the ``from`` attribute it is possible to define another
test suite as the source from which to lookup the resource, specifying as the attribute's value the identifier of the target
test suite. The lookup of the test suite using the ``from`` value is carried out as follows:

#. Look for the test suite in the same **specification** as the test case being executed.
#. If not found in the same specification, look for the test suite in the other specifications of the test case's **domain**. If across
   specifications multiple matching test suites are found, one of them will be arbitrarily picked. To avoid such a scenario
   it is obvious that you should ensure test suites used to load shared resources can be uniquely identified.

The ``type`` attribute is optional and defaults to ``binary`` denoting a general-purpose file (regardless of whether it is text-based or not). You would
likely never need to set this explicitly, however if you choose to do so you can set it set as:

* ``binary``: To load the artefact as a set of bytes without additional processing.
* ``object``: To load the artefact as a XML Document Object Model.
* ``schema``: To load the artefact as a XML Schema or Schematron file.

In case the file is text-based you also have the option of setting the ``encoding`` attribute to consider (by default set as ``UTF-8``).

Regarding the path to the resource this is the resource's path within the test suite archive (with or without the test suite ID as a prefix). As an
example consider the following test case fragment where a XML schema is loaded and set in the session context as a variable named "ublSchema". The
path specified suggests that the file is named "UBL-Invoice-2.1.xsd" and exists in a folder within the test suite archive named "resources". This example also includes
another input whose referenced resource is defined dynamically based on an external configuration parameter (at organisation level in this case).

.. code-block:: xml

    <testcase>
        <imports>
            <!--
                The "ublSchema" is loaded from a fixed resource within the test suite.
            -->
            <artifact name="ublSchema">resources/UBL-Invoice-2.1.xsd</artifact>
            <!--
                The "organisationSpecificSchema" is loaded dynamically based on an organisation-level configuration property named "xsdToUseForOrganisation".
            -->
            <artifact name="organisationSpecificSchema">$ORGANISATION{xsdToUseForOrganisation}</artifact>
        </imports>
        <steps>
            <verify handler="XmlValidator" desc="Validate invoice against UBL 2.1 Invoice Schema">
                <!-- 
                    Variable $fileContent is loaded in another step.
                -->
                <input name="xml">$fileContent</input>
                <input name="xsd">$ublSchema</input>
            </verify>
        </steps>
    </testcase>

It is also possible to :ref:`import resources from other test suites <test-suite-sharing>`. To do this you use the ``from`` attribute identifying the
test suite that contains the resource, in which case the provided path is resolved in the context of the other test suite.

.. code-block:: xml

    <testcase>
        <imports>
            <!--
                The "ublSchema" is loaded from a fixed resource within a test suite with identifier "testSuite2".
            -->
            <artifact name="ublSchema" from="testSuite2">resources/UBL-Invoice-2.1.xsd</artifact>
        </imports>
    </testcase>

.. note::
    **Test module import:** The GITB TDL schema allows also for ``module`` elements to be defined for the import of test modules (validation, 
    messaging and processing handlers). This approach is no longer supported as it required the handler implementations to be bundled within 
    the Test Bed itself. The preferred and simpler approach now is to simply define the handler in the respective test step (e.g. the ``verify``
    step's ``handler`` attribute for validators) without previously importing it.

.. index:: preliminary (Test case)
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
        <preliminary desc="Prepare your system">
            <instruct desc="Preparation instructions">"Make sure your system is up and running"</instruct>
            <request desc="Provide your configuration file" contentType="BASE64">$sutConfigFile</request>
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

.. index:: actors (Test case)
.. index:: id (Test case actors)
.. index:: name (Test case actors)
.. index:: role (Test case actors)
.. index:: displayOrder (Test case actors)
.. index:: endpoint (Test case actors)
.. _test-case-actors:

Actors
~~~~~~

The ``actors`` element is where the test case defines the actors involved in its steps and, importantly, their role. It contains
one or more ``actor`` children with the following structure:

.. csv-table::
    :stub-columns: 1
    :delim: ~
    :header: "Name", "Required?", "Description"

    @displayOrder~ no~ A number indicating the relative positioning that needs to be respected when displaying the actor in test case's execution diagram. Setting this here overrides any corresponding setting at test suite level (see :ref:`test-suite-actors` for details).
    @id~ yes~ The actor's unique (within the specification) ID. This must match an actor ID specified in the test suite.
    @name~ no~ The name to display for the actor. This can differ from the ID to display an actor name specific to the test case. Not specifying this will default to the name for actor provided in the test suite.
    @role~ no~ The actor's role in the test case. This is "SUT" if the actor is the focus of the test case, "SIMULATED" (the default value) if the actor is simulated by the Test Bed, or "MONITOR" if the actor is present for monitoring purposes.
    endpoint~ no~ An optional sequence of configuration endpoints if the actor is simulated.

The main purpose of the ``actors`` element in the test case is to identify which of the :ref:`actors defined in the test suite <test-suite-actors>`
is the SUT (the actor the target system is testing for). This is done simply by defining the ``role`` attribute as follows:

.. code-block:: xml

    <testcase>
        <gitb:actor id="sender" role="SUT"/>
        <!-- The "SIMULATED" role is considered by default. -->
        <gitb:actor id="receiver"/>
    </testcase>

Besides defining the actors involved in the test case, you can also override their presentation by means of the ``name`` and ``displayOrder``
attributes:

.. code-block:: xml

    <testcase>
        <gitb:actor id="sender" role="SUT" name="Message sender" displayOrder="0"/>
        <gitb:actor id="receiver" name="Message receiver" displayOrder="1"/>
    </testcase>

Actor ``endpoint`` elements used in test cases require a bit more explanation to understand their use. They serve a niche case for test suites
including multiple actors defined in test cases as SUTs, and for each of which actor-level configuration properties are foreseen. In practice,
a simpler and typically more flexible approach is to use several :ref:`system-level configuration properties <test-case-expressions-system>`.

If you still require actor-level configuration for such cases, you can use the actors' ``endpoint`` elements to define default configuration values for
simulated actors. Imagine a specification that defines "sender" and "receiver" actors that can both be the SUTs depending on the actor a system selects to test for.
As such, a test suite focusing on the sender will include test cases with the sender as the SUT and the receiver as being simulated. Similarly, a
test suite focusing on the receiver will define the receiver as SUT and the sender as simulated. In terms of configuration properties, the sender
might need to define a "replyToAddress" to receive replies, whereas the receiver simply needs to define his "deliveryAddress" which is where messages
are expected. In terms of :ref:`actor configuration in the test suite <test-suite-actors>` this would look like this:

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

* **Dynamically** through a :ref:`custom messaging handler <handlers-custom-handlers>`. Using this approach, the Test Bed, while in its initiation phase, will request configuration
  properties from the handler that will be mapped to the SUT's corresponding endpoint (see :ref:`test-suite-actors-endpoints-simulated`).
* **Statically** by defining the endpoint and one or more of its parameters within the test case itself.

The second option is why we are able to configure ``endpoint`` elements as part of the test case. The values configured here will be used only 
if not already specified by the response of the simulated actor's handler. The below snippet illustrates this considering the sender as the SUT:

.. code-block:: xml

    <testcase>
        <gitb:actor id="sender" name="sender" role="SUT"/>
        <gitb:actor id="receiver" name="receiver">
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

.. index:: variables (Test case)
.. index:: name (Test case variables)
.. index:: type (Test case variables)
.. index:: value (Test case variables)
.. _test-case-variables:

Variables
~~~~~~~~~

.. note::

    **Implicit variables:** Variables are automatically created for new :ref:`assignments <tdl-step-assign>` and you should typically never need
    to declare them explicitly. The exception is test cases `executed via API <https://www.itb.ec.europa.eu/docs/itb-ou/latest/api/index.html#start>`__
    that are designed to work with client-provided inputs. Such inputs must map to test case variables.

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
language in place. Using the default XPath 3.0 expression language a variable named ``myVar`` is referenced as ``$myVar``. More information on 
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
* To cover inputs provided for test sessions started via `REST API <https://www.itb.ec.europa.eu/docs/itb-ou/latest/api/index.html#start>`__.

For examples of automatic variable definition refer to the corresponding steps as well as the documentation on :ref:`expressions<test-case-variables-from-expression-output>`.
Coming back to explicitly defined variables, the following example shows two such cases, one to store a user-uploaded file and another to store a part of it, 
extracted via XPath:

.. code-block:: xml

    <testcase>
        <imports>
            <artifact name="schemaFile">artifacts/UBL/maindoc/UBL-Invoice-2.1.xsd</artifact>
        </imports>
        <variables>
            <var name="fileContent" type="object"/>
            <var name="targetElement" type="object"/>
        </variables>
        <steps>
            <!-- 
                Store the uploaded result in the fileContent variable.
            -->
            <interact desc="UBL invoice upload">
                <request desc="Upload the UBL invoice to validate" contentType="BASE64">$fileContent</request>
            </interact>
            <!-- 
                Extract a part of it and store in the targetElement variable.
            -->
            <assign to="$targetElement" source="$fileContent">/*[local-name() = 'testcase']/*[local-name() = 'steps']</assign>
            <!-- 
                Pass the targetElement for validation.
            -->
            <verify handler="XmlValidator" desc="Validate content">
                <input name="xml">$targetElement</input>
                <input name="xsd">$schemaFile</input>
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

.. index:: steps (Test case)
.. index:: stopOnError (Test case)
.. index:: logLevel
.. index:: ERROR (Test case logLevel)
.. index:: WARNING (Test case logLevel)
.. index:: INFO (Test case logLevel)
.. index:: DEBUG (Test case logLevel)

.. _test-case-steps:

Steps
~~~~~

The ``steps`` element is where the test case's testing logic is implemented. It consists of a sequence of test steps realised by means
of a GITB TDL step construct. The structure of the element is as follows:

.. csv-table::
    :stub-columns: 1
    :delim: ~
    :header: "Name", "Required?", "Description"

    @logLevel~ no~ The minimum logging level that this test case should produce. This can be (in increasing severity) ``DEBUG``, ``INFO`` (the default level), ``WARNING`` or ``ERROR``, but can also be set dynamically as a variable reference. See also the :ref:`tdl-step-log` step.
    @stopOnError~ no~ A boolean flag determining whether the test session should stop if any step fails (default is "false"). See also :ref:`tdl-steps-common-stoponerror`.

.. _test-case-steps__logging:

Test case logging
+++++++++++++++++

The test case's **logging level** affects log statements produced automatically by the Test Bed or :ref:`added explicitly by the test case<tdl-step-log>`.
While executing a test session, the Test Bed automatically produces the following log output:

* At ``DEBUG`` level, information on each step's start, end and latest status.
* At ``INFO`` level, information on key lifecycle points such as the start and end of the session.
* At ``WARNING`` level, detected issues that although not blocking for the test session could be signs of problems.
* At ``ERROR`` level, information on unexpected errors that forced the test session to fail.

When using the ``logLevel`` attribute to set the test case's log level, this defines the minimum level of messages to be added to the
session's log. A good example is setting the ``logLevel`` to ``WARN`` which will exclude all ``DEBUG`` and ``INFO`` output while including all output 
of levels ``WARNING`` and ``ERROR``. This could be interesting if you want to only share test engine errors and problematic issues signalled by
using the :ref:`tdl-step-log` step while ignoring status updates.

In certain cases you may prefer to set the test case logging level dynamically. This is achieved by using a variable reference as the 
``logLevel`` value, referring either to a :ref:`configuration property<test-case-configuration>` or a :ref:`predefined test case variable<test-case-variables>`.
Interestingly, when referring to a variable and given that the provided expression is calculated every time, you can adapt the test case's
logging level during the course of the test session. You may for example start with a ``WARNING`` level but switch to ``INFO``
for a specific set of steps. An example of this is illustrated below:

.. code-block:: xml

    <testcase>
        <variables>
            <var name="loggingLevel" type="string">
                <value>WARNING</value>
            </var>
        </variables>
        <steps logLevel="$loggingLevel">
            <!-- 
                The following log entry is ignored as we only log at WARNING level and above.
            -->
            <log level="INFO">'An info message'</log>
            <!-- 
                For the group that follows switch to INFO level.
            -->
            <assign to="loggingLevel">'INFO'</assign>
            <group>
                ...
            </group>
            <!-- 
                Switch back to WARNING level.
            -->
            <assign to="loggingLevel">'WARNING'</assign>
            ...
        </steps>
    </testcase>

.. _test-case-steps__steps:

Available steps
+++++++++++++++

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

.. index:: output (Test case)
.. index:: success (Test case output)
.. index:: failure (Test case output)
.. index:: undefined (Test case output)
.. index:: case (Test case output)
.. index:: default (Test case output)
.. index:: cond (Test case output)
.. index:: message (Test case output)
.. index:: match (Test case output)
.. index:: all (Test case output)
.. index:: first (Test case output)
.. index:: cascade (Test case output)
.. _test-case-output:

Output
~~~~~~

The ``output`` element is an optional means of defining a final result message for a given test session. It is processed once all
steps have completed, checking the data in the test session's context to display specific messages. Using this
element allows extended feedback to be returned that may be important to summarise and contextualise the steps' results.

The ``output`` element supports messages for a success, failure, or undefined result; defined as different cases with match conditions and an
overall default. The structure of the ``output`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    failure, no, The set of output cases to apply in case the test completes with a failure.
    success, no, The set of output cases to apply in case the test completes with a success.
    undefined, no, The set of output cases to apply in case the test completes with an undefined result.

The ``success``, ``failure`` and ``undefined`` elements share a common structure to define their specific cases and overall defaults:

.. csv-table::
    :stub-columns: 1
    :delim: ~
    :header: "Name", "Required?", "Description"

    @match ~ no ~ The approach to follow when matching message cases. Can be ``first`` (the default), ``all`` or ``cascade``.
    case ~ no ~ Zero or more specific cases to apply depending on the provided match conditions.
    default ~ no ~ An optional default if no specific case was found to apply.

Finally, each ``case`` element shares a common structure as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    cond, yes, Defines a condition :ref:`expression<test-case-expressions>` expected to return a "true" or "false" value.
    message, yes, Defines an :ref:`expression<test-case-expressions>` expected to return the output message (as a string).

The ``output`` section is flexible as it doesn't require you to define messages for each outcome type. In addition, you could
opt only to provide default messages without specific cases or only provide certain specific messages without generic defaults.
It is important to note that the condition expressions tolerate failures, meaning that if a condition cannot
be evaluated its relevant case will simply be skipped. In addition, if any error is raised when creating the text message itself, the output
message will be altogether skipped; under no circumstances will a test session fail due to the evaluation of its ``output`` section
(relevant warnings will however be included in the test session's log).

The following snippet illustrates how the ``output`` section could be leveraged to return user-friendly failure messages based on the executed
test steps (using the :ref:`STEP_STATUS<test-case-expressions-step-status>` variable to determine the cause of the failure). For successful
and undefined states we only include a default message to make test sessions more user friendly.

.. code-block:: xml

    <testcase>
        <steps stopOnError="true">
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
            <success>
                <default>"Test session completed successfully."</default>
            </success>
            <failure>
                <case>
                    <cond>$STEP_STATUS{checkIntegrity} = 'ERROR'</cond>
                    <message>"Please verify the integrity of your data and re-submit."</message>
                </case>
                <case>
                    <cond>$STEP_STATUS{checkSyntax} = 'ERROR'</cond>
                    <message>"Please verify the syntax of your data and re-submit."</message>
                </case>
                <case>
                    <cond>$STEP_STATUS{checkContent} = 'ERROR'</cond>
                    <message>"Please verify your data content and re-submit."</message>
                </case>
                <!-- The default will be applied if no specific case was found to match. -->
                <default>"Your data failed to be processed correctly. Please check the session log to determine the cause of the failure."</default>
            </failure>
            <undefined>
                <default>"Test session stopped before producing a result."</default>
            </undefined>
        </output>
    </testcase>

It may seem at first unintuitive to provide output messages for anything other than failures. However, there are a few cases where this could be
particularly useful:

* A success message as a **user-friendly confirmation** that the test succeeded.
* A success message to **highlight a warning** (steps resulting in warnings are successful).
* An undefined message for an :ref:`exit step <tdl-step-exit>` that terminated with an undefined result.

As mentioned earlier you can choose to omit certain outcomes altogether. For example, the following test case defines only failure messages
(a specific one and a default one):

.. code-block:: xml

    <testcase>
        <steps stopOnError="true">
            ...
        </steps>
        <output>
            <failure>
                <case>
                    <cond>$STEP_STATUS{checkContent} = 'ERROR'</cond>
                    <message>"Please verify your data content and re-submit."</message>
                </case>
                <default>"Your data failed to be processed correctly. Please check the session log to determine the cause of the failure."</default>
            </failure>
        </output>
    </testcase>

The examples up to this point consider outputting a single message depending on the result. It is possible to have **several messages** be 
returned which could be used as an alternative to test step reports for simple test cases. This is achieved through the ``match`` attribute
of the ``failure``, ``success`` and ``undefined`` elements that supports the following values:

* ``first`` (the default), meaning that the first (sequentially) condition to match will be considered. If none match the
  default applies.
* ``all``, meaning that all conditions with passing conditions will be considered. If none match the default applies.
* ``cascade``, meaning that once a first condition is found to pass, the check will cascade to subsequent conditions and include them if
  they also pass. If a condition check fails then the cascade stops even if later conditions would also pass. If none match
  the default applies.

To illustrate this with an example, consider the following test case output section:

.. code-block:: xml

    <output>
        <failure match="all">
            <case>
                <cond>$flag1</cond>
                <message>"The integrity check on your data failed."</message>
            </case>
            <case>
                <cond>$flag2</cond>
                <message>"The syntax check on your data failed."</message>
            </case>
            <default>"Your data failed to be processed correctly. Please check the session log to determine the cause of the failure."</default>
        </failure>
    </output>

In the above example given that match is set to ``all``, both messages could be displayed if both ``flag1`` and ``flag2`` are true.
If ``match`` was set to ``first`` then only the first message would be generated. Finally, the ``cascade`` option allows you to
fine-tune the display of multiple related messages, without forcing an all or nothing approach.

.. note::
    **Messages are also expressions:** Output messages are themselves :ref:`expressions<test-case-expressions>` allowing dynamic
    output to be returned (e.g. concatenating text with session variable values). For a fixed message make sure to enclose the text
    in quotes.

.. index:: scriptlets (Test case)
.. _test-case-scriptlets:

Scriptlets
~~~~~~~~~~

The ``scriptlets`` element is meant to define reusable blocks of steps that can be called during the test case's execution.
They resemble function blocks in programming languages considering that they can be called multiple times with different
inputs and produce different outputs.

Scriptlets are typically defined as separate XML documents that can be used across test cases or even across test suites.
Defining a scriptlet within a test case results in it being private to its containing test case. To define such private
scriptlets, include the ``scriptlets`` element with one or more ``scriptlet`` children.

Details on how each ``scriptlet`` element is defined are provided in the :ref:`scriptlet documentation<scriptlets>`. This
includes the :ref:`differences to consider<scriptlets_embedded>` when comparing scriptlets embedded in test cases and ones
that are defined as standalone XML documents.

Calling a scriptlet from a test case is achieved through the :ref:`call<tdl-step-call>` step. The following example
illustrates the definition of a scriptlet within a test case to validate XML documents. This is called twice for each
of the inputs provided by the user.

.. code-block:: xml

    <testcase>
        <steps>
            <!-- 
                Request two files to be uploaded.
            -->
            <interact id="userData" desc="Upload files">
                <request desc="Upload the first file" name="file1" contentType="BASE64"/>
                <request desc="Upload the second file" name="file2" contentType="BASE64"/>
            </interact>
            <!-- 
                Call the scriptlet for the first file and store the result under variable "call1".
            -->
            <call id="call1" path="validateDocument">
                <input name="contentToValidate">$userData{file1}</input>
            </call>
            <!-- 
                Call the scriptlet for the second file and store the result under variable "call2".
            -->
            <call id="call2" path="validateDocument">
                <input name="contentToValidate">$userData{file2}</input>
            </call>
            <!--
                Log the root element names of the validated files.
            -->
            <log>"File 1: " || $call1{rootName}</log>
            <log>"File 2: " || $call2{rootName}</log>
        </steps>
        <scriptlets>
            <scriptlet id="validateDocument">
                <imports>
                    <artifact name="schemaToUse">resources/aSchemaFile.xsd</artifact>
                    <artifact name="schematronToUse">resources/aSchematronFile.sch</artifact>
                </imports>
                <params>
                    <var name="contentToValidate" type="object"/>
                </params>
                <steps>
                    <verify handler="XmlValidator" desc="Validate XML structure">
                        <input name="xml">$contentToValidate</input>
                        <input name="xsd">$schemaToUse</input>
                    </verify>
                    <verify handler="XmlValidator" desc="Validate business rules">
                        <input name="xml">$contentToValidate</input>
                        <input name="schematron">$schematronToUse</input>
                    </verify>
                </steps>
                <output name="rootName" source="$contentToValidate">name(/*)</output>
            </scriptlet>
        </scriptlets>
    </testcase>

.. note::
    **Using scriptlets across test cases:** Scriptlets defined within test cases are private to that test case. If you want
    to use a scriptlet across several test cases, within the test suite or across test suites, you need to define it in
    its own XML document. See the :ref:`scriptlet documentation<scriptlets>` for details on this.
