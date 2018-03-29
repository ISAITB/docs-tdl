TDL step constructs
===================

Overview
--------

TDL step constructs are used to capture the core testing logic. They are used in test cases and
also in :ref:`test-case-scriptlets` to define their sequence of test steps. The available test
steps are described in the sections that follow, organised in four main categories:

* **Messaging steps** used to exchange information between actors.
* **Processing steps** to perform complex arbitrary processing.
* **Flow steps** to manage the execution flow of the test case.
* **Support steps** to introduce support features to test cases`.

.. _tdl-messaging-steps:

Messaging steps
---------------

Messaging steps allow the test case to handle the exchange of messages between actors. The actual implementation
allowing content to be sent or received is implemented by a messaging handler (see :ref:`introduction-concepts-messaging-handlers`).

.. _tdl-step-btxn:

btxn
~~~~

The ``btxn`` step stands for "Begin transaction". Its purpose is to define a scope around a set of messaging
steps that have a logical relation to each other. This scope remains active until a ``etxn`` element is
encountered to end it. The structure of the ``btxn`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @txnid, yes, A string identifier for the transaction.
    @from, yes, The ID of the actor that acts as the messaging source (see :ref:`test-case-actors`).
    @to, yes, The ID of the actor that acts as the messaging target (see :ref:`test-case-actors`).
    @handler, yes, A string value identifying the the messaging handler for the transaction (see :ref:`handlers-implementation`).
    config, no, Zero or more elements to provide configuration when creating the transaction. Each ``config`` element has a ``name`` attribute and a text content as value.

Executing the ``btxn`` step results in a call to messaging handler specified by the ``handler`` attribute. This gives it an 
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

.. _tdl-step-etxn:

Note that ``btxn`` steps are not presented to the user.

etxn
~~~~

The ``etxn`` step stands for "End transaction" and acts as the counterpart to a ``btxn`` element by referencing its transaction
ID. It is structured as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @txnid, yes, The identifier of the transaction to end.

Executing the ``etxn`` results in a call to the transaction's messaging handler to take necessary action such as resource cleanup.

.. code-block:: xml
    :emphasize-lines: 6

    <btxn from="Actor1" to="Actor2" txnId="t1" handler="SoapMessaging"/>
    <send id="dataSend" desc="Send data" from="Actor1" to="Actor2" txnId="t1">
        <config name="soap.version">1.2</config>
        <input name="soap_message">$soapMessage</input>
    </send>
    <receive id="dataReceive" desc="Receive data" from="Actor2" to="Actor1" txnId="t1"/>
    <etxn txnId="t1"/>

.. _tdl-step-send:

Note that ``etxn`` steps are not presented to the user.

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
    @desc, yes, A description to display to the user for this test step.
    config, no, Zero or more elements containing configuration values pertinent to sending.  Each ``config`` element has a ``name`` attribute and a text content as value.
    input, no, Zero or more elements for the input parameters. See :ref:`handlers-inputs-outputs` for details.

The ``send`` step results in the transaction's messaging handler to be notified that it needs to send content. Recall that the actual
sending always takes place through the message handlier implementation. The ``send`` step simply acts as the signal to do so.

.. code-block:: xml
    :emphasize-lines: 2,3,4,5

    <btxn from="Actor1" to="Actor2" txnId="t1" handler="SoapMessaging"/>
    <send id="dataSend" desc="Send data" from="Actor1" to="Actor2" txnId="t1">
        <config name="soap.version">1.2</config>
        <input name="soap_message">$soapMessage</input>
    </send>
    <etxn txnId="t1"/>

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
    @desc, yes, A description to display to the user for this test step.
    config, no, Zero or more elements containing configuration values pertinent to receiving.  Each ``config`` element has a ``name`` attribute and a text content as value.
    input, no, Zero or more elements for the signal's input parameters. See :ref:`handlers-inputs-outputs` for details.
    output, no, Zero or more elements for the output parameters. See :ref:`handlers-inputs-outputs` for details.

When the test bed execute the ``receive`` step it performs two actions:

#. It signals the transaction's messaging handler that content is expected to be received.
#. It blocks waiting for a callback from the messaging handler that will contain the received data.

Regarding the ``input`` elements provided these act as information provided to the messaging handler that are relevant to the
message's reception. They act as a counterpart to ``config`` elements to allow evaluated expressions (e.g. variable values) to 
be passed considering that ``config`` elements may only contain static strings. The ``output`` elements provided are optional
and serve only to restrict the messaging handler's output (returned via its callback to the test bed) to the specified values.
If not specified all available output values are returned.

