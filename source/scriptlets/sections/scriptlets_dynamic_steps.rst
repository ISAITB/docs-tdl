Similar to :ref:`dynamically changing labels and actors<scriptlets_dynamic_references>`, a scriptlet can also have its contained
steps be dynamically defined. This is achieved via specific ``boolean`` flags, covered in this section, that in the case of
scriptlets can be set as :ref:`variable references<test-case-referring-to-variables>` as long as their values can be determined
at test case load time and remain constant during execution. Such flags, defining the visual representation of a test case, need
to remain fixed given that a test case's presentation cannot change during test execution.

Dynamically changing a scriptlet's contained steps can be achieved via the following flags:

* The ``hidden`` attribute :ref:`supported by all test steps<tdl-steps-common-hidesteps>`. This allows you to hide steps while
  ensuring they **will still be executed**.
* The ``static`` attribute of :ref:`if steps<tdl-step-if>`. This allows you to conditionally include steps, ensuring that non-included
  steps are **not executed**.

In the case of the ``hidden`` attribute, setting this to "true" will result in a step being executed but not displayed. 
Setting this to a :ref:`variable reference<test-case-referring-to-variables>` is allowed only within scriptlets
in which case the variable needs to match one of the scriptlet's :ref:`parameters<scriptlets_elements_params>`. The variable reference
is evaluated when the test case is loaded, with the resulting value being determined either from the 
scriptlet's inputs (provided via its :ref:`call step<tdl-step-call>`) or the default value defined for the parameter.

The :ref:`if step's<tdl-step-if>` ``static`` attribute goes further, allowing you to fully skip sets of steps. In this
case, when ``static`` is set to "true" the test engine will expect to find the step's condition (its ``cond`` element) set
with a :ref:`variable reference<test-case-referring-to-variables>`. It is this variable reference that is then evaluated at load
time to determine whether to include the steps in the ``then`` block (if "true"), or the ``else`` block (if "false" and if defined).
One important additional point here is that a statically evaluated :ref:`if step<tdl-step-if>` **does not display a boundary box and title**,
but rather directly displays the steps of the selected branch. This means that you can use a static ``if`` as other languages use
"include" and "import" constructs.

.. note::
    A similar result to an :ref:`if step<tdl-step-if>` defined as ``static`` can be achieved by using a hidden ``if`` step with an
    explicitly visible ``then`` block. Check the :ref:`if step's documentation<tdl-step-if>` for more details.

The cases described above are presented in the sample scriptlet below to highlight their use.

.. code-block:: xml

    <scriptlet id="receiveData" xmlns="http://www.gitb.com/tdl/v1/">
        <params>
            <!-- Parameter that must be specified as an input. -->
            <var name="hideMessage" type="boolean"/>
            <!-- Parameter that may be optionally specified as an input. -->
            <var name="validateAsExpression" type="boolean">
                <value>true</value>
            </var>
        </params>
        <steps>
            ...
            <!-- If 'hideMessage' is set to true, this message exchange will take place but will not be displayed. -->
            <receive hidden="$hideMessage" id="receiveStep" desc="Receive message" from="Actor1" to="Actor2" txnId="t1">
                <input name="countryCode">$ORGANISATION{countryCode}</input>
            </receive>
            <!-- 
                Static 'if' to determine the steps used to process the received message.
                No 'if' boundary will be displayed and the steps of the rejected branch will be excluded.
            -->
            <if static="true">
                <cond>$validateAsExpression</cond>
                <then>
                    <log>"Validating input as a TDL expression"</log>
                    <verify handler="ExpressionValidator" desc="Validate value">
                        <input name="expression">$value = $expectedValue</input>
                    </verify>
                </then>
                <else>
                    <log>"Validating input as a regular expression"</log>
                    <verify handler="RegExpValidator" desc="Validate value">
                        <input name="input">$value</input>
                        <input name="expression">'^REF\-\d+$'</input>
                    </verify>    
                </else>
            </if>             
            ...
        </steps>
    </scriptlet>

