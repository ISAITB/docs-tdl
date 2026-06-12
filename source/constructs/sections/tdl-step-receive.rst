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
    @skipped | no | A boolean value or variable reference (default being "false") which will result in the step being skipped if "true". See also :ref:`tdl-steps-common-skipped`.
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
