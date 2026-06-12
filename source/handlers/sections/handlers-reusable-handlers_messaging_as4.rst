.. note::
  The built-in :ref:`DomibusMessaging <handlers-DomibusMessaging>` handler allows eDelivery message exchanges via a Domibus
  Access Point without needing an external service.

The AS4 messaging component allows integration with a `Domibus eDelivery access point <https://ec.europa.eu/digital-building-blocks/wikis/display/DIGITAL/Domibus>`_
to send and receive messages over eDelivery using the AS4 protocol. This component allows you to:

* **Send** a message using a prepared header and payload as inputs.
* **Check** for an acknowledgement received for a given message (previously sent) based on a message identifier.
* **Receive** a message based on an expected message identifier. This will poll the Domibus backend API until the specific message is received.

To use this component, pull the `isaitb/asx-messaging-v4 <https://hub.docker.com/r/isaitb/asx-messaging-v4>`_ Docker image to install it on your
environment. The resulting service must be accessible by the Test Bed, and also have access to the relevant Domibus backend API that will handle the messaging.

To **send** a message use a :ref:`send<tdl-step-send>` step with the following inputs:

* ``as4.send.header``, the AS4 message XML header (of type ``string``, ``object`` or ``binary``).
* ``as4.send.payload``, the list of payloads to send (of type ``list[binary]``).

.. code-block:: xml
    :emphasize-lines: 5-8

    <steps>
       <btxn from="sender" to="received" txnId="t1" handler="http://localhost:8080/ms/api/as4messaging?wsdl"/>
       <assign to="header">$headerToUse</assign>
       <assign to="payloads" append="true">$payloadToInclude</assign>
       <send id="data" desc="Send message" from="sender" to="receiver" txnId="t1">
          <input name="as4.send.header">$header</input>
          <input name="as4.send.payload">$payloads</input>
       </send>
       <!-- Assigned message ID returned as "as4.send.messageId" -->
       <log>$data{as4.send.messageId}</log>
       <!--
          Sent content returned as a map named "sentContent" including entries:
          - "as4.send.header": The header of the message.
          - "as4.send.payload": The list of payloads.
       -->
       <assign to="headerSent">$data{sentContent}{as4.send.header}</assign>
       <assign to="firstPayloadSent">$data{sentContent}{as4.send.payload}{0}</assign>
       <etxn txnId="t1"/>
    </steps>

To check for a message's **acknowledgement** use a :ref:`receive<tdl-step-receive>` step with the following inputs:

* ``as4.receive.messageId``, the ID of the message to check for (of type ``string``).
* ``as4.receive.type``, set to "ack_check".

.. code-block:: xml
    :emphasize-lines: 9-12

    <steps>
       <btxn from="sender" to="received" txnId="t1" handler="http://localhost:8080/ms/api/as4messaging?wsdl"/>
       ...
       <send id="data" desc="Send message" from="sender" to="receiver" txnId="t1">
          <input name="as4.send.header">$header</input>
          <input name="as4.send.payload">$payloads</input>
       </send>
       ...
       <receive id="ackData" desc="Receive acknowledgement" from="receiver" to="sender" txnId="t1">
          <input name="as4.receive.messageId">$data{as4.send.messageId}</input>
          <input name="as4.receive.type">'ack_check'</input>
       </receive>
       <!--
          The receive step returns a map of two entries:
          - "as4.send.messageId": The message ID that was acknowledged.
          - "as4.message.send.reason": The acknowledgement text.
          Whether the step is a success or failure depends on the state returned and the service's "messaging.ackFailureStates" environment variable.
       -->
       <log>$ackData{as4.send.messageId}</log>
       <log>$ackData{as4.message.send.reason}</log>
       <etxn txnId="t1"/>
    </steps>

To **receive** a message use a :ref:`receive<tdl-step-receive>` step with the following input:

* ``as4.receive.messageId``, the ID of the message to lookup (of type ``string``).

.. code-block:: xml
    :emphasize-lines: 3-5

    <steps>
       <btxn from="sender" to="receiver" txnId="t1" handler="http://localhost:8080/ms/api/as4messaging?wsdl"/>
       <receive id="data" desc="Receive message" from="sender" to="receiver" txnId="t1">
          <input name="as4.receive.messageId">$messageId</input>
       </receive>
       <!--
          The receive step returns a map of the following entries:
          - "header": The message header (of type "object").
          - "payload": A map containing the payload data. Each separate payload received is added in a separate map named "payload.N" (where N is a 1-based index). Each of these includes the following entries:
             - "payload.id": The ID of the payload (of type string).
             - "payload.contentType": The payload's content/mime type (of type string).
             - "payload.content": The payload's data (of type binary).
       -->
       <assign to="firstReceivedPayload">$data{payload}{payload.1}{payload.content}</assign>
       <etxn txnId="t1"/>
    </steps>