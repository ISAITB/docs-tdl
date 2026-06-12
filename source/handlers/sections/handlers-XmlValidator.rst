Used to validate an XML document against an XML Schema (XSD) and/or zero or more Schematron files.

.. csv-table::
    :header: "Input name", "Required?", "Type", "Description"
    :delim: |

    ``schematron`` | No | ``list[schema]`` | The list of Schematron files to validate the document's content against.
    ``schematronType`` | No | ``string`` | The type of Schematron file to consider (``xslt`` or ``sch``) in case this cannot be determined from the files' suffix. The overall default considered is ``sch``.
    ``schemaVersion`` | No | ``string`` | The XML Schema specification version to consider (``1.0`` or ``1.1``). If unspecified, this is defined by the schema's root ``minVersion`` element defaulting to ``1.0``.
    ``showLocationPaths`` | No | ``boolean`` | Whether or reported items for Schematron rules should include in their location information the path for the relevant element (default is ``false``).
    ``showSchematronTests`` | No | ``boolean`` | Whether or not the Schematron assertions applied should be displayed for each reported finding (default is ``false``).
    ``showValidationArtefacts`` | No | ``boolean`` | Whether or not the XSDs and/or Schematrons used for the validation should be included in the step's report (default is "true").
    ``sortBySeverity`` | No | ``boolean`` | Whether findings should be sorted by severity ("true") or by location in the XML content (``false`` - the default).
    ``stopOnXsdErrors`` | No | ``boolean`` | Whether or not XSD errors should prevent validation from proceeding with Schematron validations (default is "true").
    ``xml`` | Yes | ``object`` | The XML document to validate.
    ``xsd`` | No | ``schema`` | The XSD to validate the document's structure against.

Regarding inputs, if you need to supply a single Schematron file you don't need to create a ``list`` and pass it as such. You can
simply pass the Schematron file as-is and the test engine will automatically convert it to a single-element ``list``. Note that considering
that both the ``xsd`` and ``schematron`` inputs are optional, if you provide neither, the validation will simply succeed with an empty report.

.. note::
    **XSLT vs SCH Schematron files:** XSLT versions of Schematron files are pre-processed files and offer significantly better
    performance for complex rule cases. In addition, if Schematron rules import other resources or use functions (built-in or custom),
    use of XSLT files is required. The ``XmlValidator`` considers SCH as the default format so you if you use XSLT-formatted Schematron,
    you must define the ``schematronType`` input with a value of ``xslt``.

The following examples illustrate how the ``XmlValidator`` can be used in various scenarios:

.. code-block:: xml

    <!--
        Validate against an XSD.
    -->
    <verify handler="XmlValidator" desc="XML validation">
        <input name="xml">$content</input>
        <input name="xsd">$schema</input>
    </verify>
    <!--
        Validate against an XSD and force the use of XML Schema 1.1.
    -->
    <verify handler="XmlValidator" desc="XML validation with XML Schema 1.1">
        <input name="xml">$content</input>
        <input name="xsd">$schema</input>
        <input name="schemaVersion">"1.1"</input>
    </verify>
    <!--
        Validate against a single Schematron file.
    -->
    <verify handler="XmlValidator" desc="XML validation">
        <input name="xml">$content</input>
        <input name="schematron">$schematron</input>
    </verify>
    <!--
        Validate against a single Schematron file and include path locations in the resulting report.
    -->
    <verify handler="XmlValidator" desc="XML validation">
        <input name="xml">$content</input>
        <input name="schematron">$schematron</input>
        <input name="showLocationPaths">true()</input>
    </verify>
    <!--
        Validate against two Schematron files.
    -->
    <assign to="schematrons" append="true">$schematron1</assign>
    <assign to="schematrons" append="true">$schematron2</assign>
    <verify handler="XmlValidator" desc="XML validation">
        <input name="xml">$content</input>
        <input name="schematron">$schematrons</input>
    </verify>
    <!--
        Validate against an XSD and two Schematron files:
        - Without stopping for XSD errors.
        - Sorting findings by severity.
        - Hiding the XSD and Schematrons used.
    -->
    <assign to="schematrons" append="true">$schematron1</assign>
    <assign to="schematrons" append="true">$schematron2</assign>
    <verify handler="XmlValidator" desc="XML validation">
        <input name="xml">$content</input>
        <input name="schematron">$schematrons</input>
        <input name="stopOnXsdErrors">false()</input>
        <input name="sortBySeverity">true()</input>
        <input name="showValidationArtefacts">false()</input>
    </verify>

When comparing with the similar :ref:`handlers-XSDValidator` and :ref:`handlers-SchematronValidator`, the  ``XmlValidator`` is more
flexible and simple to use. In addition, it allows a better fine-tuning of how validation steps are presented. If for example validating
XML content requires validation against an XSD and two Schematron files, using the :ref:`handlers-XSDValidator` and :ref:`handlers-SchematronValidator`
will present three distinct validation steps in the session execution diagram. Using the ``XmlValidator`` you may still display each such
validation separately but you also have the option of making a single validation for all artefacts. Doing so is typically preferred because:

* It presents a **single logical step**, rather than expose the different resources involved.
* It aggregates all findings in a **single report**.

For the sake of comparison, the following examples illustrate how two distinct validations carried out with the :ref:`handlers-XSDValidator` and :ref:`handlers-SchematronValidator`
can be replicated via a single use of the ``XmlValidator``:

.. code-block:: xml

    <!--
        Using the XsdValidator and SchematronValidator.
    -->
    <verify handler="XsdValidator" desc="Validate content">
        <input name="xml">$docToValidate</input>
        <input name="xsd">$schemaFile</input>
    </verify>
    <verify handler="SchematronValidator" desc="Validate content">
        <input name="xml">$docToValidate</input>
        <input name="schematron">$schematronFile</input>
    </verify>
    <!--
        Equivalent validation using the XmlValidator.
    -->
    <verify handler="XmlValidator" desc="Validate content">
        <input name="xml">$docToValidate</input>
        <input name="xsd">$schemaFile</input>
        <input name="schematron">$schematronFile</input>
        <input name="stopOnXsdErrors">false()</input>
    </verify>