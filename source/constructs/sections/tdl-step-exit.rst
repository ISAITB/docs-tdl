The ``exit`` step is used to immediately exit the test case from any execution branch. The structure of the ``exit`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @desc, no, A description for the step to display to the user and to include in the test session log. Within scriptlets this can also be a :ref:`variable reference<scriptlets_dynamic_references>`.
    @hidden, no, A boolean flag determining whether or not the step is displayed to users (default is "false"). Note that within scriptlets this can also be a :ref:`variable reference<scriptlets_dynamic_references>`. See :ref:`tdl-steps-common-hidesteps` for further details.
    @skipped, no, A boolean value or variable reference (default being "false") which will result in the step being skipped if "true". See also :ref:`tdl-steps-common-skipped`.
    @success, no, Whether or not this step should be considered as a success or failure (the default). This is provided as a ``boolean`` or a :ref:`variable reference<test-case-referring-to-variables>`.
    @undefined, no, Whether or not this step should complete the test session with an undefined outcome (considered only if ``success`` is not set to ``true``). This is provided as a ``boolean`` or a :ref:`variable reference<test-case-referring-to-variables>`.
    documentation, no, Rich text content that provides further information on the current step.

The following example shows a test case that exits as a success based on the user's input:

.. code-block:: xml
    :emphasize-lines: 8

    <assign to="inputValue">'NO'</assign>
    <interact desc="Provide your choice">
        <request desc="Enter 'YES' to end the test">$inputValue</request>
    </interact>
    <if>
        <cond>$inputValue = 'YES'</cond>
        <then>
            <exit desc="Terminate test" success="true"/>
        </then>
        <else>
            <interact desc="You chose to continue">
                <instruct desc="Test continues"/>
            </interact>
            <verify handler="XmlValidator" desc="Validate content">
                <input name="xml">$document</input>
                <input name="xsd">$schemaFile"</input>
            </verify>
        </else>
    </if>

The result type of the ``exit`` step can also be determined via variable reference. The example that follows exits as a success or failure depending
on whether or not the user provides a "true" of "false" input:

.. code-block:: xml

    <interact desc="Decide outcome">
        <request desc="Succeed?">$choice</request>
    </interact>
    <exit desc="Finished" success="$choice"/>

Besides being used to forcibly succeed or fail a test session, you can also use the ``exit`` step to terminate a test session without a specific
result. This is achieved by setting the ``undefined`` attribute to ``true`` (or to a :ref:`variable reference<test-case-referring-to-variables>`
evaluating to ``true``). An example of when this could be interesting is to handle errors that come up during a test case's setup phase. In such
a case, you want to prevent the test session from proceeding, but also want to make it clear that the test session could not be processed as opposed
to having failed its assertions. The following example illustrates such a scenario: 

.. code-block:: xml
    :emphasize-lines: 14

    <!--
        Group of send steps that POST test datasets to the SUT as part of the test's setup.
    -->
    <group id="setup" title="Setup" stopOnChildError="true">
        <send desc="Load test dataset 1" handler="HttpMessagingV2">...</send>
        <send desc="Load test dataset 2" handler="HttpMessagingV2">...</send>
    </group>
    <!-- 
        If any of the setup steps failed, terminate the test session with an undefined result.
    -->
    <if hidden="true">
        <cond>$STEP_STATUS{setup} = 'ERROR'</cond>
        <then hidden="false">
            <exit desc="Ensure correct setup" undefined="true"/>
        </then>
    </if>
