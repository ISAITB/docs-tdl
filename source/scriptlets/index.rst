.. index:: Scriptlets
.. _scriptlets:

Scriptlets
==========

.. _scriptlets_overview:

Overview
--------

Scriptlets are reusable blocks of test steps that can be called during a test case's execution. They are similar to
function blocks in programming languages considering that:

* They can receive inputs and may produce outputs.
* They can define their own imports.
* They operate in their own isolated scope.

Extracting reusable sections of test steps is a common need in test suites where certain set of tasks are frequently
encountered. Consider for example a test suite that contains numerous test cases that at some point need to validate
certain received data against the same common validation artefacts. Rather than repeating the same set of steps and
artefact imports across all test cases, these validation steps can be defined in a scriptlet that is called wherever
necessary. Scriptlets are defined as separate XML documents within a test suite, but can also be defined
:ref:`embedded within specific test cases<scriptlets_embedded>`.

The following example is a complete scriptlet that is used to validate a provided XML document against an XML Schema
and a Schematron file. As output the scriptlet returns the name of the file's root element:

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <scriptlet id="validateDocument" xmlns="http://www.gitb.com/tdl/v1/">
        <imports>
            <artifact type="schema" encoding="UTF-8" name="schemaToUse">resources/aSchemaFile.xsd</artifact>
            <artifact type="schema" encoding="UTF-8" name="schematronToUse">resources/aSchematronFile.sch</artifact>
        </imports>
        <params>
            <var name="contentToValidate" type="object"/>
        </params>
        <steps>
            <verify handler="XSDValidator" desc="Validate XML structure">
                <input name="xsddocument">$schemaToUse</input>
                <input name="xmldocument">$contentToValidate</input>
            </verify>
            <verify handler="SchematronValidator" desc="Validate XML content">
                <input name="schematron">$schematronToUse</input>
                <input name="xmldocument">$contentToValidate</input>
            </verify>
            <assign to="rootName" source="$contentToValidate">name(/*)</assign>
        </steps>
        <output name="rootName"/>
    </scriptlet>

Scriptlets are used in test cases or other scriptlets by means of the :ref:`call step<tdl-step-call>`. Considering the
previous example, an example call from a test case would be as follows:

.. code-block:: xml

    <!--
        Assume the scriptlet is defined in the same test suite in file "scriptlets/validateDocument.xml".
    -->
    <call id="call1" path="scriptlets/validateDocument.xml">
        <!--
            The variable "anXmlFile" contains the XML content to validate.
        -->
        <input name="contentToValidate">$anXmlFile</input>
    </call>
    <!--
        Log the XML file's root element name.
    -->
    <log>$call1{rootName}</log>

The following table provides an overview of the attributes and child elements that a ``scriptlet`` may have. A more detailed discussion per case
follows in the subsequent sections.

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @id, Yes, A unique identifier for the scriptlet.
    metadata, No, A block containing the metadata used to describe the scriptlet.
    namespaces, No, An optional set of namespaces to define the expression languages used in the scriptlet.
    imports, No, An optional set of imports used to load additional resources.
    params, No, An optional set of parameters that the scriptlet expects as input when called.
    variables, No, An optional set of variables that are used locally within the scriptlet.
    steps, Yes, The sequence of steps that this scriptlet foresees.
    output, No, An optional set of output values that the scriptlet will return to its caller.

.. _scriptlets_elements:

Elements
--------

The following sections discuss the purpose and use of each element contained within the ``scriptlet`` definition.

.. index:: metadata (Scriptlets)
.. _scriptlets_elements_metadata:

Metadata
~~~~~~~~

The structure and content of the ``metadata`` element is identical to the one defined for the :ref:`test suite<test-suite-metadata>`
and its :ref:`test cases<test-case-metadata>`. In the case of scriptlets however, no such metadata is displayed to users
meaning that any information provided here is purely for test developers. In contrast to test suites and test cases,
the ``metadata`` block for scriptlets is optional and can be fully skipped.

For further information on this element check the :ref:`test case metadata documentation<test-case-metadata>`.

.. index:: namespaces (Scriptlets)
.. _scriptlets_elements_namespaces:

Namespaces
~~~~~~~~~~

The ``namespaces`` element is identical in structure and purpose to the one defined for :ref:`test cases<test-case-namespaces>`.
It is used to define one or more expression languages that are used in test case constructs that support them.

For further information on this element check the :ref:`test case namespaces documentation<test-case-namespaces>`.

.. note::
    **GITB software support:** Expression languages other than the default XPath 1.0 are not supported. As such the ``namespaces`` element is currently ignored.

.. index:: imports (Scriptlets)
.. _scriptlets_elements_imports:

Imports
~~~~~~~

The ``imports`` element allows the use of arbitrary resources from the same or another test suite. The purpose and structure
of this element is identical to the one defined for :ref:`test cases<test-case-imports>`. One notable difference in the case
of scriptlets however, is that imports lacking an explicit ``from`` attribute are loaded from the test suite containing the
scriptlet, which is not necessarily the same as the test suite containing the currently executing test case.

For further information on this element check the :ref:`test case imports documentation<test-case-imports>`.

.. index:: params
.. index:: name (params)
.. index:: type (params)
.. _scriptlets_elements_params:

Params
~~~~~~

The ``params`` element is used to define the inputs that a scriptlet expects. Each such input is defined as a ``var``
element with the following structure:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @name, Yes, The name of the parameter. It is with this name that the parameter can be referenced within the scriptlet.
    @type, Yes, The type of the parameter. One of the GITB data types can be used (see :ref:`test-case-types`).

Whenever a scriptlet is called using a :ref:`call<tdl-step-call>` step, each of its declared parameters must be matched
by a provided input. Once provided, such parameters are available in the scope of the scriptlet and can be used in the same way
as other variables. For scriptlets that are not :ref:`embedded within test cases<scriptlets_embedded>`, using parameters
is the only way to provide state from the test session's context to the scriptlet.

.. index:: variables (Scriptlets)
.. _scriptlets_elements_variables:

Variables
~~~~~~~~~

The ``variables`` element can be defined to create one or more variables that will be used during the scriptlet's execution.
The purpose and structure of this element is identical to the one defined for :ref:`test cases<test-case-variables>`.
One notable point is that a scriptlet's variables, as well as all other data in its scope, is isolated from the test
case or scriptlet that is calling it, unless this is a scriptlet :ref:`embedded within a test case<scriptlets_embedded>`.

For further information on this element check the :ref:`test case variables documentation<test-case-variables>`.

.. index:: steps (Scriptlets)
.. _scriptlets_elements_steps:

Steps
~~~~~

The ``steps`` element is where the scriptlet's testing logic is implemented. It consists of a sequence of test steps, each realised by means
of a GITB TDL step construct. The purpose and structure of this element is identical to the one defined for :ref:`test cases<test-case-steps>`,
with the only notable difference being how **actors** are considered in :ref:`messaging steps<tdl-messaging-steps>`.

A test case's :ref:`actors<test-case-actors>` represent the parties involved in the test. Each actor is assigned an ``id``
which is then referenced in :ref:`messaging steps<tdl-messaging-steps>` to determine the flow of communications. A scriptlet
does not include such an actor definition block given that it is not the scriptlet's role to determine which actors
are simulated and which is the system under test (SUT). If a scriptlet includes :ref:`messaging steps<tdl-messaging-steps>`
the test developer needs to ensure that the actor IDs that are referenced will match those defined by its calling test cases.

For further information on the scriptlet's ``steps`` element and the different constructs it can include check the :ref:`test case steps documentation<test-case-steps>`.

.. index:: output (Scriptlets)
.. index:: name (Scriptlet output)
.. index:: lang (Scriptlet output)
.. index:: source (Scriptlet output)
.. index:: asTemplate (Scriptlet output)
.. _scriptlets_elements_output:

Output
~~~~~~

A scriptlet's results are returned by the ``output`` elements that it defines. These outputs are the only means a scriptlet
has to communicate information back to its caller, be it a test case or another scriptlet, but can be omitted in case
no results are expected. The structure of each ``output`` element is as follows:

.. csv-table::
    :header: "Name", "Required?", "Description"
    :stub-columns: 1
    :delim: |

    @name | Yes | A name with which the output can be referenced.
    @lang | No | The expression language prefix to use to evaluate the contained expression (see :ref:`test-case-namespaces` and :ref:`test-case-expressions`).
    @source | No | A variable reference to identify a source ``object`` variable upon which the contained expression should be evaluated.
    @asTemplate | No | Whether or not the result will be considered as a template for placeholder replacement (see :ref:`test-case-expressions-template-files`). By default this is "false".

The content of the ``output`` element is an :ref:`expression<test-case-expressions>` that is used once the scriptlet's processing
is complete to calculate the output's value. In the special case where a result needs to be calculated from an XPath expression
on an XML document, the ``source`` attribute can be used to define the document to consider. The value returned by an ``output``
element is calculated as follows:

#. If the ``output`` element is non-empty, its content is used as an expression to calculate the return value.
#. If the ``output`` element is empty the ``name`` of the ``output`` element must match the name of a variable present
   in the scriptlet's scope. If matched, the value of the variable is returned.

Taking the sample scriptlet presented previously, we want to return from the scriptlet a single output named "rootName"
which is set with the validated document's root element name. A first way to achieve this is to reference a variable
from the scriptlet's scope that contains the calculated value.

.. code-block:: xml

    <scriptlet id="validateDocument" xmlns="http://www.gitb.com/tdl/v1/">
        <steps>
            ...
            <!--
                Calculate the document's root name and assign it to a variable named "rootName".
            -->
            <assign to="rootName" source="$contentToValidate">name(/*)</assign>
        </steps>
        <!--
            Reference the "rootName" variable from the scriptlet's scope.
        -->
        <output name="rootName"/>
    </scriptlet>

Using this approach we treat the ``output`` element simply as a definition for which we need to ensure that, once complete,
the scriptlet's scope will include a similarly named variable. Note how the ``output`` element is itself empty given that
it should not evaluate any expression to return its value. An alternative approach would be to use this expression to
define the value as follows:

.. code-block:: xml

    <scriptlet id="validateDocument" xmlns="http://www.gitb.com/tdl/v1/">
        <steps>
            ...
            <assign to="documentRootName" source="$contentToValidate">name(/*)</assign>
        </steps>
        <!--
            Reference the "documentRootName" variable from the scriptlet's scope.
        -->
        <output name="rootName">$documentRootName</output>
    </scriptlet>

This uses a simple :ref:`variable expression<test-case-referring-to-variables>` and is largely equivalent. We could of course
provide a more elaborate :ref:`expression<test-case-expressions>` here to e.g. enclose the value in square brackets:

.. code-block:: xml

    <output name="rootName">concat('[', $documentRootName, ']')</output>

Finally, when using the ``output`` element's expression to calculate its return value we could in our example skip the previous
:ref:`assign<tdl-step-assign>` step altogether as follows:

.. code-block:: xml

    <scriptlet id="validateDocument" xmlns="http://www.gitb.com/tdl/v1/">
        <steps>
            ...
        </steps>
        <!--
            Calculate the value to return in the output element itself.
        -->
        <output name="rootName" source="$contentToValidate">name(/*)</output>
    </scriptlet>

Keep in mind that you may return any number of ``output`` elements you want. As an example, we could adapt our scriptlet
to return also information on the result of the individual checks it made, in case the caller wants to take specific actions
per case.

.. code-block:: xml

    <scriptlet id="validateDocument" xmlns="http://www.gitb.com/tdl/v1/">
        <steps>
            <verify id="xsdCheck" handler="XSDValidator" desc="Validate XML structure">
                ...
            </verify>
            <verify id="schCheck" handler="SchematronValidator" desc="Validate XML content">
                ...
            </verify>
        </steps>
        <!--
            Return the name of the document's root element and the individual validation steps' results.
        -->
        <output name="rootName" source="$contentToValidate">name(/*)</output>
        <output name="isValidForSchema">$xsdCheck</output>
        <output name="isValidForSchematron">schCheck</output>
    </scriptlet>

In this example note how the :ref:`verify<tdl-step-verify>` steps are set with a specific ``id``. Once each validation is
completed the overall results are recorded as ``boolean`` variables which are then used to return the scriptlet's outputs
(see the :ref:`verify<tdl-step-verify>` step documentation for further details on this).

.. note::
    **Selecting outputs:** When a scriptlet returns multiple outputs its caller may choose to select only a subset of these to use. For details
    on this and how returned outputs can be used check the :ref:`call<tdl-step-call>` step's documentation.

.. _scriptlets_embedded:

Scriptlets embedded in test cases
---------------------------------

Scriptlets are typically defined as separate XML documents within a test suite that can be used across its test cases or
externally from other test suites. It is also possible however, to define scriptlets within a specific test case in its
:ref:`scriptlets<test-case-scriptlets>` element. Such embedded scriptlets are processed in the same way but have certain
key differences:

* **Scope inheritance:** The scope of an embedded scriptlet extends, without replacing, the scope of its included test case.
  This means that looking up a variable within the scriptlet will first check the scriptlet's scope and, if not found, will
  check the test case's scope. Across multiple embedded scriptlets however, scopes remain isolated.
* **Private access:** Embedded scriptlets are only usable from the test case that contains them and from its other embedded
  scriptlets.
* **Identification:** Embedded scriptlets rely on their ``id`` attribute to be identified and referenced in :ref:`call<tdl-step-call>`
  steps. As such, this must be unique within the scope of their containing test case.

For more details on defining embedded scriptlets check the documentation of the test case's :ref:`scriptlets<test-case-scriptlets>`
element. For information on how a scriptlet, embedded or not, is called check the documentation of the :ref:`call<tdl-step-call>`
step.