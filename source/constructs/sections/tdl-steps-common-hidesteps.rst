The purpose of most test steps, apart from carrying out their respective actions, is to also communicate progress and results to the user.
Depending on how specific steps are rendered, they can present the test session's control flow and reports that include a step's input, output
and validation results (in case of a :ref:`verify<tdl-step-verify>` step).

Depending on the purpose of a given step it could nonetheless be preferrable to hide it from the test session's display. Doing so could be interesting
in case this step is used as a complementary action that is not important from a testing perspective but is required to e.g. clean up resources or make
internal updates. Examples of such cases include:

* :ref:`Sending<tdl-step-send>` a finalisation message to a given test service (e.g. a :ref:`custom messaging service<handlers>`).
* Making a :ref:`processing call<tdl-step-process>` to record statistics (e.g. via a :ref:`custom processing service<handlers>`).
* Validating content via a :ref:`verify<tdl-step-verify>` step at warning level as an internal check to determine subsequent actions.
* Additional control flow steps (e.g. :ref:`if<tdl-step-if>` steps) to determine finalisation actions to make.
* Making a manual verification of test session data through an :ref:`administrator interaction <tdl-step-interact_admin_interactions>`.

Hiding an otherwise visible test step is supported by means of the ``hidden`` attribute. This takes a ``boolean`` value that determines whether the
step should be included in the test session's display. When set to false, the step is not presented but is executed by the test engine as expected.
In other words, hiding a step affects only its visual representation, not its processing.

The following example includes a :ref:`verify<tdl-step-verify>` step that is not meant to be displayed to the user but is used to determine subsequent
processing. Note how the verification is forced at warning level to not impact the test session's result:

.. code-block:: xml

    <!-- This check will not be presented in the test session display. -->
    <verify id="internalCheck" handler="StringValidator" hidden="true" level="WARNING">
        <input name="actual">$valueToCheck</input>
        <input name="expected">'CASE1'</input>
    </verify>
    <!-- Conditional branch based on previous (hidden) check result. -->
    <if>
        <cond>$internalCheck</cond>
        <then>
            ...
        </then>
    </if>

In case you need to take multiple internal actions that you want to hide, a good approach is to use the :ref:`group<tdl-step-group>` step. To do so 
place all such internal steps within a :ref:`group<tdl-step-group>` and set the group itself to be ``hidden``. Any steps included in a
step that takes child steps (e.g. :ref:`group<tdl-step-group>`, :ref:`if<tdl-step-if>`, :ref:`foreach<tdl-step-foreach>`, :ref:`flow<tdl-step-flow>`)
which is set as ``hidden`` will be altogether removed from the display. This is the case regardless of how the ``hidden`` attibute may be set
on child steps.

The following example illustrates how to hide a :ref:`group<tdl-step-group>` of steps:

.. code-block:: xml

    <group hidden="true">
        <verify id="internalCheck" handler="StringValidator" level="WARNING">
            ...
        </verify>
        <process>
            ...
        </process>
        <send>
            ...
        </send>
    </group>

The ``hidden`` attribute is supported on all test steps that can be visually represented. Check the documentation of each step to see whether displaying or
hiding it is applicable.

.. note::
    **Visible process steps:** The default value for the ``hidden`` attribute is true, resulting in the relevant steps being displayed. The exception
    is the :ref:`process<tdl-step-process>` step that is hidden by default. Setting ``hidden`` to false on a :ref:`process<tdl-step-process>` step will
    result in it being displayed, providing a report that includes the step's output.