.. code-block:: xml
    :emphasize-lines: 2,3,4

    <btxn from="Actor1" to="Actor2" txnId="t1" handler="SoapMessaging"/>
    <receive id="dataReceive" desc="Receive data" from="Actor2" to="Actor1" txnId="t1">
        <config name="soap.version">1.2</config>
    </receive>
    <etxn txnId="t1"/>

.. _tdl-step-listen:

listen
~~~~~~

The ``listen`` step is used to tell the test bed to act as a proxy between messages sent to and from two actors defined as SUTs. 
Similar to the ``send`` and ``receive`` steps, this step is expected to take place within a transaction created by ``btxn``, the 
identifier of which it references. The structure of the ``listen`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @txnid, yes, The ID of the transaction this ``listen`` belongs to.
    @from, yes, The ID of the actor that will be sending the message (see :ref:`test-case-actors`).
    @to, yes, The ID of the actor that will be receiving the message (see :ref:`test-case-actors`).
    config, no, Zero or more elements containing configuration values pertinent to the message exchange.  Each ``config`` element has a ``name`` attribute and a text content as value.
    input, no, Zero or more elements for for the messaging handler to consider. See :ref:`handlers-inputs-outputs` for details.
    output, no, Zero or more elements for the output parameters reported to the test case. See :ref:`handlers-inputs-outputs` for details.

.. note::
    **GITB software support:** The ``listen`` step is currently not supported by the GITB test bed software. As a general note, 
    interoperability tests involving multiple actors as SUTs are not currently supported.

.. _tdl-processing-steps:

Processing steps
----------------

Processing steps are used to handle complex manipulations on information in the test session context that are domain-specific
or too elaborate to be implemented using simple constructs such as the :ref:`tdl-step-assign` step. The actual implementation
that carries out operations is implemented by a processing handler (see :ref:`introduction-concepts-processing-handlers`).

Note that processing steps are not presented to the user.

.. _tdl-step-bptxn:

bptxn
~~~~~

Similar to :ref:`tdl-messaging-steps`, processing occurs in the context of a transaction that acts as a grouping mechanism
over related operations. The ``bptxn`` step (the name stands for "Begin processing transaction") is the construct used to
signal that a processing transaction should be considered as started as is assigned an identifier. Subsequent relevant 
operations will be accompanied by this transaction ID to allow their processing handler to carry them out accordingly.
The structure of the ``btxn`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @txnid, yes, A string identifier for the transaction.
    @handler, yes, A string value identifying the the processing handler for the transaction (see :ref:`handlers-implementation`).
    config, no, Zero or more elements to provide configuration when creating the transaction. Each ``config`` element has a ``name`` attribute and a text content as value.

The ``bptxn`` step results in a call to the configured processing handler to signal it that a new transaction is going to 
start.

.. code-block:: xml
    :emphasize-lines: 1

    <bptxn txnId="t1" handler="https://PROCESSING_SERVICE?wsdl"/>
    <process id="result" txnId="t1">
        <operation>action</operation>
        <input name="anInput">$aValue</input>
    </process>
    <eptxn txnId="t1"/>

.. _tdl-step-eptxn:

eptxn
~~~~~

