Used to extract values or complete elements from JSON content based on a provided `JSON Pointer expression <https://datatracker.ietf.org/doc/html/rfc6901>`_.
The following operation is supported:

.. csv-table::
    :header: "Operation", "Description", "Input(s)", "Output(s)"
    :delim: |

    ``process`` | Use a JSON Pointer expression to extract a value or full elements from the provided JSON content. | Yes | A ``string`` named ``output`` in the resulting step's ``map``.

The input parameters expected by the ``process`` operation are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``asYaml`` | No | A ``boolean`` flag on whether to treat the input as YAML (and produce YAML output).
    ``content`` | Yes | A ``string`` representing the JSON content to process.
    ``pointer`` | Yes | A ``string`` representing the JSON Pointer expression to use.

The following example illustrates how the ``JsonPointerProcessor`` can be used to extract a value from JSON content provided by the user via
an :ref:`interact<tdl-step-interact>` step:

.. code-block:: xml

    <!--
        Have the user enter the JSON content to process.
    -->
    <interact id="data" desc="Receive input">
        <request name="json" desc="Enter JSON content:" inputType="CODE" mimeType="application/json"/>
    </interact>
    <!--
        Retrieve a value from the provided JSON content. The content is expected to structured as follows:
        {
            "user": {
                "address": {
                    "streetName": "An address"
                }
            }
        }
    -->
    <process handler="JsonPointerProcessor" operation="process" output="result">
        <input name="content">$data{json}</input>
        <input name="pointer">"/user/address/streetName"</input>
    </process>
    <!--
        Log the result.
    -->
    <log>$result</log>