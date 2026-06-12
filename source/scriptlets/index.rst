.. index:: Scriptlets
.. _scriptlets:

Scriptlets
==========

.. _scriptlets_overview:

Overview
--------

Scriptlets are reusable blocks of test steps that can be called during a test case's execution. They are similar to
function blocks in programming languages considering that:

* They can receive **inputs** and may produce **outputs**.
* They can define their own **imports**.
* They define a **new scope** but can also read data from **parent scopes**.

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

.. include:: /scriptlets/sections/scriptlets_elements.rst

.. index:: params
.. index:: name (params)
.. index:: type (params)
.. index:: optional (params)
.. index:: defaultEmpty (params)
.. _scriptlets_elements_params:

Params
~~~~~~

.. include:: /scriptlets/sections/scriptlets_elements_params.rst

.. index:: hidden
.. index:: static
.. _scriptlets_dynamic_steps:

Dynamic steps in scriptlets
---------------------------

.. include:: /scriptlets/sections/scriptlets_dynamic_steps.rst

.. _scriptlets_embedded:

Scriptlets embedded in test cases
---------------------------------

.. include:: /scriptlets/sections/scriptlets_embedded.rst
