.. _tdl-steps:

Test steps
==========

Overview
--------

TDL test steps are used to capture a test case's core testing logic. They are used in test cases and
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
.. index:: skipped (btxn)
.. index:: stopOnError (btxn)
.. index:: property (btxn)
.. index:: config (btxn)
.. index:: handlerTimeout (btxn)
.. _tdl-step-btxn:

btxn
~~~~

The ``btxn`` step stands for "Begin transaction". Its purpose is to define a scope around a set of messaging
steps that have a logical relation to each other. This scope remains active until a ``etxn`` element is
encountered to end it. Apart from wrapping together related messaging steps, the key purpose of the ``btxn`` step
is to declare the :ref:`messaging handler<handlers>` that is used and the involved actors.

Use of a messaging transaction is not necessary as you can also define the handler and involved actors on the individual
messaging steps. It could nonetheless still be interesting to use transactions to avoid repeating the handler's definition.

The structure of the ``btxn`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @from, no, The ID of the actor that acts as the messaging source (see :ref:`test-case-actors`). Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.
    @handler, yes, A string value or variable reference identifying the messaging handler to use for the transaction (see :ref:`handlers-implementation`).
    @handlerTimeout, no, A number or variable reference with the maximum time (in milliseconds) to wait for the handler service call to complete (in case of an external test service being used as a handler). See also :ref:`tdl-steps-common-handlerTimeouts`.
    @skipped, no, A boolean value or variable reference (default being "false") which will result in the step being skipped if "true" See also :ref:`tdl-steps-common-skipped`.
    @stopOnError, no, A boolean flag determining whether the test session should end if this step fails (default is "false"). See also :ref:`tdl-steps-common-stoponerror`.
    @to, no, The ID of the actor that acts as the messaging target (see :ref:`test-case-actors`). Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.
    @txnid, yes, A string ID for the transaction.
    config, no, Zero or more elements to provide configuration when creating the transaction. Each ``config`` element has a ``name`` attribute and a text content or variable reference as value.
    property, no, Zero or more elements to provide configuration regarding the setup of the messaging handler call that are not passed to the handler. Each ``property`` element has a ``name`` attribute and a text content or variable reference as value.

Executing the ``btxn`` step results in a call to the messaging handler specified by the ``handler`` attribute. This gives it an 
opportunity to take any actions needed for the upcoming transaction and apply specific configurations for its related ``send``
and ``receive`` calls.

.. code-block:: xml
    :emphasize-lines: 1

    <btxn txnId="t1" handler="HttpMessagingV2"/>
    <send id="dataSend" desc="Send data" txnId="t1">
        <input name="uri">"https://my.sut.org/api/get"</input>
        <input name="method">"GET"</input>
    </send>
    <receive id="dataReceive" desc="Receive data" txnId="t1">
        <input name="method">"GET"</input>
    </receive>
    <etxn txnId="t1"/>

Note that ``btxn`` steps are not presented to the user.

.. index:: etxn
.. index:: txnid (etxn)
.. index:: skipped (etxn)
.. index:: stopOnError (etxn)
.. _tdl-step-etxn:

etxn
~~~~

The ``etxn`` step stands for "End transaction" and acts as the counterpart to a ``btxn`` element by referencing its transaction
ID. It is structured as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @skipped, no, A boolean value or variable reference (default being "false") which will result in the step being skipped if "true" See also :ref:`tdl-steps-common-skipped`.
    @stopOnError, no, A boolean flag determining whether the test session should end if this step fails (default is "false"). See also :ref:`tdl-steps-common-stoponerror`.
    @txnid, yes, The identifier of the transaction to end.

Executing the ``etxn`` results in a call to the transaction's messaging handler to take necessary actions such as resource clean-up.

.. code-block:: xml
    :emphasize-lines: 9

    <btxn from="Actor1" to="Actor2" txnId="t1" handler="HttpMessagingV2"/>
    <send id="dataSend" desc="Send data" from="Actor1" to="Actor2" txnId="t1">
        <input name="uri">"https://my.sut.org/api/get"</input>
        <input name="method">"GET"</input>
    </send>
    <receive id="dataReceive" desc="Receive data" from="Actor2" to="Actor1" txnId="t1">
        <input name="method">"GET"</input>
    </receive>
    <etxn txnId="t1"/>

Note that ``etxn`` steps are not presented to the user.

.. index:: listen
.. index:: txnid (listen)
.. index:: from (listen)
.. index:: to (listen)
.. index:: id (listen)
.. index:: skipped (listen)
.. index:: stopOnError (listen)
.. index:: documentation (listen)
.. index:: config (listen)
.. index:: input (listen)
.. index:: output (listen)
.. index:: hidden (listen)
.. index:: reply (listen)
.. index:: handler (listen)
.. index:: level (listen)
.. index:: ERROR (listen)
.. index:: WARNING (listen)    
.. index:: invert (listen)
.. _tdl-step-listen:

listen
~~~~~~

The ``listen`` step is used to instruct the Test Bed to act as a proxy between messages sent to and from two actors defined as SUTs. 
Similar to the ``send`` and ``receive`` steps, this step is expected to take place within a transaction created by ``btxn``, the 
identifier of which it references. The structure of the ``listen`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"
    :delim: |

    @from | no | The ID of the actor (defaulting to the SUT actor) that will be sending the message (see :ref:`test-case-actors`). Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.
    @handler | no | The :ref:`messaging handler<handlers>` to use for this messaging step. If not specified (for transactional messaging) the ``txnid`` attribute is required.
    @hidden | no | A boolean flag determining whether or not the step is displayed to users (default is "false"). Note that within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`. See :ref:`tdl-steps-common-hidesteps` for further details.
    @id | no | The ID for the step. This is also the name of a ``map`` variable in the session context in which output will be stored.
    @invert | no | A boolean flag determining whether the step's result should be inverted (default is "false"). Setting to "true" will expect a communication failure to complete the step as a success.
    @level | no | The severity level to be considered when this step fails. Can be set to ``ERROR`` (the default) or ``WARNING``, or be defined dynamically via :ref:`variable reference<test-case-referring-to-variables>`. See :ref:`tdl-steps-common-level` for further details.
    @reply | no | A boolean flag indicating that this communication should be presented as a reply (default is "false"). Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.
    @skipped | no | A boolean value or variable reference (default being "false") which will result in the step being skipped if "true" See also :ref:`tdl-steps-common-skipped`.
    @stopOnError | no, A boolean flag determining whether the test session should end if this step fails (default is "false"). See also :ref:`tdl-steps-common-stoponerror`.
    @to | no | The ID of the actor (defaulting to the non-SUT actor if one is defined) that will be receiving the message (see :ref:`test-case-actors`). Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.
    @txnid | no | The ID of the transaction this ``listen`` belongs to. If not specified (for non-transactional messaging) the ``handler`` attribute is required.
    config | no | Zero or more elements containing configuration values pertinent to the message exchange.  Each ``config`` element has a ``name`` attribute and a text content or variable reference as value.
    documentation | no | Rich text content that provides further information on the current step.
    input | no | Zero or more elements for for the messaging handler to consider. See :ref:`handlers-inputs-outputs` for details.
    output | no | Zero or more elements for the output values reported back to the test case. See :ref:`handlers-inputs-outputs` for details.
    property | no | Zero or more elements to provide configuration regarding the setup of the messaging handler call that are not passed to the handler. Each ``property`` element has a ``name`` attribute and a text content or variable reference as value.

.. note::
    **GITB software support:** The ``listen`` step is currently not supported. As a general note, 
    interoperability tests involving multiple actors as SUTs are not currently possible.

.. index:: receive
.. index:: txnid (receive)
.. index:: from (receive)
.. index:: to (receive)
.. index:: desc (receive)
.. index:: id (receive)
.. index:: timeout (receive)
.. index:: timeoutFlag (receive)
.. index:: timeoutIsError (receive)
.. index:: skipped (receive)
.. index:: stopOnError (receive)
.. index:: documentation (receive)
.. index:: config (receive)
.. index:: input (receive)
.. index:: output (receive)
.. index:: hidden (receive)
.. index:: reply (receive)
.. index:: handler (receive)
.. index:: level (receive)
.. index:: ERROR (receive)
.. index:: WARNING (receive)  
.. index:: invert (receive)
.. index:: handlerTimeout (receive)
.. index:: handlerTimeoutFlag (receive)
.. _tdl-step-receive:

receive
~~~~~~~

The ``receive`` step is the counterpart of ``send`` signalling that an actor is expected to receive a message from another. This 
operation may be defined as part of a transaction created by ``btxn``, in which case it references the transaction's identifier. 
Alternatively, it may work without a transaction by specifying directly the :ref:`messaging handler implementation<handlers-implementation>` to use.

The structure of the ``receive`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"
    :delim: |

    @desc | no | A description for this step to display to the user and to include in the test session log. Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.
    @from | no | The ID of the actor (defaulting to the SUT actor) that will be sending the message (see :ref:`test-case-actors`). Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.
    @handler | no | The :ref:`messaging handler<handlers>` to use for this messaging step. If not specified (for transactional messaging) the ``txnid`` attribute is required.
    @handlerTimeout | no | A number or variable reference with the maximum time (in milliseconds) to wait for the handler service call to complete (in case of an external test service being used as a handler). See also :ref:`tdl-steps-common-handlerTimeouts`.
    @handlerTimeoutFlag | no | A string value with the name of a boolean variable to set informing whether or not a handler timeout occurred. See also :ref:`tdl-steps-common-handlerTimeouts`.
    @hidden | no | A boolean flag determining whether or not the step is displayed to users (default is "false"). Note that within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`. See :ref:`tdl-steps-common-hidesteps` for further details.
    @id | no | The ID for the step. This is also the name of a ``map`` variable in the session context in which output will be stored.
    @invert | no | A boolean flag determining whether the step's result should be inverted (default is "false"). Setting to "true" will expect a communication failure to complete the step as a success.
    @level | no | The severity level to be considered when this step fails. Can be set to ``ERROR`` (the default) or ``WARNING``, or be defined dynamically via :ref:`variable reference<test-case-referring-to-variables>`. See :ref:`tdl-steps-common-level` for further details.
    @reply | no | A boolean flag indicating that this communication should be presented as a reply (default is "false"). Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.
    @skipped | no | A boolean value or variable reference (default being "false") which will result in the step being skipped if "true" See also :ref:`tdl-steps-common-skipped`.
    @stopOnError | no | A boolean flag determining whether the test session should end if this step fails (default is "false"). See also :ref:`tdl-steps-common-stoponerror`.
    @timeout | no | An optional timeout (in milliseconds) on the time to wait for a message to be received. This is provided as a ``number`` or a variable reference.
    @timeoutFlag | no | An optional name for a boolean flag to record whether or not the timeout was triggered that will be stored in the result ``map`` named using the ``id`` attribute. This is provided as a ``string`` or a variable reference.
    @timeoutIsError | no | Whether or not a timeout being triggered should be considered as an error or success (the default). This is provided as a ``boolean`` or a variable reference.
    @to | no | The ID of the actor (defaulting to the non-SUT actor if one is defined) that will be receiving the message (see :ref:`test-case-actors`). Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.
    @txnid | no | The ID of the transaction this ``receive`` belongs to. If not specified (for non-transactional messaging) the ``handler`` attribute is required.
    config | no | Zero or more elements containing configuration values pertinent to receiving.  Each ``config`` element has a ``name`` attribute and a text content or variable reference as value.
    documentation | no | Rich text content that provides further information on the current step.
    input | no | Zero or more elements for the signal's input parameters. See :ref:`handlers-inputs-outputs` for details.
    output | no | Zero or more elements for the resulting output values. See :ref:`handlers-inputs-outputs` for details.
    property | no | Zero or more elements to provide configuration regarding the setup of the messaging handler call that are not passed to the handler. Each ``property`` element has a ``name`` attribute and a text content or variable reference as value.

When the Test Bed executes the ``receive`` step it performs two actions:

#. It signals the step's messaging handler that content is expected to be received.
#. It blocks waiting for a call-back from the messaging handler that will contain the received data, or until the configured timeout has elapsed.

Regarding the ``input`` elements provided these act as information provided to the messaging handler that are relevant to the
message's reception. They act as a counterpart to ``config`` elements whose purpose is more to signal parameters for the communication
setup rather than the involved message. The ``output`` elements provided are optional and serve only to restrict the messaging handler's
output (returned via its call-back to the Test Bed) to the specified values. If not specified all available output values are returned.

.. code-block:: xml
    :emphasize-lines: 1,2,3,7,8,9,10,11

    <!-- 
        Example receiving a message sent by the SUT actor to the (assumed single) non-SUT actor.
    -->
    <receive id="dataReceive" desc="Receive data" handler="HttpMessagingV2">
        <input name="method">"GET"</input>
    </receive>
    <!--
        Example specifying the step's "to" actor in case we have multiple non-SUT actors defined.
    -->
    <receive id="dataReceiveWithExplicitRecipient" desc="Receive data" to="Actor1" handler="HttpMessagingV2">
        <input name="method">"GET"</input>
    </receive>
    <!--
        Example specifying the step's "to" and "from" actors in case we want to show a different
        messaging direction.
    -->
    <receive id="dataReceiveWithExplicitActors" desc="Receive data" from="Actor1" to="Actor2" handler="SimulatedMessaging"/>
    <!--
        Example using timeouts (that are considered as an error).
    -->
    <receive 
      id="dataReceiveTimeout" desc="Receive data with timeout" handler="HttpMessagingV2"
      timeout="$maxWaitTime" timeoutFlag="timeoutOccurred" timeoutIsError="true">
        <input name="method">"GET"</input>
    </receive>
    <!--
        Check to see if timeout took place or not and inform the user.
    -->
    <interact desc="Check timeout status">
        <instruct desc="Timeout occurred:">$dataReceiveTimeout{timeoutOccurred}</instruct>
    </interact>

You may also choose to define a ``receive`` step as part of a transaction (via :ref:`btxn<tdl-step-btxn>`). The following is an example of such a
step that makes use of an :ref:`external service handler<handlers-implementation>` (a messaging service) to carry out the messaging:

.. code-block:: xml

    <btxn txnId="t1" from="Actor1" to="Actor2" handler="$DOMAIN{messagingService}"/>
    <receive id="dataReceive" desc="Receive data" from="Actor1" to="Actor2" txnId="t1">
        <input name="expectedMessageId">$messageId</input>
    </receive>
    <etxn txnId="t1"/>

.. note::
    **Parallel receives:** In case you use the ``receive`` step within a :ref:`flow<tdl-step-flow>` step's threads and a
    :ref:`custom messaging service<handlers>`, you need to make sure your service manages the specific receive call's identifier.
    Check the `messaging service documentation`_ for details on how to do this.

.. _tdl-step-receive_actors:

Explicitly specifying from and to actors
++++++++++++++++++++++++++++++++++++++++

The ``receive`` step forces the test session to pause until a message is received. In terms of actors, this means in most cases that 
the actor marked as the System Under Test (SUT) will be sending the message, whereas the actor simulated by the test engine will
be the recipient. Furthermore, often only a single simulated actor is defined in a test case, meaning that the ``from`` and ``to``
actors of the ``receive`` step can be automatically determined.

Considering the above, defining the ``from`` actor is optional as it defaults to the test case SUT actor. Similarly, the ``to``
actor is also optional as long as the test case defines only a single non-SUT actor. You may still need to explicitly define the
``from`` and ``to`` actors to cover the following cases:

* Your test case defines multiple non-SUT actors meaning that you must specify the ``to`` actor.
* The ``receive`` step is handled by the :ref:`SimulatedMessaging handler <handlers-simulatedmessaging>` and you want to specify
  an exchange between actors with a direction other than the default.

.. note::
    Omitting the ``receive`` step's ``to`` actor when the test case does not have a single non-SUT (simulated) actor will fail test suite validation.

.. index:: send
.. index:: txnid (send)
.. index:: from (send)
.. index:: to (send)
.. index:: desc (send)
.. index:: id (send)
.. index:: skipped (send)
.. index:: stopOnError (send)
.. index:: documentation (send)
.. index:: config (send)
.. index:: input (send)
.. index:: hidden (send)
.. index:: reply (send)
.. index:: handler (send)
.. index:: level (send)
.. index:: ERROR (send)
.. index:: WARNING (send)    
.. index:: invert (send)
.. index:: handlerTimeout (send)
.. index:: handlerTimeoutFlag (send)
.. _tdl-step-send:

send
~~~~

The ``send`` step allows the Test Bed to signal that content needs to be sent from one actor to another. This 
operation may be defined as part of a transaction created by ``btxn``, in which case it references the transaction's identifier. 
Alternatively, it may work without a transaction by specifying directly the :ref:`messaging handler implementation<handlers-implementation>` to use.

The structure of the ``send`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"
    :delim: |

    @desc | no | A description for this step to display to the user and to include in the test session log. Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.
    @from | no | The ID of the actor (defaulting to the non-SUT actor if one is defined) that will be sending the message (see :ref:`test-case-actors`). Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.
    @handler | no | The :ref:`messaging handler<handlers>` to use for this messaging step. If not specified (for transactional messaging) the ``txnid`` attribute is required.
    @handlerTimeout | no | A number or variable reference with the maximum time (in milliseconds) to wait for the handler service call to complete (in case of an external test service being used as a handler). See also :ref:`tdl-steps-common-handlerTimeouts`.
    @handlerTimeoutFlag | no | A string value with the name of a boolean variable to set informing whether or not a handler timeout occurred. See also :ref:`tdl-steps-common-handlerTimeouts`.
    @hidden | no | A boolean flag determining whether or not the step is displayed to users (default is "false"). Note that within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`. See :ref:`tdl-steps-common-hidesteps` for further details.
    @id | no | The ID for the step. This is also the name of a ``map`` variable in the session context in which output will be stored.
    @invert | no | A boolean flag determining whether the step's result should be inverted (default is "false"). Setting to "true" will expect a communication failure to complete the step as a success.
    @level | no | The severity level to be considered when this step fails. Can be set to ``ERROR`` (the default) or ``WARNING``, or be defined dynamically via :ref:`variable reference<test-case-referring-to-variables>`. See :ref:`tdl-steps-common-level` for further details.
    @reply | no | A boolean flag indicating that this communication should be presented as a reply (default is "false"). Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.
    @skipped | no | A boolean value or variable reference (default being "false") which will result in the step being skipped if "true" See also :ref:`tdl-steps-common-skipped`.
    @stopOnError | no | A boolean flag determining whether the test session should end if this step fails (default is "false"). See also :ref:`tdl-steps-common-stoponerror`.
    @to | no | The ID of the actor (defaulting to the SUT actor) that will be receiving the message (see :ref:`test-case-actors`). Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.
    @txnid | no | The ID of the transaction this ``send`` belongs to. If not specified (for non-transactional messaging) the ``handler`` attribute is required.
    config | no | Zero or more elements containing configuration values pertinent to sending.  Each ``config`` element has a ``name`` attribute and a text content or variable reference as value.
    documentation | no | Rich text content that provides further information on the current step.
    input | no | Zero or more elements for the input parameters. See :ref:`handlers-inputs-outputs` for details.
    property | no | Zero or more elements to provide configuration regarding the setup of the messaging handler call that are not passed to the handler. Each ``property`` element has a ``name`` attribute and a text content or variable reference as value.

The ``send`` step results in the messaging handler being notified that it needs to send content. Recall that the actual
sending always takes place through the message handler implementation. The ``send`` step simply acts as the signal to do so.

.. code-block:: xml

    <!-- 
        Example sending a message to the SUT actor from the (assumed single) non-SUT actor.
    -->
    <send id="dataSend" desc="Send data" handler="HttpMessagingV2">
        <input name="uri">"https://my.sut.org/api/get"</input>
        <input name="method">"GET"</input>
    </send>
    <!--
        Example specifying the step's "from" actor in case we have multiple non-SUT actors defined.
    -->    
    <send id="dataSendWithExplicitSender" desc="Send data" from="Actor2" handler="HttpMessagingV2">
        <input name="uri">"https://my.sut.org/api/get"</input>
        <input name="method">"GET"</input>
    </send>
    <!--
        Example specifying the step's "to" and "from" actors in case we want to show a different
        messaging direction.
    -->
    <send id="dataSendWithExplicitActors" desc="Send data" from="Actor1" to="Actor2" handler="SimulatedMessaging"/>

You may also choose to define a ``send`` step as part of a transaction (via :ref:`btxn<tdl-step-btxn>`). The following is an example of such a
step that makes use of an :ref:`external service handler<handlers-implementation>` (a messaging service) to carry out the messaging:

.. code-block:: xml

    <btxn from="Actor1" to="Actor2" txnId="t1" handler="$DOMAIN{messagingService}"/>
    <send id="dataSend" desc="Send data" from="Actor1" to="Actor2" txnId="t1">
        <input name="messageId">$messageId</input>
        <input name="messagePayload">$messagePayload</input>
    </send>
    <etxn txnId="t1"/>

.. _tdl-step-send_actors:

Explicitly specifying from and to actors
++++++++++++++++++++++++++++++++++++++++

The ``step`` results in the test engine sending a synchronous request to a recipient. In terms of actors, this means in most cases
that the actor marked as the System Under Test (SUT) will be receiving the message, whereas the actor simulated by the test engine will
be the sender. Furthermore, often only a single simulated actor is defined in a test case, meaning that the ``from`` and ``to``
actors of the ``send`` step can be automatically determined.

Considering the above, defining the ``to`` actor is optional as it defaults to the test case SUT actor. Similarly, the ``from``
actor is also optional as long as the test case defines only a single non-SUT actor. You may still need to explicitly define the
``from`` and ``to`` actors to cover the following cases:

* Your test case defines multiple non-SUT actors meaning that you must specify the ``from`` actor.
* The ``send`` step is handled by the :ref:`SimulatedMessaging handler <handlers-simulatedmessaging>` and you want to specify
  an exchange between actors with a direction other than the default.

.. note::
    Omitting the ``send`` step's ``from`` actor when the test case does not have a single non-SUT (simulated) actor will fail test suite validation.

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
.. index:: skipped (bptxn)
.. index:: stopOnError (bptxn)
.. index:: property (bptxn)
.. index:: config (bptxn)
.. index:: handlerTimeout (bptxn)
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

    @handler, yes, A string value or variable reference identifying the the processing handler for the transaction (see :ref:`handlers-implementation`).
    @handlerTimeout, no, A number or variable reference with the maximum time (in milliseconds) to wait for the handler service call to complete (in case of an external test service being used as a handler). See also :ref:`tdl-steps-common-handlerTimeouts`.
    @skipped, no, A boolean value or variable reference (default being "false") which will result in the step being skipped if "true" See also :ref:`tdl-steps-common-skipped`.
    @stopOnError, no, A boolean flag determining whether the test session should end if this step fails (default is "false"). See also :ref:`tdl-steps-common-stoponerror`.
    @txnid, yes, A string identifier for the transaction.
    config, no, Zero or more elements to provide configuration when creating the transaction. Each ``config`` element has a ``name`` attribute and a text content or variable reference as value.
    property, no, Zero or more elements to provide configuration regarding the setup of the processing handler call that are not passed to the handler. Each ``property`` element has a ``name`` attribute and a text content or variable reference as value.

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
.. index:: skipped (eptxn)
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
    @skipped, no, A boolean value or variable reference (default being "false") which will result in the step being skipped if "true" See also :ref:`tdl-steps-common-skipped`.
    @stopOnError, no, A boolean flag determining whether the test session should end if this step fails (default is "false"). See also :ref:`tdl-steps-common-stoponerror`.

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
.. index:: skipped (process)
.. index:: stopOnError (process)
.. index:: documentation (process)
.. index:: operation (process)
.. index:: input (process)
.. index:: output (process)
.. index:: hidden (process)
.. index:: level (process)
.. index:: ERROR (process)
.. index:: WARNING (process)
.. index:: invert (process)
.. index:: handlerTimeout (process)
.. index:: handlerTimeoutFlag (process)
.. index:: actor (process)
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
    @skipped | no | A boolean value or variable reference (default being "false") which will result in the step being skipped if "true" See also :ref:`tdl-steps-common-skipped`.
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

.. index:: exit
.. index:: desc (exit)
.. index:: success (exit)
.. index:: undefined (exit)
.. index:: documentation (exit)
.. index:: hidden (exit)
.. _tdl-step-exit:

exit
~~~~

The ``exit`` step is used to immediately exit the test case from any execution branch. The structure of the ``exit`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @desc, no, A description for the step to display to the user and to include in the test session log. Within scriptlets this can also be a :ref:`variable reference<scriptlets_dynamic_references>`.
    @hidden, no, A boolean flag determining whether or not the step is displayed to users (default is "false"). Note that within scriptlets this can also be a :ref:`variable reference<scriptlets_dynamic_references>`. See :ref:`tdl-steps-common-hidesteps` for further details.
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

