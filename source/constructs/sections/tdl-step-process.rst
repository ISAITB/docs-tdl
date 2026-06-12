The ``process`` step is where the actual processing work takes place. This may be defined within the context of a
processing transaction started by a ``bptxn`` step, the ID of which is referenced. Alternatively, if a transaction 
is not required by the underlying processing handler, the transaction ID reference can be skipped and the handler
can be defined on the ``process`` step itself (see also :ref:`tdl-step-bptxn` for additional details).

The structure of the ``process`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"
    :delim: |

    @actor | no | The identifier of an :ref:`actor <test-case-actors>` under which to display the step (if visible). See also :ref:`tdl-steps-common-step-actor`.
    @desc | no | A description for this step to display to the user (meaningful if ``hidden`` is "false") and to include in the test session log. Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.
    @handler | no | A string value or variable reference identifying the processing handler for this step (see :ref:`handlers-implementation`). This is omitted in favour of the ``txnId`` in case a transaction is referenced.
    @handlerTimeout | no | A number or variable reference with the maximum time (in milliseconds) to wait for the handler service call to complete (in case of an external test service being used as a handler). See also :ref:`tdl-steps-common-handlerTimeouts`.
    @handlerTimeoutFlag | no | A string value with the name of a boolean variable to set informing whether or not a handler timeout occurred. See also :ref:`tdl-steps-common-handlerTimeouts`.
    @hidden | no | A boolean flag determining whether or not the step is displayed to users (default is "true"). Note that within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`. See :ref:`tdl-steps-common-hidesteps` for further details.
    @id | no | The ID for the step. This is also the name of a ``map`` variable in the session context in which output will be stored.
    @input | no | An alternative to input elements to provide a single input when the processing handler expects a single input or (if multiple) a single mandatory input. See also :ref:`tdl-step-process__simplified`.
    @invert | no | A boolean flag determining whether the step's result should be inverted (default is "false"). Setting to "true" will expect a processing failure to complete the step as a success.
    @level | no | The severity level to be considered when this step fails. Can be set to ``ERROR`` (the default) or ``WARNING``, or be defined dynamically via :ref:`variable reference<test-case-referring-to-variables>`. See :ref:`tdl-steps-common-level` for further details.
    @operation | no | An alternative to the operation element providing the operation to carry out by the processing handler. See also :ref:`tdl-step-process__simplified`.
    @output | no | The name to use for the session context variable to store the processing output as an alternative to using the ``id``. See also :ref:`tdl-step-process__simplified`.
    @skipped | no | A boolean value or variable reference (default being "false") which will result in the step being skipped if "true". See also :ref:`tdl-steps-common-skipped`.
    @stopOnError | no | A boolean flag determining whether the test session should end if this step fails (default is "false"). See also :ref:`tdl-steps-common-stoponerror`.
    @txnId | no | The ID of the transaction to which this processing step belongs. Can be omitted if a transaction is not needed but in this case the ``handler`` attribute must be defined.
    documentation | no | Rich text content that provides further information on the current step (meaningful if ``hidden`` is "false").
    input | no | Zero or more elements for the input parameters to the processing step. See :ref:`handlers-inputs-outputs` for details.
    operation | no | An optional ``string`` to identify an operation the handler is expected to perform.
    property | no | Zero or more elements to provide configuration regarding the setup of the processing handler call that are not passed to the handler. Each ``property`` element has a ``name`` attribute and a text content or variable reference as value.

Setting an ``operation`` is relevant for processing handlers that can support more than one task. Use of multiple operations under
the same transaction renders processing services quite powerful in that they can perform any number of related operations
and be extended with additional ones if needed. The operation to perform can be provided either via child element or attribute. If both
are provided, the child element takes precedence.

