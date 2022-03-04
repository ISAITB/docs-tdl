.. _tdl-steps:

TDL step constructs
===================

Overview
--------

TDL step constructs are used to capture a test case's core testing logic. They are used in test cases and
also in :ref:`test-case-scriptlets` to define their sequence of test steps. The available test
steps are described in the sections that follow, organised in four main categories:

* **Messaging steps** used to exchange information between actors.
* **Processing steps** to perform complex arbitrary processing.
* **Flow steps** to manage the execution flow of the test case.
* **Support steps** to introduce support features to test cases.

.. index:: Messaging steps
.. _tdl-messaging-steps:

Messaging steps
---------------

Messaging steps allow the test case to handle the exchange of messages between actors. The actual implementation
allowing content to be sent or received is implemented by a messaging handler (see :ref:`introduction-concepts-messaging-handlers`).

.. index:: btxn
.. index:: txnid (btxn)
.. index:: to (btxn)
.. index:: handler (btxn)
.. index:: stopOnError (btxn)
.. index:: property (btxn)
.. index:: config (btxn)
.. _tdl-step-btxn:

btxn
~~~~

The ``btxn`` step stands for "Begin transaction". Its purpose is to define a scope around a set of messaging
steps that have a logical relation to each other. This scope remains active until a ``etxn`` element is
encountered to end it. The structure of the ``btxn`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @txnid, yes, A string ID for the transaction.
    @from, yes, The ID of the actor that acts as the messaging source (see :ref:`test-case-actors`).
    @to, yes, The ID of the actor that acts as the messaging target (see :ref:`test-case-actors`).
    @handler, yes, A string value or variable reference identifying the messaging handler to use for the transaction (see :ref:`handlers-implementation`).
    @stopOnError, no, A boolean flag determining whether the test session should end if this step fails (default is ``false``). See also :ref:`tdl-steps-common-stoponerror`.
    property, no, Zero or more elements to provide configuration regarding the setup of the messaging handler call that are not passed to the handler. Each ``property`` element has a ``name`` attribute and a text content or variable reference as value.
    config, no, Zero or more elements to provide configuration when creating the transaction. Each ``config`` element has a ``name`` attribute and a text content or variable reference as value.

Executing the ``btxn`` step results in a call to the messaging handler specified by the ``handler`` attribute. This gives it an 
opportunity to take any actions needed for the upcoming transaction and apply specific configurations for its related ``send``
and ``receive`` calls.

.. code-block:: xml
    :emphasize-lines: 1

    <btxn from="Actor1" to="Actor2" txnId="t1" handler="SoapMessaging"/>
    <send id="dataSend" desc="Send data" from="Actor1" to="Actor2" txnId="t1">
        <input name="soap_message">$soapMessage</input>
    </send>
    <receive id="dataReceive" desc="Receive data" from="Actor2" to="Actor1" txnId="t1"/>
    <etxn txnId="t1"/>

Note that ``btxn`` steps are not presented to the user.

.. index:: etxn
.. index:: txnid (etxn)
.. index:: stopOnError (etxn)
.. _tdl-step-etxn:

etxn
~~~~

The ``etxn`` step stands for "End transaction" and acts as the counterpart to a ``btxn`` element by referencing its transaction
ID. It is structured as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @txnid, yes, The identifier of the transaction to end.
    @stopOnError, no, A boolean flag determining whether the test session should end if this step fails (default is ``false``). See also :ref:`tdl-steps-common-stoponerror`.

Executing the ``etxn`` results in a call to the transaction's messaging handler to take necessary actions such as resource clean-up.

.. code-block:: xml
    :emphasize-lines: 7

    <btxn from="Actor1" to="Actor2" txnId="t1" handler="SoapMessaging"/>
    <send id="dataSend" desc="Send data" from="Actor1" to="Actor2" txnId="t1">
        <config name="soap.version">1.2</config>
        <input name="soap_message">$soapMessage</input>
    </send>
    <receive id="dataReceive" desc="Receive data" from="Actor2" to="Actor1" txnId="t1"/>
    <etxn txnId="t1"/>

Note that ``etxn`` steps are not presented to the user.

.. index:: send
.. index:: txnid (send)
.. index:: from (send)
.. index:: to (send)
.. index:: desc (send)
.. index:: id (send)
.. index:: stopOnError (send)
.. index:: documentation (send)
.. index:: config (send)
.. index:: input (send)
.. index:: hidden (send)
.. index:: reply (send)
.. _tdl-step-send:

send
~~~~

The ``send`` step allows the test bed to signal that content needs to be sent from one actor to another. This operation needs to be
part of a transaction created by ``btxn``, the identifier of which it references. The structure of the ``send`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @txnid, yes, The ID of the transaction this ``send`` belongs to.
    @from, yes, The ID of the actor that will be sending the message (see :ref:`test-case-actors`).
    @to, yes, The ID of the actor that will be receiving the message (see :ref:`test-case-actors`).
    @desc, no, A description to display to the user for this test step.
    @id, no, The ID for the step. This is also the name of a ``map`` variable in the session context in which output will be stored.
    @stopOnError, no, A boolean flag determining whether the test session should end if this step fails (default is ``false``). See also :ref:`tdl-steps-common-stoponerror`.
    @hidden, no, A boolean flag determining whether or not the step is displayed to users (default is ``false``). See also :ref:`tdl-steps-common-hidesteps`.
    @reply, no, A boolean flag indicating that this communication should be presented as a reply.
    documentation, no, Rich text content that provides further information on the current step.
    config, no, Zero or more elements containing configuration values pertinent to sending.  Each ``config`` element has a ``name`` attribute and a text content or variable reference as value.
    input, no, Zero or more elements for the input parameters. See :ref:`handlers-inputs-outputs` for details.

The ``send`` step results in the transaction's messaging handler to be notified that it needs to send content. Recall that the actual
sending always takes place through the message handler implementation. The ``send`` step simply acts as the signal to do so.

.. code-block:: xml
    :emphasize-lines: 2,3,4,5

    <btxn from="Actor1" to="Actor2" txnId="t1" handler="SoapMessaging"/>
    <send id="dataSend" desc="Send data" from="Actor1" to="Actor2" txnId="t1">
        <config name="soap.version">1.2</config>
        <input name="soap_message">$soapMessage</input>
    </send>
    <etxn txnId="t1"/>

.. index:: receive
.. index:: txnid (receive)
.. index:: from (receive)
.. index:: to (receive)
.. index:: desc (receive)
.. index:: id (receive)
.. index:: timeout (receive)
.. index:: timeoutFlag (receive)
.. index:: timeoutIsError (receive)
.. index:: stopOnError (receive)
.. index:: documentation (receive)
.. index:: config (receive)
.. index:: input (receive)
.. index:: output (receive)
.. index:: hidden (receive)
.. index:: reply (receive)
.. _tdl-step-receive:

receive
~~~~~~~

The ``receive`` step is the counterpart of ``send`` signalling that an actor is expected to receive a message from another. This 
operation needs to be defined as part of a transaction created by ``btxn``, the identifier of which it references. The structure 
of the ``receive`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @txnid, yes, The ID of the transaction this ``receive`` belongs to.
    @from, yes, The ID of the actor that will be sending the message (see :ref:`test-case-actors`).
    @to, yes, The ID of the actor that will be receiving the message (see :ref:`test-case-actors`).
    @desc, no, A description to display to the user for this test step.
    @id, no, The ID for the step. This is also the name of a ``map`` variable in the session context in which output will be stored.
    @timeout, no, An optional timeout (in milliseconds) on the time to wait for a message to be received. This is provided as a ``number`` or a variable reference.
    @timeoutFlag, no, An optional name for a boolean flag to record whether or not the timeout was triggered that will be stored in the result ``map`` named using the ``id`` attribute. This is provided as a ``string`` or a variable reference.
    @timeoutIsError, no, Whether or not a timeout being triggered should be considered as an error or success (the default). This is provided as a ``boolean`` or a variable reference.
    @stopOnError, no, A boolean flag determining whether the test session should end if this step fails (default is ``false``). See also :ref:`tdl-steps-common-stoponerror`.
    @hidden, no, A boolean flag determining whether or not the step is displayed to users (default is ``false``). See also :ref:`tdl-steps-common-hidesteps`.
    @reply, no, A boolean flag indicating that this communication should be presented as a reply.
    documentation, no, Rich text content that provides further information on the current step.
    config, no, Zero or more elements containing configuration values pertinent to receiving.  Each ``config`` element has a ``name`` attribute and a text content or variable reference as value.
    input, no, Zero or more elements for the signal's input parameters. See :ref:`handlers-inputs-outputs` for details.
    output, no, Zero or more elements for the resulting output values. See :ref:`handlers-inputs-outputs` for details.

When the test bed executes the ``receive`` step it performs two actions:

#. It signals the transaction's messaging handler that content is expected to be received.
#. It blocks waiting for a call-back from the messaging handler that will contain the received data, or until the configured timeout has elapsed.

Regarding the ``input`` elements provided these act as information provided to the messaging handler that are relevant to the
message's reception. They act as a counterpart to ``config`` elements whose purpose is more to signal parameters for the communication
setup rather than the involved message. The ``output`` elements provided are optional and serve only to restrict the messaging handler's
output (returned via its call-back to the test bed) to the specified values. If not specified all available output values are returned.

.. code-block:: xml
    :emphasize-lines: 2,3,4,8,9,10,18

    <btxn from="Actor1" to="Actor2" txnId="t1" handler="SoapMessaging"/>
    <receive id="dataReceive" desc="Receive data" from="Actor2" to="Actor1" txnId="t1">
        <config name="soap.version">1.2</config>
    </receive>
    <!--
        Example using timeouts (that are considered as an error).
    -->
    <receive 
      id="dataReceiveTimeout" desc="Receive data with timeout" from="Actor2" to="Actor1" txnId="t1"
      timeout="$maxWaitTime" timeoutFlag="timeoutOccurred" timeoutIsError="true">
        <config name="soap.version">1.2</config>
    </receive>
    <etxn txnId="t1"/>
    <!--
        Check to see if timeout took place or not and inform the user.
    -->
    <interact desc="Check timeout status">
        <instruct desc="Timeout occurred:" with="Actor2">$dataReceiveTimeout{timeoutOccurred}</instruct>
    </interact>

.. note::
    **Parallel receives:** In case you use the ``receive`` step within a :ref:`flow<tdl-step-flow>` step's threads and a
    :ref:`custom messaging service<handlers>`, you need to make sure your service manages the specific receive call's identifier.
    Check the `messaging service documentation`_ for details on how to do this.

.. index:: listen
.. index:: txnid (listen)
.. index:: from (listen)
.. index:: to (listen)
.. index:: id (listen)
.. index:: stopOnError (listen)
.. index:: documentation (listen)
.. index:: config (listen)
.. index:: input (listen)
.. index:: output (listen)
.. index:: hidden (listen)
.. index:: reply (listen)
.. _tdl-step-listen:

listen
~~~~~~

The ``listen`` step is used to instruct the test bed to act as a proxy between messages sent to and from two actors defined as SUTs. 
Similar to the ``send`` and ``receive`` steps, this step is expected to take place within a transaction created by ``btxn``, the 
identifier of which it references. The structure of the ``listen`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @txnid, yes, The ID of the transaction this ``listen`` belongs to.
    @from, yes, The ID of the actor that will be sending the message (see :ref:`test-case-actors`).
    @to, yes, The ID of the actor that will be receiving the message (see :ref:`test-case-actors`).
    @id, no, The ID for the step. This is also the name of a ``map`` variable in the session context in which output will be stored.
    @stopOnError, no, A boolean flag determining whether the test session should end if this step fails (default is ``false``). See also :ref:`tdl-steps-common-stoponerror`.
    @hidden, no, A boolean flag determining whether or not the step is displayed to users (default is ``false``). See also :ref:`tdl-steps-common-hidesteps`.
    @reply, no, A boolean flag indicating that this communication should be presented as a reply.
    documentation, no, Rich text content that provides further information on the current step.
    config, no, Zero or more elements containing configuration values pertinent to the message exchange.  Each ``config`` element has a ``name`` attribute and a text content or variable reference as value.
    input, no, Zero or more elements for for the messaging handler to consider. See :ref:`handlers-inputs-outputs` for details.
    output, no, Zero or more elements for the output values reported back to the test case. See :ref:`handlers-inputs-outputs` for details.