.. index:: flow
.. index:: title (flow)
.. index:: desc (flow)
.. index:: skipped (flow)
.. index:: stopOnError (flow)
.. index:: documentation (flow)
.. index:: thread (flow)
.. index:: hidden (flow)
.. index:: hidden (thread)
.. index:: collapsed (flow)
.. index:: stopOnChildError (flow)
.. _tdl-step-flow:

flow
~~~~

The ``flow`` step is used to perform sequences of steps in parallel rather that sequentially as is the default. This can be useful
in scenarios where you want to process data in parallel or trigger messaging to actors concurrently. The flow of execution will be 
joined at the end of the ``flow`` step to continue sequential execution. The structure of the ``flow`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @collapsed, no, A boolean flag determining whether or not the step is displayed as initially collapsed (default is "false"). See also :ref:`tdl-steps-common-collapsed`.
    @desc, no, A description for this thread fork to display to the user and to include in the test session log. Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.
    @hidden, no, A boolean flag determining whether or not the step is displayed to users (default is "false"). Note that within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`. See :ref:`tdl-steps-common-hidesteps` for further details.
    @skipped, no, A boolean value or variable reference (default being "false") which will result in the step being skipped if "true" See also :ref:`tdl-steps-common-skipped`.
    @stopOnError, no, A boolean flag determining whether the test session should end if this step fails (default is "false"). See also :ref:`tdl-steps-common-stoponerror`.
    @title, no, A short title to display for this step (default is "flow"). Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.
    documentation, no, Rich text content that provides further information on the current step.
    thread, yes, One or more elements containing as children any sequence of steps to execute in the thread (including other ``flow`` steps).

The following example makes a HTTP GET to two API endpoints in parallel and proceeds to call a third one when both requests have completed.

.. code-block:: xml

    <flow desc="Contact endpoints api1 and api2 in parallel">
        <thread>
            <!--
                Call api1 and wait for response.
            -->
            <send id="dataSend1" desc="Call API 1" from="Sender" to="Receiver" handler="HttpMessagingV2">
                <input name="uri">"https://my.sut.org/api1/get"</input>
                <input name="method">"GET"</input>
            </send>
        </thread>
        <thread>
            <!--
                Call api2 and wait for response.
            -->
            <send id="dataSend1" desc="Call API 2" from="Sender" to="Receiver" handler="HttpMessagingV2">
                <input name="uri">"https://my.sut.org/api2/get"</input>
                <input name="method">"GET"</input>
            </send>
        </thread>
    </flow>
    <!-- 
        After both requests have completed make a new HTTP GET to api3.
    -->
    <send id="dataSend" desc="Call API 3"  from="Sender" to="Receiver" handler="HttpMessagingV2">
        <input name="uri">"https://my.sut.org/api3/get"</input>
        <input name="method">"GET"</input>
    </send>

A ``flow`` step's separate threads can also be **individually hidden** through use of their ``hidden`` attribute. This could be useful 
if you need to execute and display a set of parallel test branches, but at the same time carry out a parallel operation that
shouldn't be displayed. The following example illustrates a scenario where two messages are sent in parallel, with an additional
notification that is hidden.

.. code-block:: xml
    :emphasize-lines: 10

    <flow desc="Send messages">
        <thread>
            <log>"Calling API 1"</log>
            <send id="dataSendA" desc="Call API 1" from="Sender" to="Receiver" handler="HttpMessagingV2">...</send>
        </thread>
        <thread>
            <log>"Calling API 2"</log>
            <send id="dataSendB" desc="Call API 2" from="Sender" to="Receiver" handler="HttpMessagingV2">...</send>
        </thread>
        <thread hidden="true">
            <log>"Calling clean-up API"</log>
            <send id="cleanUp" from="Sender" to="Receiver" handler="HttpMessagingV2">...</send>
        </thread>
    </flow>

