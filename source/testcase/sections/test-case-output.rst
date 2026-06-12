The ``output`` element is an optional means of defining a final result message for a given test session. It is processed once all
steps have completed, checking the data in the test session's context to display specific messages. Using this
element allows extended feedback to be returned that may be important to summarise and contextualise the steps' results.

The ``output`` element supports messages for a success, failure, or undefined result; defined as different cases with match conditions and an
overall default. The structure of the ``output`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    failure, no, The set of output cases to apply in case the test completes with a failure.
    success, no, The set of output cases to apply in case the test completes with a success.
    undefined, no, The set of output cases to apply in case the test completes with an undefined result.

The ``success``, ``failure`` and ``undefined`` elements share a common structure to define their specific cases and overall defaults:

.. csv-table::
    :stub-columns: 1
    :delim: ~
    :header: "Name", "Required?", "Description"

    @match ~ no ~ The approach to follow when matching message cases. Can be ``first`` (the default), ``all`` or ``cascade``.
    case ~ no ~ Zero or more specific cases to apply depending on the provided match conditions.
    default ~ no ~ An optional default if no specific case was found to apply.

Finally, each ``case`` element shares a common structure as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    cond, yes, Defines a condition :ref:`expression<test-case-expressions>` expected to return a "true" or "false" value.
    message, yes, Defines an :ref:`expression<test-case-expressions>` expected to return the output message (as a string).

The ``output`` section is flexible as it doesn't require you to define messages for each outcome type. In addition, you could
opt only to provide default messages without specific cases or only provide certain specific messages without generic defaults.
It is important to note that the condition expressions tolerate failures, meaning that if a condition cannot
be evaluated its relevant case will simply be skipped. In addition, if any error is raised when creating the text message itself, the output
message will be altogether skipped; under no circumstances will a test session fail due to the evaluation of its ``output`` section
(relevant warnings will however be included in the test session's log).

The following snippet illustrates how the ``output`` section could be leveraged to return user-friendly failure messages based on the executed
test steps (using the :ref:`STEP_STATUS<test-case-expressions-step-status>` variable to determine the cause of the failure). For successful
and undefined states we only include a default message to make test sessions more user friendly.

.. code-block:: xml

    <testcase>
        <steps stopOnError="true">
            ...
            <verify id="checkIntegrity" desc="Validate integrity">
                ...
            </verify>
            <verify id="checkSyntax" desc="Validate syntax">
                ...
            </verify>
            <verify id="checkContent" desc="Validate business rules">
                ...
            </verify>
            ...
        </steps>
        <output>
            <success>
                <default>"Test session completed successfully."</default>
            </success>
            <failure>
                <case>
                    <cond>$STEP_STATUS{checkIntegrity} = 'ERROR'</cond>
                    <message>"Please verify the integrity of your data and re-submit."</message>
                </case>
                <case>
                    <cond>$STEP_STATUS{checkSyntax} = 'ERROR'</cond>
                    <message>"Please verify the syntax of your data and re-submit."</message>
                </case>
                <case>
                    <cond>$STEP_STATUS{checkContent} = 'ERROR'</cond>
                    <message>"Please verify your data content and re-submit."</message>
                </case>
                <!-- The default will be applied if no specific case was found to match. -->
                <default>"Your data failed to be processed correctly. Please check the session log to determine the cause of the failure."</default>
            </failure>
            <undefined>
                <default>"Test session stopped before producing a result."</default>
            </undefined>
        </output>
    </testcase>

It may seem at first unintuitive to provide output messages for anything other than failures. However, there are a few cases where this could be
particularly useful:

* A success message as a **user-friendly confirmation** that the test succeeded.
* A success message to **highlight a warning** (steps resulting in warnings are successful).
* An undefined message for an :ref:`exit step <tdl-step-exit>` that terminated with an undefined result.

As mentioned earlier you can choose to omit certain outcomes altogether. For example, the following test case defines only failure messages
(a specific one and a default one):

.. code-block:: xml

    <testcase>
        <steps stopOnError="true">
            ...
        </steps>
        <output>
            <failure>
                <case>
                    <cond>$STEP_STATUS{checkContent} = 'ERROR'</cond>
                    <message>"Please verify your data content and re-submit."</message>
                </case>
                <default>"Your data failed to be processed correctly. Please check the session log to determine the cause of the failure."</default>
            </failure>
        </output>
    </testcase>

The examples up to this point consider outputting a single message depending on the result. It is possible to have **several messages** be 
returned which could be used as an alternative to test step reports for simple test cases. This is achieved through the ``match`` attribute
of the ``failure``, ``success`` and ``undefined`` elements that supports the following values:

* ``first`` (the default), meaning that the first (sequentially) condition to match will be considered. If none match the
  default applies.
* ``all``, meaning that all conditions with passing conditions will be considered. If none match the default applies.
* ``cascade``, meaning that once a first condition is found to pass, the check will cascade to subsequent conditions and include them if
  they also pass. If a condition check fails then the cascade stops even if later conditions would also pass. If none match
  the default applies.

To illustrate this with an example, consider the following test case output section:

.. code-block:: xml

    <output>
        <failure match="all">
            <case>
                <cond>$flag1</cond>
                <message>"The integrity check on your data failed."</message>
            </case>
            <case>
                <cond>$flag2</cond>
                <message>"The syntax check on your data failed."</message>
            </case>
            <default>"Your data failed to be processed correctly. Please check the session log to determine the cause of the failure."</default>
        </failure>
    </output>

In the above example given that match is set to ``all``, both messages could be displayed if both ``flag1`` and ``flag2`` are true.
If ``match`` was set to ``first`` then only the first message would be generated. Finally, the ``cascade`` option allows you to
fine-tune the display of multiple related messages, without forcing an all or nothing approach.

.. note::
    **Messages are also expressions:** Output messages are themselves :ref:`expressions<test-case-expressions>` allowing dynamic
    output to be returned (e.g. concatenating text with session variable values). For a fixed message make sure to enclose the text
    in quotes.
