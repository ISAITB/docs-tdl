The ``RdfUtils`` processor is used to perform different types of manipulations on RDF models. This processing handler
does not require a processing transaction to be established. The following operations are supported:

.. csv-table::
    :header: "Operation", "Description", "Input(s)", "Output(s)"
    :delim: |

    ``ask`` | Run a SPARQL ASK query on an RDF model. | Yes | Yes, a ``boolean`` value with the ask query's result.
    ``construct`` | Run a SPARQL CONSTRUCT query on an RDF model to produce a new RDF graph. | Yes | Yes, a ``string`` representation of the resulting RDF graph.
    ``convert`` | Convert a provided RDF model from its current RDF syntax to another. | Yes | Yes, a ``string`` representation of the resulting RDF model.
    ``merge`` | Merge multiple RDF models to a single aggregate one. | Yes | Yes, a ``string`` representation of the resulting RDF model.
    ``select`` | Run a SPARQL SELECT query on an RDF model to produce an RDF result set. | Yes | Yes, a ``string`` representation of the resulting RDF result set.

Manipulating RDF models using this processor's operations grants you the flexibility needed to handle any kind of RDF processing in test cases.

.. _handlers-RdfUtils_ask:

RdfUtils - ask
^^^^^^^^^^^^^^

The ``ask`` operation enables you to run `SPARQL ASK queries <https://www.w3.org/TR/rdf-sparql-query/#ask>`__ against a provided RDF model, to check whether
they produce a match (returned as a ``boolean`` result). The input parameters supported by this operation are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``inputContentType`` | Yes | The ``string`` content type of the provided model.
    ``model`` | Yes | The ``string`` RDF model to run the query against.
    ``query`` | Yes | The ``string`` SPARQL query to execute.

The following example illustrates the ``ask`` operation's use:

.. code-block:: xml

    <!--
        Define the ASK query to use.
    -->
    <assign to="query"><![CDATA["PREFIX ns1: <http://itb.ec.europa.eu/sample/po#>

    ASK WHERE {
      ?item a ns1:Item ;
              ns1:quantity ?q .
    FILTER(?q > 1000)
    }"]]></assign>
    <!--
        Run the query.
    -->
    <process output="askResult" handler="RdfUtils" operation="ask">
        <input name="model">$ttlSample</input>
        <input name="inputContentType">"text/turtle"</input>
        <input name="query">$query</input>
    </process>
    <log>$askResult</log>


.. _handlers-RdfUtils_construct:

RdfUtils - construct
^^^^^^^^^^^^^^^^^^^^

The ``construct`` operation enables you to run `SPARQL CONSTRUCT queries <https://www.w3.org/TR/rdf-sparql-query/#construct>`__ against a provided RDF model,
to generate from it a new graph based on the resulting triples (returned as a ``string`` result). The input parameters supported by this operation are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``inputContentType`` | Yes | The ``string`` content type of the provided model.
    ``model`` | Yes | The ``string`` RDF model to run the query against.
    ``outputContentType`` | No | The ``string`` content type to use for the produced graph. If not provided, the content type of the input model will be used.
    ``query`` | Yes | The ``string`` SPARQL query to execute.

The following example illustrates the ``construct`` operation's use:

.. code-block:: xml

    <!--
        Define the CONSTRUCT query.
    -->
    <assign to="query"><![CDATA["PREFIX ns1: <http://itb.ec.europa.eu/sample/po#>
    PREFIX : <http://my.sample.po/summary#>

    CONSTRUCT {
        ?item :summaryOf ?productName ;
        :totalCost ?cost .
    }
    WHERE {
        ?item a ns1:Item ;
        ns1:productName ?productName ;
        ns1:quantity ?q ;
        ns1:priceEUR ?p .
        BIND(?q * ?p AS ?cost)
    }"]]></assign>
    <!--
        Run the query.
    -->
    <process output="constructResult" handler="RdfUtils" operation="construct">
        <input name="model">$ttlSample</input>
        <input name="inputContentType">"text/turtle"</input>
        <!--
            Optional. If not provided defaults to the input model's content type.
        -->
        <input name="outputContentType">"application/rdf+xml"</input>
        <input name="query">$query</input>
    </process>
    <log>$constructResult</log>

.. _handlers-RdfUtils_convert:

RdfUtils - convert
^^^^^^^^^^^^^^^^^^

The ``convert`` operation enables you to convert an RDF model to another RDF syntax. The input parameters supported by this operation are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``inputContentType`` | Yes | The ``string`` content type of the provided model.
    ``model`` | Yes | The ``string`` RDF model to convert.
    ``outputContentType`` | Yes | The ``string`` content type to use for the output model.

The following example illustrates the ``convert`` operation's use:

.. code-block:: xml

    <!--
        Convert the provided Turtle model to RDF/XML.
    -->
    <process output="convertResult" handler="RdfUtils" operation="convert">
        <input name="model">$ttlSample</input>
        <input name="inputContentType">"text/turtle"</input>
        <input name="outputContentType">"application/rdf+xml"</input>
    </process>
    <!--
        Output the converted model.
    -->
    <log>$convertResult</log>

.. _handlers-RdfUtils_merge:

RdfUtils - merge
^^^^^^^^^^^^^^^^

The ``merge`` operation enables you to merge multiple RDF models to a new one, aggregating their combined triples into as a new graph.
The input parameters supported by this operation are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``inputContentType`` | No | The ``string`` content type for all provided models. Either this or the ``inputContentTypes`` must be provided.
    ``inputContentTypes`` | No | A ``list`` of ``string`` content types for the provided models. Either this or the ``inputContentType`` must be provided.
    ``models`` | Yes | The ``string`` RDF model to convert.
    ``outputContentType`` | No | The ``string`` content type to use for the output model. If not provided the first provided input content type will be used.

The following example illustrates the ``merge`` operation's use:

.. code-block:: xml

    <!--
        Merge models with different RDF syntaxes and return the resulting model in RDF/XML.
    -->
    <assign to="models" append="true">$ttlSample</assign>
    <assign to="models" append="true">$rdfXmlSample</assign>
    <assign to="inputContentTypes" append="true">"text/turtle"</assign>
    <assign to="inputContentTypes" append="true">"application/rdf+xml"</assign>
    <process output="mergeResult" handler="RdfUtils" operation="merge">
        <input name="models">$models</input>
        <input name="inputContentTypes">$inputContentTypes</input>
        <!--
            Optional. If not provided the output type will match the one of the first input model.
        -->
        <input name="outputContentType">"application/rdf+xml"</input> Optional
    </process>
    <log>$mergeResult</log>

    <!--
        Merge models that are all in Turtle format and return the resulting model in Turtle.
        We don't specify here the outputContentType as it defaults to the input type.
    -->
    <assign to="ttlModels" append="true">$ttlSample1</assign>
    <assign to="ttlModels" append="true">$ttlSample2</assign>
    <process output="mergeResult2" handler="RdfUtils" operation="merge">
        <input name="models">$models</input>
        <input name="inputContentType">"text/turtle"</input>
    </process>
    <log>$mergeResult2</log>

.. _handlers-RdfUtils_select:

RdfUtils - select
^^^^^^^^^^^^^^^^^

The ``select`` operation enables you to run `SPARQL SELECT queries <https://www.w3.org/TR/rdf-sparql-query/#select>`__ against a provided RDF model,
resulting in an RDF result set (returned as a ``string`` result). The input parameters supported by this operation are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``inputContentType`` | Yes | The ``string`` content type of the provided model.
    ``model`` | Yes | The ``string`` RDF model to run the query against.
    ``outputContentType`` | No | The ``string`` content type to use for the produced result set. If not provided, ``application/sparql-results+xml`` will be used as a default.
    ``query`` | Yes | The ``string`` SPARQL query to execute.

The following example illustrates the ``select`` operation's use:

.. code-block:: xml

    <!--
        Define the SELECT query.
    -->
    <assign to="query"><![CDATA["PREFIX ns1: <http://itb.ec.europa.eu/sample/po#>

    SELECT ?item ?productName ?quantity ?priceEUR ?totalCost WHERE {
      ?item a ns1:Item ;
            ns1:productName ?productName ;
            ns1:quantity ?quantity ;
            ns1:priceEUR ?priceEUR .
      BIND(?quantity * ?priceEUR AS ?totalCost)
    }"]]></assign>
    <!--
        Run the query.
    -->
    <process output="selectResult" handler="RdfUtils" operation="select">
        <input name="model">$ttlSample</input>
        <input name="inputContentType">"text/turtle"</input>
        <!--
            Optional. If not provided defaults to application/sparql-results+xml.
        -->
        <input name="outputContentType">"text/csv"</input>
        <input name="query">$query</input>
    </process>
    <log>$selectResult</log>