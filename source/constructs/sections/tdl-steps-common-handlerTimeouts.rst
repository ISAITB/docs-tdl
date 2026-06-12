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