.. note::
    **GITB software support:** The ``listen`` step is currently not supported. As a general note, 
    interoperability tests involving multiple actors as SUTs are not currently possible.

.. index:: Processing steps
.. _tdl-processing-steps:

Processing steps
----------------

Processing steps are used to handle complex manipulations on information in the test session context that are domain-specific
or too elaborate to be implemented using simple constructs such as the :ref:`tdl-step-assign` step. The actual implementation
that carries out operations is implemented by a processing handler (see :ref:`introduction-concepts-processing-handlers`).

Note that processing steps are not presented to the user.

.. index:: bptxn
.. index:: txnid (bptxn)
.. index:: handler (bptxn)
.. index:: stopOnError (bptxn)
.. index:: property (bptxn)
.. index:: config (bptxn)
.. _tdl-step-bptxn:

bptxn
~~~~~

Similar to :ref:`tdl-messaging-steps`, processing occurs in the context of a transaction that acts as a grouping mechanism
over related operations. The ``bptxn`` step (the name stands for "Begin processing transaction") is the construct used to
signal that a processing transaction should be considered as started and is assigned an identifier. Subsequent relevant 
operations will be accompanied by this transaction ID to allow their processing handler to carry them out accordingly.

Use of a processing transaction is not always required. For processing steps that are simple in nature and don't require
state to be maintained across calls, you may skip the definition of a transaction and simply refer to the processing handler
from the ``process`` step itself (see :ref:`tdl-step-process` for details). Whether or not skipping a transaction's definition is 
supported depends on the specific processing handler; typically however, even if a processing handler doesn't require a transaction
and is signalled to create one this will simply be ignored. In terms of whether you need or not to define a processing transaction 
you can consider this rule of thumb:

* **Transaction needed:** When the processing handler is expected to maintain state across individual ``process`` calls and eventually 
  perform some clean up operations.
* **Transaction not needed:** When the processing handler is stateless.

The structure of the ``bptxn`` element (defined when a processing transaction is needed) is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @txnid, yes, A string identifier for the transaction.
    @handler, yes, A string value or variable reference identifying the the processing handler for the transaction (see :ref:`handlers-implementation`).
    @stopOnError, no, A boolean flag determining whether the test session should end if this step fails (default is ``false``). See also :ref:`tdl-steps-common-stoponerror`.
    property, no, Zero or more elements to provide configuration regarding the setup of the processing handler call that are not passed to the handler. Each ``property`` element has a ``name`` attribute and a text content or variable reference as value.
    config, no, Zero or more elements to provide configuration when creating the transaction. Each ``config`` element has a ``name`` attribute and a text content or variable reference as value.

The ``bptxn`` step results in a call to the configured processing handler to signal that a new transaction is going to 
start.

.. code-block:: xml
    :emphasize-lines: 1

    <bptxn txnId="t1" handler="https://PROCESSING_SERVICE?wsdl"/>
    <process id="result" txnId="t1">
        <operation>action</operation>
        <input name="anInput">$aValue</input>
    </process>
    <eptxn txnId="t1"/>

.. index:: eptxn
.. index:: txnid (eptxn)
.. index:: stopOnError (eptxn)
.. _tdl-step-eptxn:

eptxn
~~~~~

The ``eptxn`` step (the name stands for "End processing transaction") is the counterpath of the ``bptxn`` step and is used to
close a transaction the ID of which it references. The structure of the ``eptxn`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @txnid, yes, A string identifier for the processing transaction to end.
    @stopOnError, no, A boolean flag determining whether the test session should end if this step fails (default is ``false``). See also :ref:`tdl-steps-common-stoponerror`.

The ``eptxn`` step results in a call to the transaction's processing handler to signal that it should consider the transaction as
completed and proceed with any needed actions such as resource clean-up.

.. code-block:: xml
    :emphasize-lines: 6

    <bptxn txnId="t1" handler="https://PROCESSING_SERVICE?wsdl"/>
    <process id="result" txnId="t1">
        <operation>action</operation>
        <input name="anInput">$aValue</input>
    </process>
    <eptxn txnId="t1"/>

.. index:: process
.. index:: txnid (process)
.. index:: id (process)
.. index:: desc (process)
.. index:: handler (process)
.. index:: stopOnError (process)
.. index:: documentation (process)
.. index:: operation (process)
.. index:: input (process)
.. index:: output (process)
.. index:: hidden (process)
.. _tdl-step-process:

process
~~~~~~~

The ``process`` step is where the actual processing work takes place. This may be defined within the context of a
processing transaction started by a ``bptxn`` step, the ID of which is referenced. Alternatively, if a transaction 
is not required by the underlying processing handler, the transaction ID reference can be skipped and the handler
can be defined on the ``process`` step itself (see also :ref:`tdl-step-bptxn` for additional details).

The structure of the ``process`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @txnId, no, The ID of the transaction to which this processing step belongs. Can be omitted if a transaction is not needed but in this case the ``handler`` attribute must be defined.
    @id, no, The ID for the step. This is also the name of a ``map`` variable in the session context in which output will be stored.
    @desc, no, A description to display to the user on the purpose of the check (meaningful if ``hidden`` is ``false``).
    @handler, no, A string value or variable reference identifying the processing handler for this step (see :ref:`handlers-implementation`). This is omitted in favour of the ``txnId`` in case a transaction is referenced.
    @stopOnError, no, A boolean flag determining whether the test session should end if this step fails (default is ``false``). See also :ref:`tdl-steps-common-stoponerror`.
    @hidden, no, A boolean flag determining whether or not the step is displayed to users (default is ``true``).
    @input, no, An alternative to input elements to provide a single input when the processing handler expects a single input or (if multiple) a single mandatory input. See also :ref:`tdl-step-process__simplified`.
    @operation, no, An alternative to the operation element providing the operation to carry out by the processing handler. See also :ref:`tdl-step-process__simplified`.
    @output, no, The name to use for the session context variable to store the processing output as an alternative to using the ``id``. See also :ref:`tdl-step-process__simplified`.
    documentation, no, Rich text content that provides further information on the current step (meaningful if ``hidden`` is ``false``).
    operation, no, An optional ``string`` to identify an operation the handler is expected to perform.
    input, no, Zero or more elements for the input parameters to the processing step. See :ref:`handlers-inputs-outputs` for details.

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
step in the test session presentation by setting its ``hidden`` attribute to ``false`` (the default value is ``true`` for ``process`` steps). An 
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
    only case where the default value is assumed to be ``true``. For further information on the steps' ``hidden`` attribute check the  
    :ref:`tdl-steps-common-hidesteps` section.

.. _tdl-step-process__simplified:

Simplified processing steps
+++++++++++++++++++++++++++

Test cases often include basic processing steps as utilities that don't need transactions and multiple inputs, or produce only single
output values. To reduce the verbosity of the ``process`` step in such cases, you can make use of three syntax alternatives:

    * The ``input`` attribute to provide a single input. This is possible when a single input is expected, or in case of multiple expected
      inputs, there is one mandatory one.
    * The ``operation`` attribute to define the operation.
    * The ``output`` attribute to directly name the result rather than use an intermediate ``map``.

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

