.. note::
    The built-in :ref:`JsonValidator <handlers-JsonValidator>` allows the validation of JSON without needing an external service.

The JSON validation service allows you to validate JSON content by means of one or more `JSON Schema <https://json-schema.org/>`_
definitions. It is the default, generic configuration of the Test Bed's `JSON validator component <https://hub.docker.com/r/isaitb/json-validator>`_ that
expects the schemas to apply as inputs alongside the content to validate.

.. note::

    The generic JSON validator is also available for standalone use via `user interface <https://www.itb.ec.europa.eu/json/any/upload>`__,
    `REST API <https://www.itb.ec.europa.eu/json/swagger-ui/index.html>`__ and `SOAP API <https://www.itb.ec.europa.eu/json/soap/any/validation?wsdl>`__.
    Furthermore, a custom validator with a predefinined configuration and specific settings can be defined following the Test Bed's
    `JSON validation guide <https://www.itb.ec.europa.eu/docs/guides/latest/validatingJSON/index.html>`_. The API of such a custom instance is identical to
    the generic instance presented here.

You can use the JSON validator by one of two approaches:

* **Locally**, by pulling the `isaitb/json-validator <https://hub.docker.com/r/isaitb/json-validator>`_ Docker image.
* **As a service**, by setting the handler to ``https://www.itb.ec.europa.eu/json/soap/any/validation?wsdl``.

The validator supports several inputs to customise the validation to take place. The available inputs are listed in the service's
`SOAP API documentation <https://www.itb.ec.europa.eu/docs/guides/latest/validatingJSON/index.html#validation-via-soap-web-service-api>`__,
where all listed inputs match exactly those that can be used in test cases through :ref:`verify<tdl-step-verify>` steps.

The following test case sample illustrates how to use the validator for the most common use case of validating JSON content against a schema:

.. code-block:: xml

    <steps>
        <!--
            You can validate against any number of schemas in one go. In this case we use one schema (defined in $schema)
            that is typically provided as an import but could also be loaded from configuration or even generated on the
            fly in a previous test case step.
         -->
        <assign to="schema1{schema}">$schema</assign>
        <!-- Set embeddingMethod to "STRING" if the content is defined as a string ("BASE64" corresponds to binary). -->
        <assign to="schema1{embeddingMethod}">"BASE64"</assign>
        <assign to="schemasToUse" append="true">$schema1</assign>
        <!--
            Call the validator.
        -->
        <verify handler="https://www.itb.ec.europa.eu/json/soap/any/validation?wsdl" desc="Validate JSON file">
            <input name="contentToValidate">$fileToValidate</input>
            <input name="externalSchemas">$schemasToUse</input>
            <!-- Set embeddingMethod to "STRING" if the contentToValidate is defined as a string ("BASE64" corresponds to binary). -->
            <input name="embeddingMethod">"BASE64"</input>
        </verify>
    </steps>