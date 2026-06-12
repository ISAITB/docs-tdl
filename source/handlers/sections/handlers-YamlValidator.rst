Used to validate a YAML document (with support also for JSON) against JSON Schema(s).

.. csv-table::
    :header: "Input name", "Required?", "Type", "Description"
    :delim: ~

    ``yaml`` ~ Yes ~ ``string`` ~ The YAML content to validate.
    ``schema`` ~ No ~ ``string`` or ``list[string]`` ~ One or more JSON schemas to use for the validation. If no schema is provided the validation will by default succeed.
    ``schemaCombinationApproach`` ~ No ~ ``string`` ~ The combined validation approach when multiple schemas are used. This can be ``allOf`` (the default), ``anyOf``, or ``oneOf``.
    ``showSchema`` ~ No ~ ``boolean`` ~ Whether or not the schema(s) used will be displayed in the resulting step report. By default this is "true".
    ``supportJson`` ~ No ~ ``boolean`` ~ Whether or not the provided content for the ``yaml`` input can also be JSON. By default this is "false".

Regarding inputs, if you need to supply a single schema file you don't need to create a ``list`` and pass it as such. You can
simply pass the file as-is and the test engine will automatically convert it to a single-element ``list``.

The following examples illustrate how the ``YamlValidator`` can be used in various scenarios:

.. code-block:: xml

    <!--
        Validate YAML content against a single schema.
    -->
    <verify desc="Validate content" handler="YamlValidator">
        <input name="yaml">$yamlContent</input>
        <input name="schema">$schema</input>
    </verify>
    <!--
        Validate YAML content against multiple schemas.
    -->
    <assign to="schemas" append="true">$jsonSchema1</assign>
    <assign to="schemas" append="true">$jsonSchema2</assign>
    <verify desc="Validate content" handler="YamlValidator">
        <input name="yaml">$yamlContent</input>
        <input name="schema">$schemas</input>
        <!-- This could have been skipped as "allOf" is the default. -->
        <input name="schemaCombinationApproach">"allOf"</input>
    </verify>
    <!--
        Validate content that can be either YAML or JSON against a single schema.
        In addition, don't show the schema that was used for the validation.
    -->
    <verify desc="Validate content" handler="YamlValidator">
        <input name="yaml">$yamlContent</input>
        <input name="schema">$schema</input>
        <input name="supportYaml">true()</input>
        <input name="showSchema">false()</input>
    </verify>

When validation needs are complex it can be useful to **split schemas into separate ones** and reuse them as needed. In case such schemas
are published online you can reference through their published URI. In case schemas are not published,
the `JSON Schema specification <https://json-schema.org/draft/2020-12/json-schema-core>`__ leaves the handling of local schema references
up to each implementation, which in the case of the Test Bed is addressed as described below.

The first step is to define your schemas as test suite resources. As an example consider a ``PurchaseOrder.schema.json`` that reuses
an ``Address.schema.json`` schema, which could be defined as follows in the test suite:

.. code-block:: none

    testSuite
    ├── schemas
    │   ├── common
    │   │   └── Address.schema.json
    │   └── PurchaseOrder.schema.json
    ├── tests
    │   └── testCase1.xml
    └── testSuite.xml

The common schema defines (as all schemas do) its ``$id`` element as a unique identifier. For example, ``Address.schema.json`` could define this
as follows:

.. code-block:: none

    {
        "$id": "http://itb.ec.europa.eu/sample/Address.schema.json",
        ...
    }

This identifier allows its reference from ``PurchaseOrder.schema.json``, which does so as follows:

.. code-block:: none

    {
        "$id": "http://itb.ec.europa.eu/sample/PurchaseOrder.schema.json",
        ...
        "shipTo": { "$ref": "Address.schema.json" },
        ...
    }