As you see the scriptlet expects two parameters, ``hideMessage`` and ``validateAsExpression``, that determine the display
and content of the scriptlet. They are used respectively in the :ref:`receive step's<tdl-step-receive>` ``hidden`` attribute,
and the condition of the :ref:`if step<tdl-step-if>` that is marked as ``static``.

When calling this scriptlet we need to ensure that both these variables can be evaluated at load time. The ``validateAsExpression``
parameter already has a default value, so a :ref:`call step<tdl-step-call>` needs to only override it if needed. The following
example illustrates this (note how ``false()`` is used as opposed to "false" given that this is an :ref:`expression<test-case-expressions>`).

.. code-block:: xml

    <call path="scriptlets/receiveData.xml">
        <input name="hideMessage">false()</input>
        <!--
            No need to specify 'validateAsExpression' as true given
            that this is already the default.

            <input name="validateAsExpression">true()</input>
        -->
    </call>

.. _scriptlets_scope:

Scriptlet variable scope
------------------------

Scriptlets define their own scope which acts as a child scope of their parent (either a test case or another scriptlet). Any variables
created when a scriptlet is called are added to its own scope and do not propagate to the parent one, thus ensuring that no unintended
changes are made to the state of its caller. This covers:

* :ref:`Input parameters <scriptlets_elements_params>` passed to the scriptlet from its parent via the :ref:`call step <tdl-step-call>`.
* :ref:`Imports <scriptlets_elements_imports>` defined by the scriptlet.
* :ref:`Variables <scriptlets_elements_variables>` defined explicitly by the scriptlet.
* Variables created implicitly by the scriptlet through the :ref:`assign step <tdl-step-assign>`.

While new variables are always created in the scriptlet's own scope, it is possible to read variables from a parent
scope as long as these are looked up using a :ref:`variable reference <test-case-referring-to-variables>`. When referring to a variable
that does not exist in the scriptlet's own scope it is the responsibility of the test developer to ensure that these will be available
when the scriptlet is called. Although such references are not prevented, the better practice is to pass information from the caller to
the scriptlet as :ref:`input parameters <scriptlets_elements_params>` to ensure that the scriptlet is always portable and not impose
requirements to its caller for its correct functioning that are not obvious.

.. note::
    Variables referenced by a scriptlet that are expected to be available in its parent scope are reported as part of a test suite's validation.

The following example, and its included comments, highlights how scriptlet scopes work:

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <scriptlet id="validateDocument" xmlns="http://www.gitb.com/tdl/v1/">
        <imports>
            <!-- 
                This import defines a 'schemaToUse' variable in the scriptlet's own scope.
            -->
            <artifact name="schemaToUse">resources/aSchemaFile.xsd</artifact>
        </imports>
        <params>
            <!-- 
                This parameter defines a 'contentToValidate' variable in the scriptlet's own scope that
                is passed from the parent in a call step.
            -->
            <var name="contentToValidate" type="object"/>
        </params>
        <variables>
            <!-- 
                This explicit variable declaration defines a 'stopOnXsdErrors' variable in the
                scriptlet's own scope.
            -->
            <var name="stopOnXsdErrors" type="boolean"><value>true</value></var>
        </variables>
        <steps>
            <!-- 
                The assign step implicitly creates the 'showValidationArtefacts' variable in its own
                scope even if such a variable exists also in the parent scope.
            -->
            <assign to="showValidationArtefacts">true()</assign>
            <verify handler="XmlValidator" desc="Validate XML structure">
                <!-- 
                    Reference the 'contentToValidate' parameter.
                -->
                <input name="xml">$contentToValidate</input>
                <!-- 
                    Reference the 'schemaToUse' import.
                -->
                <input name="xsd">$schemaToUse</input>
                <!-- 
                    Reference the 'showValidationArtefacts' variable defined via the earlier assign step.
                -->
                <input name="showValidationArtefacts">$showValidationArtefacts</input>
                <!-- 
                    Given that no variable named 'sortBySeverity' exists in the scriptlet's scope this
                    will be looked up from the parent and must be found. This means that the parent will
                    need to already define this before calling the scriptlet. A better approach would
                    have been to pass this flag as an additional input parameter.
                -->
                <input name="sortBySeverity">$sortBySeverity</input>
            </verify>
        </steps>
    </scriptlet>