.. index:: if
.. index:: title (if)
.. index:: desc (if)
.. index:: stopOnError (if)
.. index:: documentation (if)
.. index:: cond (if)
.. index:: then (if)
.. index:: else (if)
.. index:: hidden (if)
.. index:: collapsed (if)
.. _tdl-step-if:

if
~~

The ``if`` step is used to run one of more steps if a condition is met. Its structure is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @title, no, A short title to display for this step (default is "decision").
    @desc, no, A description to display to the user on the purpose of the check.
    @stopOnError, no, A boolean flag determining whether the test session should end if this step fails (default is ``false``). See also :ref:`tdl-steps-common-stoponerror`.
    @hidden, no, A boolean flag determining whether or not the step is displayed to users (default is ``false``). See also :ref:`tdl-steps-common-hidesteps`.
    @collapsed, no, A boolean flag determining whether or not the step is displayed as initially collapsed (default is ``false``). See also :ref:`tdl-steps-common-collapsed`.
    documentation, no, Rich text content that provides further information on the current step.
    cond, yes, The condition to verify in order to execute the ``then`` set of steps (if true) or ``else`` (if false). This is provided as an expression (see :ref:`test-case-expressions`).
    then, yes, Contains as children any sequence of steps to execute if the condition results to true.
    else, no, Contains as children any sequence of steps to execute if the condition results to false.

.. code-block:: xml

    <if desc="Check process type">
        <cond>$processType = 'process1'</cond>
        <then>
            <assign to="$formatType">'XML'</assign>
            <verify handler="https://VALIDATOR?wsdl" desc="Validate as XML">
                <input name="source" source="$document"/>
                <input name="validationType">$formatType</input>
            </verify>
        </then>
        <else>
            <assign to="$formatType">'CSV'</assign>
        </else>
    </if>

.. index:: while
.. index:: title (while)
.. index:: desc (while)
.. index:: stopOnError (while)
.. index:: documentation (while)
.. index:: cond (while)
.. index:: do (while)
.. index:: hidden (while)
.. index:: collapsed (while)
.. _tdl-step-while:

while
~~~~~

The ``while`` step is the most useful looping construct. It allows a sequence of steps to be continuously executed as long as a condition
continues to be true. The structure of the ``while`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @title, no, A short title to display for this step (default is "loop").
    @desc, no, A description to display to the user on the purpose of the loop.
    @stopOnError, no, A boolean flag determining whether the test session should end if this step fails (default is ``false``). See also :ref:`tdl-steps-common-stoponerror`.
    @hidden, no, A boolean flag determining whether or not the step is displayed to users (default is ``false``). See also :ref:`tdl-steps-common-hidesteps`.
    @collapsed, no, A boolean flag determining whether or not the step is displayed as initially collapsed (default is ``false``). See also :ref:`tdl-steps-common-collapsed`.
    documentation, no, Rich text content that provides further information on the current step.
    cond, yes, The condition to verify in order to execute the contained steps. This is provided as an expression (see :ref:`test-case-expressions`).
    do, yes, Contains as children any sequence of steps to execute if the loop's condition results to true.

The following example validates the name of each attachment defined in an XML document using a ``while`` loop:

.. code-block:: xml

    <!--
        Initialise maximum iteration count based on the number of "Attachment" nodes in the document.
    -->
    <assign to="$iterationCount" source="$document">count(//*[local-name() = "Attachment"]</assign>
    <assign to="$iteration">1</assign>
    <while desc="Validate attachment names">
        <cond>$iteration &lt;= $iterationCount</cond>
        <do>
            <verify handler="XPathValidator" desc="The attachment is named as expected">
                <input name="xmldocument" source="$document"/>
                <!-- 
                    Construct the XPath expression to apply using the iteration variable.
                -->
                <input name="xpathexpression">concat("//*[local-name() = 'Attachment'][", $iteration, "]/text() = 'file_", $iteration, ".xml'")</input>
            </verify>
            <!--
                Increment iteration counter.
            -->
            <assign to="$iteration">$iteration + 1</assign>
        </do>
    </while>

.. index:: repuntil
.. index:: title (repuntil)
.. index:: desc (repuntil)
.. index:: stopOnError (repuntil)
.. index:: documentation (repuntil)
.. index:: do (repuntil)
.. index:: cond (repuntil)
.. index:: hidden (repuntil)
.. index:: collapsed (repuntil)
.. _tdl-step-repuntil:

repuntil
~~~~~~~~

The `repuntil` step allows you to execute a sequence of steps at least once, checking at the end a condition to see if another iteration
should take place. The structure of the ``repuntil`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @title, no, A short title to display for this step (default is "loop").
    @desc, no, A description to display to the user on the purpose of the loop.
    @stopOnError, no, A boolean flag determining whether the test session should end if this step fails (default is ``false``). See also :ref:`tdl-steps-common-stoponerror`.
    @hidden, no, A boolean flag determining whether or not the step is displayed to users (default is ``false``). See also :ref:`tdl-steps-common-hidesteps`.
    @collapsed, no, A boolean flag determining whether or not the step is displayed as initially collapsed (default is ``false``). See also :ref:`tdl-steps-common-collapsed`.
    documentation, no, Rich text content that provides further information on the current step.
    do, yes, Contains as children any sequence of steps to execute at least once and then again if the condition in ``cond`` is true.
    cond, yes, The condition to verify in order to execute again the steps contained in ``do``. This is provided as an expression (see :ref:`test-case-expressions`).

.. code-block:: xml

    <assign to="$iteration">1</assign>
    <assign to="$maxIteration">3</assign>
    <repuntil desc="Do iteration">
        <do>
            <interact desc="Message to user" with="User">
                <instruct desc="Iteration: " with="User" type="string">concat($iteration, " of ", $maxIteration)</instruct>
            </interact>
            <assign to="$iteration">$iteration + 1</assign>
        </do>
        <cond>$iteration &lt;= $maxIteration</cond>
    </repuntil>

.. note::
    **Do-while:** Step ``repuntil`` stands for "repeat until". Considering this you could assume that the steps in ``do`` will be executed until
    the condition in ``cond`` is true. This is actually not the case currently as steps are executed while the condition in ``cond`` remains true
    (i.e. the logic is actually inversed). The naming of this step is thus unfortunate; it would be more appropriate if this was named ``dowhile``
    reflecting accurately how the condition is considered.

.. index:: foreach
.. index:: title (foreach)
.. index:: desc (foreach)
.. index:: start (foreach)
.. index:: end (foreach)
.. index:: counter (foreach)
.. index:: stopOnError (foreach)
.. index:: documentation (foreach)
.. index:: do (foreach)
.. index:: hidden (foreach)
.. index:: collapsed (foreach)
.. _tdl-step-foreach:

foreach
~~~~~~~

The ``foreach`` step allows you to execute a sequence of steps for a specific number of iterations. Its structure is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @title, no, A short title to display for this step (default is "loop").
    @desc, no, A description to display to the user on the purpose of the loop.
    @start, yes, A number to initialise the iteration index to. This is provided as a constant or as a variable reference.
    @end, yes, A number that is considered as the maximum iteration count plus 1. This is provided as a constant or as a variable reference.
    @counter, no, A name for the variable through which to expose the iteration counter (default is "i").
    @stopOnError, no, A boolean flag determining whether the test session should end if this step fails (default is ``false``). See also :ref:`tdl-steps-common-stoponerror`.
    @hidden, no, A boolean flag determining whether or not the step is displayed to users (default is ``false``). See also :ref:`tdl-steps-common-hidesteps`.
    @collapsed, no, A boolean flag determining whether or not the step is displayed as initially collapsed (default is ``false``). See also :ref:`tdl-steps-common-collapsed`.
    documentation, no, Rich text content that provides further information on the current step.
    do, yes, Contains as children any sequence of steps to execute for a loop iteration.

The ``start`` and ``end`` values define the number of iterations to perform. Specifically, the loop will continue as long as
``start`` is less than ``end`` with ``start`` getting incremented by one at the end of each iteration.

.. code-block:: xml

    <!-- 
        The loop will execute 2 times (start must be less than end). The currentIndex variable will be 5 in the first 
        iteration and then 6. Note that referring to this is done as a variable reference (if not specified the variable
        would be named "i" and referred to as "$i").
    -->
    <foreach desc="Do iteration" counter="currentIndex" start="5" end="7">
        <do>
            <interact desc="Message to user" with="User">
                <instruct desc="Iteration: " with="User" type="string">concat("Iteration ", $currentIndex)</instruct>
            </interact>
        </do>
    </foreach>
    <!-- In the following case the loop's boundaries are set dynamically. -->
    <assign to="$start">5</assign>
    <assign to="$end">$start + 2</assign>
    <foreach desc="Do iteration" counter="currentIndex" start="$start" end="$end">
        <do>
            <interact desc="Message to user" with="User">
                <instruct desc="Iteration: " with="User" type="string">concat("Iteration ", $currentIndex)</instruct>
            </interact>
        </do>
    </foreach>	

.. index:: flow
.. index:: title (flow)
.. index:: desc (flow)
.. index:: stopOnError (flow)
.. index:: documentation (flow)
.. index:: thread (flow)
.. index:: hidden (flow)
.. index:: collapsed (flow)
.. _tdl-step-flow:

flow
~~~~