The reference is achieved through the ``$ref`` property which can either be set with an absolute URI (``http://itb.ec.europa.eu/sample/Address.schema.json``),
or a relative one (``Address.schema.json``) as seen above. In the latter case, the full URI will be determined considering the current schema's
``$id`` property. Note that all that matters are the schema ``$id`` properties, not their specific folder placement. In other words, the fact
that ``Address.schema.json`` is under a ``common`` folder does not need to be reflected in the ``$id`` or ``$ref`` properties.

On the side of the test case, you would then need to import the main ``PurchaseOrder.schema.json`` schema but also tell the test engine where
to look for any dependent schemas. To achieve this, you provide the schema to the ``JsonValidator`` not as a ``string`` but as a ``map``
that contains two keys:

* ``schema``, a ``string`` element with the main schema's content.
* ``sharedSchemaPaths``, a ``list`` of ``string`` elements with the **paths**, not the actual contents, of the referred-to schemas.

You may wonder why there is a need for the ``sharedSchemaPaths`` information, as opposed to simply loading schemas from a location
relative to the current one. The reason is that schema URIs (specified via the ``$id`` property) do not represent necessarily
the physical location of schemas. Making a comparison with XML Schema where namespaces and schema locations are distinct, there
is no such distinction in JSON Schema, at least when working with local schemas.

.. note::

    When :ref:`importing schemas from other test suites <test-suite-sharing>`, the ``sharedSchemaPaths`` are resolved with respect to the target test suite.

The following examples cover various scenarios of validating against schemas with dependencies:

.. code-block:: xml

    <testcase>
        ...
        <imports>
            <!--
                Import the main schema to use.
            -->
            <artifact name="jsonSchema">schemas/PurchaseOrder.schema.json</artifact>
            ...
        </imports>
        ...
        <steps>
            <!--
                Example with a schema having a single dependent schema.
                Define the schema as a map and under the "schema" key set the schema content
                (imported from the test suite).
            -->
            <assign to="schema{schema}">$jsonSchema</assign>
            <!--
                Under "sharedSchemaPaths" define the path from which dependent schemas should be read from.
            -->
            <assign to="schema{sharedSchemaPaths}" append="true">"schemas/common/Address.schema.json"</assign>
            <verify desc="Validate YAML" handler="YamlValidator">
                <input name="yaml">$yamlContent</input>
                <input name="schema">$schema</input>
            </verify>
            <!--
                Example with a schema having a multiple dependent schemas.
            -->
            <assign to="schemaWithDependencies{schema}">$jsonSchema</assign>
            <assign to="schemaWithDependencies{sharedSchemaPaths}" append="true">"schemas/common/Address.schema.json"</assign>
            <assign to="schemaWithDependencies{sharedSchemaPaths}" append="true">"schemas/common/OrderItem.schema.json"</assign>
            <verify desc="Validate YAML" handler="YamlValidator">
                <input name="yaml">$yamlContent</input>
                <input name="schema">$schemaWithDependencies</input>
            </verify>
            <!--
                Example with multiple schemas, each with dependent schemas.
            -->
            <assign to="example1Schema1{schema}">$schema1</assign>
            <assign to="example1Schema1{sharedSchemaPaths}" append="true">"schemas/common/Address.schema.json"</assign>
            <assign to="example1Schema2{schema}">$schema2</assign>
            <assign to="example1Schema2{sharedSchemaPaths}" append="true">"schemas/common/OrderItem.schema.json"</assign>
            <assign to="example1Schemas" append="true">$example1Schema1</assign>
            <assign to="example1Schemas" append="true">$example1Schema2</assign>
            <verify desc="Validate YAML" handler="YamlValidator">
                <input name="yaml">$yamlContent</input>
                <input name="schema">$example1Schemas</input>
                <input name="schemaCombinationApproach">"allOf"</input>
            </verify>
            <!--
                Example with multiple schemas, only one of which has dependencies.
                For schemas without dependencies you don't need to define them as a map.
            -->
            <assign to="example2Schema1{schema}">$schema1</assign>
            <assign to="example2Schema1{sharedSchemaPaths}" append="true">"schemas/common/Address.schema.json"</assign>
            <assign to="example2Schemas" append="true">$example2Schema1</assign>
            <!-- Add the second schema directly. -->
            <assign to="example2Schemas" append="true">$schema2</assign>
            <verify desc="Validate YAML" handler="YamlValidator">
                <input name="yaml">$yamlContent</input>
                <input name="schema">$schemasToUse</input>
                <input name="schemaCombinationApproach">"allOf"</input>
            </verify>
            ...
        </steps>
    </testcase>