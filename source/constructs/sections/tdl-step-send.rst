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
    @skipped | no | A boolean value or variable reference (default being "false") which will result in the step being skipped if "true". See also :ref:`tdl-steps-common-skipped`.
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
