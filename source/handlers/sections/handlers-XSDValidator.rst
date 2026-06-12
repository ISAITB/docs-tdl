.. note::

  For XSD validation consider using the :ref:`handlers-XmlValidator` handler which offering more features and flexibility.

Used to validate an XML document against an XML Schema (XSD) instance.

.. csv-table::
    :header: "Input name", "Required?", "Type", "Description"
    :delim: |

    ``schemaVersion`` | No | ``string`` | The XML Schema specification version to consider (``1.0`` or ``1.1``). If unspecified, this is defined by the schema's root ``minVersion`` element defaulting to ``1.0``.
    ``showSchema``| No | ``boolean`` | Whether to include in the step's report the XSD used for the validation (default is "true").
    ``sortBySeverity``| No | ``boolean`` | Whether to sort findings by severity ("true") or location in the input (``false`` - the default).
    ``xml``| Yes | ``object`` | The XML document to validate.
    ``xsd``| Yes | ``schema`` | The XSD to validate the document against.

.. code-block:: xml

    <verify handler="XsdValidator" desc="Validate content">
        <input name="xml">$docToValidate</input>
        <input name="xsd">$schemaFile</input>
        <!-- Following inputs are optional. -->
        <input name="showSchema">false()</input>
        <input name="sortBySeverity">true()</input>
    </verify>