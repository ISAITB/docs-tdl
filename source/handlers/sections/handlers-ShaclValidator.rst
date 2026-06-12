Used to validate an RDF model against SHACL shape files.

.. csv-table::
    :header: "Input name", "Required?", "Type", "Description"
    :delim: ~

    ``contentType`` ~ Yes ~ ``string`` ~ The content type of the RDF model provided for validation.
    ``loadImports`` ~ No ~ ``boolean`` ~ Whether or not ``owl:import`` statements found in the input model will be evaluated and included for the validation. By default this is "false".
    ``model`` ~ Yes ~ ``string`` ~ The RDF model to validate.
    ``reportContentType`` ~ No ~ ``string`` ~ The content type to use for the `SHACL validation report <https://www.w3.org/TR/shacl/#validation-report>`__ in case ``showReport`` is set to "true". By default the input model's ``contentType`` will be used.
    ``shapes`` ~ No ~ ``map`` or ``list[map]`` ~ One or more files with SHACL shapes to use for the validation (provided along with their respective content types). If no shape files are provided the validation will by default succeed.
    ``showReport`` ~ No ~ ``boolean`` ~ Whether or not the step's report will display the resulting `SHACL validation report <https://www.w3.org/TR/shacl/#validation-report>`__. By default this is "false".
    ``showShapes`` ~ No ~ ``boolean`` ~ Whether or not the step's report will display the shapes used for the validation (using the first shape file's content type in case of multiple). By default this is "true".
    ``sortBySeverity`` ~ No ~ ``boolean`` ~ Whether or not the resulting report items will be sorted by severity. By default this is "false".

Regarding providing the shape file to consider for the ``shapes`` input, this is provided as a ``map`` containing two keys:

* ``content``, for the shape file content.
* ``contentType``, for the content type to consider when reading the content.

In case multiple shape files are needed, you provide the ``shapes`` input as a ``list`` where each element is a ``map`` corresponding
to a shape file as described above.

The following examples illustrate how the ``ShaclValidator`` can be used in various scenarios:

.. code-block:: xml

    <!--
        Validate an RDF model with single shape file.
    -->
    <assign to="shapeFile{content}">$myShapes1</assign>
    <assign to="shapeFile{contentType}">"text/turtle"</assign>
    <verify desc="Validate model" handler="ShaclValidator">
        <input name="model">$ttlSample</input>
        <input name="shapes">$shapeFile</input>
        <input name="contentType">"text/turtle"</input>
    </verify>
    <!--
        Validate an RDF model with multiple shape files.
    -->
    <assign to="shape1{content}">$myShapes1</assign>
    <assign to="shape1{contentType}">"text/turtle"</assign>
    <assign to="shape2{content}">$myShapes2</assign>
    <assign to="shape2{contentType}">"text/turtle"</assign>
    <assign to="shapes" append="true">$shape1</assign>
    <assign to="shapes" append="true">$shape2</assign>
    <verify desc="Validate model" handler="ShaclValidator">
        <input name="model">$ttlSample</input>
        <input name="contentType">"text/turtle"</input>
        <input name="shapes">$shapes</input>
    </verify>
    <!--
        Validate an RDF model and for the step's report:
        - Hide the shapes used for the validation.
        - Include the SHACL validation report in RDF/XML format.
        - Sort reported items by severity.
    -->
    <assign to="shapeFileExample{content}">$myShapes1</assign>
    <assign to="shapeFileExample{contentType}">"text/turtle"</assign>
    <verify desc="Validate model" handler="ShaclValidator">
        <input name="model">$ttlSample</input>
        <input name="shapes">$shapeFileExample</input>
        <input name="contentType">"text/turtle"</input>
        <input name="showShapes">false()</input>
        <input name="showReport">false()</input>
        <input name="reportContentType">"application/rdf+xml"</input>
        <input name="sortBySeverity">true()</input>
    </verify>