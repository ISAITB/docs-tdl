Used to extract values, calculations, or complete elements from JSON content based on a provided `JSONPath expression <https://www.rfc-editor.org/rfc/rfc9535.html>`_.
This processing handler does not require a processing transaction to be established. The following operations are supported:

.. csv-table::
    :header: "Operation", "Description", "Input(s)", "Output(s)"
    :delim: |

    ``count`` | Run a JSONPath expression on a given JSON content and return the number of matching elements. | Yes | Yes, a ``number`` representing the match count.
    ``exists`` | Check whether a JSONPath expression matches any elements in a given JSON content. | Yes | Yes, a ``boolean`` representing the check result.
    ``process`` | Extract values or entire JSON blocks using a JSONPath expression on a given JSON content. | Yes | Yes, depending on the selected ``outputType``.

.. _handlers-JsonPathProcessor_count:

JsonPathProcessor - count
^^^^^^^^^^^^^^^^^^^^^^^^^

The ``count`` operation is used to check the number of matches returned when running a JSONPath expression on a given JSON content
(returned as a ``number`` result). The input parameters supported by this operation are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``asYaml`` | No | A ``boolean`` flag on whether to treat the input as YAML (and produce YAML output).
    ``content`` | Yes | The ``string`` JSON content to consider.
    ``expression`` | Yes | The ``string`` JSONPath expression to use.

The following example illustrates the ``count`` operation's use:

.. code-block:: xml

    <!--
      Return the number of books that have a high price.
    -->
    <process handler="JsonPathProcessor" operation="count" output="matchingBooks">
       <input name="content">$json</input>
       <input name="expression">"$.store.book[?(@.price > 100)]"</input>
    </process>
    <!--
      Log the count.
    -->
    <log>"Matched " || $matchingBooks || " book(s)."</log>

.. _handlers-JsonPathProcessor_exists:

JsonPathProcessor - exists
^^^^^^^^^^^^^^^^^^^^^^^^^^

The ``exists`` operation is used to check whether any matches are returned when running a JSONPath expression on a given JSON content
(returned as a ``boolean`` result). The input parameters supported by this operation are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``asYaml`` | No | A ``boolean`` flag on whether to treat the input as YAML (and produce YAML output).
    ``content`` | Yes | The ``string`` JSON content to consider.
    ``expression`` | Yes | The ``string`` JSONPath expression to use.

The following example illustrates the ``exists`` operation's use:

.. code-block:: xml

    <!--
      Check to see if any books are expensive.
    -->
    <process handler="JsonPathProcessor" operation="exists" output="hasExpensiveBook">
       <input name="content">$json</input>
       <input name="expression">"$.store.book[?(@.price > 100)]"</input>
    </process>
    <!--
      If there are expensive books skip the following validation.
    -->
    <verify desc="Validate books" handler="..." skipped="$hasExpensiveBook">
       ...
    </verify>

.. _handlers-JsonPathProcessor_process:

JsonPathProcessor - process
^^^^^^^^^^^^^^^^^^^^^^^^^^^

The ``process`` operation is used to run a JSONPath expression on a given JSON content, to retrieve matching
values or entire JSON blocks. The input parameters supported by this operation are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``asYaml`` | No | A ``boolean`` flag on whether to treat the input as YAML (and produce YAML output).
    ``content`` | Yes | The ``string`` JSON content to consider.
    ``expression`` | Yes | The ``string`` JSONPath expression to use.
    ``outputType`` | No | The type of output to return. This can be ``default`` (the default), ``list`` or ``raw``.

Regarding the ``outputType`` input, the values it supports bear the following meaning:

* ``default`` returns a list with the results. If resulting in a leaf node it will convert it to a basic type (e.g. a string or a list of string for an array). This is the default approach considered.
* ``list`` is very similar to the ``default`` approach but will force a list for the result. This means that even if your expression results in a simple string property value, it will be wrapped in a list. This allows you to consistently check if your expression produced results (and how many if needed).
* ``raw`` returns a JSON string with the raw result. This is always formatted as a JSON array containing the expression's matches.

The following example illustrates the ``process`` operation's use:

.. code-block:: xml

    <!--
      Prompt the user to provide JSON content, the expression to use, and the type of output to return.
    -->
    <interact id="data" desc="Provide inputs">
       <request desc="JSON" name="json" inputType="CODE" mimeType="application/json"/>
       <request desc="Expression" name="expression"/>
       <request desc="Output" name="outputType" inputType="SELECT_SINGLE" options="default,raw,list"/>
    </interact>
    <!--
      Run the expression on the content and return the output.
    -->
    <process handler="JsonPathProcessor" operation="process" output="result">
       <!-- The JSON input to process. -->
       <input name="content">$data{json}</input>
       <!-- The JSONPath expression to use. -->
       <input name="expression">$data{expression}</input>
       <!-- The type of output to return (default is "default"). -->
       <input name="outputType">$data{outputType}</input>
    </process>
    <!--
      Log the result.
    -->
    <log>$result</log>