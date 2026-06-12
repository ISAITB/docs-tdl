The ``namespaces`` optional element is used to declare namespace mappings for use within the test case. The primary use cases of these namespaces is to allow
the definition of prefixes used in XML and XPath expressions. In principle they could be applied to any type of language or expression that has such a concept
(e.g. JSON-LD, Turtle) but currently their use is limited to XML.

Each namespace to declare is defined as a child ``ns`` element with the following structure:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @prefix, yes, The namespace prefix that will be used in expressions.

The value to which the ``prefix`` is mapped is provided as the ``ns`` element's text content.

Namespaces declared using this approach can be used in two cases:

* Within any GITB TDL step that supports :ref:`expressions<test-case-expressions>`.
* As the expression to apply for the :ref:`XPathValidator<handlers-XPathValidator>` embedded validation handler.

The following example illustrates how namespaces can be used for XML-based processing. The sample test case:

#. Requests an invoice from the user.
#. Extracts the invoice's type using namespaces in an :ref:`assign step<tdl-step-assign>` and then logs it.
#. Validates the invoice's type using namespaces with the :ref:`XPathValidator<handlers-XPathValidator>`.

.. code-block:: xml

    <testcase>
        <!-- 
            Declare the namespaces to use used in XPath expressions.
        -->
        <namespaces>
           <ns prefix="inv">urn:oasis:names:specification:ubl:schema:xsd:Invoice-2</ns>
           <ns prefix="cbc">urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2</ns>
        </namespaces>
        <steps>
            <!-- 
                Request the user to upload the invoice to validate.
            -->
            <interact id="input" desc="Upload invoice">
                <request desc="File upload" name="xml" inputType="UPLOAD"/>
            </interact>
            <!--
                Use an XPath expression to extract the invoice type as an XML element.
            -->
            <assign to="invoiceTypeElement" type="object" source="$input{xml}">/inv:Invoice/cbc:InvoiceTypeCode</assign>
            <!--
                Log the extracted element.
            -->
            <log>$invoiceTypeElement</log>
            <!--
                Use XPath to validate the invoice.
            -->
            <verify handler="XPathValidator" desc="Check invoice type">
                <input name="xml">$input{xml}</input>
                <input name="expression">"/inv:Invoice/cbc:InvoiceTypeCode/text() = '380'"</input>
            </verify>  
        </steps>
    </testcase>