.. note::
    **Parallel receives:** In case you use the :ref:`receive<tdl-step-receive>` step within a ``flow`` step's threads and a
    :ref:`custom messaging service<handlers>`, you need to make sure your service manages the specific receive call's identifier.
    Check the `messaging service documentation`_ for details on how to do this.

.. index:: foreach
.. index:: title (foreach)
.. index:: desc (foreach)
.. index:: start (foreach)
.. index:: end (foreach)
.. index:: counter (foreach)
.. index:: skipped (foreach)
.. index:: stopOnError (foreach)
.. index:: documentation (foreach)
.. index:: do (foreach)
.. index:: hidden (foreach)
.. index:: collapsed (foreach)
.. index:: stopOnChildError (foreach)
.. index:: item (foreach)
.. index:: of (foreach)
.. _tdl-step-foreach:

foreach
~~~~~~~

The ``foreach`` step allows you to execute a sequence of steps for a specific number of iterations or to iterate over a set of items. Its structure is as follows:

.. csv-table::
    :stub-columns: 1
    :delim: |
    :header: "Name", "Required?", "Description"

    @collapsed | no | A boolean flag determining whether or not the step is displayed as initially collapsed (default is "false"). See also :ref:`tdl-steps-common-collapsed`.
    @counter | no | A name for the variable through which to expose the iteration counter (default is "i").
    @desc | no | A description for this loop to display to the user and to include in the test session log. Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.
    @end | no | A number considered as the maximum iteration count plus 1, provided as a constant or as a variable reference. If the ``of`` attribute is not present, the `end` attribute becomes mandatory.
    @hidden | no | A boolean flag determining whether or not the step is displayed to users (default is "false"). Note that within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`. See :ref:`tdl-steps-common-hidesteps` for further details.
    @item | no | The name of a variable that will hold the current iteration's item. This applies only when iterating over a collection provided through the ``of`` attribute.
    @of | no | A variable reference for a collection (map or list) to iterate over. If this is not provided, the ``end`` attribute becomes mandatory.
    @skipped | no | A boolean value or variable reference (default being "false") which will result in the step being skipped if "true" See also :ref:`tdl-steps-common-skipped`.
    @start | no | A number to initialise the zero-based iteration index with, provided as a constant or as a variable reference.
    @stopOnError | no | A boolean flag determining whether the test session should end if this step fails (default is "false"). See also :ref:`tdl-steps-common-stoponerror`.
    @title | no | A short title to display for this step (default is "loop"). Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.
    do | yes | Contains as children any sequence of steps to execute for a loop iteration.
    documentation | no | Rich text content that provides further information on the current step.

The ``foreach`` step allows you to run a specific number of iterations, as defined by the ``start`` and ``end`` values.
When used in this way, iterations will continue as long as ``start`` is less than ``end`` with ``start`` getting incremented by one at the end of each iteration.
If not specified, ``start`` is assumed to be zero.

.. code-block:: xml

    <!--
        The loop will execute 2 times (start must be less than end). The currentIndex variable will be 5 in the first
        iteration and then 6. Note that referring to this is done as a variable reference (if not specified the variable
        would be named "i" and referred to as "$i").
    -->
    <foreach desc="Do iteration" counter="currentIndex" start="5" end="7">
        <do>
            <interact desc="Message to user">
                <instruct desc="Iteration">"Iteration " || $currentIndex</instruct>
            </interact>
        </do>
    </foreach>
    <!-- In the following case the loop's boundaries are set dynamically. -->
    <assign to="start" type="number">5</assign>
    <assign to="end" type="number">$start + 2</assign>
    <foreach desc="Do iteration" counter="currentIndex" start="$start" end="$end">
        <do>
            <interact desc="Message to user">
                <instruct desc="Iteration">"Iteration " || $currentIndex</instruct>
            </interact>
        </do>
    </foreach>

Another way of using the ``foreach`` step is as an iterator over a map or a list. In this case ``of`` becomes the key attribute
to consider, that is set with the reference to the target map or list. With this approach you would typically also
define the ``item`` attribute with the name of a variable to hold the current iteration item. When iterating over a map,
the iterated items are the map's entries, each being exposed as a map with ``key`` and ``value`` entries.

.. code-block:: xml

    <!--
      Prepare a map.
    -->
    <assign to="myMap{key1}">"value1"</assign>
    <assign to="myMap{key2}">"value2"</assign>
    <assign to="myMap{key3}">"value3"</assign>
    <!--
      Iterate over the map's entries
    -->
    <foreach item="entry" of="$myMap" counter="index" desc="Iterate entries">
       <do>
          <!--
            Exposing and using the index counter is not needed, but we can use it if useful.
          -->
          <log>"In foreach of entries (" || $index || "): " || $entry{key} || ":" || $entry{value}</log>
       </do>
    </foreach>
    <!--
      Prepare a list.
    -->
    <assign to="myList" append="true">"value1"</assign>
    <assign to="myList" append="true">"value2"</assign>
    <assign to="myList" append="true">"value3"</assign>
    <!--
      Iterate over the list.
    -->
    <foreach item="item" of="$myList" desc="Iterate values">
       <do>
          <log>$item</log>
       </do>
    </foreach>

When using the ``item`` and ``of`` attributes to iterate over a map or list, you can still make use of the ``start`` and ``end`` attributes.
In this case, when ``start`` is present it defines the zero-based index to start the iteration from, whereas ``end``
defines the end index. The ``start`` and ``end`` attributes can be used together, one at a time, or altogether skipped.

.. index:: if
.. index:: title (if)
.. index:: desc (if)
.. index:: skipped (if)
.. index:: stopOnError (if)
.. index:: documentation (if)
.. index:: cond (if)
.. index:: then (if)
.. index:: else (if)
.. index:: hidden (if)
.. index:: hidden (then)
.. index:: collapsed (if)
.. index:: static (if)
.. index:: stopOnChildError (if)
.. _tdl-step-if:

if
~~

The ``if`` step is used to run one of more steps if a condition is met. Its structure is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @collapsed, no, A boolean flag determining whether or not the step is displayed as initially collapsed (default is "false"). See also :ref:`tdl-steps-common-collapsed`.
    @desc, no, A description for this check to display to the user and to include in the test session log. Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.
    @hidden, no, A boolean flag determining whether or not the step is displayed to users (default is "false"). Note that within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`. See :ref:`tdl-steps-common-hidesteps` for further details.
    @skipped, no, A boolean value or variable reference (default being "false") which will result in the step being skipped if "true" See also :ref:`tdl-steps-common-skipped`.
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

.. index:: repuntil
.. index:: title (repuntil)
.. index:: desc (repuntil)
.. index:: skipped (repuntil)
.. index:: stopOnError (repuntil)
.. index:: documentation (repuntil)
.. index:: do (repuntil)
.. index:: cond (repuntil)
.. index:: hidden (repuntil)
.. index:: collapsed (repuntil)
.. index:: stopOnChildError (repuntil)
.. _tdl-step-repuntil:

repuntil
~~~~~~~~

The ``repuntil`` step allows you to execute a sequence of steps at least once, checking at the end a condition to see if another iteration
should take place. The structure of the ``repuntil`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @collapsed, no, A boolean flag determining whether or not the step is displayed as initially collapsed (default is "false"). See also :ref:`tdl-steps-common-collapsed`.
    @desc, no, A description for this loop to display to the user and to include in the test session log. Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.
    @hidden, no, A boolean flag determining whether or not the step is displayed to users (default is "false"). Note that within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`. See :ref:`tdl-steps-common-hidesteps` for further details.
    @skipped, no, A boolean value or variable reference (default being "false") which will result in the step being skipped if "true" See also :ref:`tdl-steps-common-skipped`.
    @stopOnError, no, A boolean flag determining whether the test session should end if this step fails (default is "false"). See also :ref:`tdl-steps-common-stoponerror`.
    @title, no, A short title to display for this step (default is "loop"). Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.
    cond, yes, The condition to verify in order to execute again the steps contained in ``do``. This is provided as an expression (see :ref:`test-case-expressions`).
    do, yes, Contains as children any sequence of steps to execute at least once and then again if the condition in ``cond`` is true.
    documentation, no, Rich text content that provides further information on the current step.

.. code-block:: xml

    <assign to="iteration" type="number">1</assign>
    <assign to="maxIteration" type="number">3</assign>
    <repuntil desc="Do iteration">
        <do>
            <interact desc="Message to user">
                <instruct desc="Iteration">$iteration || " of " || $maxIteration</instruct>
            </interact>
            <assign to="iteration">$iteration + 1</assign>
        </do>
        <cond>$iteration &lt;= $maxIteration</cond>
    </repuntil>

.. note::
    **Do-while:** Step ``repuntil`` stands for "repeat until". Considering this you could assume that the steps in ``do`` will be executed until
    the condition in ``cond`` is true. This is actually not the case currently as steps are executed while the condition in ``cond`` remains true
    (i.e. the logic is actually inversed). The naming of this step is thus unfortunate; it would be more appropriate if this was named ``dowhile``
    reflecting accurately how the condition is considered.

.. index:: while
.. index:: title (while)
.. index:: desc (while)
.. index:: skipped (while)
.. index:: stopOnError (while)
.. index:: documentation (while)
.. index:: cond (while)
.. index:: do (while)
.. index:: hidden (while)
.. index:: collapsed (while)
.. index:: stopOnChildError (while)
.. _tdl-step-while:

while
~~~~~

The ``while`` step is the most useful looping construct. It allows a sequence of steps to be continuously executed as long as a condition
continues to be true. The structure of the ``while`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @collapsed, no, A boolean flag determining whether or not the step is displayed as initially collapsed (default is "false"). See also :ref:`tdl-steps-common-collapsed`.
    @desc, no, A description for this loop to display to the user and to include in the test session log. Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.
    @hidden, no, A boolean flag determining whether or not the step is displayed to users (default is "false"). Note that within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`. See :ref:`tdl-steps-common-hidesteps` for further details.
    @skipped, no, A boolean value or variable reference (default being "false") which will result in the step being skipped if "true" See also :ref:`tdl-steps-common-skipped`.
    @stopOnError, no, A boolean flag determining whether the test session should end if this step fails (default is "false"). See also :ref:`tdl-steps-common-stoponerror`.
    @title, no, A short title to display for this step (default is "loop"). Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.
    cond, yes, The condition to verify in order to execute the contained steps. This is provided as an expression (see :ref:`test-case-expressions`).
    do, yes, Contains as children any sequence of steps to execute if the loop's condition results to true.
    documentation, no, Rich text content that provides further information on the current step.

The following example validates the name of each attachment defined in an XML document using a ``while`` loop:

.. code-block:: xml

    <!--
        Initialise maximum iteration count based on the number of "Attachment" nodes in the document.
    -->
    <assign to="iterationCount" source="$document" type="number">count(//*[local-name() = "Attachment"]</assign>
    <assign to="iteration" type="number">1</assign>
    <while desc="Validate attachment names">
        <cond>$iteration &lt;= $iterationCount</cond>
        <do>
            <verify handler="XPathValidator" desc="The attachment is named as expected">
                <input name="xml" source="$document"/>
                <!-- 
                    Construct the XPath expression to apply using the iteration variable.
                -->
                <input name="expression">"//*[local-name() = 'Attachment'][" || $iteration || "]/text() = 'file_" || $iteration || ".xml'"</input>
            </verify>
            <!--
                Increment iteration counter.
            -->
            <assign to="iteration">$iteration + 1</assign>
        </do>
    </while>

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
.. index:: byValue (assign)
.. index:: skipped (assign)    
.. index:: stopOnError (assign)
.. _tdl-step-assign:

assign
~~~~~~

The ``assign`` step is a frequently used construct in GITB TDL. It is a step that is not visible to the user, used for the manipulation 
of data in the test session's context. It can be used to assign values to variables but also as a means of 
performing simple processing or :ref:`conversion between data types <test-case-types-type-conversions>`. 
The processing and assignment result is determined by an :ref:`expression <test-case-expressions>` provided as the text content of the ``assign`` element. 
The element's structure is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @append, no, Used if the ``to`` variable is a ``list`` to append the result to. Can be "true" or "false".
    @asTemplate, no, Whether or not the result will be considered as a :ref:`template for placeholder replacement <test-case-expressions-template-files>`. By default this is "false".
    @byValue, no, Whether adding a value to a ``map`` or ``list`` will be by-value as opposed to by-reference. By default this is "false" (i.e., by-reference).
    @lang, no, Not used currently (and defaulting to XPath as the built-in :ref:`expression language <test-case-expressions>`).
    @skipped, no, A boolean value or variable reference (default being "false") which will result in the step being skipped if "true" See also :ref:`tdl-steps-common-skipped`.
    @source, no, A variable reference to identify a source ``object`` variable upon which the expression should be evaluated.
    @stopOnError, no, A boolean flag determining whether the test session should end if this step fails (default is "false"). See also :ref:`tdl-steps-common-stoponerror`.
    @to, yes, The target variable to assign the result of the expression to.
    @type, no, Used to explicitly specify the type of variable to create (e.g. if the ``to`` is an entry in a ``map``).

The ``to`` attribute of an ``assign`` step determines the target variable to which the expression's output will be assigned to. This can be:

    * An **existing variable**, defined in the test case's :ref:`variables<test-case-variables>` section or from previous steps.
    * A **new variable** that will be created once this step completes.

You typically specify the ``to`` attribute with the target variable name (e.g. ``myVariable``) in which case the variable will either be set if it 
already exists, or created otherwise. You can make this behaviour explicit by prefixing the attribute name with a dollar sign (``$``),
in which case it becomes a :ref:`variable reference <test-case-referring-to-variables>` (e.g. ``$myVariable``). In this case, if the referenced
variable does not already exist the test case will fail. You will likely never need to use a variable reference in an assign step, unless you 
have very specific needs (such as :ref:`referencing a variable from a scriptlet's parent scope <scriptlets_scope>`).

When defining a new variable its type is determined based on the result of the expression. This can also be affected by additional context information
from the way the ``assign`` step is used, specifically the ``append`` attribute that would suggest a ``list``, as well as the ``to`` expression that 
could suggest a ``map`` (e.g. if this defines ``myMap{myKey}``). You can also explicitly define the variable's type by means of the
``type`` attribute.

Given that the GITB TDL's built-in :ref:`expression language <test-case-expressions>` is XPath, the ``assign`` step can also be leveraged
to complete more advanced tasks. Specifically:

    * Make **XPath lookups** in XML content (replacing use of the :ref:`XPathProcessor <handlers-XPathProcessor>`).
    * Make **conditional variable assignments** (replacing use of :ref:`if steps <tdl-step-if>`).

The following snippet illustrates simple and more advanced use cases of the ``assign`` step:

.. code-block:: xml

    <!-- 
        Assign a text to a string variable.
    -->
    <assign to="value1">"My value"</assign>
    <!--
        Assign a number to a number variable.
    -->
    <assign to="value2" type="number">1</assign>
    <!--
        Increment a number.
    -->
    <assign to="value2">$value2 + 1</assign>
    <!--
        Assign a variable's value conditionally (no need for an if step to do this).
    -->
    <assign to="value3">if ($flag) then "Value 1" else "Value 2"</assign>
    <!--
        Lookup a text value from an XML document (no need for a process step and XPathProcessor).
    -->
    <assign to="value4" source="$xmlContent">//po:shipTo[@country = "BE"]/po:name</assign>
    <!--
        Lookup an XML block from an XML document.
    -->
    <assign to="value5" source="$xmlContent" type="object">//po:shipTo[@country = "BE"]</assign>

Further examples can be found in the documentation on :ref:`expressions<test-case-expressions>`. Examples are also provided 
here on how variables are :ref:`dynamically created<test-case-variables-from-expression-output>` if not already defined.

.. index:: byValue (assign)

.. _tdl-step-assign-by-value:

Assignment by-reference and by-value
++++++++++++++++++++++++++++++++++++

The assignment carried out by the ``assign`` step is in almost all cases done **by-value**. This means that the step's expression is calculated
and the resulting value is assigned to the target variable. 

The exception to this rule is when the target value is a **container type**, specifically a ``map`` or ``list``, and the step's expression is a :ref:`variable reference <test-case-referring-to-variables>`.
In this case the referenced variable is looked up and added **by-reference** to the target ``map`` or ``list``. This means that if following the 
assignment the referenced variable changes value, this will also be reflected in the assigned value of the ``map`` or ``list``.
Although this may seem inconsistent it follows the typical practice in most programming languages. A ``map`` or ``list`` is a complex type that
wraps references to its contained objects, whereas other types are treated like primitives and are always assigned by value.

In case you are assigning to a ``map`` or a ``list`` you can override this default behaviour by using the ``byValue`` attribute. Setting this to
"true" in this case you can force the test engine to use a copy of the referenced variable's value instead of assigning the variable itself. Note 
that when the ``assign`` step targets a non-container variable, this attribute is ignored as the assignment is always by-value.

To summarise, the assignment behaviour of the ``assign`` step follows these rules (in sequence):

#. If the step's expression is not a :ref:`variable reference <test-case-referring-to-variables>`, the assignment is **by-value**.
#. Otherwise, if the the target variable is not a ``map`` or ``list``, the assignment is **by-value**.
#. Otherwise, if ``byValue`` is set to "true", the assignment is **by-value**.
#. Otherwise the assignment is **by-reference**.

The following example illustrates the above with various assignment cases:

.. code-block:: xml

    <assign to="variable1">"v1"</assign>
    <assign to="variable2">"v2"</assign>
    <!-- 
        Prints "Values: v1 v2".
    -->
    <log>"Values: " || $variable1 || " " || $variable2</log>

    <assign to="list" append="true">$variable1</assign>
    <assign to="list" append="true" byValue="true">$variable2</assign>
    <!--
        Prints "List: v1 v2".
    -->
    <log>"List: " || $list{0} || " " || $list{1}</log>

    <assign to="map{key1}">$variable1</assign>
    <assign to="map{key2}" byValue="true">$variable2</assign>
    <!--
        Prints "Map: v1 v2".
    -->
    <log>"Map: " || $map{key1} || " " || $map{key2}</log>

    <assign to="variable1">"v1_updated"</assign>
    <assign to="variable2">"v2_updated"</assign>
    <!--
        Prints "Values: v1_updated v2_updated".
    -->
    <log>"Values: " || $variable1 || " " || $variable2</log>
    <!--
        Prints "List: v1_updated v2".
        The second list item was not updated as the assignment was by-value.
    -->
    <log>"List: " || $list{0} || " " || $list{1}</log>
    <!--
        Prints "Map: v1_updated v2".
        The second map entry was not updated as the assignment was by-value.
    -->
    <log>"Map: " || $map{key1} || " " || $map{key2}</log>


.. index:: call
.. index:: id (call)
.. index:: path (call)
.. index:: from (call)
.. index:: skipped (call)
.. index:: stopOnError (call)
.. index:: input (call)
.. index:: output (call)
.. index:: hidden (call)
.. index:: stopOnChildError (call)
.. _tdl-step-call:

call
~~~~

The ``call`` step is used to invoke a set of steps defined as a ``scriptlet`` (see :ref:`scriptlets`). If we consider that a scriptlet resembles a function
with input, output and local variables, the ``call`` step can be considered as the function's invocation. Its purpose is to identify the ``scriptlet`` to call, pass
its required input parameters and receive its output. The structure of the ``call`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @from, no, The identifier of the test suite from which the scriptlet will be loaded. If not provided this is assumed to be the current test suite.
    @hidden, no, A boolean flag determining whether or not the step is displayed to users (default is "false"). Note that within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`. See :ref:`tdl-steps-common-hidesteps` for further details.
    @id, no, The ID for the step. This is also the name of a ``map`` variable in the session context in which output will be stored.
    @input, no, An alternative to input elements to provide a single input when the scriptlet expects a single input or (if multiple) a single mandatory input. See also :ref:`tdl-step-call__simplified`.
    @output, no, The name to use for the session context variable to store the scriptlet output as an alternative to using the ``id``. See also :ref:`tdl-step-call__simplified`.
    @path, yes, The identifier scriptlet to call. The value provided here depends on the whether the scriptlet is :ref:`external to the test case<scriptlets>` or :ref:`defined within it<test-case-scriptlets>`.
    @skipped, no, A boolean value or variable reference (default being "false") which will result in the step being skipped if "true" See also :ref:`tdl-steps-common-skipped`.
    @stopOnError, no, A boolean flag determining whether the test session should end if this step fails (default is "false"). See also :ref:`tdl-steps-common-stoponerror`.
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
the path to the scriptlet's XML file (relative to the test suite root or, as a fallback, to the test suite definition file).
When the ``from`` attribute is specified the scriptlet is always considered to be
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

.. index:: group
.. index:: desc (group)
.. index:: skipped (group)    
.. index:: stopOnError (group)
.. index:: documentation (group)
.. index:: hidden (group)
.. index:: collapsed (group)
.. index:: title (group)
.. index:: stopOnChildError (group)
.. index:: hiddenContainer (group)
.. _tdl-step-group:

group
~~~~~

The ``group`` step is a construct used to visually and behaviourally group together a sequence of steps. By itself it has no effect on
the test execution but it can be used to apply a common behaviors to all child steps. Its structure is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @collapsed, no, A boolean flag determining whether or not the step is displayed as initially collapsed (default is "false"). See also :ref:`tdl-steps-common-collapsed`.
    @desc, no, A description for this group to display to the user and to include in the test session log. Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.
    @hidden, no, A boolean flag determining whether or not the step is displayed to users (default is "false"). Note that within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`. See :ref:`tdl-steps-common-hidesteps` for further details.
    @hiddenContainer, no, A boolean flag determining whether or not the group's boundary will be displayed or not (default is "false").
    @skipped, no, A boolean value or variable reference (default being "false") which will result in the step being skipped if "true" See also :ref:`tdl-steps-common-skipped`.
    @stopOnChildError, no, A boolean flag determining whether the execution of this step's children should stop in case of a failure (default is "false"). See also :ref:`tdl-steps-common-stoponerror`.
    @stopOnError, no, A boolean flag determining whether the test session should end if this step fails (default is "false"). See also :ref:`tdl-steps-common-stoponerror`.
    @title, no, A short title to display for this step (default is "group"). Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.
    documentation, no, Rich text content that provides further information on the current step.

The children of the ``group`` element can be any number of steps supported by GITB TDL. The following example creates a group around a set of 
related validations.

.. code-block:: xml

    <group desc="Validate payload">
        <verify handler="XmlValidator" desc="Validate body">
            <input name="xml">$body</input>
            <input name="xsd">$bodySchema"</input>
        </verify>
        <verify handler="XmlValidator" desc="Validate header">
            <input name="xml">$header</input>
            <input name="xsd">$headerSchema"</input>
        </verify>
    </group>

Using a ``group`` can provide a useful means of structuring a test case's presentation. In addition, it allows several steps to be considered
together and determine how they are processed and presented. Specifically:

* Used with the :ref:`hidden<tdl-steps-common-hidesteps>` attribute, to completely hide a set of steps.
* Used with the :ref:`collapsed<tdl-steps-common-collapsed>` attribute, to define the group's display as initially collapsed.
* Used with the :ref:`stopOnError <tdl-steps-common-stoponerror>` and :ref:`stopOnChildError <tdl-steps-common-stoponchilderror>` attributes, to
  define how errors in child steps are managed.
* Used with the ``hiddenContainer`` attribute, to display the group's child steps but not its own boundary. This flag is particularly
  useful when you want to apply behavior to a group of steps but don't want to otherwise adapt the display.

Use of these attributes is illustrated in the following TDL snippet:

.. code-block:: xml

    <!-- 
        Hide both validations. This could be interesting to make an internal check to drive subsequent control flow.
    -->
    <group id="checkResult" hidden="true">
        <verify handler="XmlValidator" desc="Validate body" level="WARNING">...</verify>
        <verify handler="XmlValidator" desc="Validate header" level="WARNING">...</verify>
    </group>
    <!-- 
        Show the two validations in a group and present as initially collapsed.
    -->
    <group desc="Validate document" collapsed="true">
        <verify handler="XmlValidator" desc="Validate body">...</verify>
        <verify handler="XmlValidator" desc="Validate header">...</verify>
    </group>
    <!-- 
        Show the two validations in a group and present it fully (the default).
    -->
    <group desc="Validate document">
        <verify handler="XmlValidator" desc="Validate body">...</verify>
        <verify handler="XmlValidator" desc="Validate header">...</verify>
    </group>
    <!--
        Ensure all validations within the group take place regardless of failures.
    -->
    <group desc="Validate document" stopOnChildError="false">
        <verify handler="XmlValidator" desc="Validate body">...</verify>
        <verify handler="XmlValidator" desc="Validate header">...</verify>
    </group>
    <!--
        Ensure validations immediately stop execution of the group's steps, and
        hide the group's own boundary.
    -->
    <group desc="Validate document" hiddenContainer="true" stopOnChildError="true">
        <verify handler="XmlValidator" desc="Validate body">...</verify>
        <verify handler="XmlValidator" desc="Validate header">...</verify>
    </group>

.. index:: interact
.. index:: id (interact)
.. index:: title (interact)
.. index:: desc (interact)
.. index:: with (interact)
.. index:: inputTitle (interact)
.. index:: skipped (interact)    
.. index:: stopOnError (interact)
.. index:: documentation (interact)
.. index:: hidden (interact)
.. index:: collapsed (interact)
.. index:: admin (interact)
.. index:: blocking (interact)
.. index:: timeout (interact)
.. index:: handler (interact)
.. index:: handlerEnabled (interact)
.. index:: handlerConfig (interact)
.. index:: handlerTimeout (interact)
.. index:: handlerTimeoutFlag (interact)
.. index:: actor (interact)
.. _tdl-step-interact:

interact
~~~~~~~~

The ``interact`` step is used to exchange information with the user executing the test case by means of a popup dialog.
It can be used both to present information to the user, as well as request inputs, depending on the child elements it defines.
These child elements include:

* **Instructions:** Informative messages or data to be presented to a user.
* **Requests:** Prompts to a user to provide input.

The structure of the ``interact`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @actor, no, The identifier of an :ref:`actor <test-case-actors>` to replace the "Test engine" in the step's display. See also :ref:`tdl-steps-common-step-actor`.
    @blocking, no, A boolean flag determining whether or not the interaction step will force the test session to wait until the interation completes (default is "true").
    @collapsed, no, A boolean flag determining whether or not the step is displayed as initially collapsed (default is "false"). See also :ref:`tdl-steps-common-collapsed`.
    @desc, no, A description for the user interaction to display to the user and to include in the test session log. Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.
    @handler, no, The endpoint address for a `GITB messaging service <https://www.itb.ec.europa.eu/docs/services/latest/messaging/index.html>`__ that will be delegated the handling of this interaction in case ``handlerEnabled`` is true. See also :ref:`tdl-step-interact_handler`.
    @handlerEnabled, no, A boolean flag (by default "false") provided as a constant or a :ref:`variable reference <test-case-referring-to-variables>` to determine whether the interaction should be delegated to an external service (specified via the ``handler`` attribute). See also :ref:`tdl-step-interact_handler`.
    @handlerTimeout, no, A number or variable reference with the maximum time (in milliseconds) to wait for the handler service call to complete (in case of an external test service being used as a handler). See also :ref:`tdl-steps-common-handlerTimeouts`.
    @handlerTimeoutFlag, no, A string value with the name of a boolean variable to set informing whether or not a handler timeout occurred. See also :ref:`tdl-steps-common-handlerTimeouts`.
    @hidden, no, A boolean flag determining whether or not the step is displayed to users (default is "false"). Note that within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`. See :ref:`tdl-steps-common-hidesteps` for further details.
    @id, no, Used as the name of a ``map`` variable that will be used to store provided input (if no per-input assignment is provided).
    @inputTitle, no, A custom text to display as the title of the user input popup (default is "Server interaction"). Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.
    @skipped, no, A boolean value or variable reference (default being "false") which will result in the step being skipped if "true" See also :ref:`tdl-steps-common-skipped`.
    @stopOnError, no, A boolean flag determining whether the test session should end if this step fails (default is "false"). See also :ref:`tdl-steps-common-stoponerror`.
    @timeout, no, An optional timeout (in milliseconds) on the time to wait for the interaction to be completed. This is provided as a ``number`` or a variable reference.
    @title, no, A short title to display for this step (default is "interact"). Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.
    @with, no, The ID of the actor this interaction refers to. If not specified is is assumed to be the test case actor defined as the SUT. Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.
    documentation, no, Rich text content that provides further information on the current step.
    handlerConfig, no, Contains a set of ``input`` elements (see :ref:`handlers-inputs-outputs`) to define the inputs passed to an external ``handler`` service in case ``handlerEnabled`` is "true". If ``handlerEnabled`` is "false" this element is ignored.
    instruct, no, Zero or more elements to appear as instructions to the user.
    request, no, Zero or more input requests for the user.

.. index:: instruct (interact)
.. index:: desc (instruct)
.. index:: with (instruct)
.. index:: name (instruct)
.. index:: type (instruct)
.. index:: source (instruct)
.. index:: asTemplate (instruct)
.. index:: mimeType (instruct)
.. index:: forceDisplay (instruct)
.. index:: showControls (instruct)
.. index:: level (instruct)
.. index:: ERROR (instruct)
.. index:: WARNING (instruct)
.. index:: INFO (instruct)
.. index:: SUCCESS (instruct)
.. index:: NONE (instruct)
.. index:: report (instruct)

The ``instruct`` elements define what is going to presented to the user. They have the following structure:

.. csv-table::
    :stub-columns: 1
    :delim: ~
    :header: "Name", "Required?", "Description"

    @asTemplate~ no~ Whether or not the result will be considered as a template for placeholder replacement (see :ref:`test-case-expressions-template-files`). By default this is "false".
    @desc~ yes~ The label to display to the user.
    @forceDisplay~ no~ Whether the content should be always displayed inline rather than in an editor. By default this is "false".
    @level~ no~ A severity level used to stylise the presentation of the displayed text. Can be set to ``ERROR``, ``WARNING``, ``INFO``, ``SUCCESS`` and ``NONE`` (the default), and also be provided via :ref:`variable reference <test-case-referring-to-variables>`.
    @mimeType~ no~ A `mime type`_ value (e.g. ``text/xml``) provided as a string or a variable reference, to hint how this value should be highlighted when displayed. In case an invalid or unsupported mime type is provided no such highlighting will be applied.
    @name~ no~ In case of ``instruct`` elements that used to share binary content, this is used as the name of the file presented for download.
    @report~ no~ Whether To include this element in the step's report. By default this is "false".
    @showControls~ no~ Whether or not to display user interface controls for this elements (copy, view and download). By default such controls are displayed.
    @source~ no~ A pure variable reference identifying a source variable. Used as the target upon which to evaluate the contained expression.
    @type~ no~ The ``type`` to consider for the displayed value. If this is not specified the ``type`` will be inferred from the referred variable (if defined) or default to ``string``.
    @with~ no~ The ID of the actor this interaction refers to. If not specified this is taken from the ``interact`` parent element (which itself defaults to the test case's SUT actor). Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.

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
.. index:: report (request)
.. index:: fileName (request)
.. index:: required(request)
.. index:: size(request)
.. index:: accept(request)
.. index:: default(request)

The ``request`` elements define how information shall be requested from the user. Their structure is as follows:

.. csv-table::
    :stub-columns: 1
    :delim: ~
    :header: "Name", "Required?", "Description"

    @asTemplate~ no~ Whether or not the result will be considered as a :ref:`template for placeholder replacement <test-case-expressions-template-files>`. By default this is "false".
    @contentType~ no~ Defines how the specified variable's value is to be set ("STRING", "BASE64" or "URI"). The default is "STRING".
    @default~ no~ The default value to display in the presented input field. This can be provided inline or via :ref:`variable reference <test-case-referring-to-variables>`.
    @desc~ yes~ The label to display to the user.
    @encoding~ no~ Used in case of text binary input to specify the character encoding to consider. The default is "UTF-8".
    @fileName~ no~ The name of a variable or output map property that will record the file name for the uploaded file (in case of a file upload). This can be set as a fixed ``string`` or be defined dynamically via :ref:`variable reference<test-case-referring-to-variables>`.
    @inputType~ no~ The input control type to use when prompting users for the relevant value. By default a value of ``TEXT`` is assumed unless this input is mapped to an existing ``binary`` variable.
    @mimeType~ no~ In case the ``inputType`` is set as ``CODE`` (i.e. a code editor) this is the content's expected `mime type`_ (e.g. ``text/xml``), provided as a string or a variable reference, to be considered for presenting appropriate syntax highlighting.
    @multiple~ no~ A ``boolean`` value to determine whether the dropdown list (if the ``options`` attribute is defined) shall be a single or multiple selection list (default is "false" for single selection).
    @name~ no~ In case of ``request`` elements this name is the key to be used for the map entry to hold the provided data.
    @optionLabels~ no~ Used as the labels for the option values (comma-separated values, a reference to a string variable of comma-separated values, or a reference to a list variable of strings). If provided the number of values needs to match the options. If not provided the option values are used.
    @options~ no~ Used to render a dropdown list by providing the option values to consider (comma-separated values, a reference to a string variable of comma-separated values, or a reference to a list variable of strings).
    @required~ no~ Whether or not this input is mandatory (by default "false"). When set to "true" the relevant input control will appear as mandatory and the interaction will not be able to be completed unless a value is provided. Note that this flag can also be set as a :ref:`variable reference <test-case-referring-to-variables>`.
    @report~ no~ Whether or not this value will be included in the presentation of the test step's report (by default "true"). When set to "false" the requested value will be stored in the test session context but not displayed in the step's report.
    @size~ no~ A number defining the size of the input control in terms of its height. This is considered for the rows of ``MULTILINE_TEXT`` inputs, the pixels of ``CODE`` editors, and the presented items of ``SELECT_MULTIPLE`` selections.
    @with~ no~ The ID of the actor this interaction refers to. If not specified this is taken from the ``interact`` parent element (which itself defaults to the test case's SUT actor). Within scriptlets this can also be a :ref:`variable reference<scriptlets_dynamic_references>`.

Both instructions and requests can be included in the same ``interact`` step to display and/or request multiple sets of
information in one go. The sequence with which these elements are defined determine also their display sequence in the
``interact`` step's popup. The attributes for the ``interact``, ``instruct`` and ``request`` elements determine the precise
behaviour in presenting and requesting data to and from the user. Detailed information on their possible options as well as several
examples are provided in the sections that follow.

.. _tdl-step-interact_instruct_presentation:

Displaying data using instruct elements
+++++++++++++++++++++++++++++++++++++++

The ``instruct`` element is used to present data from the test session to the user. The presented data is labelled using
the ``desc`` attribute's value, whereas the data itself is determined by evaluating the element's content as an :ref:`expression <test-case-expressions>`,
ranging from a **constant value**, to a :ref:`variable reference <test-case-referring-to-variables>`, to a complete **XPath expression** to process. In case of
simple messages, you can also define the ``instruct`` element as empty in which case only the ``desc`` value is presented.

.. code-block:: xml

    <interact desc="User instructions">
        <!-- Display a simple inline message -->
        <instruct desc="Send the message to continue"/>
        <!-- Display a labelled value -->
        <instruct desc="Message type to use:">"Request message"</instruct>
        <!-- Display a labelled file for download -->
        <instruct desc="Message to send:">$file</instruct>
    </interact>

When the ``instruct`` element's content is defined, the way in which it is presented depends on the :ref:`type <test-case-types>`
of the resulting expression, which can also be forced through the ``type`` attribute. Depending on the value's type (inferred or explicitly set),
it is presented as follows:

* For a ``binary``, ``object`` or ``schema`` type, controls will be displayed allowing to **download** the value as a file
  and to view it in a **code editor** (if text-based).
* For all other types, the value is displayed **inline as text**. This is also the default approach if the value's type was not
  set and could not be inferred.

Instruction elements are considered as informational features and by default only figure in the step's presentation on
the user interface when the step is being executed. If you want to ensure certain ``instruct`` elements are presented
also for completed steps, both on the user interface and produced step reports, you can set their ``report`` attribute
to true.

When displaying the **value inline** it could be the case that the text is too long. In this case the user will instead
be provided with controls to download it as a file or open it in a code editor (as in the case of e.g. binary content).
You can override this behaviour by setting the ``forceDisplay`` attribute to true, which will result in an inline display
regardless of the value's size. In addition, you can set the ``showControls`` attribute to false, to hide the display of
user interface copy and view controls.

When conveying feedback, status or instructions, you may also find useful the ``level`` attribute. This is used to adapt
the display of ``instruct`` elements to convey severity by means of colour highlighting. It supports values ``ERROR``,
``WARNING``, ``INFO``, ``SUCCESS`` and ``NONE`` (the default), that result in the instruction being presented with a
coloured background (e.g. green background for ``SUCCESS``). The value for this attribute can also be determined at
runtime by providing it as a :ref:`variable reference <test-case-referring-to-variables>`. Using the ``level`` attribute
to display stylised messages, can be optionally complemented by the ``forceDisplay`` and ``showControls`` attributes to
ensure that the message is presented without truncation and without superfluous UI controls.

.. code-block:: xml

    <!--
        Display a blue information message ensuring that no UI controls are displayed.
    -->
    <interact inputTitle="Next step" desc="Show instructions">
        <instruct level="INFO" forceDisplay="true" showControls="false">"Please send a request referring to identifier " || $expectedIdentifier</instruct>
    </interact>

As a complement to the ``type``, you can also specify the ``mimeType`` attribute. This is meaningful for binary or large text content
as it serves two purposes: it allows you to specify the content type and **file extension** to use when the content is downloaded as a file, and it
provides a hint for appropriate **syntax highlighting** when displaying the content in a code editor. Considering file
downloads, you can also set the ``name`` attribute to specify the name of the downloaded file.

It is important to note that the ``mimeType`` attribute has no effect when the value is presented inline. In fact, an inline
presentation is always as simple text, with no additional formatting or highlighting. In case you are looking for an inline
display of something more elaborate (e.g. a rich text message styled as HTML content), you should consider defining this
as :ref:`documentation for the step <tdl-steps-common-documentation>` using a ``documentation`` element within the ``interact``
step.

In case the ``interact`` step only defines ``instruct`` elements you can also use the ``blocking`` attribute to specify that 
it **should not block the test session execution**. This could be interesting in case you want to allow subsequent test steps to
proceed without necessarily waiting for the user to complete the interaction. An example where this could be particularly
useful is in case you are using the ``interact`` step as an information popup, before expecting a message to be sent to the 
test engine through a :ref:`receive step <tdl-step-receive>`. In such a scenario, users may proceed with
sending the expected message before closing the interaction, which depending on the :ref:`handler implementation <handlers>`, might
result in the test engine ignoring the (prematurely sent) message.

The configuration to avoid this issue would be as follows:

.. code-block:: xml

    <!-- 
        Use an interact step to show an information popup informing the user of the expected identifier
        to use in a request. Setting this as not "blocking" will result in the popup being displayed
        but the test session proceeding to run the next "receive" step.

        We also set the interaction as "hidden" as we don't need this to be displayed on the test
        execution diagram.
    -->
    <log>"Expecting GET call quoting identifier " || $expectedIdentifier</log>
    <interact inputTitle="Instructions" desc="Show instructions" blocking="false" hidden="true">
        <instruct>"Please send a request referring to identifier " || $expectedIdentifier</instruct>
    </interact>
    <receive id="receive" desc="Receive message" handler="HttpMessagingV2">
        <input name="uriExtension">"/message/" || $expectedIdentifier</input>
    </receive>

Using the ``blocking`` attribute is optional, and if not specified defaults to "true". You can also set this dynamically 
as a :ref:`variable reference <test-case-referring-to-variables>` in case the blocking behaviour depends on configuration
or your current state. Note finally that in case the ``interact`` step includes ``request`` elements for user input, the
``blocking`` attribute is ignored as the interaction is always blocking.

.. _tdl-step-interact_form_inputs:

Requesting inputs using request elements
++++++++++++++++++++++++++++++++++++++++

The ``request`` element is used to request input from the user. The element's definition determines the two important aspects
related to such input:

* The way in which the provided input will be recorded for subsequent use.
* The form control used by the user to provide the input.

Regarding the first point, how to **record the provided input**, you can either set it as the value of an existing variable,
or record it in a map corresponding to the ``interact`` step's output. To set an existing variable you specify the ``request``
element's content as a :ref:`variable reference <test-case-referring-to-variables>`, referring to the variable that will
receive the value. In this case you need to ensure that the referenced variable already exists, for example through an earlier
:ref:`assign step <tdl-step-assign>` or a :ref:`variable declaration <test-case-variables>`.

.. code-block:: xml

    <assign to="aValue">"Initial value"</assign>
    ...
    <interact id="data" desc="Provide input">
        <!-- Stored as $aValue.  -->
        <request desc="Enter a text value:">$aValue</request>
    </interact>
    <!-- Log the provided value. -->
    <log>$aValue</log>

The alternative to this approach is to store received inputs in a map corresponding to the ``interact`` step's output. In
this case you leave the ``request`` element empty, specifying instead the ``name`` attribute that will be used as the value's
key in the step's output map. The output map added to the test session context is named after the ``interact`` step's ``id``.

.. code-block:: xml

    <interact id="data" desc="Provide input">
        <!-- Stored as $data{aValue}.  -->
        <request desc="Enter a text value:" name="aValue"/>
    </interact>
    <!-- Log the provided value. -->
    <log>$data{aValue}</log>

This second approach is considered the **better practice** as it is typically simpler to use. Firstly, there is no need to predefine
variables to receive inputs, and secondly, it becomes simpler to manage ``interact`` steps resulting in multiple inputs.

.. code-block:: xml

    <interact id="data" desc="Provide inputs">
        <request desc="Invoice number:" name="invoiceNumber"/>
        <request desc="Seller VAT number:" name="sellerVat"/>
        <request desc="Buyer VAT number:" name="buyerVAT"/>
    </interact>
    <!-- Log the provided values. -->
    <log>"Invoice "|| $data{invoiceNumber} || " sent from " || $data{sellerVat} || " to " || $data{buyerVAT}</log>

The second key aspect of a ``request`` element is configuring the **form control** to present to the user. The label for the
input is defined through the ``desc`` attribute, whereas the type of input presented is determined by the ``inputType`` attribute
that supports the following values:

.. csv-table::
    :delim: ~
    :header: "Input type", "Description"

    ``TEXT`` ~ A simple text field (the default if not specified).
    ``MULTILINE_TEXT`` ~ A textarea supporting input of multiple lines. You can also set in this case the ``size`` attribute to define the control's height (in rows).
    ``SECRET`` ~ A control to add a secret value such as a password.
    ``UPLOAD`` ~ A file upload control. You can also set in this case the ``fileName`` attribute to record the name of the uploaded file (the ``fileName`` value being the key to use in the step's output map). The ``accept`` attribute can also be used to limit the types of accepted files.
    ``CODE`` ~ A code editor. To complement this you can also specify the ``mimeType`` attribute with a `mime type`_ (e.g. ``text/xml``) to have appropriate syntax highlighting. The ``size`` attribute can also be used to define the editor's height (in pixels).
    ``SELECT_SINGLE`` ~ A single-select dropdown list, specifying the options via the ``options`` and ``optionLabels`` attributes.
    ``SELECT_MULTIPLE`` ~ A multi-select dropdown list using similarly the ``options`` and ``optionLabels`` attributes. You can also set in this case the ``size`` attribute to define the number of presented items.

.. note::
    The value received from a ``SELECT_MULTIPLE`` input will be a comma-separated string in which the individual parts match the selected values.

The following example produces a popup presenting a text field, textarea and file upload.

.. code-block:: xml

    <interact id="data" desc="Provide inputs">
        <request desc="Invoice number:" name="invoiceNumber"/>
        <request desc="Invoice comment:" name="invoiceComment" inputType="MULTILINE_TEXT"/>
        <request desc="Invoice file:" name="invoiceFile" fileName="invoiceFileName" inputType="UPLOAD"/>
    </interact>
    <!-- Log the provided values. -->
    <log>"Invoice "|| $data{invoiceNumber} || " uploaded as " || $data{invoiceFileName} || " with comment: " || $data{invoiceComment}</log>

In the example above, the types of uploaded files could also be controlled by using the ``accept`` attribute to specify the accepted file
types as `mime types <https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types/Common_types>`__. This can
be specified inline as a comma-separated string or provided dynamically by referencing a ``string`` or ``list`` variable.

.. code-block:: xml

    <assign to="acceptedImageTypes" append="true">"image/png"</assign>
    <assign to="acceptedImageTypes" append="true">"image/jpeg"</assign>
    <interact id="data" desc="Provide inputs">
        <request desc="Evidence screenshot" name="screenshot" fileName="evidenceFile" inputType="UPLOAD" accept="$acceptedImageTypes"/>
        <request desc="Invoice file" name="invoiceFile" fileName="invoiceFileName" inputType="UPLOAD" accept="application/pdf"/>
    </interact>

For certain input types, notably ``MULTILINE_TEXT``, ``CODE`` and ``SELECT_MULTIPLE``, you can specify the ``size`` attribute
to control the height of the resulting control. In all cases this needs to be a positive integer, provided inline or via
:ref:`variable reference <test-case-referring-to-variables>`. For ``MULTILINE_TEST`` inputs this is interpreted as the
number of rows displayed by default, for ``CODE`` inputs as the height in pixels of the editor, and for ``SELECT_MULTIPLE``
as the number of visible items in the selection list.

.. code-block:: xml

    <interact id="data" desc="Provide inputs">
        <request desc="Invoice comment:" name="invoiceComment" inputType="MULTILINE_TEXT" size="5"/>
        <request desc="Invoice content:" name="invoiceContent" inputType="CODE" size="500"/>
    </interact>

Where is is meaningful to present a default value in an input control you can use the ``default`` attribute. With this
you can specify a default value inline, or provide a :ref:`variable reference <test-case-referring-to-variables>` pointing
to a :ref:`configuration value <test-case-configuration>` or another value from the test session context.

.. code-block:: xml

    <interact id="data" desc="Provide inputs">
        <request desc="The name of your software" name="softwareName" inputType="TEXT" default="$SYSTEM{shortName}" required="true"/>
    </interact>

Prior to GITB TDL version 1.14.0, the way to determine the input control to use was the ``contentType`` attribute. Although less expressive, this approach is
still supported as follows:

* Specifying "BASE64" results in a file upload presented to the user.
* Specifying "STRING" (the default) or "URI" results in a simple text input. Note that only "STRING" can be used in case the request is defined as a dropdown list (i.e. the ``options`` attribute is defined).

It is interesting to note that any available context information is always considered to reduce the configuration you need to provide. For example, if for a ``request``
you are referencing an already defined ``binary`` variable, you can skip the ``inputType`` or ``contentType`` definitions as this will anyway result in a file upload.
Similarly, if for a ``request`` you define ``options`` and the ``multiple`` attribute, you don't need to define the ``inputType`` as well as this is considered to be
by default ``SELECT_MULTIPLE``.

Besides defining the type of input control to display, you can also specify whether the input should be presented as ``required``. This is achieved through the
similarly named attribute that is set either with a boolean value or with a :ref:`variable reference <test-case-referring-to-variables>`. The latter approach
might be interesting in case whether or not an input is mandatory depends on your configuration or test session state. Note that specifying the ``required``
attribute is optional, with a value of "false" (non-mandatory) being considered as the default.

.. code-block:: xml

    <interact id="data" desc="Provide inputs">
        <!-- The invoice number is required. -->
        <request desc="Invoice number:" name="invoiceNumber" required="true"/>
        <!-- The invoice comment is optional. -->
        <request desc="Invoice comment:" name="invoiceComment" inputType="MULTILINE_TEXT"/>
        <!-- The invoice file is required. -->
        <request desc="Invoice file:" name="invoiceFile" required="true" inputType="UPLOAD"/>
    </interact>

An ``interact`` step containing one or more ``request`` elements set as ``required`` will not be able to complete until at least the mandatory inputs are provided.
Presentation-wise, mandatory inputs are highlighted as such on the Test Bed's user interface with error messages displayed if the user attempts to complete the
interaction without specifying them. Note that it is always possible to minimise the interaction and completed it at a later point. If the interaction is
:ref:`delegated to an external service <tdl-step-interact_handler>`, ``required`` inputs must be returned otherwise the interaction will fail.

.. _tdl-step-interact_admin_interactions:

Interactions for administrators
+++++++++++++++++++++++++++++++

Interactions are by default presented to the tester, but can also be reserved for an administrator by setting the ``interact`` element's ``admin`` flag to true.
This could be useful in case you need to pause a test session while an administrator makes a manual verification, or to make a manual check on user-provided
evidence data that cannot be automatically verified.

In case the administrator needs to input internal information, you can fine-tune what gets reported to users. You can skip
the reporting of specific information by setting the ``report`` flag on the relevant ``request`` elements to false. You can
even :ref:`hide the interaction step <tdl-steps-common-hidesteps>` altogether by setting ``hidden`` to true.

.. code-block:: xml

    <interact id="adminData" desc="Confirm results" admin="true">
        <request desc="Comments" inputType="MULTILINE_TEXT" name="comments"/>
        <!-- The "code" input will be recorded but not added to reports. -->
        <request desc="Internal code" name="code" report="false"/>
    </interact>
    <log>$adminData{code}</log>

.. _tdl-step-interact_customise_display:

Customising the interact step's display
+++++++++++++++++++++++++++++++++++++++

The ``interact`` step's presentation can be customised using the element's attributes. Besides :ref:`commons options <tdl-steps-common>`
applicable to all TDL steps, you can adapt the step's description (``desc``) and ``title`` for the displayed step's boundary, as well as
provide a custom ``inputTitle`` for the popup used to present the step's instructions and form controls.

.. code-block:: xml

    <interact desc="User instructions" inputTitle="Instructions">
        <instruct desc="Send the message to continue"/>
    </interact>

For an ``interaction`` step that is not a key part of the test case, for example a popup to guide the user in next steps,
a typical configuration is to set it as :ref:`hidden <tdl-steps-common-hidesteps>`. This ensures the interaction is
executed but will not be displayed on the test execution diagram or included in test case reports. A lighter alternative
is to set the step as ``collapsed`` in which case it will be included in the execution diagram and reports, but will
be presented as initially collapsed. This approach could be interesting for example if the step in question requests inputs
that you still want to make available for later review by expanding the step's display and selecting the step's report.

.. code-block:: xml

    <interact desc="User instructions" inputTitle="Instructions" collapsed="true">
        <instruct desc="Send the message to continue"/>
    </interact>

.. _tdl-step-interact_timeout:

Background execution and timeouts
+++++++++++++++++++++++++++++++++

When the ``interact`` step is executed the test execution pauses until the step is completed by the user. This is meaningful
when the relevant test is executed interactively but could be a point to consider if the test is executed in the background.
Moreover, even if executed interactively, you may want to prevent the test session from blocking indefinitely due to a simple information
popup that has no bearing on the overall test.

To avoid blocking test sessions and to better control how interactions are completed in background executions, the ``interact``
step can be set with a configurable ``timeout``. The value of this is a number corresponding to the milliseconds to
wait for until automatically completing the interaction. It can be set as a fixed value or as a :ref:`variable reference <test-case-referring-to-variables>`,
set dynamically or referring to a :ref:`configuration property <test-case-configuration>` (similar to the timeouts of :ref:`receive steps <tdl-step-receive>`).

.. note::

    Background test sessions with pending user interactions, can be still be completed by both users and administrators
    through the GITB Test Bed's user interface (under the **active sessions** in the
    `session history <https://www.itb.ec.europa.eu/docs/itb-ta/latest/validateTestSetup/index.html#active-test-sessions>`__
    and `session dashboard <https://www.itb.ec.europa.eu/docs/itb-ta/latest/sessionDashboard/index.html#active-test-sessions>`__ screens).

