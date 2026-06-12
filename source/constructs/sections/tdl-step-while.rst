The ``while`` step is the most useful looping construct. It allows a sequence of steps to be continuously executed as long as a condition
continues to be true. The structure of the ``while`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @collapsed, no, A boolean flag determining whether or not the step is displayed as initially collapsed (default is "false"). See also :ref:`tdl-steps-common-collapsed`.
    @desc, no, A description for this loop to display to the user and to include in the test session log. Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.
    @hidden, no, A boolean flag determining whether or not the step is displayed to users (default is "false"). Note that within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`. See :ref:`tdl-steps-common-hidesteps` for further details.
    @skipped, no, A boolean value or variable reference (default being "false") which will result in the step being skipped if "true". See also :ref:`tdl-steps-common-skipped`.
    @stopOnError, no, A boolean flag determining whether the test session should end if this step fails (default is "false"). See also :ref:`tdl-steps-common-stoponerror`.
    @title, no, A short title to display for this step (default is "loop"). Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.
    cond, yes, The condition to verify in order to execute the contained steps. This is provided as an expression (see :ref:`test-case-expressions`).
    do, yes, Contains as children any sequence of steps to execute if the loop's condition results to true.
    documentation, no, Rich text content that provides further information on the current step.

The following example validates the name of each attachment defined in an XML document using a ``while`` loop:

.. code-block:: xml

    <!--
        Initialise maximum iteration count based on the number of "Attachment" nodes in the document.
    -->
    <assign to="iterationCount" source="$document" type="number">count(//*[local-name() = "Attachment"]</assign>
    <assign to="iteration" type="number">1</assign>
    <while desc="Validate attachment names">
        <cond>$iteration &lt;= $iterationCount</cond>
        <do>
            <verify handler="XPathValidator" desc="The attachment is named as expected">
                <input name="xml" source="$document"/>
                <!-- 
                    Construct the XPath expression to apply using the iteration variable.
                -->
                <input name="expression">"//*[local-name() = 'Attachment'][" || $iteration || "]/text() = 'file_" || $iteration || ".xml'"</input>
            </verify>
            <!--
                Increment iteration counter.
            -->
            <assign to="iteration">$iteration + 1</assign>
        </do>
    </while>

.. index:: Support steps

Support steps
-------------

Support steps are those that perform specific actions not related to messaging, processing or flow control. 
