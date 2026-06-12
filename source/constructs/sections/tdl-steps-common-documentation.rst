Test steps that are meant to be presented to users can be defined with an additional ``documentation`` element to include extended rich text documentation as HTML. This complements the limited label
attached to each step (via attribute ``desc``), allowing further instructions, context and references to be provided. The structure of this element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    encoding, no, In case an ``import`` reference is defined this can be used to specify the file's encoding. If not provided ``UTF-8`` is considered.
    from, no, The identifier of a test suite from which the ``import`` file will be loaded. If unspecified the current test suite is assumed.
    import, no, A reference to a separate file within the test suite archive that defines the documentation content.

Using the above attributes to specify a reference to a separate file is not mandatory. The documentation's content can also be provided as the element's text content,
typically enclosed within a CDATA section if this includes HTML elements (in which case the ``from``, ``import`` and ``encoding`` attributes are omitted).
When loading documentation from a separate file, it is also possible to lookup this file from another test suite. This is
done by specifying as the value of the ``from`` attribute the ``id`` of the target test suite. This is used to lookup the
target test suite as follows:

#. Look for the test suite in the same **specification** as the current test case.
#. If not found in the same specification, look for the test suite in the other specifications of the test case's **domain**.
   If across specifications multiple matching test suites are found, one of them will be arbitrarily picked. To avoid such
   a scenario it is obvious that you should ensure test suites used to load shared resources can be uniquely identified.

This documentation can provide further information on the context of the test step, diagrams or reference information that are useful to understand how it is to be completed. The content supplied supports
several HTML features:

    * Structure elements (e.g. headings, text blocks, lists).
    * In-line styling.
    * Tables.
    * Links.
    * Images.

The simplest way to provide such information is to enclose the HTML content in a ``CDATA`` section to ensure the test case XML remains well-formed. The
example that follows illustrates two examples, one defining a simple additional text, and another with more comprehensive HTML content.

.. code-block:: xml

    <!-- Additional documentation as simple text. -->
    <verify handler="XmlValidator" desc="Validate invoice against schema">
        <documentation>This is an extra documentation item.</documentation>
        <input name="xml">$invoice</input>
        <input name="xsd">$schema</input>
    </verify>
    <!-- Additional documentation as rich HTML content. -->
    <verify handler="XmlValidator" desc="Validate invoice against business rules" level="WARNING">
        <documentation><![CDATA[
        <p>This is <b>important information!</b> Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum. </p>
        <div style="width:100%; text-align:center"><img src="https://www.itb.ec.europa.eu/docs/services/latest/_images/ValidationService.png"/></div>
        <b><u>Steps for testing:</u></b>
        <p>
            <ol>
                <li>Prepare correctly</li>
                <li>Submit the correct file</li>
                <li>Validate results</li>
            </ol>
        </p>
        <p>
            <table style="border: 1px solid black; width:100%">
                <tr style="border: 1px solid black; font-weight: bold;">
                    <td>COL1</td><td>COL2</td><td>COL3</td><td>COL4</td><td>COL5</td>
                </tr>
                <tr>
                    <td>1</td><td>2</td><td>3</td><td>4</td><td>5</td>
                </tr>
                <tr>
                    <td>test1</td><td>test2</td><td>test3</td><td>test4</td><td>test5</td>
                </tr>
            </table>
        </p>
        <p>After this make sure to check the docs <a href="https://www.itb.ec.europa.eu/docs/tdl/latest">here</a>.</p>
        ]]></documentation>
        <input name="xml">$invoice</input>
        <input name="schematron" source="$schematron"/>
    </verify>

.. note::
    Documentation such as this is also supported for the overall :ref:`test suite<test-suite-metadata>` and the :ref:`test cases<test-case-metadata>` included in the test suite.

.. _tdl-steps-common-stop:

Stop execution in case of errors
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

During the course of a test session you will need to consider step failures and define how these are to affect the test session's execution.
Such failures can broadly be classified as:

* **Expected** errors resulting from :ref:`validations<tdl-step-verify>` and :ref:`message exchanges<tdl-messaging-steps>` due to violations of the specifications' requirements.
* **Unexpected** errors due to services being unavailable or unforeseen processing problems.

Any step failure that occurs will ultimately result in the overall test session to fail. However, whether or not you want to continue a test session
once a failure has occurred depends on the design of your test case. For example, if a test case involves constructing a message, validating it, and then 
using it to start a series of message exchanges, it is probably meaningless to proceed with messaging steps if the message is invalid to begin with. In such
a case, any failure when validating the message should immediately fail the test session.

Immediately stopping execution may however not always be desired. Consider a case where you receive a message from a system and then proceed to use it for a series of validations,
each focusing on a different complementary aspect (e.g. integrity, syntax and business rules). In such a scenario you want any failures to be recorded but not prevent subsequent
steps so that the user receives a complete validation outcome.

To manage all such cases and fine-tune your error handling, test steps provide specialised error-handling flags. Specifically:

* The ``stopOnError`` flag, available on all steps, to :ref:`terminate the test session <tdl-steps-common-stoponerror>` in case of error.
* The ``stopOnChildError`` flag, available on steps containing child steps, to :ref:`skip processing child steps <tdl-steps-common-stoponchilderror>` in case of error.