For a test session executing in the background, the test engine treats ``interact`` steps in the following way:

1. If the step contains only ``instruct`` elements and ``blocking`` is set (or resolves) to "false", the interaction is skipped.
2. If the step is set with a ``timeout``, the interaction remains pending until completed by the user or until the timeout expires.
3. If no ``timeout`` is set but the ``interact`` step is set as an :ref:`administrator interaction <tdl-step-interact_admin_interactions>`
   (i.e. ``admin`` is set to true), the test session will pause indefinitely until completed by an administrator.
4. In all other cases the step is completed automatically with a log warning, effectively skipping it and resulting in empty inputs (if requested).

.. code-block:: xml

    <!-- Assume the test session is executing in the background. -->

    <!-- This interaction is automatically skipped. -->
    <interact desc="User instructions">
        <instruct desc="Send the message to continue"/>
    </interact>

    <!-- This interaction will remain pending until manually completed or until the timeout elapses. -->
    <interact id="userData" desc="User input" timeout="$DOMAIN{userInteractionTimeout}">
        <request desc="Identifier to check" name="identifier"/>
    </interact>

    <!-- This interaction will remain pending until manually completed. -->
    <interact id="adminData" desc="Confirm results" admin="true">
        <request desc="Comments" inputType="MULTILINE_TEXT" name="comments"/>
    </interact>

    <!-- This interaction will remain pending until manually completed or until the timeout elapses. -->
    <interact id="adminData" desc="Confirm results" admin="true" timeout="$DOMAIN{adminInteractionTimeout}">
        <request desc="Comments" inputType="MULTILINE_TEXT" name="comments"/>
    </interact>

