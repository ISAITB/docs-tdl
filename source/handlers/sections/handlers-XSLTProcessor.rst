Used to transform XML content using an XSLT style sheet, both being provided as inputs, and output the result.

.. csv-table::
    :header: "Operation", "Description", "Input(s)", "Output(s)"
    :delim: |

    ``process`` | Process XML content using an XSLT style sheet and return the transformed result. | Yes | Yes, a ``string`` named ``output`` in the resulting step's ``map``.

The input parameters expected by the ``process`` operation are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``xml`` | Yes | The XML content to transform.
    ``xslt`` | Yes | The XSLT style sheet to use for the transformation.

The following example illustrates usage of the ``XsltProcessor`` to transform the provided "xmlContent" (the XML input) using the "xsltContent" (the XSLT style sheet).
These variables may be provided in any manner, for example the "xmlContent" could be uploaded via a :ref:`user interaction step<tdl-step-receive>` whereas the "xsltContent" could
be :ref:`imported<test-case-imports>` from the test suite's resources.

.. code-block:: xml

    <process output="result" handler="XsltProcessor">
        <input name="xml">$xmlContent</input>
        <input name="xslt">$xsltContent</input>
    </process>
    <log>$result</log>