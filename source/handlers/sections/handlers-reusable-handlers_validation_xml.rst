.. note::
    Built-in validators for XML are also available, notably the :ref:`handlers-XmlValidator` for validation against XML Schema and Schematron, as
    well as the :ref:`handlers-XmlMatchValidator` for validation against expected templates.

The XML validation service allows you to validate XML content by means of one or more `XML Schemas <https://www.w3.org/standards/xml/schema>`_
and `Schematron <https://schematron.com/>`_. It is the default, generic configuration of the Test Bed's
`XML validator component <https://hub.docker.com/r/isaitb/xml-validator>`_ that expects the schemas and Schematrons to apply as inputs alongside
the content to validate.

.. note::

    The generic XML validator is also available for standalone use via `user interface <https://www.itb.ec.europa.eu/xml/upload>`__,
    `REST API <https://www.itb.ec.europa.eu/xml/swagger-ui/index.html>`__ and `SOAP API <https://www.itb.ec.europa.eu/xml/api/validation?wsdl>`__.
    Furthermore, a custom validator with a predefinined configuration and specific settings can be defined following the Test Bed's
    `XML validation guide <https://www.itb.ec.europa.eu/docs/guides/latest/validatingXML/index.html>`_. The API of such a custom instance is identical to
    the generic instance presented here.

You can use the XML validator by one of two approaches:

* **Locally**, by pulling the `isaitb/xml-validator <https://hub.docker.com/r/isaitb/xml-validator>`_ Docker image.
* **As a service**, by setting the handler to ``https://www.itb.ec.europa.eu/xml/api/validation?wsdl``.

The validator supports several inputs to customise the validation to take place. The available inputs are listed in the service's
`SOAP API documentation <https://www.itb.ec.europa.eu/docs/guides/latest/validatingXML/index.html#validation-via-soap-web-service-api>`__,
where all listed inputs match exactly those that can be used in test cases through :ref:`verify<tdl-step-verify>` steps.

The following test case sample illustrates how to use the validator for the most common use case of validating XML content against an XML Schema and a
Schematron rule file:

.. code-block:: xml

    <steps>
        <!--
            You can validate against any number of schemas in one go. In this case we use one schema (defined in $schema)
            that is typically provided as an import but could also be loaded from configuration or even generated on the
            fly in a previous test case step.
         -->
        <assign to="schema1{content}">$schema</assign>
        <!-- Set embeddingMethod to "STRING" if the content is defined as a string ("BASE64" corresponds to binary). -->
        <assign to="schema1{embeddingMethod}">"BASE64"</assign>
        <assign to="schemasToUse" append="true">$schema1</assign>
        <!--
            Prepare also the Schematron rules to use (defined in $schematron) that could similarly be imported, loaded from
            configuration or generated on the fly.
        -->
        <assign to="schematron1{content}">$schematron</assign>
        <!--
            The Schematron type could be "xsl" (for Schematron transformed to XSLT), or "sch" for the raw Schematron format. Besides
            experimentation or very simple cases, XSLT rules (so a "xsl" type value) should always be preferred.
        -->
        <assign to="schematron1{type}">"xsl"</assign>
        <!-- Set embeddingMethod to "STRING" if the content is defined as a string ("BASE64" corresponds to binary). -->
        <assign to="schematron1{embeddingMethod}">"BASE64"</assign>
        <assign to="schematronsToUse" append="true">$schematron1</assign>
        <!--
            Call the validator.
        -->
        <verify handler="https://www.itb.ec.europa.eu/xml/api/validation?wsdl" desc="Validate XML file">
            <input name="xml">$fileToValidate</input>
            <input name="externalSchema">$schemasToUse</input>
            <input name="externalSchematron">$schematronsToUse</input>
            <!-- Set embeddingMethod to "STRING" if the contentToValidate is defined as a string ("BASE64" corresponds to binary). -->
            <input name="embeddingMethod">"BASE64"</input>
        </verify>
    </steps>

.. note::
    When using the generic XML validator you don't need to always validate using XML Schema and Schematron. For example you could skip schema
    validation and only validate against a set of generated Schematron rules.