Generally speaking, it is a **good practice** to always define timeouts for ``interact`` steps as this gives you full control over
their execution. This becomes even more important if you are expecting manual user inputs, but still want your test cases to
support background execution. In this case you can even foresee a fallback solution for user input that was not provided,
checking the ``interact`` step's resulting value(s) and assigning defaults as needed.

.. code-block:: xml

    <log>"Requesting user input..."</log>
    <interact id="userData" desc="Request user input" timeout="3000">
        <request desc="Input value" name="inputValue"/>
    </interact>
    <!-- If the step was skipped the requested value will be empty. -->
    <assign to="userData{inputValue}">if ($userData{inputValue} = "") then "Default value" else $userData{inputValue}</assign>
    <log>"Value to use: " || $userData{inputValue}</log>

.. note::

    When executing test sessions through the GITB Test Bed's `REST API <https://www.itb.ec.europa.eu/docs/itb-ta/latest/api/index.html#start>`__
    you can provide input data as part of the service call. Such data could serve to replace inputs that would otherwise be
    requested via ``interact`` steps. Note that for such headless test sessions you may also want to :ref:`delegate mandatory interactions <tdl-step-interact_handler>`
    to an external service.

.. _tdl-step-interact_handler:

Delegating to external handler services
+++++++++++++++++++++++++++++++++++++++

The default implementation for an ``interact`` step is to display a **popup dialog** to the user to :ref:`present instructions <tdl-step-interact_instruct_presentation>`
and :ref:`request inputs <tdl-step-interact_form_inputs>`. You may want to replace this default behaviour with a different implementation
that will delegate the handling of the interaction to an external service. Scenarios where you would want to do this are:

* When launching tests via the Test Bed's `REST API <https://www.itb.ec.europa.eu/docs/itb-ta/latest/api/index.html#start>`__, in which case
  instead of blocking the test, you want to provide requested inputs via a different approach.
* When the interaction should involve **external solutions** such as sending emails and collecting inputs via another interface.

Delegating an ``interact`` to an alternate implementation requires using an **external custom service** that will handle the provision
of any requested inputs. This service needs to implement the `GITB messaging service API <https://www.itb.ec.europa.eu/docs/services/latest/messaging/index.html>`__,
the same way that a :ref:`receive step <tdl-step-receive>` would do to receive a message asynchronously. Even though using a messaging
service API for a user interaction may seem initially unintuitive, semantically both represent the same scenario: the test session pauses until
data is received. The specific messaging operation that needs to be implemented by the service to handle ``interact`` steps is the
`receive operation <https://www.itb.ec.europa.eu/docs/services/latest/messaging/index.html#receive>`__.

To delegate an ``interact`` step to an external service you use the ``handler`` attribute. This is set exactly as in the case of a :ref:`receive step <tdl-step-receive>`,
and would typically be set via a :ref:`domain configuration parameter <test-case-expressions-domain>`. You also have available a ``boolean``
flag named ``handlerEnabled`` that determines whether the handler service should be used. This can be set to fixed "true" or "false" values
("false" being the default), but can also be a :ref:`variable reference <test-case-referring-to-variables>` allowing this decision
to be made at runtime based on your configuration or runtime state.

