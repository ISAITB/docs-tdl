Any test step may be **skipped** during execution by setting the step's ``skipped`` attribute to "true". Considering that setting this with a fixed "true" or "false"
(the default if missing) would not have much meaning, this attribute also accepts a :ref:`variable reference <test-case-referring-to-variables>` to set it
**dynamically**.

The following example shows an :ref:`interact step <tdl-step-interact>` that will be skipped if a specific condition on the current session state is met:

.. code-block:: xml

    <!--
        Calculate the condition to skip the next step.
    -->
    <assign to="skipNextMessage">$messageType = 'final'</assign>
    <!--
        Skip the interaction if unnecessary.
    -->
    <interact desc="Instruct users" skipped="$skipNextMessage" hidden="true">
        <instruct desc="Proceed to send the next message"/>
    </interact>

Besides calculating the ``skipped`` value based on the variable state of a test session, you could also refer to :ref:`configuration parameters <test-case-configuration>`.
The following example shows how you could use a configuration flag set as a :ref:`domain parameter <test-case-expressions-domain>` to skip all 
interactions:

.. code-block:: xml

    <!--
        Skip the interaction if configured to do so.
    -->
    <interact desc="Instruct users" skipped="$DOMAIN{skipInstructions}" hidden="true">
        <instruct desc="Proceed to send the next message"/>
    </interact>

A further interesting approach would be to skip steps based based on a variable defined in the test case's :ref:`variables section <test-case-variables>`.
This allows you to skip steps if the test has started via the Test Bed's `REST API <https://www.itb.ec.europa.eu/docs/itb-ta/latest/api/index.html#start>`__,
in which case an override value can be provided for the variable in question as part of the REST call's payload.

.. code-block:: xml

    <testcase>
        ...
        <variables>
            <!--
                Predefine the skipInstructions variable to allow REST API calls to set
                it with an input value.
            -->
            <var name="skipInstructions" type="boolean">
                <!-- 
                    Set the default value to "false" to cover regular interactive executions.
                    This will be overriden when the test is started via the REST API.
                -->
                <value>false</value>
            </var>
        </variables>
        ...
        <steps>
            ...
            <!-- 
                Refer to the skipInstructions variable to see if we skip the step or not.
            -->
            <interact desc="Instruct users" skipped="$skipInstructions" hidden="true">
                <instruct desc="Proceed to send the next message"/>
            </interact>
            ...
        </steps>
    </testcase>

The previous examples have shown how to skip a single step. Very ofter you will however need to **skip entire sections** of
your test case including multiple steps. To avoid setting the ``skipped`` attribute on each step, you can use a :ref:`group step <tdl-step-group>`
which allows you to apply common behaviour to all contained steps in one go. To avoid including the group's display in the test execution
diagram you can also set ``hiddenContainer`` to "true":

.. code-block:: xml

    <!--
        Skip an entire set of steps.
    -->
    <group hiddenContainer="true" skipped="$skipNextMessage">
        <interact desc="Instruct users" hidden="true">
            <instruct desc="Proceed to send the next message"/>
        </interact>
        <receive desc="Receive message" handler="HttpMessagingV2">
            ...
        </receive>
        <verify desc="Validate received message" handler="JsonValidator">
            ...
        </verify>
    </group>

An alternative to using the ``skipped`` attribute to skip steps is to use an :ref:`if step <tdl-step-if>` that will check a condition
and execute its contained steps if "true". Although the result would be similar, it requires you to write additional test steps
resulting in longer and less intuitive test definitions. An :ref:`if step <tdl-step-if>` would nonetheless still be the better option if:

* You want to show the steps in question on the **test execution diagram**, to provide additional context to users on the test's flow.
* You want to alternate between two sets of steps following **if/then/else** logic.
* Skipping steps requires checking a **complete expression** rather than referring to a variable.

The following example shows both approaches being used in the same test case:

.. code-block:: xml

    <!-- 
        Use the skipped attribute to skip specific steps.
    -->
    <process skipped="$skipNextMessage" handler="$DOMAIN{processingService}">
        ...
    </process>
    ...
    <!-- 
        Later on, use an if step to present the alternative flows to the user, showing
        explicitly what was skipped and what was executed.
    -->
    <if desc="Complete processing depending on whether or not a message is expected">
        <cond>$skipNextMessage</cond>
        <then>
            <!-- 
                Go through a step of steps in case skipNextMessage is true.
            -->
            <log>"Next message not expected"</log>
            ...
        </then>
        <else>
            <!-- 
                Go through an alternate step of steps otherwise.
            -->
            <log>"Next message expected"</log>
            ...
        </else>
    </if>

.. index:: collapsed
.. _tdl-steps-common-collapsed:

Collapse sets of test steps
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Test cases that include numerous test steps may appear as complex to users. Introducing :ref:`grouping<tdl-step-group>` of related steps can provide better
visual structuring and make them easier to follow. If further simplification is needed a good approach is to additionally define such groups as being by 
default collapsed by setting their ``collapsed`` attribute to "true".

Collapsing a group of steps results in them being presented as initially minimised and displaying only:

* Their description (if defined).
* Their :ref:`documentation<tdl-steps-common-documentation>` link (if defined).
* Their overall progress status and result.

After a :ref:`group<tdl-step-group>` is diplayed as collapsed, the user can always expand it to view its details and child steps. However, the initial minimised 
display helps significantly in reducing the amount of visual information on steps that you may not want to prominently display. Good examples are actions to
provide acknowledgements for received messages or sets of validations that are not the test case's main focus. It may also be interesting to display as ``collapsed``
sets of steps that repeat in several locations to avoid visual clutter.

The ``collapsed`` attribute is not only applicable to :ref:`group<tdl-step-group>` steps, but also to any other step that is presented as a block with contained details.
It can be set on:

* Steps with child steps, notably :ref:`group<tdl-step-group>`, :ref:`while<tdl-step-while>`, :ref:`repuntil<tdl-step-repuntil>`, 
  :ref:`foreach<tdl-step-foreach>`, :ref:`flow<tdl-step-flow>`, :ref:`if<tdl-step-if>`.
* The :ref:`interact<tdl-step-interact>` step used to trigger user actions.

The default value for the ``collapsed`` attribute is always "false", meaning that all relevant steps will by default be presented as
fully expanded.

The following example illustrates how a set of tests on XML content can be displayed as initially collapsed to simplify the display:

.. code-block:: xml

    <!-- 
        The three verify steps will be initially hidden, showing instead only their containing group.
        The group can be at any point expanded to view internal details (e.g. if a validation error is reported).
    -->
    <group desc="Validate XML message" collapsed="true">
        <verify handler="XmlValidator" desc="Validate message structure">...</verify>
        <verify handler="XmlValidator" desc="Validate core business rules">...</verify>
        <verify handler="XmlValidator" desc="Validate additional contraints">...</verify>
    </group>

.. note::
    **Collapsed vs Hidden:** Apart from collapsing a group of steps, steps can also be set as :ref:`hidden<tdl-steps-common-hidesteps>`.
    Use ``collapsed`` when you want to still include steps in the test session display but simplify their presentation. Use ``hidden``
    for purely internal steps that you want to completely remove from the display. In case ``hidden`` is set to "true" the ``collapsed``
    attribute is effectively ignored.
