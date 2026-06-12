Used to extract information from XML content using an XPath expression.

.. csv-table::
    :header: "Operation", "Description", "Input(s)", "Output(s)"
    :delim: |

    ``process`` | Process XML content with an XPath expression return the transformed result. | Yes | Yes, a ``string`` named ``output`` in the resulting step's ``map``.

The input parameters expected by the ``process`` operation are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``expression`` | Yes | The XPath expression to use.
    ``input`` | Yes | The XML content to process.
    ``type`` | No | The :ref:`GITB type <test-case-types>` for the expression's output (default is ``string`` - for XML output use ``object``).

The ``XPathProcessor`` leverages **XML namespaces** as defined by the test case's :ref:`namespaces element <test-case-namespaces>`, allowing
you to construct namespace-aware expressions. The following examples illustrate several data extraction scenarios from a provided XML document.

.. code-block:: xml

    <!--
        Define the namespace(s) used at the level of the test case (or scriptlet).
    -->
    <namespaces>
        <ns prefix="po">http://itb.ec.europa.eu/sample/po</ns>
    </namespaces>
    <steps>
        ...
        <!--
            Extract a text value.
        -->
        <process handler="XPathProcessor" output="result1">
            <input name="input">$xmlContent</input>
            <input name="expression">"/po:purchaseOrder/po:shipTo/po:name/text()"</input>
        </process>
        <log>"Extracted string: " || $result1</log>
        <!--
            Extract a value and force it to be considered as a number.
        -->
        <process handler="XPathProcessor" output="result2">
            <input name="input">$xmlContent</input>
            <input name="expression">"/po:purchaseOrder/po:amount/text()"</input>
            <input name="type">"number"</input>
        </process>
        <log>"Extracted number: " || $result2</log>
        <!--
            Extract an XML block for further processing.
        -->
        <process handler="XPathProcessor" output="result3">
            <input name="input">$xmlContent</input>
            <input name="expression">"/po:purchaseOrder/po:shipTo"</input>
            <input name="type">"object"</input>
        </process>
        <log>"Extracted XML block: " || $result3</log>
        <!--
            Extract a text value (alternative using namespace wildcards).
        -->
        <process handler="XPathProcessor" output="result4">
            <input name="input">$xmlContent</input>
            <input name="expression">"/*[local-name() = 'purchaseOrder']/*[local-name() = 'shipTo']/po:name/text()"</input>
        </process>
        <log>"Extracted string: " || $result4</log>
        ...
    </steps>

Using namespaces as illustrated in the above examples is not mandatory. You could also use expressions with **namespace wildcards**,
in which case however you would most probably need to check elements' local names. Using specific namespaces should be preferred
when possible, as they result in more simple and understandable expressions, but also ensure you are matching precisely what you
expect. The following examples illustrates a lookup using namespace wildcards:

.. code-block:: xml

    <!--
        Use wildcards in place of specific namespaces. Besides being more complex, the expression could have ambiguous
        results in case several namespaces in the XML document define similarly named elements.
    -->
    <process handler="XPathProcessor" output="result">
        <input name="input">$xmlContent</input>
        <input name="expression">"/*[local-name() = 'purchaseOrder']/*[local-name() = 'shipTo']/po:name/text()"</input>
    </process>
    <log>"Extracted string: " || $result</log>

You may have noticed in all examples above that the ``expression`` input is provided as a string and needs to be quoted. This contrasts
:ref:`other steps accepting expressions <test-case-expressions-where>` where XPath expressions can be provided directly. The reason for
this is because you may use the ``XPathProcessor`` also with expressions that are generated, for example by concatenating several strings.
Support for such generated XPath expressions is also what differentiates this handler from using a simple :ref:`assign step <tdl-step-assign>`.
The following examples illustrates how this handler is often equivalent to an :ref:`assign step <tdl-step-assign>`, as well as how it differs
when you need to generate the expression to use:

.. code-block:: xml

    <assign to="country">"BE"</assign>
    <!--
        Using the XPath processor to look up a value, with a variable used in a condition.
    -->
    <process handler="XPathProcessor" output="result1">
        <input name="input">$xmlContent</input>
        <input name="expression">"/po:purchaseOrder/po:shipTo[@country = $country]/po:name"</input>
    </process>
    <!--
        Using an assign step that gives the same result as the previous step.
    -->
    <assign to="result2" source="$xmlContent">/po:purchaseOrder/po:shipTo[@country = $country]/po:name</assign>
    <!--
        Using the XPath processor but generating the expression beforehand.
    -->
    <assign to="countryAttributeName">"country"</assign>
    <assign to="expressionToUse">"/po:purchaseOrder/po:shipTo[@" || $countryAttributeName || " = $country]/po:name"</assign>
    <process handler="XPathProcessor" output="result3">
        <input name="input">$xmlContent</input>
        <input name="expression">$expressionToUse</input>
    </process>

    <log>"Results are the same: [" || $result1 || "] [" || $result2 || "] [" || $result3 || "]"</log>

.. note::

    **Using assign for simple extractions:** Using an :ref:`assign step <tdl-step-assign>` is the simplest way to extract content from XML.
    Provide the XML content as the ``source``, the output value as the ``to``, and if needed the specific target type as the ``type``, with
    the XPath expression as the step's value:

    ``<assign to="result" source="$xmlContent">//po:shipTo[@country = $country]/po:name</assign>``

    The ``XPathProcessor`` should be preferred to create expressions dynamically or to reuse expressions across lookups.