To provide context to the handler service, the ``interact`` step also supports a ``handlerConfig`` child element that is used
to define the inputs to send to the service. With this you can pass values and state collected from previous test steps to help
the service complete the interaction. If any values were requested by the interaction via ``request`` elements, these can be provided
as **similarly named service outputs**. Regarding inputs and outputs you need to keep in mind the following:

* If a handler service  is delegated the interaction, the step's ``instruct`` and ``request`` elements are ignored. You must pass
  expected information as ``input`` elements under a ``handlerConfig`` block.
* Any outputs returned by the service that do not correspond to ``request`` elements are ignored.
* Any missing outputs for ``request`` elements set as ``required`` will cause the ``interact`` step to fail.

.. note::
    When delegating an interaction to a handler service that is expected to returned inputs, it is best to set the step's
    ``request`` elements with ``name`` values. This allows **explicit name-based matching** to returned service outputs.

The following example illustrates how an ``interact`` step can delegate its implementation to a handler service:

.. code-block:: xml

    <!-- 
        The handler attribute defines an external service (via configuration) to manage the interaction.

        If the handlerEnabled flag evaluates to "true" when the step is executed, the delegation will
        take place. Otherwise the handler is ignored and the default UI-based approach to complete
        the step will be used.
    -->
    <interact id="review" handlerEnabled="$delegateInteractions" handler="$DOMAIN{serviceAddress}" desc="Review test data">
        <request name="outcome" desc="Review outcome" inputType="SELECT_SINGLE" options="OK,NOK"/>
        <request name="comments" desc="Comments" inputType="MULTILINE_TEXT"/>
        <!-- 
            Provide inputs to the handler service as context for the review, loaded from prior test steps.
            These are ignored if we're not delegating to a handler service.
        -->
        <handlerConfig>
            <input name="receivedMessage">$message</input>
            <input name="expectedCode">$expectedCode</input>
        </handlerConfig>
    </interact>
    <log>"Outcome: " || $review{outcome}</log>
    <log>"Comments: " || $review{comments}</log>

Using a handler to delegate interactions can be a powerful mechanism when required interactions are not possible to complete manually. This
is typically the case for tests launched through the Test Bed's `REST API <https://www.itb.ec.europa.eu/docs/itb-ta/latest/api/index.html#start>`__.
To allow such tests to work both when launched interactively and via REST call, we can have the ``handlerEnabled`` flag check 
a :ref:`predefined test case variable <test-case-variables>` that is :ref:`set accordingly through the REST call's inputs <tdl-step-interact_api>`:

.. code-block:: xml

    <variables>
        <!-- 
            Set the delegateInteractions flag to "false" by default to cover manual execution.
            This will be overriden when to "true" when the test is started via the REST API.
        -->
        <var name="delegateInteractions" type="boolean">
            <value>false</value>
        </var>
    </variables>
    <steps>
        ...
        <!-- 
            Check the delegateInteractions flag to see whether we are delegating or not.
        -->
        <interact id="review" handlerEnabled="$delegateInteractions" handler="$DOMAIN{serviceAddress}" desc="Review test data">
            ...
        </interact>
        ...
    </steps>

.. note::
    If an ``interact`` step is optional you can also :ref:`skip it <tdl-steps-common-skipped>` using the ``skipped`` attribute.

.. _tdl-step-interact_api:

Interactions in REST API-initiated tests
++++++++++++++++++++++++++++++++++++++++

The Test Bed provides a rich `REST API <https://www.itb.ec.europa.eu/docs/itb-ta/latest/api/>`__ to, among other operations, 
`launch test sessions <https://www.itb.ec.europa.eu/docs/itb-ta/latest/api/index.html#start>`__ as part of automated processes. Doing
so presents a challenge for tests relying on user interactions, given that in this case interactions will likely not be possible.

In case tests are launched via REST API but can later be interacted with by users, the typical approach would be to
:ref:`use timeouts <tdl-step-interact_timeout>` so that users have enough time to provide inputs. In case however such
interactions are not possible, the main options to manage ``interact`` steps are:

* Setting ``skipped`` to "true" to :ref:`skip interactions <tdl-steps-common-skipped>` that are not mandatory (e.g. instruction popups).
* Setting a ``handler`` service and ``handlerEnabled`` to "true" to :ref:`delegate to an external service <tdl-step-interact_handler>`.

To accommodate the same test cases being used regardless of whether they were launched manually or via REST call, the best approach
is to set ``skipped`` and ``handlerEnabled`` as :ref:`references <test-case-referring-to-variables>` to :ref:`predefined test case variables <test-case-variables>`.
When defining such variables you would set them by default to what you need to enable manual execution, while expecting them to be 
overriden when the test is executed via REST call.

The following example illustrates how you could configure this:

.. code-block:: xml

    <variables>
        <!-- 
            Define the flags to control the interactions. These are set to false to reflect the
            default behavior when executed via the UI. They will be replaced by inputs when the
            test session is triggered via the REST API.
        -->
        <var name="skipInteractions" type="boolean">
            <value>false</value>
        </var>
        <var name="delegateInteractions" type="boolean">
            <value>false</value>
        </var>
    </variables>
    <steps>
        ...
        <interact id="review" skipped="$skipInteractions" handlerEnabled="$delegateInteractions" handler="$DOMAIN{serviceAddress}" desc="Review test data">
            <request name="outcome" desc="Review outcome" inputType="SELECT_SINGLE" options="OK,NOK"/>
            <request name="comments" desc="Comments" inputType="MULTILINE_TEXT"/>
            <!-- 
                Provide inputs to the handler service as context for the review, loaded from prior
                test steps. These are ignored if we're not delegating to a handler service.
            -->
            <handlerConfig>
                <input name="receivedMessage">$message</input>
                <input name="expectedCode">$expectedCode</input>
            </handlerConfig>
        </interact>
        <log>"Outcome: " || $review{outcome}</log>
        <log>"Comments: " || $review{comments}</log>
        ...
    </steps>

When this test case is launched via the UI, both ``skipInteractions`` and ``delegateInteractions`` flags are set to their default "false"
values, resulting in an interaction that is not skipped and not delegated to the external service. When executed via the REST API however,
we can override the flags via inputs provided to the `start operation <https://www.itb.ec.europa.eu/docs/itb-ta/latest/api/index.html#start>`__
as follows:

.. code-block:: json

    {
        "system": "7039...E9B0",
        "actor": "7704...09E6",
        "testCase": [ "testCase1" ],
        "inputMapping": [
            { "input": { "name": "skipInteractions", "value": "false" } },
            { "input": { "name": "delegateInteractions", "value": "true" } }
        ]
    }

Not all REST-initiated tests that trigger interactions must have them skipped or delegated. You may have certain interactions
that you still expect to **complete manually**, such as manual :ref:`administrator verification <tdl-step-interact_admin_interactions>`
of collected evidences. In such cases you should define :ref:`interaction timeouts <tdl-step-interact_timeout>`
to set the maximum time tests will pause for waiting for such inputs to be provided.

.. _tdl-step-interact_examples:

Example interact steps
++++++++++++++++++++++

The following examples illustrate user interactions presenting instructions and also requesting information:

.. code-block:: xml

    <interact desc="Some information and inputs">
        <!-- type="string" omitted as default. Displays the text as a message to the user. -->
        <instruct desc="This is a simple message"/>
        <instruct desc="A text value:">"A text value " || $aTextValue</instruct>
        <!-- Present a download button and XML editor for file "schema.xsd" (not specifying a name would produce a "downloadedFile" file). -->
        <instruct name="schema.xsd" desc="A file to download:" mimeType="text/xml">$schemaFile</instruct>
        <!-- Present an instruction forcing an inline display rather than using an editor. -->
        <instruct desc="Message:" forceDisplay="true">"A long text such as detailed instructions, that would otherwise be displayed in an editor rather than follow an inline display."</instruct>
        <!-- Present a text input field storing the result in variable aStringInputValue. -->
        <request desc="Enter a text value:" inputType="TEXT">$aStringInputValue</request>
        <!-- Present a text area input storing the result in variable aLongStringInputValue. -->
        <request desc="Enter a long text value:" inputType="MULTILINE_TEXT">$aLongStringInputValue</request>
        <!-- Present a secret value input storing the result in variable aSecretValue. -->
        <request desc="Enter a secret value:" inputType="SECRET">$aSecretValue</request>
        <!-- Present a single selection dropdown list storing the result in variable aSelectedInputValue. -->
        <request desc="Enter a text value:" options="v1, v2" optionLabels="Value 1, Value 2">$aSelectedInputValue</request>
        <!-- Present a file upload storing the result in variable aBinaryVariable and its file name in aBinaryVariableFileName. -->
        <request desc="Upload a file:" fileName="aBinaryVariableFileName">$aBinaryVariable</request>
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
        <!-- Equivalent to the above but using the inputType. In addition, the file name is recorded and can be retrieved as userInput{anotherFileName}. -->
        <request name="anotherFile" fileName="anotherFileName" desc="Upload another file:" inputType="UPLOAD"/>
    </interact>

To better illustrate how dropdown selections can be defined, the following code sample presents the different ways to define them:

.. code-block:: xml

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

Finally the following code sample illustrates some of the more advanced interaction features, considering an information step to the tester
followed by an administrator-level verification:

.. code-block:: xml

    <!--
        Display an information prompt to the tester, closing it automatically after 10 seconds. The timeout could also be set
        via configuration using for example a domain parameter ($DOMAIN{infoTimeout}).

        The step is hidden as it is not interesting to include in the graphical execution diagram, and the message is set as being
        forced to display as-is, to avoid it being presented in a code editor if it exceeds the inline display limit.
    -->
    <interact hidden="true" desc="Test information" timeout="10000">
        <instruct desc="Message:" forceDisplay="true">"Please wait until the administrator validates your results."</instruct>
    </interact>

    <!--
        Display an interaction prompt to an administrator to provide inputs.

        The administrator is also expected to provide an internal code that will not be presented in the step's report but can
        be subsequently used in other test steps.
    -->
    <interact id="adminData" desc="Confirm results" admin="true">
        <request desc="Comments" inputType="MULTILINE_TEXT" name="comments"/>
        <!-- The "code" input will be recorded but not added to reports. -->
        <request desc="Internal code" name="code" report="false"/>
    </interact>

    <log>$adminData{code}</log>

.. index:: log
.. index:: lang (log)
.. index:: source (log)
.. index:: asTemplate (log)
.. index:: skipped (log)
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

The log output is determined by an :ref:`expression <test-case-expressions>` provided as the text content of the ``log`` element.
The element's structure is as follows:

.. csv-table::
    :stub-columns: 1
    :delim: ~
    :header: "Name", "Required?", "Description"

    @asTemplate~ no~ Whether or not the result will be considered as a template for placeholder replacement (see :ref:`test-case-expressions-template-files`). By default this is "false".
    @lang~ no~ Not used currently (and defaulting to XPath as the built-in :ref:`expression language <test-case-expressions>`).
    @level~ no~ The severity level to consider for the log entry. This can be (in increasing severity) ``DEBUG``, ``INFO`` (the default level), ``WARNING`` or ``ERROR``. It can also be provided as a variable reference. See :ref:`tdl-steps-common-level` for further details.
    @skipped~ no~ A boolean value or variable reference (default being "false") which will result in the step being skipped if "true" See also :ref:`tdl-steps-common-skipped`.
    @source~ no~ A variable reference to identify a source ``object`` variable upon which the expression should be evaluated.
    @stopOnError~ no~ A boolean flag determining whether the test session should end if this step fails (default is "false"). See also :ref:`tdl-steps-common-stoponerror`.

The most typical usage of the ``log`` step is to print a message, potentially also referencing variables form the session's state. This can be
easily done using the XPath concatenation operator (``||``) or even by defining the ``log`` step's value as a template and having it processed as
such by setting ``asTemplate`` to "true":

.. code-block:: xml

    <!-- Generate a UUID. -->
    <process handler="TokenGenerator" output="uuid" operation="uuid"/>
    <!-- Log using string concatenation. -->
    <log>"Expecting UUID '" || $uuid || "' to be referenced in message."</log>
    <!-- Log using a string template. -->
    <log asTemplate="true">"Expecting UUID '${uuid}' to be referenced in message."</log>

The following example illustrates further ways the ``log`` step can be used, considering in this case input provided by the
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
    <log>'You selected: ' || $input{flag}</log>
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

Using the ``log`` step provides flexibility to test developers for conveying information to users that may be difficult to present on the test execution
diagram. When considering such log contributions, the ``log`` step is complemented by the `logging capabilities`_ of `custom test services`_ used as
:ref:`remote service handlers<handlers>` for messaging (:ref:`send<tdl-step-send>`, :ref:`receive<tdl-step-receive>`), processing (:ref:`process<tdl-step-process>`)
and validation (:ref:`verify<tdl-step-verify>`) steps. Such custom services can contribute to the test session log via service call to the Test Bed.

.. note::
    **Test case log level:** You can configure the :ref:`minimum log level for a test case<test-case-steps>` to control which log
    messages are included in the session log.

.. index:: verify
.. index:: id (verify)
.. index:: desc (verify)
.. index:: handler (verify)
.. index:: level (verify)
.. index:: skipped (verify)    
.. index:: stopOnError (verify)
.. index:: output (verify)
.. index:: documentation (verify)
.. index:: property (verify)
.. index:: config (verify)
.. index:: input (verify)
.. index:: hidden (verify)
.. index:: ERROR (verify)
.. index:: WARNING (verify)
.. index:: invert (verify)
.. index:: handlerTimeout (verify)
.. index:: handlerTimeoutFlag (verify)
.. index:: actor (verify)
.. _tdl-step-verify:

