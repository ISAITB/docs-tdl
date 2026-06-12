The ``if`` step is used to run one of more steps if a condition is met. Its structure is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @collapsed, no, A boolean flag determining whether or not the step is displayed as initially collapsed (default is "false"). See also :ref:`tdl-steps-common-collapsed`.
    @desc, no, A description for this check to display to the user and to include in the test session log. Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.
    @hidden, no, A boolean flag determining whether or not the step is displayed to users (default is "false"). Note that within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`. See :ref:`tdl-steps-common-hidesteps` for further details.
    @skipped, no, A boolean value or variable reference (default being "false") which will result in the step being skipped if "true". See also :ref:`tdl-steps-common-skipped`.
    @static, no, A boolean flag determining whether the step's conditions is evaluated at test case load time ("true") or at runtime ("false" - the default). See also :ref:`scriptlets_dynamic_steps`.
    @stopOnError, no, A boolean flag determining whether the test session should end if this step fails (default is "false"). See also :ref:`tdl-steps-common-stoponerror`.
    @title, no, A short title to display for this step (default is "decision"). Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.
    cond, yes, The condition to verify in order to execute the ``then`` set of steps (if true) or ``else`` (if false). This is provided as an expression (see :ref:`test-case-expressions`).
    documentation, no, Rich text content that provides further information on the current step.
    else, no, Contains as children any sequence of steps to execute if the condition results to false.
    then, yes, Contains as children any sequence of steps to execute if the condition results to true.

The following example illustrates use of the ``if`` step to conditionally validate received content based on a condition.

.. code-block:: xml

    <if desc="Check process type">
        <cond>$processType = 'process1'</cond>
        <then>
            <assign to="formatType">'XML'</assign>
            <verify handler="https://VALIDATOR?wsdl" desc="Validate as XML">
                <input name="source" source="$document"/>
                <input name="validationType">$formatType</input>
            </verify>
        </then>
        <else>
            <assign to="formatType">'CSV'</assign>
        </else>
    </if>

.. note::
    **Setting variables conditionally:** If you simply need to set a variable's value conditionally you don't require 
    an ``if`` step. A simpler approach is to use the :ref:`assign step <tdl-step-assign>` with an XPath if expression
    such as the following (more expression examples available :ref:`here <test-case-expressions>`):

    ``<assign to="variable">if ($flag) then "Value 1" else "Value 2"</assign>``

    Using an ``if`` step is more appropriate when you need to conditionally execute multiple steps, or when the condition
    branches themselves are defined in the specifications you are testing for.

.. _tdl-step-if_hide_boundary:

Displaying contained steps without a boundary
+++++++++++++++++++++++++++++++++++++++++++++

Aside from using an ``if`` step to represent logical branches to users, we can also use it as an **internal control structure** for 
our testing logic. In such a case, we may want to only present the step's included child steps, and not the boundary structure,
title and description of the ``if`` step itself.

Displaying only an ``if`` step's children is possible via two approaches:

* Use of the ``if`` step's ``static`` attribute.
* Use of the ``hidden`` attribute on the ``if`` step and its contained ``then`` block.

The **first approach**, defining a static ``if``, means that the step's condition is evaluated when the test case is loaded as opposed
to a runtime evaluation when the step is executed. The result of this is the inclusion of either the step's ``then`` or ``else``
block in the test case without the ``if`` step's overall boundary. Using this feature is meaningful for tests triggered via API call,
or when ``if`` steps appear within :ref:`scriptlets<scriptlets>` as it allows their content to be dynamically adapted based on the needs of the given test case.
For more details on this check :ref:`how scriptlets can dynamically define their steps <scriptlets_dynamic_steps>`.

The **second approach**, using the ``hidden`` attribute, achieves a similar effect as the ``static`` flag but with the key difference
that the ``if`` condition is evaluated at runtime. To use this approach you need to set the ``if`` step as ``hidden`` but also
specify its ``then`` block as explicitly visible (``hidden`` set to "true"). This results in hiding the ``if`` step's boundary
and displaying directly the steps contained within the ``then`` block. These steps may subsequently be skipped (and displayed as
such) if the ``if`` step's condition evaluates to "false". Displaying steps directly, only to potentially mark them as skipped
may seem confusing but could be useful for single optional steps. An example scenario is including a check to stop execution
which me way want to display as an :ref:`exit step<tdl-step-exit>` that ends up getting skipped. In such a case, whether you
show or not such a step's containing ``if`` structure is effectively the same, and only affects the display you want to achieve.

The following example illustrates exactly this use case of including an :ref:`exit step<tdl-step-exit>` directly and displaying it
as skipped if the exit condition is not met.

.. code-block:: xml

    <!-- Validate content. -->
    <receive id="receiveData" from="Actor1" to="Actor2" handler="..."/>

    <!-- 
        Check and exit if needed. We set 'hidden' to 'true' to hide the if step's boundary.
    -->
    <if hidden="true">
        <cond>$receiveData{messageType} != $expectedType</cond>
        <!-- 
            Only the 'exit' step will be displayed and skipped if the condition is not matched.
            This is achieved by setting 'hidden' explicitly to 'false'.
        -->
        <then hidden="false">
            <exit desc="Stop due to unexpected message type"/>
        </then>
    </if>     

.. note::
    A hidden ``if`` step can only have a visible ``then`` block. If an ``else`` block is defined it will never be displayed although
    it may be executed in case the ``if`` condition evaluates to "false". If you want to conditionally include and display either the ``then`` or
    the ``else`` block, you should check out how a static ``if`` can be :ref:`used within scriptlets<scriptlets_dynamic_steps>`.
