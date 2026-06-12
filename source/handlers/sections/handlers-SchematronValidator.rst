.. note::

  For Schematron validation consider using the :ref:`handlers-XmlValidator` handler which offering more features and flexibility.

Used to validate an XML document against a Schematron file.

.. csv-table::
    :header: "Input name", "Required?", "Type", "Description"

    ``schematron``, Yes, ``schema``, The Schematron file to use for the validation (XSTL or SCH).
    ``showLocationPaths``, No, ``boolean``, Whether or reported items should include in their location information the path for the relevant element (default is ``false``).
    ``showSchematron``, No, ``boolean``, Whether or not to include in the step's report the Schematron used for the validation (default is "true").
    ``showTests``, No, ``boolean``, Whether or not to include in the step's report the assertion performed for each finding (default is ``false``).
    ``sortBySeverity``, No, ``boolean``, Whether to sort findings by severity ("true") or location in the input (``false`` - the default).
    ``type``, No, ``string``, The type of Schematron file to consider (``xslt`` or ``sch``) in case this cannot be determined from the resource's file suffix. The overall default considered is ``sch``.
    ``xml``, Yes, ``object``, The XML document to validate.

.. note::
    **XSLT vs SCH Schematron files:** XSLT versions of Schematron files are pre-processed files and offer significantly better
    performance for complex rule cases. In addition, if Schematron rules import other resources or use functions (built-in or custom),
    use of XSLT files is required. The ``SchematronValidator`` considers SCH as the default format so you if you use XSLT-formatted Schematron,
    you must define the ``type`` input with a value of ``xslt``.

.. code-block:: xml

    <verify handler="SchematronValidator" desc="Validate content">
        <input name="xml">$docToValidate</input>
        <input name="schematron">$schematronFile</input>
        <!-- Following inputs are optional. -->
        <input name="showSchematron">false()</input>
        <input name="sortBySeverity">true()</input>
        <input name="showTests">true()</input>
        <input name="showLocationPaths">true()</input>
    </verify>