The ``eptxn`` step (the name stands for "End processing transaction") is the counterpath of the ``bptxn`` step and is used to
close a transaction the ID of which it references. The structure of the ``eptxn`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @txnid, yes, A string identifier for the processing transaction to end.

The ``eptxn`` acts results in a call to the transaction's processing handler to signal that it should consider the transaction as
completed and proceed with any actions such as resource cleanup.

.. code-block:: xml
    :emphasize-lines: 6

    <bptxn txnId="t1" handler="https://PROCESSING_SERVICE?wsdl"/>
    <process id="result" txnId="t1">
        <operation>action</operation>
        <input name="anInput">$aValue</input>
    </process>
    <eptxn txnId="t1"/>

.. _tdl-step-process:

process
~~~~~~~

The ``process`` step is where the actual processing work takes place. This needs to be defined within the context of a
processing transaction started by a ``bptxn`` step, the ID of which is referenced. The structure of the ``process``
element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @txnid, yes, The ID of the transaction to which this processing step belongs.
    @desc, yes, A description for the action taking place within the processing step.
    operation, no, An optional ``string`` to identify an operation the handler is expected to perform.
    input, no, Zero or more elements for the input parameters to the processing step. See :ref:`handlers-inputs-outputs` for details.

The ``operation`` is relevant for processing handlers that can support more than one task. Use of multiple operations under
the same transaction renders processing services quite powerful in that they can perform any number of related operations
and be extended with additional ones if needed.

Carrying out processing operations in a transaction is important as it gives the handler an opportunity to manage
correctly its resources. Moreover, for processing handlers supporting more than one operation, a transaction provides
much needed context to logically connect operations. As an example consider a processing service that is used to read the 
contents from a ZIP archive. If the test case needs to read multiple files at different points in its execution it would be 
possible but very inefficient to pass the ZIP archive in each call. Defining a transaction allows the test case to pass the 
archive once allowing the processing handler to cache it and ultimately remove it upon transaction end. In addition, the 
presence of a transaction provides context and makes operations such as "initialize" (to pass the archive to consider),
"extract" (to get a file's contents), "checkExistence" (to check if a file exists but not return it) possible. Use of such a 
processing service is illustrated in the following example:

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

Flow steps
----------

Flow steps are used to control the processing flow of a test case. The consstructs available are similar to the
flow control structures available in programming languages.

.. _tdl-step-if:

if
~~

The ``if`` step is used to run one of more steps if a condition is met. Its structure is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @desc, yes, A description to display to the user on the purpose of the check.
    cond, yes, The condition to verify in order to execute the ``then`` steps (if true) or ``else`` (if false). This is provided as an expression (see :ref:`test-case-expressions:`).
    then, yes, Contains as children any sequence of steps to execute if the condition results to true.
    else, yes, Contains as children any sequence of steps to execute if the condition results to false.

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

.. note::
    **IF without ELSE:** The TDL specification currently requires that an ``else`` element is always defined for an ``if``. This means that
    even if you don't need to specify an ``else`` block you need to, even if it means adding a step that is not meaningful (e.g. an ``assign``
    that has no effect. This is expected to be adapted in a future version of the specification to skip the ``else`` if not needed.

.. _tdl-step-while:

while
~~~~~

The ``while`` step is the most useful looping construct. It allows a sequence of steps to be continuously executed as long as a condition
continues to result to true. The structure of the ``while`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @desc, yes, A description to display to the user on the purpose of the loop.
    cond, yes, The condition to verify in order to execute the contained steps. This is provided as an expression (see :ref:`test-case-expressions:`).
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
            <verify handler="XPathValidator" desc="The referenced attachment is in the container">
                <input name="xmldocument" source="$document"/>
                <!-- 
                    Construct the XPath expression to use using the iteration variable.
                -->
                <input name="xpathexpression">concat("//*[local-name() = "Attachment"][", $iteration, "]/text() = 'file_", $iteration, ".xml'")</input>
            </verify>
            <!--
                Increment iteration counter.
            -->
            <assign to="$iteration">$iteration + 1</assign>
        </do>
    </while>

.. _tdl-step-repuntil:

repuntil
~~~~~~~~

The `repuntil` step allows you to execute a sequence of steps at least once, checking at the end a condition to see if another iteration
should take place. The structure of the ``repuntil`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @desc, yes, A description to display to the user on the purpose of the loop.
    do, yes, Contains as children any sequence of steps to execute at least once and then again if the condition in ``cond`` is true.
    cond, yes, The condition to verify in order to execute again the steps contained in ``do``. This is provided as an expression (see :ref:`test-case-expressions:`).

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

.. _tdl-step-foreach:

foreach
~~~~~~~

The ``foreach`` step allows you to execute a sequence of steps for a specific number of iterations. Its structure is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @desc, yes, A description to display to the user on the purpose of the loop.
    @start, yes, A number to initialise the iteration index to.
    @end, yes, A number for that is considered as the maximum iteration count plus 1.
    @counter, no, A name for the variable through which to expose the iteration counter (default is "i").
    do, yes, Contains as children any sequence of steps to execute for a loop iteration.

The ``start`` and ``end`` values define the number of iterations to perform. Specifically, the loop will continue as long as
``start`` is less than ``end`` with ``start`` getting incremented by one at the end of each iteration.

.. code-block:: xml

    <!-- 
        The loop will execute 2 times (start must be less than end). The current iteration counter will be
        5 and then 6. Note that referring to this is done as a variable reference (if not specified the variable
        would be named "i" and referred to as "$i").
    -->
    <foreach desc="Do iteration" counter="currentIndex" start="5" end="7">
        <do>
            <interact desc="Message to user" with="User">
                <instruct desc="Iteration: " with="User" type="string">concat("Iteration ", $currentIndex)</instruct>
            </interact>
        </do>
    </foreach>

.. note::
    **Variables for foreach indexes:** The ``foreach`` step currently expects that the ``start`` and ``end`` indexes are provided 
    as fixed numbers. Considering that variable references can't be used this diminishes the effectiveness of this construct. 
    This will likely be addressed in a future version of GITB TDL to allow pure variable references to be used as well.

.. _tdl-step-flow:

flow
~~~~

The ``flow`` step is used to perform sequences of steps in parallel rather that sequentially as is the default. This can be useful
in scenarios where you want to process data in parallel or trigger messaging to actors concurrently. The flow of execution will be 
joined at the end of the ``flow`` step to continue sequential execution. The structure of the ``flow`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @desc, yes, A description to display to the user on the purpose of the forking.
    thread, yes, One or more elements containing as children any sequence of steps to execute in the thread (including other ``flow`` steps).

