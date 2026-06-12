.. note::
    The built-in :ref:`ShaclValidator <handlers-ShaclValidator>` allows validating RDF data without needing an external service.

The RDF validation service allows you to validate RDF content via `SHACL shapes <https://www.w3.org/TR/shacl/>`_
definitions. It is the default, generic configuration of the Test Bed's `RDF validator component <https://hub.docker.com/r/isaitb/shacl-validator>`_ that
expects the shapes to apply as inputs alongside the content to validate.

.. note::

    The generic RDF validator is also available for standalone use via `user interface <https://www.itb.ec.europa.eu/shacl/any/upload>`__,
    `REST API <https://www.itb.ec.europa.eu/shacl/swagger-ui/index.html>`__ and `SOAP API <https://www.itb.ec.europa.eu/shacl/soap/any/validation?wsdl>`__.
    Furthermore, a custom validator with a predefinined configuration and specific settings can be defined following the Test Bed's
    `RDF validation guide <https://www.itb.ec.europa.eu/docs/guides/latest/validatingRDF/index.html>`_. The API of such a custom instance is identical to
    the generic instance presented here.

You can use the RDF validator by one of two approaches:

* **Locally**, by pulling the `isaitb/shacl-validator <https://hub.docker.com/r/isaitb/shacl-validator>`_ Docker image.
* **As a service**, by setting the handler to ``https://www.itb.ec.europa.eu/shacl/soap/any/validation?wsdl``.

The validator supports several inputs to customise the validation to take place. The available inputs are listed in the service's
`SOAP API documentation <https://www.itb.ec.europa.eu/docs/guides/latest/validatingRDF/index.html#validation-via-soap-web-service-api>`__,
where all listed inputs match exactly those that can be used in test cases through :ref:`verify<tdl-step-verify>` steps.

The following test case sample illustrates how to use the validator for the most common use case of validating RDF content against a shape graph:

.. code-block:: xml

    <steps>
        <!--
            You can validate against any number of shape graph files in one go. In this case we use one file (defined in $shapes)
            that is typically provided as an import but could also be loaded from configuration or even generated on the
            fly in a previous test case step.
         -->
        <assign to="shapes1{ruleSet}">$shapes</assign>
        <assign to="shapes1{ruleSyntax}">"application/turtle"</assign>
        <!-- Set embeddingMethod to "STRING" if the content is defined as a string ("BASE64" corresponds to binary). -->
        <assign to="shapes1{embeddingMethod}">"BASE64"</assign>
        <assign to="shapesToUse" append="true">$shapes1</assign>
        <!--
            Call the validator.
        -->
        <verify handler="https://www.itb.ec.europa.eu/shacl/soap/any/validation?wsdl" desc="Validate RDF file">
            <input name="contentToValidate">$fileToValidate</input>
            <input name="contentSyntax">"application/rdf+xml"</input>
            <input name="externalRules">$shapesToUse</input>
            <!-- Set embeddingMethod to "STRING" if the contentToValidate is defined as a string ("BASE64" corresponds to binary). -->
            <input name="embeddingMethod">"BASE64"</input>
        </verify>
    </steps>