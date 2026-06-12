The ``YamlConverter`` processor is used to convert JSON content to YAML and vice-versa. It supports the following operations:

.. csv-table::
    :header: "Operation", "Description", "Input(s)", "Output(s)"
    :delim: |

    ``jsonToYaml`` | Convert JSON content to YAML. | Yes | Yes, the resulting YAML content.
    ``yamlToJson`` | Convert YAML content to JSON. | Yes | Yes, the resulting JSON content.

.. _handlers-YamlConverter_jsonToYaml:

YamlConverter - jsonToYaml
^^^^^^^^^^^^^^^^^^^^^^^^^^

The ``jsonToYaml`` operation is used to convert JSON content to YAML. The inputs it expects are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``content`` | Yes | The JSON content to convert.

The following example illustrates the operation's use:

.. code-block:: xml

    <!-- Convert JSON content to YAML. -->
    <process handler="YamlConverter" output="yamlResult" operation="jsonToYaml">
       <input name="content">$jsonSample</input>
    </process>
    <log>$yamlResult</log>

.. _handlers-YamlConverter_yamlToJson:

YamlConverter - yamlToJson
^^^^^^^^^^^^^^^^^^^^^^^^^^

The ``yamlToJson`` operation is used to convert YAML content to JSON. The inputs it expects are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``content`` | Yes | The YAML content to convert.

The following example illustrates the operation's use:

.. code-block:: xml

    <!-- Convert YAML content to JSON. -->
    <process handler="YamlConverter" output="jsonResult" operation="yamlToJson">
       <input name="content">$yamlSample</input>
    </process>
    <log>$jsonResult</log>