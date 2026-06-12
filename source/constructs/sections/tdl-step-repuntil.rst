The ``repuntil`` step allows you to execute a sequence of steps at least once, checking at the end a condition to see if another iteration
should take place. The structure of the ``repuntil`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @collapsed, no, A boolean flag determining whether or not the step is displayed as initially collapsed (default is "false"). See also :ref:`tdl-steps-common-collapsed`.
    @desc, no, A description for this loop to display to the user and to include in the test session log. Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.
    @hidden, no, A boolean flag determining whether or not the step is displayed to users (default is "false"). Note that within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`. See :ref:`tdl-steps-common-hidesteps` for further details.
    @skipped, no, A boolean value or variable reference (default being "false") which will result in the step being skipped if "true". See also :ref:`tdl-steps-common-skipped`.
    @stopOnError, no, A boolean flag determining whether the test session should end if this step fails (default is "false"). See also :ref:`tdl-steps-common-stoponerror`.
    @title, no, A short title to display for this step (default is "loop"). Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.
    cond, yes, The condition to verify in order to execute again the steps contained in ``do``. This is provided as an expression (see :ref:`test-case-expressions`).
    do, yes, Contains as children any sequence of steps to execute at least once and then again if the condition in ``cond`` is true.
    documentation, no, Rich text content that provides further information on the current step.

.. code-block:: xml

    <assign to="iteration" type="number">1</assign>
    <assign to="maxIteration" type="number">3</assign>
    <repuntil desc="Do iteration">
        <do>
            <interact desc="Message to user">
                <instruct desc="Iteration">$iteration || " of " || $maxIteration</instruct>
            </interact>
            <assign to="iteration">$iteration + 1</assign>
        </do>
        <cond>$iteration &lt;= $maxIteration</cond>
    </repuntil>

.. note::
    **Do-while:** Step ``repuntil`` stands for "repeat until". Considering this you could assume that the steps in ``do`` will be executed until
    the condition in ``cond`` is true. This is actually not the case currently as steps are executed while the condition in ``cond`` remains true
    (i.e. the logic is actually inversed). The naming of this step is thus unfortunate; it would be more appropriate if this was named ``dowhile``
    reflecting accurately how the condition is considered.
