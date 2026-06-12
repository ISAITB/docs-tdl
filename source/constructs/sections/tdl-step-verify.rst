The ``verify`` step is used to trigger validation of content. Similar to :ref:`tdl-messaging-steps` and  :ref:`tdl-processing-steps`, validation
takes place using a validation handler implementation that can either be an embedded Test Bed component or a remote service that implements the
`GITB validation service API`_. The content to validate is provided by the test case to the handler in terms of configuration and input, for which
a test report is returned in the `GITB TRL (Test Reporting Language) format`_. The structure of the ``verify`` element is as follows:

.. _GITB validation service API: https://www.itb.ec.europa.eu/specs/latest/gitb_vs.wsdl
.. _GITB TRL (Test Reporting Language) format: https://www.itb.ec.europa.eu/specs/latest/gitb_tr.xsd

.. csv-table::
    :stub-columns: 1
    :delim: ~
    :header: "Name", "Required?", "Description"

    @actor~ no~ The identifier of an :ref:`actor <test-case-actors>` under which to display the step. See also :ref:`tdl-steps-common-step-actor`.
    @desc~ no~ A description for this validation to display to the user and to include in the test session log. Within scriptlets this can also be a :ref:`variable reference<scriptlets_dynamic_references>`.
    @handler~ yes~ A string value or variable reference identifying the the validation handler (see :ref:`handlers-implementation`).
    @handlerTimeout ~ no ~ A number or variable reference with the maximum time (in milliseconds) to wait for the handler service call to complete (in case of an external test service being used as a handler). See also :ref:`tdl-steps-common-handlerTimeouts`.
    @handlerTimeoutFlag ~ no ~ A string value with the name of a boolean variable to set informing whether or not a handler timeout occurred. See also :ref:`tdl-steps-common-handlerTimeouts`.
    @hidden~ no~ A boolean flag determining whether or not the step is displayed to users (default is "false"). Note that within scriptlets this can also be a :ref:`variable reference<scriptlets_dynamic_references>`. See :ref:`tdl-steps-common-hidesteps` for further details.
    @id~ no~ The ID for the step. This is also the name of a ``boolean`` variable in the session context in which the validation result will be recorded ("true" for success).
    @invert~ no~ A boolean flag determining whether the step's result should be inverted (default is "false"). Setting to "true" will expect a validation failure to complete the step as a success.
    @level~ no~ The severity level to be considered when this step fails validation. Can be set to ``ERROR`` (the default) or ``WARNING``, or be defined dynamically via :ref:`variable reference<test-case-referring-to-variables>`. See :ref:`tdl-steps-common-level` for further details.
    @output~ no~ A string value determining the name of the variable to be set with the output of the step (if any). If this is not set the output is displayed but is not recorded in the test session context.
    @skipped~ no~ A boolean value or variable reference (default being "false") which will result in the step being skipped if "true". See also :ref:`tdl-steps-common-skipped`.
    @stopOnError~ no~ A boolean flag determining whether the test session should end if this step fails (default is "false"). See also :ref:`tdl-steps-common-stoponerror`.
    config~ no~ Zero or more elements to provide configuration for the validation. Each ``config`` element has a ``name`` attribute and a text content or variable reference as value.
    documentation~ no~ Rich text content that provides further information on the current step.
    input~ yes~ One more elements for the validation's input parameters. See :ref:`handlers-inputs-outputs` for details.
    property~ no~ Zero or more elements to provide configuration regarding the setup of the validation handler call that are not passed to the handler. Each ``property`` element has a ``name`` attribute and a text content or variable reference as value.

A ``verify`` step that is set at warning level (through attribute ``level``) will never result in an overall failure for the test session. If validation fails,
the result will be indicated as a warning but without further impact. Note that a validation service returning a detailed validation report for a ``verify`` step 
at warning level may have its resulting report adapted accordingly. The report will be set as ``WARNING`` (if it was ``FAILURE``) and any error-level report 
items will be listed as warnings.

The following example includes three ``verify`` steps, the first one using an :ref:`handlers-XmlValidator`, followed by a second one at warning level which uses a remote
validation service. The third ``verify`` step replicates the previous one but defines its level dynamically:

.. code-block:: xml

    <!-- 
        Validation using the embedded XmlValidator.
    -->
    <verify handler="XmlValidator" desc="Validate invoice against schema">
        <input name="xml">$document</input>
        <input name="xsd">$schema"</input>
    </verify>
    <!-- 
        Warning-level validation using a remote validation service.
    -->
    <verify handler="https://VALIDATION_SERVICE_ADDRESS?wsdl" level="WARNING" desc="Validate against remote service">
        <input name="aDocument">$document</input>
    </verify>
    <!-- 
        Validation using a remote validation service with a dynamically set severity level.
    -->
    <assign to="levelToUse">'WARNING'</assign>
    <verify handler="https://VALIDATION_SERVICE_ADDRESS?wsdl" level="$levelToUse" desc="Validate against remote service">
        <input name="aDocument">$document</input>
    </verify>
    <!-- 
        Validation using the embedded XmlValidator and invert the result (i.e. succeed the step if validation fails).
    -->
    <verify handler="XmlValidator" desc="Validate bad content against schema" invert="true">
        <input name="xml">$expectedBadDocument</input>
        <input name="xsd">$schema"</input>
    </verify>

.. note::
    **Remote or local validators:** Simple validations such as those evaluating an XPath expression against a document can be implemented using 
    :ref:`handlers-predefined-validation-handlers`. When validation logic however is complex it is always best to decouple this into an external validation service. 
    This is the case even when validating XML content since this usually involves multiple validation steps using an XSD and one or more Schematron files. It is more
    concise to present this as a single validation step with one report. This also enhances maintainability of the test cases considering that use of the embedded
    :ref:`handlers-XSDValidator` and :ref:`handlers-SchematronValidator` means that you need to bundle (and maintain) the validation artefacts in each test suite. 
    When decoupled as a service artefacts can be updated without needing new test suite versions aside from the benefit that your service can also be invoked 
    outside the Test Bed using any SOAP client.

It may be the case that the ``verify`` step also produces output that needs to be leveraged further on in the test session. This could be interesting in case an 
:ref:`embedded validation handler<handlers-predefined-validation-handlers>` is used, the inputs of which are determined dynamically via an expression. Usually 
however you would want to record output if validation is done via a custom service which, apart from returning a validation report, calculates and returns
additional information. As an example consider a validator that checks the integrity of a provided file and also returns its hash code which is used in further
processing. Recording a ``verify`` step's output is done by means of the ``output`` attribute which defines the name of the variable to set. Once validation
completes, this variable will be set to anything returned as the `validation report context`_.

.. code-block:: xml

    <!-- 
        Validate and return as the report's context a map containing data with the key "identifier".
        The map is recorded in the session context under "validationOutput".
    -->
    <verify output="validationOutput">
        ...
    </verify>
    <log>$validationOutput{identifier}</log>

If no ``output`` attribute is set, the context data from the step's report will be displayed but not recorded in the session context.

.. _tdl-steps-common:

Common step concepts 
--------------------

The following section documents common concepts that apply to all test steps.