The ``flow`` step is used to perform sequences of steps in parallel rather that sequentially as is the default. This can be useful
in scenarios where you want to process data in parallel or trigger messaging to actors concurrently. The flow of execution will be 
joined at the end of the ``flow`` step to continue sequential execution. The structure of the ``flow`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @title, no, A short title to display for this step (default is "flow").
    @desc, no, A description to display to the user on the purpose of the forking.
    @stopOnError, no, A boolean flag determining whether the test session should end if this step fails (default is ``false``). See also :ref:`tdl-steps-common-stoponerror`.
    @hidden, no, A boolean flag determining whether or not the step is displayed to users (default is ``false``). See also :ref:`tdl-steps-common-hidesteps`.
    @collapsed, no, A boolean flag determining whether or not the step is displayed as initially collapsed (default is ``false``). See also :ref:`tdl-steps-common-collapsed`.
    documentation, no, Rich text content that provides further information on the current step.
    thread, yes, One or more elements containing as children any sequence of steps to execute in the thread (including other ``flow`` steps).

The following example sends a SOAP request to two actors in parallel and proceeds to send a third one when both actors have replied.

.. code-block:: xml

    <flow desc="Contact actor ReceiverA and ReceiverB in parallel">
        <thread>
            <!--
                Send message to ReceiverA and wait for response.
            -->
            <btxn from="Sender" to="ReceiverA" txnId="t1" handler="SoapMessaging"/>
            <send id="dataSend" desc="Send data to A" from="Sender" to="ReceiverA" txnId="t1">
                <config name="soap.version">1.2</config>
                <input name="soap_message">$soapMessageForA</input>
            </send>
            <receive id="dataReceive" desc="Receive data from A" from="ReceiverA" to="Sender" txnId="t1"/>
            <etxn txnId="t1"/>
        </thread>
        <thread>
            <!--
                Send message to ReceiverB and wait for response.
            -->
            <btxn from="Sender" to="ReceiverB" txnId="t2" handler="SoapMessaging"/>
            <send id="dataSend" desc="Send data to B" from="Sender" to="ReceiverB" txnId="t2">
                <config name="soap.version">1.2</config>
                <input name="soap_message">$soapMessageForB</input>
            </send>
            <receive id="dataReceive" desc="Receive data from B" from="ReceiverB" to="Sender" txnId="t2"/>
            <etxn txnId="t2"/>
        </thread>
    </flow>
    <!-- 
        After ReceiverA and ReceiverB have responded send a message to ReceiverC.
    -->
    <btxn from="Sender" to="ReceiverC" txnId="t3" handler="SoapMessaging"/>
    <send id="dataSend" desc="Send data to C" from="Sender" to="ReceiverC" txnId="t3">
        <config name="soap.version">1.2</config>
        <input name="soap_message">$soapMessageForC</input>
    </send>
    <etxn txnId="t3"/>

.. note::
    **Parallel receives:** In case you use the :ref:`receive<tdl-step-receive>` step within a ``flow`` step's threads and a
    :ref:`custom messaging service<handlers>`, you need to make sure your service manages the specific receive call's identifier.
    Check the `messaging service documentation`_ for details on how to do this.

.. index:: exit
.. index:: desc (exit)
.. index:: success (exit)
.. index:: documentation (exit)
.. index:: hidden (exit)
.. _tdl-step-exit:

exit
~~~~

The ``exit`` step is used to immediately exit the test case from any execution branch. The structure of the ``exit`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @desc, no, A description to display for the ``exit`` step.
    @success, no, Whether or not this step should be considered as a success or failure (the default). This is provided as a ``boolean`` or a variable reference.
    @hidden, no, A boolean flag determining whether or not the step is displayed to users (default is ``false``). See also :ref:`tdl-steps-common-hidesteps`.
    documentation, no, Rich text content that provides further information on the current step.

The following example shows a test case that exits as a success based on the user's input:

.. code-block:: xml
    :emphasize-lines: 8

    <assign to="$inputValue">'NO'</assign>
    <interact desc="Provide your choice" with="User">
        <request desc="Enter 'YES' to end the test" with="User">$inputValue</request>
    </interact>
    <if>
        <cond>$inputValue = 'YES'</cond>
        <then>
            <exit desc="Terminate test" success="true"/>
        </then>
        <else>
            <interact desc="You chose to continue" with="User">
                <instruct desc="Test continues" with="User" type="string">""</instruct>
            </interact>
            <verify handler="XSDValidator" desc="Validate content">
                <input name="xmldocument">$document</input>
                <input name="xsddocument">$schemaFile"</input>
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

.. index:: Support steps

Support steps
-------------

Support steps are those that perform specific actions not related to messaging, processing or flow control. 

.. index:: assign
.. index:: to (assign)
.. index:: append (assign)
.. index:: type (assign)
.. index:: lang (assign)
.. index:: source (assign)
.. index:: asTemplate (assign)
.. index:: stopOnError (assign)
.. _tdl-step-assign:

assign
~~~~~~

The ``assign`` step is a frequently used construct in GITB TDL. It is a step that is not visible to the user, used for the manipulation 
of data in the test session's context. It can be used to assign values to variables but also as a means of 
performing simple processing or conversion between data types (see :ref:`test-case-types-type-conversions`). 
The processing and assignment result is determined by an expression provided as the text content of the ``assign`` element (see :ref:`test-case-expressions`). 
The element's structure is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @to, yes, The target variable to assign the result of the expression to.
    @append, no, Used if the ``to`` variable is a ``list`` to append the result to. Can be "true" or "false".
    @type, no, Used to explicitly specify the type of variable to create (e.g. if the ``to`` is an entry in a ``map``).
    @lang, no, The expression language prefix to use to evaluate the contained expression (see :ref:`test-case-namespaces` and :ref:`test-case-expressions`).
    @source, no, A variable reference to identify a source ``object`` variable upon which the expression should be evaluated.
    @asTemplate, no, Whether or not the result will be considered as a template for placeholder replacement (see :ref:`test-case-expressions-template-files`). By default this is "false".
    @stopOnError, no, A boolean flag determining whether the test session should end if this step fails (default is ``false``). See also :ref:`tdl-steps-common-stoponerror`.

The following example illustrates assigning a value to a ``number`` variable and also counting the nodes in an XML document:

.. code-block:: xml

    <assign to="$value">1</assign>
    <assign to="$nodeCount" source="$document">count(//*[local-name() = "Attachment"]</assign>

The ``to`` attribute of an ``assign`` step determines the target variable to which the expression's output will be assigned to. This can be:

    * An **existing variable**, defined as part of the test case's :ref:`variables<test-case-variables>` section or from previous steps.
    * A **new variable** that will be created once this step completes.

When defining a new variable its type is determined based on the result of the expression. This can also be affected by additional context information
from the way the ``assign`` step is used, specifically the ``append`` attribute that would suggest a ``list``, as well as the ``to`` expression that 
could suggest a ``map`` (e.g. if this defines ``$myMap{myKey}``).

Numerous examples of the ``assign`` step can be found in the documentation on :ref:`expressions<test-case-expressions>`. Examples are also provided 
here on how variables are :ref:`dynamically created<test-case-variables-from-expression-output>` if not already defined.

.. note::
    **Using '$' to define the assignment target:** In the provided examples the ``to`` attribute of an ``assign`` step is always 
    prefixed by a ``$`` given that these are :ref:`variable references<test-case-referring-to-variables>`. In the case of ``assign``
    steps this is optional given that the ``to`` can only ever refer to a variable. As such, a ``to`` value of ``myVariable`` is valid 
    and considered the same as ``$myVariable``.

.. index:: log
.. index:: lang (log)
.. index:: source (log)
.. index:: asTemplate (log)
.. index:: stopOnError (log)
.. index:: level (log)
.. index:: ERROR (log)
.. index:: WARNING (log)
.. index:: INFO (log)
.. index:: DEBUG (log)

.. _tdl-step-log:

log
~~~

The ``log`` step is used to add information to the test session's log output at various severity levels. The step itself is not visible on a test case's
diagram but users can inspect its output in the recorded test session log. This step can be used both as a development utility
for test case developers and also as a means of providing additional information to testers. The latter case can be valuable
in providing e.g. technical details to complement a validation step if needed to inspect further details.

The log output is determined by an expression provided as the text content of the ``log`` element (see :ref:`test-case-expressions`).
The element's structure is as follows:

.. csv-table::
    :stub-columns: 1
    :delim: ~
    :header: "Name", "Required?", "Description"

    @lang~ no~ The expression language prefix to use to evaluate the contained expression (see :ref:`test-case-namespaces` and :ref:`test-case-expressions`).
    @source~ no~ A variable reference to identify a source ``object`` variable upon which the expression should be evaluated.
    @asTemplate~ no~ Whether or not the result will be considered as a template for placeholder replacement (see :ref:`test-case-expressions-template-files`). By default this is "false".
    @stopOnError~ no~ A boolean flag determining whether the test session should end if this step fails (default is ``false``). See also :ref:`tdl-steps-common-stoponerror`.
    @level~ no~ The severity level to consider for the log entry. This can be (in increasing severity) ``DEBUG``, ``INFO`` (the default level), ``WARNING`` or ``ERROR``. It can also be provided as a variable reference.

The following example illustrates the various ways the ``log`` step can be used, considering in this case input provided by the
user by means of a :ref:`user interaction step<tdl-step-interact>`:

.. code-block:: xml

    <!-- Add a static message to the log. -->
    <log>'Starting execution of test case'</log>
    <!-- Request certain information from the user. -->
    <interact id="input" desc="User input">
        <request desc="Provide a boolean flag" name="flag" options="true,false"/>
        <request desc="Provide an XML file" contentType="BASE64" name="file"/>
    </interact>
    <!-- Log the provided flag value. -->
    <log>$input{flag}</log>
    <!-- Log a message including the provided flag value. -->
    <log>concat('You selected: ', $input{flag})</log>
    <!-- Print the id attribute of the XML file's root element. -->
    <log source="$input{file}">string(/*[local-name() = "myRootElement"]/@id)</log>
    <!-- Define a template text. -->
    <assign to="message">'A value of ${input{flag}} was provided.'</assign>
    <!-- Will process 'message' as a template to produce the log output. -->
    <log asTemplate="true">$message</log>
    <!-- Will process 'message' as a simple text and log its contents without replacing placeholders. -->
    <log>$message</log>
    <!-- Equivalent to the previous case (template processing is disabled by default). -->
    <log asTemplate="false">$message</log>
    <!-- Log a message at a different severity level (a warning in this case). -->
    <log level="WARNING">'The value should normally be received by your service directly.'</log>
    <!-- Log a message at a dynamically defined severity level. -->
    <assign to="logLevel">'WARNING'</assign>
    <log level="$logLevel">'The value should normally be received by your service directly.'</log>

.. note::
    **Test case log level:** You can configure the :ref:`minimum log level for a test case<test-case-steps>` to control which log
    messages are included in the session log.

.. index:: group
.. index:: desc (group)
.. index:: stopOnError (group)
.. index:: documentation (group)
.. index:: hidden (group)
.. index:: collapsed (group)
.. index:: title (group)
.. _tdl-step-group:

group
~~~~~

The ``group`` step is a construct used to visually group together a sequence of steps. It has no effect on the test execution adding only
a visual grouping and label to the display. Its structure is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @title, no, A short title to display for this step (default is "group").
    @desc, no, The description for the group.
    @stopOnError, no, A boolean flag determining whether the test session should end if this step fails (default is ``false``). See also :ref:`tdl-steps-common-stoponerror`.
    @hidden, no, A boolean flag determining whether or not the step is displayed to users (default is ``false``). See also :ref:`tdl-steps-common-hidesteps`.
    @collapsed, no, A boolean flag determining whether or not the step is displayed as initially collapsed (default is ``false``). See also :ref:`tdl-steps-common-collapsed`.
    documentation, no, Rich text content that provides further information on the current step.

The children of the ``group`` element can be any number of steps supported by GITB TDL. The following example creates a group around a set of 
related validations.

.. code-block:: xml

    <group desc="Validate document">
        <verify handler="XSDValidator" desc="Against schema">
            <input name="xmldocument">$document</input>
            <input name="xsddocument">$schema"</input>
        </verify>
        <verify handler="SchematronValidator" desc="Against Schematron 1">
            <input name="xmldocument">$document</input>
            <input name="schematron">$schematron1"</input>
        </verify>
        <verify handler="SchematronValidator" desc="Against Schematron 2">
            <input name="xmldocument">$document</input>
            <input name="schematron">$schematron2"</input>
        </verify>
    </group>

Using a ``group`` can provide a useful means of structuring a test case's presentation. In addition, it allows several steps to be considered
together and determine how they are presented. Specifically:

* Used with the :ref:`hidden<tdl-steps-common-hidesteps>` attribute, to completely hide a set of steps.
* Used with the :ref:`collapsed<tdl-steps-common-collapsed>` attribute, to define the group's display as initially collapsed.

Use of these attributes is illustrated in the following TDL snippet:

.. code-block:: xml

    <!-- 
        Hide both validations. This could be interesting to make an internal check to drive subsequent control flow.
    -->
    <group id="checkResult" hidden="true">
        <verify handler="XSDValidator" desc="Against schema" level="WARNING">...</verify>
        <verify handler="SchematronValidator" desc="Against Schematron" level="WARNING">...</verify>
    </group>
    <!-- 
        Show the two validations in a group and present as initially collapsed.
    -->
    <group desc="Validate document" collapsed="true">
        <verify handler="XSDValidator" desc="Against schema">...</verify>
        <verify handler="SchematronValidator" desc="Against Schematron">...</verify>
    </group>
    <!-- 
        Show the two validations in a group and present it fully (the default).
    -->
    <group desc="Validate document">
        <verify handler="XSDValidator" desc="Against schema">...</verify>
        <verify handler="SchematronValidator" desc="Against Schematron">...</verify>
    </group>

.. index:: verify
.. index:: id (verify)
.. index:: desc (verify)
.. index:: handler (verify)
.. index:: level (verify)
.. index:: stopOnError (verify)
.. index:: output (verify)
.. index:: documentation (verify)
.. index:: property (verify)
.. index:: config (verify)
.. index:: input (verify)
.. index:: hidden (verify)
.. index:: ERROR (verify)
.. index:: WARNING (verify)

.. _tdl-step-verify:

verify
~~~~~~

The ``verify`` step is used to trigger validation of content. Similar to :ref:`tdl-messaging-steps` and  :ref:`tdl-processing-steps`, validation
takes place using a validation handler implementation that can either be an embedded test bed component or a remote service that implements the
`GITB validation service API`_. The content to validate is provided by the test case to the handler in terms of configuration and input, for which
a test report is returned in the `GITB TRL (Test Reporting Language) format`_. The structure of the ``verify`` element is as follows:

.. _GITB validation service API: https://www.itb.ec.europa.eu/specs/latest/gitb_vs.wsdl
.. _GITB TRL (Test Reporting Language) format: https://www.itb.ec.europa.eu/specs/latest/gitb_tr.xsd

.. csv-table::
    :stub-columns: 1
    :delim: ~
    :header: "Name", "Required?", "Description"

    @id~ no~ The ID for the step. This is also the name of a ``boolean`` variable in the session context in which the validation result will be recorded (``true`` for success).
    @desc~ no~ The description for the validation.
    @handler~ yes~ A string value or variable reference identifying the the validation handler (see :ref:`handlers-implementation`).
    @level~ no~ The severity level to be considered when this step fails validation. Can be set to ``ERROR`` (the default) or ``WARNING``, or be defined dynamically via :ref:`variable reference<test-case-referring-to-variables>`.
    @stopOnError~ no~ A boolean flag determining whether the test session should end if this step fails (default is ``false``). See also :ref:`tdl-steps-common-stoponerror`.
    @output~ no~ A string value determining the name of the variable to be set with the output of the step (if any). If this is not set the output is displayed but is not recorded in the test session context.
    @hidden~ no~ A boolean flag determining whether or not the step is displayed to users (default is ``false``). See also :ref:`tdl-steps-common-hidesteps`.
    documentation~ no~ Rich text content that provides further information on the current step.
    property~ no~ Zero or more elements to provide configuration regarding the setup of the validation handler call that are not passed to the handler. Each ``property`` element has a ``name`` attribute and a text content or variable reference as value.
    config~ no~ Zero or more elements to provide configuration for the validation. Each ``config`` element has a ``name`` attribute and a text content or variable reference as value.
    input~ yes~ One more elements for the validation's input parameters. See :ref:`handlers-inputs-outputs` for details.

A ``verify`` step that is set at warning level (through attribute ``level``) will never result in an overall failure for the test session. If validation fails,
the result will be indicated as a warning but without further impact. Note that a validation service returning a detailed validation report for a ``verify`` step 
at warning level may have its resulting report adapted accordingly. The report will be set as ``WARNING`` (if it was ``FAILURE``) and any error-level report 
items will be listed as warnings.

The following example includes three ``verify`` steps, the first one using an :ref:`handlers-XSDValidator`, followed by a second one at warning level which uses a remote
validation service. The third ``verify`` step replicates the previous one but defines its level dynamically:

.. code-block:: xml

    <!-- 
        Validation using the embedded XSDValidator.
    -->
    <verify handler="XSDValidator" desc="Validate invoice against schema">
        <input name="xmldocument">$document</input>
        <input name="xsddocument">$schema"</input>
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

.. note::
    **Remote or local validators:** Simple validations such as those evaluating an XPath expression against a document can be implemented using 
    :ref:`handlers-predefined-validation-handlers`. When validation logic however is complex it is always best to decouple this into an external validation service. 
    This is the case even when validating XML content since this usually involves multiple validation steps using an XSD and one or more Schematron files. It is more
    concise to present this as a single validation step with one report. This also enhances maintainability of the test cases considering that use of the embedded
    :ref:`handlers-XSDValidator` and :ref:`handlers-SchematronValidator` means that you need to bundle (and maintain) the validation artefacts in each test suite. 
    When decoupled as a service artefacts can be updated without needing new test suite versions aside from the benefit that your service can also be invoked 
    outside the test bed using any SOAP client.

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

.. index:: call
.. index:: id (call)
.. index:: path (call)
.. index:: from (call)
.. index:: stopOnError (call)
.. index:: input (call)
.. index:: output (call)
.. index:: hidden (call)
.. _tdl-step-call:

call
~~~~

The ``call`` step is used to invoke a set of steps defined as a ``scriptlet`` (see :ref:`scriptlets`). If we consider that a scriptlet resembles a function
with input, output and local variables, the ``call`` step can be considered as the function's invocation. Its purpose is to identify the ``scriptlet`` to call, pass
its required input parameters and receive its output. The structure of the ``call`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @id, no, The ID for the step. This is also the name of a ``map`` variable in the session context in which output will be stored.
    @path, yes, The identifier scriptlet to call. The value provided here depends on the whether the scriptlet is :ref:`external to the test case<scriptlets>` or :ref:`defined within it<test-case-scriptlets>`.
    @from, no, The identifier of the test suite from which the scriptlet will be loaded. If not provided this is assumed to be the current test suite.
    @stopOnError, no, A boolean flag determining whether the test session should end if this step fails (default is ``false``). See also :ref:`tdl-steps-common-stoponerror`.
    @hidden, no, A boolean flag determining whether or not the step is displayed to users (default is ``false``). See also :ref:`tdl-steps-common-hidesteps`.
    @input, no, An alternative to input elements to provide a single input when the scriptlet expects a single input or (if multiple) a single mandatory input. See also :ref:`tdl-step-call__simplified`.
    @output, no, The name to use for the session context variable to store the scriptlet output as an alternative to using the ``id``. See also :ref:`tdl-step-call__simplified`.
    input, no, Zero or more elements for the ``scriptlet``'s input parameters. See :ref:`handlers-inputs-outputs` for details.
    output, no, Zero or more elements for the ``scriptlet``'s output parameters to specify which outputs you require.

Scriptlets can be defined in :ref:`separate XML files<scriptlets>`, in which case they can be used by any test case, or as
:ref:`internal to a specific test case<test-case-scriptlets>`, in which case they are considered private. How a scriptlet
is looked up depends on its type, which defines how the ``path`` and ``from`` attributes are used. Specifically:

* **Internal scriptlet:** The ``path`` attribute is set with the ``id`` value of the scriptlet to call and the ``from`` attribute is omitted.
* **External scriptlet:** The ``from`` attribute is set with the ``id`` of the test suite to load the scriptlet from, and
  the ``path`` is set with the file path to the scriptlet's XML file (relative to the test suite's root).

If the ``from`` attribute is not specified, the test engine first attempts to load the scriptlet from the ones defined
within the test case, by matching the ``path`` value against the defined scriptlets' ``id``. If no match is found a
further lookup is made within the test case's containing test suite, in which case the ``path`` value is considered as
the path to the scriptlet's XML file. When the ``from`` attribute is specified the scriptlet is always considered to be
external to the test case, and its value is considered to be a test suite's ``id``. The lookup in this case proceeds as
follows:

#. If the value matches the current **test suite** ``id``, the lookup is made within the current test suite.
#. If not found, the lookup for a matching test suite continues within the current test suite's **specification**.
#. If not found, the lookup for a matching test suite continues within the current test suite's overall **domain**.

.. note::
    **Non-unique test suite IDs:** If multiple test suites are matched during a scriptlet's lookup, an arbitrary test suite
    will be considered. Ensure that test suites sharing common resources have a unique ``id``. A test suite's ``id`` is
    always unique within a specification but not necessarily across specifications (i.e. within the overall domain).

The following example ``call`` steps, illustrate different cases of scriptlet lookup:

.. code-block:: xml

    <!--
        Look for a scriptlet with id "script1" within the test case.
        If not found look for a file "script1" within the test case's test suite.
    -->
    <call id="call1" path="script1"/>
    <!--
        Look for the scriptlet in test suite "test_suite_1" and load it from file "scriptlets/script1.xml".
    -->
    <call id="call2" from="test_suite_1" path="scriptlets/script1.xml"/>

Once the target scriptlet has been located, the ``call`` step will calculate and pass any ``inputs`` it requires. The approach
to pass inputs is identical to the case of :ref:`inputs to handlers<handlers-inputs-outputs>`. Values can be provided as
constants or results of :ref:`expressions<test-case-expressions>`, and can optionally be considered as :ref:`templates<test-case-expressions-template-files>`
with placeholder substitutions. It is important to note that all scriptlet inputs are required; failure to provide one or
more inputs will result in a test session error.

Once a scriptlet completes, its :ref:`outputs<scriptlets_elements_output>` are recorded in a ``map`` stored in the
test session's context, that is named using the ``call`` step's ``id``. Individual outputs can then be referred
to from within this ``map`` using their name.

A ``call`` step may choose to ignore specific scriptlet outputs. This can be done by listing the specific outputs you are
interested in, naming them as part of the ``call`` step's ``output`` elements. Any outputs that don't match the listed ones
will then be discarded. Note that when the ``call`` step does not define specific ``output`` elements, all scriptlet outputs are
returned by default.

The following example illustrates potential uses of the ``call`` step:

.. code-block:: xml

    <!--
        Call a scriptlet defined within the test case and retrieve all its output.
    -->
    <call id="internalCall" path="script1">
        <input name="docToValidate">$fileContent1</input>
    </call>
    <!--
        Call a scriptlet defined in test suite "test_suite_1" and retrieve only its "outputMessage" output.
    -->
    <call id="externalCall" from="test_suite_1" path="scriptlets/script1.xml">
        <input name="docToValidate">$fileContent1</input>
        <output name="outputMessage"/>
    </call>

Further information on defining and using scriptlets is provided in the :ref:`scriptlet documentation<scriptlets>`. For
scriptlets specifically defined within test cases (i.e. private scriptlets) refer to the test case's
:ref:`scriptlets element<test-case-scriptlets>`.

.. _tdl-step-call__simplified:

Simplified call steps
+++++++++++++++++++++

Test cases often include scriptlets as utilities that don't need multiple inputs or produce only single output values. To reduce the 
verbosity of the ``call`` step in such cases, you can make use of two syntax alternatives:

    * The ``input`` attribute to provide a single input. This is possible when the scriptlet expects only a single input.
    * The ``output`` attribute to directly name the result rather than use an intermediate ``map``.

Defining an input both as an attribute and child element is superfluous. If nonetheless both are defined, the child elements take precedence.
On the other hand, the ``output`` attribute is complementary to the output child elements. When defining output child elements these result
in limiting the produced results to only the ones specified. The results are first filtered as such before using the ``output`` attribute's
value to name the resulting variable.

The following example illustrates how these alternatives can be used to simplify your test definitions. We consider here that we are 
generating two messages based on a template that includes a placeholder for a signature (named "signature"). For the first message
we use a verbose syntax whereas for the second one we use the simplifications discussed here. In both cases the signature value is created
through a scriptlet that expects an input named "valueToSign" and produces an output named "signedValue".

.. code-block:: xml

    <!--
        Verbose approach.
    -->
    <call id="signatureCall" path="signatureScript">
        <input name="valueToSign">$aValue</input>
    </call>
    <!-- 
        The output is stored in a map named using the step's id. As the template defines
        a "signature" placeholder we need to create such a variable from the result map.
    -->
    <assign to="signature">$signatureScript{signedValue}</assign>
    <assign to="message1" asTemplate="true">$messageTemplate</assign>

    <!--
        Simplified approach.
    -->
    <call output="signature" path="signatureScript" input="$aValue"/>
    <assign to="message2" asTemplate="true">$messageTemplate</assign>

.. note::
    The :ref:`tdl-step-process` step also offers :ref:`similar syntax simplifications<tdl-step-process__simplified>`. This simplified
    syntax is available for :ref:`tdl-step-process` and :ref:`tdl-step-call` steps as these typically represent utilities that are
    frequently used.

.. index:: interact
.. index:: id (interact)
.. index:: title (interact)
.. index:: desc (interact)
.. index:: with (interact)
.. index:: inputTitle (interact)
.. index:: stopOnError (interact)
.. index:: documentation (interact)
.. index:: hidden (interact)
.. index:: collapsed (interact)

.. _tdl-step-interact:

interact
~~~~~~~~

The ``interact`` step is used to exchange information with the user executing the test case. Interactions can be of two types:

* **Instructions:** Informative messages or data to be presented to a user.
* **Requests:** Prompts to a user to provide input.

Both instructions and requests can be included in the same ``interact`` step to display and/or request multiple sets of information in one go.
The structure of the ``interact`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @id, no, Used as the name of a ``map`` variable that will be used to store provided input (if no per-input assignment is provided).
    @title, no, A short title to display for this step (default is "interact").
    @desc, no, A description for the user interaction.
    @with, no, The ID of the actor this interaction refers to. If not specified is is assumed to be the test case actor defined as the SUT.
    @inputTitle, no, A custom text to display as the title of the user input popup (default is "Server interaction").
    @stopOnError, no, A boolean flag determining whether the test session should end if this step fails (default is ``false``). See also :ref:`tdl-steps-common-stoponerror`.
    @hidden, no, A boolean flag determining whether or not the step is displayed to users (default is ``false``). See also :ref:`tdl-steps-common-hidesteps`.
    @collapsed, no, A boolean flag determining whether or not the step is displayed as initially collapsed (default is ``false``). See also :ref:`tdl-steps-common-collapsed`.
    documentation, no, Rich text content that provides further information on the current step.
    instruct, no, Zero or more elements to appear as instructions to the user.
    request, no, Zero or more information requests for the user.

.. index:: instruct (interact)
.. index:: desc (instruct)
.. index:: with (instruct)
.. index:: name (instruct)
.. index:: type (instruct)
.. index:: source (instruct)
.. index:: asTemplate (instruct)
.. index:: mimeType (instruct)

The ``instruct`` elements define what is going to presented to the user. They have the following structure:

.. csv-table::
    :stub-columns: 1
    :delim: ~
    :header: "Name", "Required?", "Description"

    @desc~ yes~ The label to display to the user.
    @with~ no~ The ID of the actor this interaction refers to. If not specified this is taken from the ``interact`` parent element (which itself defaults to the test case's SUT actor).
    @name~ no~ In case of ``instruct`` elements that used to share binary content, this is used as the name of the file presented for download.
    @type~ no~ The ``type`` to consider for the displayed value. If this is not specified the ``type`` will be inferred from the referred variable (if defined) or default to ``string``.
    @mimeType~ no~ A `mime type`_ value (e.g. ``text/xml``) to hint how this value should be highlighted when displayed. In case an invalid or unsupported mime type is provided no such highlighting will be applied.
    @source~ no~ A pure variable reference identifying a source variable. Used as the target upon which to evaluate the contained expression.
    @asTemplate~ no~ Whether or not the result will be considered as a template for placeholder replacement (see :ref:`test-case-expressions-template-files`). By default this is "false".

.. index:: request (interact)
.. index:: desc (request)
.. index:: with (request)
.. index:: contentType (request)
.. index:: encoding (request)
.. index:: name (request)
.. index:: options (request)
.. index:: optionLabels (request)
.. index:: multiple (request)
.. index:: asTemplate (request)
.. index:: inputType (request)
.. index:: mimeType (request)

The ``request`` elements define how information shall be requested from the user. Their structure is as follows:

.. csv-table::
    :stub-columns: 1
    :delim: ~
    :header: "Name", "Required?", "Description"

    @desc~ yes~ The label to display to the user.
    @with~ no~ The ID of the actor this interaction refers to. If not specified this is taken from the ``interact`` parent element (which itself defaults to the test case's SUT actor).
    @contentType~ no~ Defines how the specified variable's value is to be set ("STRING", "BASE64" or "URI"). The default is "STRING".
    @encoding~ no~ Used in case of text binary input to specify the character encoding to consider. The default is "UTF-8".
    @name~ no~ In case of ``request`` elements this name is the key to be used for the map entry to hold the provided data.
    @options~ no~ Used to render a dropdown list by providing the option values to consider (comma-separated values, a reference to a string variable of comma-separated values, or a reference to a list variable of strings).
    @optionLabels~ no~ Used as the labels for the option values (comma-separated values, a reference to a string variable of comma-separated values, or a reference to a list variable of strings). If provided the number of values needs to match the options. If not provided the option values are used.
    @multiple~ no~ A ``boolean`` value to determine whether the dropdown list (if the ``options`` attribute is defined) shall be a single or multiple selection list (default is ``false`` for single selection).
    @inputType~ no~ The input control type to use when prompting users for the relevant value. By default a value of ``TEXT`` is assumed unless this input is mapped to an existing ``binary`` variable.
    @mimeType~ no~ In case the ``inputType`` is set as ``CODE`` (i.e. a code editor) this is the content's expected `mime type`_ (e.g. ``text/xml``) to be considered for presenting appropriate syntax highlighting.
    @asTemplate~ no~ Whether or not the result will be considered as a template for placeholder replacement (see :ref:`test-case-expressions-template-files`). By default this is "false".

The content of the ``instruct`` and ``request`` elements is expected to be an expression (see :ref:`test-case-expressions`) that takes different
meaning depending on the specific element type. In the case of providing information to the user through a ``instruct`` element the contained
value is a complete expression that will be evaluated to produce the value to display. In this case the ``contentType`` and ``encoding`` 
attributes are not used and are ignored if specified. What is important is the ``type`` attribute that defines how the element's expression
result is to be interpreted (see :ref:`test-case-types`):

* A ``binary``, ``object`` or ``schema`` type results in the calculated expression being computed as BASE64 content. The user will have the option to
  download the content as a file or open it in a code editor.
* All other cases result in the value being displayed inline as text. This is also the default case if the ``type`` attribute is not specified.
* Note that in case the text is too long to be displayed the user will instead be provided with controls to download it as a file or open it
  in a code editor (as in the case of e.g. binary content).

As a complement to the ``type`` attribute you can also specify the ``mimeType`` attribute. This is meaningful for binary or large text content
as it serves two purposes: it allows you to specify the content type and file extension to use when the content is downloaded as a file, and it
provides a hint for appropriate syntax highlighting when displaying the content in a code editor.

Concerning ``request`` elements, the content of the expression is expected to be a pure variable reference that identifies the variable that
will receive the input. You can also leave this empty and specify a ``name`` instead, in which case the value will be recorded in a map in the 
test session context. This map is named using the ``interact`` step's ``id`` and in which the specific input value is added with a key matching
its ``name``. These two approaches are illustrated in the snippet that follows:

.. code-block:: xml

    <interact id="data" desc="Provide inputs">
        <!-- Approach 1, stored in variable aValue. -->
        <request desc="Enter a text value:">$aValue</request>
        <!-- Approach 2, stored in variable data{value}. -->
        <request desc="Enter another text value:" name="value"/>
    <interact>

To determine the type of input control to present to the user you use the ``inputType`` attribute. The supported values for this are:

* ``TEXT`` for a simple text field (the default if not specified).
* ``UPLOAD`` for a file upload control.
* ``MULTILINE_TEXT`` for a textarea supporting input of multiple lines.
* ``SECRET`` for a control to add a secret value such as a password.
* ``CODE`` for input via a code editor. To complement this you can also specify the ``mimeType`` attribute with a `mime type`_ (e.g. ``text/xml``) to have
  appropriate syntax highlighting.
* ``SELECT_SINGLE`` for a single-select dropdown list, specifying the options via the ``options`` and ``optionLabels`` attributes.
* ``SELECT_MULTIPLE`` for a multi-select dropdown list using similarly the ``options`` and ``optionLabels`` attributes.

Prior to GITB TDL version 1.14.0, the way to determine the input control to use was the ``contentType`` attribute. Although less expressive, this approach is 
still supported as follows:

* Specifying "BASE64" results in a file upload presented to the user.
* Specifying "STRING" (the default) or "URI" results in a simple text input. Note that only "STRING" can be used in case the request is defined as a dropdown list (i.e. the ``options`` attribute is defined).

It is interesting to note that any available context information is always considered to reduce the configuration you need to provide. For example, if for a ``request``
you are referencing an already defined ``binary`` variable, you can skip the ``inputType`` or ``contentType`` definitions as this will anyway result in a file upload.
Similarly, if for a ``request`` you define ``options`` and the ``multiple`` attribute, you don't need to define the ``inputType`` as well as this is considered to be
by default ``SELECT_MULTIPLE``.

The following examples illustrate user interactions presenting instructions and also requesting information:

.. code-block:: xml

    <interact desc="Some information and inputs">
        <!-- type="string" omitted as default. Displays the text as a message to the user. -->
        <instruct desc="This is a simple message"/>
        <instruct desc="A text value:">concat("A text value ", $aTextValue)</instruct>
        <!-- Present a download button and XML editor for file "schema.xsd" (not specifying a name would produce a "downloadedFile" file). -->
        <instruct name="schema.xsd" desc="A file to download:" mimeType="text/xml">$schemaFile</instruct>
        <!-- Present a text input field storing the result in variable aStringInputValue. -->
        <request desc="Enter a text value:" inputType="TEXT">$aStringInputValue</request>
        <!-- Present a text area input storing the result in variable aLongStringInputValue. -->
        <request desc="Enter a long text value:" inputType="MULTILINE_TEXT">$aLongStringInputValue</request>
        <!-- Present a secret value input storing the result in variable aSecretValue. -->
        <request desc="Enter a secret value:" inputType="SECRET">$aSecretValue</request>
        <!-- Present a single selection dropdown list storing the result in variable aSelectedInputValue. -->
        <request desc="Enter a text value:" options="v1, v2" optionLabels="Value 1, Value 2">$aSelectedInputValue</request>
        <!-- Present a file upload storing the result in variable aBinaryVariable. -->
        <request desc="Upload a file:">$aBinaryVariable</request>
    </interact>

    <!-- Example storing all provided input in a map. This uses the "id" and "name" attributes. -->
    <interact id="userInput" desc="Some information and inputs">
        <!-- Present a text input field storing the result in variable userInput{text} (a type of "string" is assumed as the default). -->
        <request name="text" desc="Enter a text value:"/>
        <!-- Present a code editor for XML content, storing the result in variable userInput{xml} -->
        <request name="xml" desc="Enter XML content:" inputType="CODE" mimeType="text/xml"/>
        <!-- Present a code editor for JSON content, storing the result in variable userInput{json} -->
        <request name="json" desc="Enter JSON content:" inputType="CODE" mimeType="application/json"/>
        <!-- Present a file upload storing the result in variable userInput{file}. -->
        <request name="file" desc="Upload a file:" type="binary"/>
        <!-- Equivalent to the above but using the inputType -->
        <request name="anotherFile" desc="Upload another file:" inputType="UPLOAD"/>
    </interact>

To better illustrate how dropdown selections can be defined, the following code sample presents the different ways to define them:

.. code-block:: xml

    <steps>
        <!-- Assign options and labels (you may predefine variables or create them on the fly as follows) -->
        <assign to="input3_options">"v1, v2, v3"</assign>
        <assign to="input3_labels">"Value 1, Value 2, Value 3"</assign>
        <assign to="input4_options" append="true">"x1"</assign>
        <assign to="input4_options" append="true">"x2"</assign>
        <assign to="input4_options" append="true">"x3"</assign>
        <assign to="input4_labels" append="true">"VAL 1"</assign>
        <assign to="input4_labels" append="true">"VAL 2"</assign>
        <assign to="input4_labels" append="true">"VAL 3"</assign>
	
        <interact id="data" desc="Enter data">
            <!-- Single selection with options provided in the attribute values (stored as data{input1}). -->
            <request desc="Select one" options="o1, o2, o3" optionLabels="Option 1, Option 2, Option 3" name="input1"/>
            <!-- Multiple selection with options provided in the attribute values (stored as data{input2}). -->
            <request desc="Select multiple" options="o1, o2, o3" optionLabels="Option 1, Option 2, Option 3" multiple="true" name="input2"/>
            <!-- Single selection with options provided by referring to string variables (stored as data{input3}). -->
            <request desc="Select one (use string reference)" options="$input3_options" optionLabels="$input3_labels" name="input3"/>
            <!-- Single selection with options provided by referring to list variables (stored as data{input4}). -->
            <request desc="Select one (use list reference)" options="$input4_options" optionLabels="$input4_labels" name="input4"/>
        </interact>
    </steps>

.. note::
    The value received from a ``request`` element defined as a multiple selection list will be a comma-separated string in which the individual
    parts match the selected values. This value is recorded in the test session context as a variable of type ``string`` that can be passed as
    input to handlers or be processed with relevant XPath functions.

.. _tdl-steps-common:

Common step concepts 
--------------------

The following section documents common concepts that apply to all test steps.

.. index:: documentation (test case step)
.. index:: import (documentation - test case step)
.. index:: from (documentation - test case step)
.. index:: encoding (documentation - test case step)
.. _tdl-steps-common-documentation:

Rich documentation per step
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Test steps that are meant to be presented to users can be defined with an additional ``documentation`` element to include extended rich text documentation as HTML. This complements the limited label
attached to each step (via attribute ``desc``), allowing further instructions, context and references to be provided. The structure of this element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    import, no, A reference to a separate file within the test suite archive that defines the documentation content.
    from, no, The identifier of a test suite from which the ``import`` file will be loaded. If unspecified the current test suite is assumed.
    encoding, no, In case an ``import`` reference is defined this can be used to specify the file's encoding. If not provided ``UTF-8`` is considered.

Using the above attributes to specify a reference to a separate file is not mandatory. The documentation's content can also be provided as the element's text content,
typically enclosed within a CDATA section if this includes HTML elements (in which case the ``from``, ``import`` and ``encoding`` attributes are omitted).
When loading documentation from a separate file, it is also possible to lookup this file from another test suite. This is
done by specifying as the value of the ``from`` attribute the ``id`` of the target test suite. This is used to lookup the
target test suite as follows:

#. Look for the test suite in the same **specification** as the current test case.
#. If not found in the same specification, look for the test suite in the other specifications of the test case's **domain**.
   If across specifications multiple matching test suites are found, one of them will be arbitrarily picked. To avoid such
   a scenario it is obvious that you should ensure test suites used to load shared resources can be uniquely identified.

This documentation can provide further information on the context of the test step, diagrams or reference information that are useful to understand how it is to be completed. The content supplied supports
several HTML features:

    * Structure elements (e.g. headings, text blocks, lists).
    * In-line styling.
    * Tables.
    * Links.
    * Images.

The simplest way to provide such information is to enclose the HTML content in a ``CDATA`` section to ensure the test case XML remains well-formed. The
example that follows illustrates two examples, one defining a simple additional text, and another with more comprehensive HTML content.

.. code-block:: xml

    <!-- Additional documentation as simple text. -->
    <verify handler="XSDValidator" desc="Validate invoice against UBL 2.1 Invoice Schema">
        <documentation>This is an extra documentation item.</documentation>
        <input name="xmldocument">$file_content</input>
        <input name="xsddocument" source="$UBL_Invoice_Schema_File"/>
    </verify>
    <!-- Additional documentation as rich HTML content. -->
    <verify handler="SchematronValidator" desc="Validate invoice against BII RULES" level="WARNING">
        <documentation><![CDATA[
        <p>This is <b>important information!</b> Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum. </p>
        <div style="width:100%; text-align:center"><img src="https://www.itb.ec.europa.eu/docs/services/latest/_images/ValidationService.png"/></div>
        <b><u>Steps for testing:</u></b>
        <p>
            <ol>
                <li>Prepare correctly</li>
                <li>Submit the correct file</li>
                <li>Validate results</li>
            </ol>
        </p>
        <p>
            <table style="border: 1px solid black; width:100%">
                <tr style="border: 1px solid black; font-weight: bold;">
                    <td>COL1</td><td>COL2</td><td>COL3</td><td>COL4</td><td>COL5</td>
                </tr>
                <tr>
                    <td>1</td><td>2</td><td>3</td><td>4</td><td>5</td>
                </tr>
                <tr>
                    <td>test1</td><td>test2</td><td>test3</td><td>test4</td><td>test5</td>
                </tr>
            </table>
        </p>
        <p>After this make sure to check the docs <a href="https://www.itb.ec.europa.eu/docs/tdl/latest">here</a>.</p>
        ]]></documentation>
        <input name="xmldocument">$file_content</input>
        <input name="schematron" source="$BII_RULES_Invoice_Schematron_File"/>
    </verify>

.. note::
    Documentation such as this is also supported for the overall :ref:`test suite<test-suite-metadata>` and the :ref:`test cases<test-case-metadata>` included in the test suite.

.. index:: stopOnError
.. _tdl-steps-common-stoponerror:

Stop a test session upon errors
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

During the course of a test session you will need to consider step failures and define how these are to affect the overall test execution.
Such failures can broadly be classified as:

* **Expected** errors resulting from :ref:`validations<tdl-step-verify>` and :ref:`message exchanges<tdl-messaging-steps>` due to violations of the specifications' requirements.
* **Unexpected** errors due to services being unavailable or unforeseen processing problems.

Any step failure that occurs will ultimately result in the overall test session to fail. However, whether or not you want to continue a test session
once a failure has occurred depends on the design of your test case. For example, if a test case involves constructing a message, validating it, and then 
using it to start a series of message exchanges, it is probably meaningless to proceed with messaging steps if the message is invalid to begin with. In such
a case, any failure when validating the message should immediately fail the test session.

Immediately stopping execution may however not always be desired. Consider a case where you receive a message from a system and then proceed to use it for a series of validations,
each focusing on a different complementary aspect (e.g. integrity, syntax and business rules). In such a scenario you want any failures to be recorded but not prevent subsequent
steps so that the user receives a complete validation outcome.

By default a test session will continue processing all steps regardless of failures (allowing the second case described above). If you want to stop test execution upon a failure 
you can use the ``stopOnError`` flag. This flag expects a ``boolean`` value (true or false) and can be set on:

* **Individual steps**, to stop the test for failures on the step in question, or any nested step.
* **Step sequences**, to stop the test for failures on any of the sequence's steps (or nested step). Examples of step sequence elements are the ``then`` or ``else`` blocks of an :ref:`if step<tdl-step-if>`.
* The **complete test case**, to stop the test on any failure. This is done by setting the ``stopOnError`` attribute on the test case's ``steps`` element.

The following GITB TDL snippets illustrates all these cases:

.. code-block:: xml

    <!-- Stop on any failure. -->
    <steps stopOnError="true">
        ...
    </steps>

    <!-- Stop if this step fails -->
    <verify stopOnError="true">
        ...
    </verify>

    <!-- Stop if any step within the "if" fails. -->
    <if stopOnError="true">
        ...
    </if>

    <!-- Stop if any step within the "then" block fails (but continue for failures under the else block). -->
    <if>
        <cond>...</cond>
        <then stopOnError="true">
            ...
        </then>
        <else>
            ...
        </else>
    </if>

It is interesting to note that stopping a test execution could also be achieved by means of the :ref:`if<tdl-step-if>` and 
:ref:`exit<tdl-step-exit>` steps. The following snippet illustrates such a case:

.. code-block:: xml

    <verify id="check" desc="Make an important validation">
    ...
    </verify>
    <if desc="check to stop the test">
        <cond>not($STEP_SUCCESS{check})</cond>
        <then>
            <exit desc="Stop the test"/>
        </then>
    </if>

Doing this simply to prevent subsequent test steps is overly verbose and, moreover is displayed as part of the test execution diagram. It
could still be interesting to follow this approach however if you want to include additional :ref:`processing<tdl-step-process>` or :ref:`user interaction<tdl-step-interact>` 
steps before the session ends.

.. index:: hidden
.. _tdl-steps-common-hidesteps:

Hide test steps
~~~~~~~~~~~~~~~

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

Hiding an otherwise visible test step is supported by means of the ``hidden`` attribute. This takes a ``boolean`` value that determines whether the
step should be included in the test session's display. When set to false, the step is not presented but is executed by the test engine as expected.
In other words, hiding a step affects only its visual representation, not its processing.

The following example includes a :ref:`verify<tdl-step-verify>` step that is not meant to be displayed to the user but is used to determine subsequent
processing. Note how the verification is forced at warning level to not impact the test session's result:

.. code-block:: xml

    <!-- This check will not be presented in the test session display. -->
    <verify id="internalCheck" handler="StringValidator" hidden="true" level="WARNING">
        <input name="actualstring">$valueToCheck</input>
        <input name="expectedstring">'CASE1'</input>
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

.. index:: collapsed
.. _tdl-steps-common-collapsed:

Collapse sets of test steps
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Test cases that include numerous test steps may appear as complex to users. Introducing :ref:`grouping<tdl-step-group>` of related steps can provide better
visual structuring and make them easier to follow. If further simplification is needed a good approach is to additionally define such groups as being by 
default collapsed by setting their ``collapsed`` attribute to ``true``.

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

The default value for the ``collapsed`` attribute is always ``false``, meaning that all relevant steps will by default be presented as
fully expanded.

The following example illustrates how a set of tests on XML content can be displayed as initially collapsed to simplify the display:

.. code-block:: xml

    <!-- 
        The three verify steps will be initially hidden, showing instead only their containing group.
        The group can be at any point expanded to view internal details (e.g. if a validation error is reported).
    -->
    <group desc="Validate XML message" collapsed="true">
        <verify handler="XSDValidator" desc="Validate message structure">...</verify>
        <verify handler="SchematronValidator" desc="Validate core business rules">...</verify>
        <verify handler="SchematronValidator" desc="Validate additional contraints">...</verify>
    </group>

.. note::
    **Collapsed vs Hidden:** Apart from collapsing a group of steps, steps can also be set as :ref:`hidden<tdl-steps-common-hidesteps>`.
    Use ``collapsed`` when you want to still include steps in the test session display but simplify their presentation. Use ``hidden``
    for purely internal steps that you want to completely remove from the display. In case ``hidden`` is set to ``true`` the ``collapsed``
    attribute is effectively ignored.

.. _validation report context: https://www.itb.ec.europa.eu/docs/services/latest/common/index.html#constructing-a-validation-report-tar
.. _messaging service documentation: https://www.itb.ec.europa.eu/docs/services/latest/messaging/index.html#receive
.. _mime type: https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types/Common_types