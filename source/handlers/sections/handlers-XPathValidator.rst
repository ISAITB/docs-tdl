Used to evaluate an XPath 3.1 expression against a provided XML document. The result of the expression
needs to evaluate to a boolean (i.e. true for success or false for failure).

.. csv-table::
    :header: "Input name", "Required?", "Type", "Description"

    ``expression``, Yes, ``string``, The XPath 3.0 expression passed as a string.
    ``failureMessage``, No, ``string``, An optional message to display in case the check fails.
    ``successMessage``, No, ``string``, An optional message to display in case the check succeeds.
    ``xml``, Yes, ``object``, The XML document upon which the XPath expression will be evaluated.

An important note here is that the XPath expression passed in ``expression`` is meant to be a string.
This means that to run an expression as-is you need to wrap it in quotes. This is because the content of
the ``input`` element can also be an expression that you want to evaluate to give you the final expression to
use. The following example illustrates both cases:

.. code-block:: xml

    <!--
        Pass a string as the expression to use.
    -->
    <assign to="expectedValue">EXPECTED"</assign>
    <verify handler="XPathValidator" desc="Check document">
        <input name="xml">$myDocument</input>
        <!-- Define expression with variable reference -->
        <input name="expression">"contains(/toc/text(), $expectedValue)"</input>
        <input name="successMessage">"The provided XML is correct."</input>
        <input name="failureMessage">"The provided XML does not match the requirements."</input>
    </verify>
    <!--
        Evaluate an expression that will give you the final expression to use.
    -->
    <verify handler="XPathValidator" desc="Check document">
        <input name="xml">$myDocument</input>
        <!-- Define expression without a variable reference using string concatenation -->
        <input name="expression">"contains(/toc/text(), '" || $expectedValue || "')"</input>
        <input name="successMessage">"The provided XML is correct."</input>
        <input name="failureMessage">"The provided XML does not match the requirements."</input>
    </verify>

In the expressions you use for the validations (attribute ``expression``) you may also make use of XML namespaces. Doing so is actually
a best practice to ensure that you don't have ambiguous results due to elements with the same local names. To use namespaces in expressions
you first need to define their prefixes in the test case's :ref:`namespaces section<test-case-namespaces>`. Moreover, keep in mind that the
provided input (attribute ``xml``) also supports expressions with namespaces when determining the XML content to apply the XPath
expression to (if e.g. you want to validate only a part of an XML document).

The following example illustrates how you can use namespace prefixes with your XPath expressions:

.. code-block:: xml

    <testcase>
        <!--
            Declare the namespaces to be used.
        -->
        <namespaces>
           <ns prefix="ns1">urn:specification:foo</ns>
           <ns prefix="ns2">urn:specification:bar</ns>
        </namespaces>
        <steps>
            <!--
                Use the defined namespaces.
            -->
            <verify handler="XPathValidator" desc="Check document">
                <input name="xml">$myDocument</input>
                <input name="expression">"/ns1:Foo/ns2:Bar/text() = 'EXPECTED'"</input>
                <input name="successMessage">"The provided XML is correct."</input>
                <input name="failureMessage">"The provided XML does not match the requirements."</input>
            </verify>
        </steps>
    </testcase>