The following example sends a SOAP request to two actors in parallel and proceeds to send a third one when both actors have replied.

.. code-block:: xml

    <flow desc="Contact actor ReceiverA and ReceiverB in parallel">
        <thread>
            <!--
                Send message to ReceiverA and wait for response.
            -->
            <btxn from="Sender" to="ReceiverA" txnId="t1" handler="SoapMessaging"/>
            <send id="dataSend" desc="Send data" from="Sender" to="ReceiverA" txnId="t1">
                <config name="soap.version">1.2</config>
                <input name="soap_message">$soapMessageForA</input>
            </send>
            <receive id="dataReceive" desc="Receive data" from="ReceiverA" to="Sender" txnId="t1"/>
            <etxn txnId="t1"/>
        </thread>
        <thread>
            <!--
                Send message to ReceiverB and wait for response.
            -->
            <btxn from="Sender" to="ReceiverB" txnId="t2" handler="SoapMessaging"/>
            <send id="dataSend" desc="Send data" from="Sender" to="ReceiverB" txnId="t2">
                <config name="soap.version">1.2</config>
                <input name="soap_message">$soapMessageForB</input>
            </send>
            <receive id="dataReceive" desc="Receive data" from="ReceiverB" to="Sender" txnId="t2"/>
            <etxn txnId="t2"/>
        </thread>
    </flow>
    <!-- 
        After ReceiverA and ReceiverB have responded send a message to ReceiverC.
    -->
    <btxn from="Sender" to="ReceiverC" txnId="t3" handler="SoapMessaging"/>
    <send id="dataSend" desc="Send data" from="Sender" to="ReceiverC" txnId="t3">
        <config name="soap.version">1.2</config>
        <input name="soap_message">$soapMessageForC</input>
    </send>
    <etxn txnId="t3"/>

.. _tdl-step-exit:

exit
~~~~