verify
~~~~~~

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
    @skipped~ no~ A boolean value or variable reference (default being "false") which will result in the step being skipped if "true" See also :ref:`tdl-steps-common-skipped`.
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

    encoding, no, In case an ``import`` reference is defined this can be used to specify the file's encoding. If not provided ``UTF-8`` is considered.
    from, no, The identifier of a test suite from which the ``import`` file will be loaded. If unspecified the current test suite is assumed.
    import, no, A reference to a separate file within the test suite archive that defines the documentation content.

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
    <verify handler="XmlValidator" desc="Validate invoice against schema">
        <documentation>This is an extra documentation item.</documentation>
        <input name="xml">$invoice</input>
        <input name="xsd">$schema</input>
    </verify>
    <!-- Additional documentation as rich HTML content. -->
    <verify handler="XmlValidator" desc="Validate invoice against business rules" level="WARNING">
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
        <input name="xml">$invoice</input>
        <input name="schematron" source="$schematron"/>
    </verify>

.. note::
    Documentation such as this is also supported for the overall :ref:`test suite<test-suite-metadata>` and the :ref:`test cases<test-case-metadata>` included in the test suite.

.. _tdl-steps-common-stop:

Stop execution in case of errors
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

During the course of a test session you will need to consider step failures and define how these are to affect the test session's execution.
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

To manage all such cases and fine-tune your error handling, test steps provide specialised error-handling flags. Specifically:

* The ``stopOnError`` flag, available on all steps, to :ref:`terminate the test session <tdl-steps-common-stoponerror>` in case of error.
* The ``stopOnChildError`` flag, available on steps containing child steps, to :ref:`skip processing child steps <tdl-steps-common-stoponchilderror>` in case of error.

.. index:: stopOnError
.. _tdl-steps-common-stoponerror:

Stop the test session in case of error
++++++++++++++++++++++++++++++++++++++

By default a test session continues processing all steps regardless of failures. If you want to stop test execution upon a failure you can use
the boolean ``stopOnError`` flag. This flag can be set on:

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

.. index:: stopOnChildError
.. _tdl-steps-common-stoponchilderror:

Skip child steps in case of error
+++++++++++++++++++++++++++++++++

Several test steps are used to define **step sequences**, in that they include additional steps as children. The simplest of these is the
:ref:`group step <tdl-step-group>` that aggregates related steps to manage their common behavior and display them visually as a group.
For such sequence steps we may want to continue their execution until a failure occurs, and then skip remaining steps but without terminating
the test session as a whole. To achieve this effect you can use the boolean ``stopOnChildError`` flag.

The ``stopOnChildError`` flag can be an interesting alternative to the ``stopOnError`` flag, where you want to skip processing certain steps but
ensure overall processing continues. A good example of this is a test case that includes **setup** and **teardown phases** around the core testing
steps. This is illustrated in the following test case:

.. code-block:: xml

    <steps>
        <!--
            Setup steps making HTTP calls to the SUT to initialise it with test datasets.
        -->
        <group title="Setup phase" collapsed="true">
            <send desc="Store test dataset 1" handler="HttpMessagingV2">...</send>
            <send desc="Store test dataset 2" handler="HttpMessagingV2">...</send>
        </group>
        <!--
            Carry out the test case's message exchanges and validations. By setting here
            stopOnChildError to true any error will skip subsequent steps within the group
            but will not terminate the test session.
        -->
        <group hiddenContainer="true" stopOnChildError="true">
            <send desc="Update data" handler="HttpMessagingV2">...</send>
            <verify desc="Check update result">...</verify>
            <send desc="Create new data" handler="HttpMessagingV2">...</send>
            <verify desc="Check create result">...</verify>
        </group>
        <!--
            Teardown steps making HTTP calls to the SUT to remove test data. Note also
            how the severity level of the steps is set to WARNING to make sure that a
            failed teardown step can never itself result in a test session failure.
        -->
        <group title="Teardown phase" collapsed="true">
            <send desc="Delete test dataset 1" handler="HttpMessagingV2" level="WARNING">...</send>
            <send desc="Delete test dataset 2" handler="HttpMessagingV2" level="WARNING">...</send>
        </group>
    </steps>

Groups with different ``stopOnChildError`` values can also be nested to fine tune the test execution. A typical case where this
can be useful is grouping together verifications to ensure all checks are carried out and reported. The following example
shows how we skip messages following a failure, but still carry out all verifications per messaging step:

.. code-block:: xml

    <steps>
        <!--
            Setting stopOnChildError to true ensures failures prevent other child steps from continuing.
        -->
        <group hiddenContainer="true" stopOnChildError="true">
            <send desc="Update data" handler="HttpMessagingV2">...</send>
            <!-- 
                Here we override the stopOnChildError semantics so that we run through all verify steps
                regardless of their individual results.
            -->
            <group title="Validations" stopOnChildError="false">
                <verify desc="Check status code">...</verify>
                <verify desc="Check payload">...</verify>
            </group>
            <!-- 
                In case the previous group step failed, the following steps will be skipped.
            -->
            <send desc="Create new data" handler="HttpMessagingV2">...</send>
            <group title="Validations" stopOnChildError="false">
                <verify desc="Check status code">...</verify>
                <verify desc="Check payload">...</verify>
            </group>
        </group>
    </steps>

The ``stopOnChildError`` flag can be set on any step that defines a sequence of steps, specifically:

* The :ref:`group step <tdl-step-group>`.
* The :ref:`if step <tdl-step-if>`, as well as individual ``then`` and ``else`` blocks.
* The iteration steps (:ref:`while <tdl-step-while>`, :ref:`foreach <tdl-step-foreach>`, :ref:`repuntil <tdl-step-repuntil>`).
* The :ref:`flow step <tdl-step-flow>`, as well as individual ``thread`` blocks.
* The :ref:`call step <tdl-step-call>` in which case it applies to the referred scriptlet's steps.

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

.. index:: skipped
.. _tdl-steps-common-skipped:

Skip test steps
~~~~~~~~~~~~~~~

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

.. index:: level
.. _tdl-steps-common-level:

Treat step failures as warnings
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Steps that perform validations support a ``level`` attribute to define their **severity level**. This attribute, by default considered as ``ERROR``,
can be set to ``WARNING`` so that a failure is presented as a warning, and importantly does not impact the overall result of the test session. Moreover,
this attribute can be set with a fixed value, or evaluated dynamically as a :ref:`variable reference<test-case-referring-to-variables>`.

The steps that support the ``level`` attribute are:

* The :ref:`verify <tdl-step-verify>` step.
* The :ref:`process <tdl-step-process>` step.
* The :ref:`send <tdl-step-send>`, :ref:`receive <tdl-step-receive>` and :ref:`listen <tdl-step-listen>` messaging steps.

Setting a :ref:`verify <tdl-step-verify>` step's severity level to ``WARNING`` can be useful if you want to carry out **optional checks**. These will be
appropriately displayed as warnings, leaving the overall test session outcome unaffected. This is illustrated in the following example, which goes
a step further defining the severity level through configuration:

.. code-block:: xml

    <!-- 
        Check required assertions (the severity level is implicitly "ERROR").
    -->
    <verify handler="XmlValidator" desc="Check core features">
        ...
    </verify>
    <!-- 
        Check experimental assertions. The severity level is determined via a domain
        configuration parameter, set either as "WARNING" or "ERROR".
    -->
    <verify handler="XmlValidator" desc="Check optional features" level="$DOMAIN{optionalLevel}">
        ...
    </verify>

Similarly, setting the severity level on messaging and processing steps is useful given that these steps can also be used to return validation
reports. Besides managing optional validations however, setting such steps at warning level ensures that they will never result in
test session failures. Scnearios where this could be useful include:

* Using a :ref:`process <tdl-step-process>` step to trigger **downstream actions** such as collecting statistics.
* Using one or more :ref:`send <tdl-step-send>` steps to perform **teardown operations** on the SUT.

The following example illustrates two teardown HTTP calls that we would want to execute at the end of a test session, ensuring 
that any failures they produce are ignored:

.. code-block:: xml

    <group title="Teardown phase" collapsed="true">
        <send desc="Delete test dataset 1" handler="HttpMessagingV2" level="WARNING">...</send>
        <send desc="Delete test dataset 2" handler="HttpMessagingV2" level="WARNING">...</send>
    </group>

.. index:: handlerTimeout
.. index:: handlerTimeoutFlag
.. _tdl-steps-common-handlerTimeouts:

Timeouts for external service handlers
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Certain test steps support delegating processing to an external test service defined as the steps' :ref:`handler <handlers-custom-handlers>`.
This is achieved by means of the ``handler`` attribute, which in the case of an external service is set with the URL of
the service's endpoint. Moreover, this endpoint is typically defined as a :ref:`domain comfiguration parameter <test-case-expressions-domain>` for
portability and easier configuration.

.. code-block:: xml

    <!--
      Use a custom messaging service (the endpoint of which is configured as a domain parameter)
      to send a message.
    -->
    <send id="sendMessage" desc="Send message" handler="$DOMAIN{messagingService}">
        <input name="messageToSend">$message</input>
    </send>

When such a test service is called, the Test Bed will wait by default until the service completes its processing, no matter
how long this may take. You might want to limit this waiting period by specifying a **response timeout**, ensuring that your
test session does not block indefinitely. Managing such timeouts is done via two attributes:

* ``handlerTimeout``, set with the number of milliseconds to wait before timing out (can also be provided via :ref:`variable reference <test-case-referring-to-variables>`).
* ``handlerTimeoutFlag``, provided with the name of a boolean variable to set depending on whether a timeout occurred or not.

Using these attributes it is possible to fine tune your test case, to better inform the user or even to adapt your testing logic.
Taking the example of a :ref:`verify step <tdl-step-verify>` calling a remote validation service, you can use timeouts as follows:

.. code-block:: xml

    <!--
      Time out if the response if not received within 10 seconds.
      This could also have been provided via variable (e.g. $DOMAIN{handlerTimeout}).
    -->
    <verify handler="$DOMAIN{address}" handlerTimeout="10000" handlerTimeoutFlag="timeoutOccurred" desc="Call remote validator">
        <input name="contentToValidate">$content</input>
    </verify>
    <!--
      Prints 'true' if the step failed due to a timeout.
    -->
    <log>$timeoutOccurred</log>

.. note::
    These attributes have no effect when a :ref:`built-in handler <handlers-predefined-handlers>` is used. In addition, note that they
    cover response timeouts, not **connection timeouts** that may come up due to networking issues or an invalid endpoint address.

In the case of steps that are inherently asynchronous, notably the :ref:`receive <tdl-step-receive>` and :ref:`interact <tdl-step-interact>` steps,
the ``handlerTimeout`` applies in addition to the steps' ``timeout`` attribute. The distinction here is that the ``timeout`` attribute
limits how long the test session will idly wait for an update, whereas the ``handlerTimeout`` attribute limits how long to
allow the service to actively process while poroviding a synchronous response. It is unlikely that you would need to specify
both ``timeout`` and ``handlerTimeout`` attributes, but it could still be interesting if for example the synchronous processing
carried out by a messaging service called via a ``receive`` step may take time before it begins waiting for a message.

.. code-block:: xml

    <!--
      Use a custom service to receive a message from the SUT. Timeouts will be raised in two cases:
      - After 10 seconds while making the call to the test service.
      - After 5 minutes while waiting for the expected SUT message to be received by the service.
    -->
    <receive id="receiveMessage" desc="Receive message" handler="$DOMAIN{messagingService}" timeout="300000" handlerTimeout="10000">
        <input name="expectedSender">$sender</input>
    </receive>

.. index:: actor
.. _tdl-steps-common-step-actor:

Display steps under specific actors
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Test steps are displayed under :ref:`actors <test-case-actors>` depending on their type. Specifically:

* Messaging steps (:ref:`receive <tdl-step-receive>`, and :ref:`send <tdl-step-send>`) are presented as arrows between
  the ``from`` and ``to`` actors, with their direction matching the communication flow.
* Interactions (:ref:`interact <tdl-step-interact>` steps) are presented as arrows between the predefined user or
  administrator (for :ref:`administrator-related interactions <tdl-step-interact_admin_interactions>`) actor, and the
  "Test engine" actor. The direction depends on the presence of :ref:`input requests <tdl-step-interact_form_inputs>` and
  :ref:`instructions <tdl-step-interact_instruct_presentation>`.
* All other steps (:ref:`verify <tdl-step-verify>`, :ref:`process <tdl-step-process>`, and :ref:`exit <tdl-step-exit>`) are
  displayed under the predefined "Test engine" actor.

With the exception of messaging steps where related actors are explicit, all other steps support the ``actor`` attribute,
to specify the actor under which they are displayed. In doing so, you can effectively replace the "Test engine"
actor if doing so results in a more meaningful test execution diagram. The ``actor`` attribute is set in all cases with
the ``id`` of the relevant :ref:`actor element <test-case-actors>`, and in the case of :ref:`scriptlets <scriptlets>`,
can also be :ref:`set dynamically <scriptlets_dynamic_references>` at test case load time.

The following example shows a simulated actor receiving a message from the SUT and displaying its validation
under the same actor, as opposed to using the "Test engine" actor.

.. code-block:: xml

    <!--
      Receive a message. This is sent from the 'sender' (the SUT) to the 'receiver'.
    -->
    <receive id="receiveMessage" desc="Receive message" from="sender" to="receiver" handler="HttpMessagingV2">
       ...
    </receive>
    <!--
      Validate the message. Show this under the 'receiver' actor to illustrate where the validation
      conceptually takes place.
    -->
    <verify desc="Validate message" actor="receiver" handler="JsonValidator">
        <input name="json">$receiveMessage{request}{body}</input>
        <input name="schema">$schema</input>
    </verify>

Using the ``actor`` attribute only affects how the test execution diagram is presented. You would typically use it to
show that steps are conceptually executed by certain actors. In contrast, using the default "Test engine" actor could
help distinguish assertions and processing actions as test engine steps. It is up to you to determine which presentation
approach is more appropriate for your users.

.. note::
    Besides replacing the "Test engine" actor in steps, you can also :ref:`adapt its name and display order <test-case-actors>`.

.. _validation report context: https://www.itb.ec.europa.eu/docs/services/latest/common/index.html#constructing-a-validation-report-tar
.. _messaging service documentation: https://www.itb.ec.europa.eu/docs/services/latest/messaging/index.html#receive
.. _mime type: https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types/Common_types
.. _custom test services: https://www.itb.ec.europa.eu/docs/services/latest/
.. _logging capabilities: https://www.itb.ec.europa.eu/docs/services/latest/common/index.html#contributing-to-test-session-logs