The output of processing steps can be leveraged in two ways:

    * If the ``output`` attribute is defined, its value is used to name the variable in which the results are stored. If multiple results
      are produced this will be a ``map``, but for a single result this will be directly recorded.
    * If there is no ``output`` attribute, the step's ``id`` is used instead. Its value will be used as the name of a ``map`` that will
      include all outputs, using the names defined by the processing handler.

Using the ``output`` attribute is meant as a simplification when doing simple processing. It allows you to control the resulting variable's
name which could be interesting if you need it as part of :ref:`template processing<test-case-expressions-template-files>` when replacing
similarly named placeholders. For further ways to simplify basic processing steps see :ref:`tdl-step-process__simplified`.

.. _tdl-step-process__transactions:

Processing transactions
+++++++++++++++++++++++

For a processing handler that retains state, carrying out operations in a transaction is important as it provides an opportunity to manage
correctly its resources. Moreover, for processing handlers supporting more than one operation for the same data, a transaction provides
much needed context to logically connect operations. As an example consider a processing service that is used to read the 
contents from a ZIP archive. If the test case needs to read multiple files at different points in its execution it would be 
possible but very inefficient to pass the ZIP archive in each call. Defining a transaction allows the test case to pass the 
archive once allowing the processing handler to cache it and ultimately remove it upon transaction end. In addition, the 
presence of a transaction provides context and makes operations such as "initialize" (to pass the archive to consider),
"extract" (to get a file's contents), "checkExistence" (to check if a file exists but not return it) possible. Use of such a 
transaction-aware processing service is illustrated in the following example:

.. code-block:: xml

    <!--
        Create a processing transaction named "t1".
    -->
    <bptxn txnId="t1" handler="https://ZIP_PROCESSING_SERVICE?wsdl"/>
    <!-- 
        Call the "initialize" operation to pass the archive to the service.
        The service handler can read and cache the archive for the transaction.
    -->
    <process id="init" txnId="t1">
        <operation>initialize</operation>
        <input name="zip">$zipContent</input>
    </process>
    <!-- 
        Call the "checkExistence" operation to see if a given entry exists.
    -->
    <process id="exists" txnId="t1">
        <operation>checkExistence</operation>
        <input name="path">'file1.xml'</input>
    </process>
    <!-- 
        Call the "extract" operation to get an entry.
    -->
    <process id="output" txnId="t1">
        <operation>extract</operation>
        <input name="path">'file1.xml'</input>
    </process>
    <!--
        End the transaction.
        The service handler can remove the archive.
    -->
    <eptxn txnId="t1"/>

For cases where processing operations are simple, one-off actions, defining a transaction results in superfluous 
and unnecessary test steps. A good example of such a case is the :ref:`handlers-TokenGenerator` embedded processing handler
that is used to generate text tokens such as a random UUID. In this case, although possible, defining a processing transaction
is not needed, and is skipped in favour of simplification. In this case however, the ``handler`` attribute must be defined
on the ``process`` step itself (replacing the ``txnId`` reference) as illustrated in the following example:

.. code-block:: xml

    <!--
        Generate a UUID. The handler is defined without referencing a transaction ID.
    -->
    <process id="uuid" handler="TokenGenerator">
        <operation>uuid</operation>
    </process>
    <!--
        Display to the user the generated UUID.
    -->
    <interact desc="Generated UUID">
        <instruct desc="Value:">$uuid{value}</instruct>
    </interact>

.. _tdl-step-process__visibility:

Process step visibility
+++++++++++++++++++++++

The ``process`` step is by default considered to be internal and not meaningful to present to users. You could nonetheless choose to include the
step in the test session presentation by setting its ``hidden`` attribute to "false" (the default value is "true" for ``process`` steps). An 
example case where this could be useful is when you use a :ref:`custom processing service<handlers>` to transform content between syntaxes. Making
the ``process`` step visible could serve to better inform users of the conversion process and its output. In addition, keep in mind that when
presenting the step you should also consider providing a **description** (via the ``desc`` attribute) and additional **documentation** (via the 
``documentation`` element).

The following TDL snippet illustrates setting this information for a custom processing step:

.. code-block:: xml

    <!--
        Setting "hidden" to false makes this step visible.
    -->
    <process id="conversion" hidden="false" desc="Convert input to syntax B" handler="$DOMAIN{conversionServiceAddress}">
        <documentation import="docs/conversionDoc.html"/>
        <operation>convert</operation>
        <input name="input">$inputContentSyntaxA</input>
    </process>

A ``process`` step that is displayed will present its overall result and additional information linked to the processing. Regarding
this additional information:

* In the case of :ref:`embedded processing handlers<handlers-predefined-handlers-processing>` the step's visible output will be any
  output values produced by the processing.
* In the case of :ref:`custom processing handlers<handlers>` the visible output will be what is set as context on the step's report
  (which can replicate or differ from the actual outputs).

.. note::
    **Hidden steps:** The ``hidden`` attribute is supported for all steps that can be presented to users. The ``process`` step however is the
    only case where the default value is assumed to be "true". For further information on the steps' ``hidden`` attribute check the  
    :ref:`tdl-steps-common-hidesteps` section.

.. _tdl-step-process__simplified:

Simplified processing steps
+++++++++++++++++++++++++++

Test cases often include basic processing steps as utilities that don't need transactions and multiple inputs, or produce only single
output values. To reduce the verbosity of the ``process`` step in such cases, you can make use of three syntax alternatives:

    * The ``input`` attribute to provide a single input.
    * The ``operation`` attribute to define the operation.
    * The ``output`` attribute to directly name the result rather than use an intermediate ``map``.

In case the ``process`` step's handler expects multiple parameters, the single ``input`` attribute is assigned to a parameter as follows
(rules listed with decreasing priority):

    #. The first mandatory parameter matching the input's type.
    #. The first mandatory parameter regardless of type.
    #. The first optional parameter matching the input's type.
    #. The first optional parameter regardless of type.
    #. An unnamed parameter set to the input's value.

.. note::
    To avoid ambiguity, use of the simplified ``process`` syntax should be preferred when a single input is expected, or in case of multiple expected
    inputs, there is one mandatory one.

In the case of inputs and operations, defining them both as attributes and child elements is superfluous. If nonetheless both are defined,
the child elements take precedence.

The following example illustrates how these alternatives can be used to simplify your test definitions. We consider here that we are 
generating two messages based on a template that includes a placeholder for an identifier (named "messageId"). For the first message
we use a verbose syntax whereas for the second one we use the simplifications discussed here. In both cases the :ref:`handlers-TokenGenerator`
is used to generate UUIDs as alphanumeric strings with a length of ten characters.

.. code-block:: xml

    <!--
        Verbose approach.
    -->
    <process id="tokenStep" handler="TokenGenerator">
        <operation>string</operation>
        <input name="format">"[a-zA-Z\d]{10}"</input>
    </process>
    <!-- 
        The output is stored in a map named using the step's id. As the template defines
        a "messageId" placeholder we need to create such a variable from the result map.
    -->
    <assign to="messageId">$tokenStep{value}</assign>
    <assign to="message1" asTemplate="true">$messageTemplate</assign>

    <!--
        Simplified approach.
    -->
    <process output="messageId" handler="TokenGenerator" input="[a-zA-Z\d]{10}" operation="string"/>
    <assign to="message2" asTemplate="true">$messageTemplate</assign>

.. note::
    The :ref:`tdl-step-call` step also offers :ref:`similar syntax simplifications<tdl-step-call__simplified>`. This simplified
    syntax is available for :ref:`tdl-step-process` and :ref:`tdl-step-call` steps as these typically represent utilities that are
    frequently used.

.. index:: Flow steps

Flow steps
----------

Flow steps are used to control the processing flow of a test case. The constructs available are similar to the
flow control structures available in programming languages.