The ``exit`` step is used to immediately exit the test case from any execution branch. Triggerring this step will result in the 
test session having an ``UNDEFINED`` result. The structure of the ``exit`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @desc, yes, A description to display for the ``exit`` step.
    
The following example shows a test case that exits based on the user's input:

.. code-block:: xml
    :emphasize-lines: 8

    <assign to="$inputValue">'NO'</assign>
    <interact desc="Provide your choice" with="User">
        <request desc="Enter 'YES' to end the test" with="User">$inputValue</request>
    </interact>
    <if>
        <cond>$inputValue = 'YES'</cond>
        <then>
            <exit desc="Terminate test"/>
        </then>
        <else>
            <interact desc="Provide your choice" with="User">
                <instruct desc="Test continues" with="User" type="string">""</instruct>
            </interact>
            <verify handler="XSDValidator" desc="Validate content">
                <input name="xmldocument">$document</input>
                <input name="xsddocument">$schemaFile"</input>
            </verify>
        </else>
    </if>

.. note::
    **GITB software support:** The ``exit`` step currently successfully terminates a test session but this is not reflected on the
    user interface. The session appears still running with the ``exit`` step pending. The user has to manually select to ``Stop``
    the session.

Support steps
-------------

Support steps are those that perform specific actions not related to messaging, processing or flow control. 

.. _tdl-step-assign:

assign
~~~~~~

The ``assign`` step is a frequently used construct in GITB TDL. It is used to assign values to variables but also as a means of 
performing simple processing on the data stored in the session's context. The processing and assignment result is determined by
an expression provided as the text content of the ``assign`` element (see :ref:`test-case-expressions`). The element's structure 
is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @to, yes, The target variable to assign the result of the expression to.
    @append, no, Used if the ``to`` variable is a ``list`` to append the result to.
    @type, no, Used to specify the type of variable to create if the ``to`` is an entry in a ``map``.
    @lang, no, The expression language prefix to use to evaluate the contained expression (see :ref:`test-case-namespaces` and :ref:`test-case-expressions`).
    @source, no, A variable reference to identify a source ``object`` variable upon which the expression should be evaluated.

The following example illustrates assigning a value to a ``number`` variable and also counting the nodes in an XML document:

.. code-block:: xml

    <assign to="$value">1</assign>
    <assign to="$nodeCount" source="$document">count(//*[local-name() = "Attachment"]</assign>

Multiple further examples per variable type are provided in the documentation of :ref:`test-case-expressions`. Note that ``assign`` steps are not presented 
to the user.

.. _tdl-step-group:

group
~~~~~

The ``group`` step is a construct used to visually group together a sequence of steps. It has no effect on the test execution adding only
a visual grouping and label to the display. Its structure is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @desc, yes, The description for the group.

The children of the ``group`` element can be any number of steps supported by GITB TDL. The following example creates a group around a set of 
related validations.

.. code-block:: xml

    <group id="groupID" desc="Validate document">
		<verify handler="XSDValidator" desc="Validate invoice against schema">
            <input name="xmldocument">$document</input>
            <input name="xsddocument">$schema"</input>
        </verify>
        <verify handler="SchematronValidator" desc="Validate against Schematron 1">
            <input name="xmldocument">$document</input>
            <input name="schematron">$schematron1"</input>
        </verify>
        <verify handler="SchematronValidator" desc="Validate against Schematron 2">
            <input name="xmldocument">$document</input>
            <input name="schematron">$schematron2"</input>
        </verify>
    </group>

.. note::
    **GITB software support:** The ``group`` step is currently not supported. Using it will execute the contained steps but these will not be
    rendered on the user interface.

.. _tdl-step-verify:

verify
~~~~~~

The ``verify`` step is used to trigger validation of content. Similar to :ref:`tdl-messaging-steps` and  :ref:`tdl-messaging-steps`, validation
takes place using a validation handler implementation that can either be an embedded test bed component or a remote service that implements the
GITB validation service API. The content to validate is provided by the test case to the handler in terms of configuration and input, for which
a test report is returned in the GITB TRL (Test Reporting Language) format. The structure of the ``verify`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @desc, yes, The description for the validation.
    @handler, yes, A string value identifying the the validation handler (see :ref:`handlers-implementation`).
    config, no, Zero or more elements to provide configuration for the validation. Each ``config`` element has a ``name`` attribute and a text content as value.
    input, yes, One more elements for the validation's input parameters. See :ref:`handlers-inputs-outputs` for details.

The following example illustrates use of two ``verify`` steps, one using an XSDValidator and the other calling a remote validation service:

.. code-block:: xml

    <!-- 
        Validation using the embedded XSDValidator.
    -->
    <verify handler="XSDValidator" desc="Validate invoice against schema">
        <input name="xmldocument">$document</input>
        <input name="xsddocument">$schema"</input>
    </verify>
    <!-- 
        Validation using a remote validation service.
    -->
    <verify handler="https://VALIDATION_SERVICE_ADDRESS?wsdl" desc="Validate against remote service">
        <input name="aDocument">$document</input>
    </verify>

.. note::
    **Remote or local validators:** Simple validations such as those evaluating an XPath expression against a document can be implemented using embedded 
    test bed validators. When validation logic however is complex is is always best to decouple this into an external validation service. This is the case
    even in simple XML content validation since this usually involves multiple validation steps using an XSD and one or more Schematron files. It is more
    concise to present this as a single validation step with one report. This also enhances maintainability of the test cases considering that use of embedded
    ``XSDValidator`` and ``SchematronValidator`` means that you need to bundle (and maintain) the validation artefacts in each test suite. When decoupled as a service 
    artefacts can be updated transparently to related test cases aside from the benefit that your service can also be invoked outside the test bed using any
    SOAP client.

.. _tdl-step-call:

call
~~~~

.. _tdl-step-interact:

interact
~~~~~~~~

NEEDS TO SPECIFY TYPE
INSTRUCTION NEEDS TO SPECIFY A MESSAGE EVEN IF EMPTY