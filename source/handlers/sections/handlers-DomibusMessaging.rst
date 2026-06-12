Used to send or receive messages over eDelivery by means of a `Domibus Access Point <https://ec.europa.eu/digital-building-blocks/wikis/display/DIGITAL/Domibus>`__.

The ``DomibusMessaging`` handler is used to interact with a Domibus Access Point that is set up externally from the Test Bed.
The installation, configuration and management of this Access Point, are not handled by the Test Bed. In addition, note that
operations are carried out through the Domibus backend web service API which should be accessible to the test engine.

.. index:: DomibusMessaging (send - backendAddress)
.. index:: DomibusMessaging (send - backendAuthType)
.. index:: DomibusMessaging (send - backendPassword)
.. index:: DomibusMessaging (send - backendUsername)
.. index:: DomibusMessaging (send - header)
.. index:: DomibusMessaging (send - payload)
.. index:: DomibusMessaging (send - messageIdentifier)
.. _handlers-DomibusMessaging-send:

Use in send steps
^^^^^^^^^^^^^^^^^

To use this handler to **send a message** via a Domibus Access Point, set it as the ``handler`` of a :ref:`send <tdl-step-send>` step
(or the step's :ref:`transaction <tdl-step-btxn>`). In this case the supported step **inputs** are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Type", "Description"
    :delim: |

    ``backendAddress`` | Yes | ``string`` | The full URL for the WSDL of the Domibus backend web service API.
    ``backendAuthType`` | No | ``string`` | The authentication type needed to call the backend web service API. Can be "basic" (the default) or "digest".
    ``backendPassword`` | No | ``string`` | The password to use when calling the backend web service API (if authentication is needed).
    ``backendUsername`` | No | ``string`` | The username to use when calling the backend web service API (if authentication is needed).
    ``header`` | Yes | ``binary`` | The message header.
    ``payload`` | Yes | ``list[binary]`` | A list of payloads, or a single payload, to include in the message.

Once the :ref:`send <tdl-step-send>` step completes, the resulting report will include the following data:

.. csv-table::
    :header: "Output name", "Always present?", "Type", "Description"
    :delim: |

    ``messageIdentifier`` | Yes | ``string`` | The identifier that was assigned to the sent message by Domibus.
    ``header`` | Yes | ``binary`` | The header of the message that was sent.
    ``payload`` | Yes | ``list[map]`` | A list of the sent payloads. Each payload entry is a map, with keys ``identifier`` (for the payload identifier) and ``content`` (for the payload data).

The following example shows how to use the handler to send messages through Domibus. The Domibus backend API is
configured as a :ref:`domain parameter <test-case-expressions-domain>`.

.. code-block:: xml

    <!--
        Send a message with two payloads.
    -->
    <assign to="payloads" append="true">$payload1</assign>
    <assign to="payloads" append="true">$payload2</assign>
    <send id="message" desc="Send message" handler="DomibusMessaging">
        <!--
            The header could be defined via a template and processed with the TemplateProcessor
            to replace information such as the recipient identifier, using the system's
            configuration (e.g. $SYSTEM{partyIdentifier}).
        -->
        <input name="header">$header</input>
        <!--
            If sending a single payload this could be passed as-is without needing a list.
        -->
        <input name="payload">$payloads</input>
        <!--
            Set to e.g. 'http://10.0.0.4:18080/domibus/services/backend?wsdl'
        -->
        <input name="backendAddress">$DOMAIN{domibusBackend}</input>
    </send>
    <!--
        Print the generated message identifier.
    -->
    <log>$message{messageIdentifier}</log>
    <!--
        Print the header that was used.
    -->
    <log>$message{header}</log>
    <!--
        Print the payload identifiers.
    -->
    <foreach item="payload" of="$message{payload}" hidden="true">
       <do>
          <log>$payload{identifier}</log>
       </do>
    </foreach>

.. index:: DomibusMessaging (receive message - backendAddress)
.. index:: DomibusMessaging (receive message - backendAuthType)
.. index:: DomibusMessaging (receive message - backendPassword)
.. index:: DomibusMessaging (receive message - backendUsername)
.. index:: DomibusMessaging (receive message - initialPollDelay)
.. index:: DomibusMessaging (receive message - maximumPollAttempts)
.. index:: DomibusMessaging (receive message - messageIdentifier)
.. index:: DomibusMessaging (receive message - pollInterval)
.. index:: DomibusMessaging (receive message - type)
.. index:: DomibusMessaging (receive message - header)
.. index:: DomibusMessaging (receive message - payload)
.. _handlers-DomibusMessaging-receive-message:

Use in receive steps to receive messages
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To use this handler to **receive a message** through your Domibus Access Point, set it as the ``handler`` of a :ref:`receive <tdl-step-receive>` step
(or the step's :ref:`transaction <tdl-step-btxn>`). In this case the supported step **inputs** are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Type", "Description"
    :delim: |

    ``backendAddress`` | Yes | ``string`` | The full URL for the WSDL of the Domibus backend web service API.
    ``backendAuthType`` | No | ``string`` | The authentication type needed to call the backend web service API. Can be "basic" (the default) or "digest".
    ``backendPassword`` | No | ``string`` | The password to use when calling the backend web service API (if authentication is needed).
    ``backendUsername`` | No | ``string`` | The username to use when calling the backend web service API (if authentication is needed).
    ``initialPollDelay`` | No | ``number`` | The initial delay before the first polling attempt.
    ``maximumPollAttempts`` | No | ``number`` | The maximum number of polling attempts to make.
    ``messageIdentifier`` | Yes | ``string`` | The identifier of the message to wait for.
    ``pollInterval`` | No | ``number`` | The internal in milliseconds between polling attempts.
    ``type`` | No | ``string`` | Leave unspecified or set explicitly to ``message`` (the default value).

Receiving a message is based on an expected **message identifier**. The check for this message is made by polling the
Domibus backend web service API until the polling threshold elapses, or until the message is downloaded. Polling steps
are logged in the test session log to allow monitoring of the step's status.

.. code-block:: none

  ...
  Polling for message [message-123456789]...
  Message [message-123456789] not found after 1 attempt...
  Message [message-123456789] not found after 2 attempts...
  Downloaded message [message-123456789].

Once the :ref:`receive <tdl-step-receive>` step completes, the resulting report will include the following data:

.. csv-table::
    :header: "Output name", "Always present?", "Type", "Description"
    :delim: |

    ``messageIdentifier`` | Yes | ``string`` | The message's identifier.
    ``header`` | Yes | ``binary`` | The header of the received message.
    ``payload`` | Yes | ``list[map]`` | A list of the received payloads. Each payload entry is a map, with keys ``identifier`` (for the payload identifier), ``content`` (for the payload data), and ``contentType`` for the content type (if defined).

The following example shows how to use this handler to receive an expected message. In this case, the message identifier is provided
by the user in a prior :ref:`interact <tdl-step-interact>` step.

.. code-block:: xml

    <!--
        Request the message identifier to wait for.
    -->
    <interact id="userData" desc="Provide inputs">
        <request desc="Message identifier" name="identifier" required="true"/>
    </interact>
    <!--
        Wait to receive a message with the provided identifier.
    -->
    <receive id="messageData" desc="Receive message" handler="DomibusMessaging">
        <input name="messageIdentifier">$userData{identifier}</input>
        <input name="backendAddress">$DOMAIN{domibusBackend}</input>
        <!--
            Make 10 total attempts separated by 5 second intervals.
        -->
        <input name="maximumPollAttempts">10</input>
        <input name="pollInterval">5000</input>
    </receive>

.. index:: DomibusMessaging (check acknowledgement - backendAddress)
.. index:: DomibusMessaging (check acknowledgement - backendAuthType)
.. index:: DomibusMessaging (check acknowledgement - backendPassword)
.. index:: DomibusMessaging (check acknowledgement - backendUsername)
.. index:: DomibusMessaging (check acknowledgement - failureState)
.. index:: DomibusMessaging (check acknowledgement - initialPollDelay)
.. index:: DomibusMessaging (check acknowledgement - maximumPollAttempts)
.. index:: DomibusMessaging (check acknowledgement - messageIdentifier)
.. index:: DomibusMessaging (check acknowledgement - pollInterval)
.. index:: DomibusMessaging (check acknowledgement - successState)
.. index:: DomibusMessaging (check acknowledgement - type)
.. index:: DomibusMessaging (check acknowledgement - status)
.. _handlers-DomibusMessaging-receive-ack:

Use in receive steps to check for acknowledgements
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To use this handler to **check for a message acknowledgement**, set it as the ``handler`` of a :ref:`receive <tdl-step-receive>` step
(or the step's :ref:`transaction <tdl-step-btxn>`). In this case the supported step **inputs** are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Type", "Description"
    :delim: |

    ``backendAddress`` | Yes | ``string`` | The full URL for the WSDL of the Domibus backend web service API.
    ``backendAuthType`` | No | ``string`` | The authentication type needed to call the backend web service API. Can be "basic" (the default) or "digest".
    ``backendPassword`` | No | ``string`` | The password to use when calling the backend web service API (if authentication is needed).
    ``backendUsername`` | No | ``string`` | The username to use when calling the backend web service API (if authentication is needed).
    ``failureState`` | No | ``list[string]`` | The list of final status values that will result in a failed acknowledgement check (by default ``SEND_FAILURE`` and ``NOT_FOUND``).
    ``initialPollDelay`` | No | ``number`` | The initial delay before the first polling attempt.
    ``maximumPollAttempts`` | No | ``number`` | The maximum number of polling attempts to make.
    ``messageIdentifier`` | Yes | ``string`` | The identifier of the message to check for an acknowledgement.
    ``pollInterval`` | No | ``number`` | The internal in milliseconds between polling attempts.
    ``successState`` | No | ``list[string]`` | The list of final status values that will result in a successful acknowledgement check (by default ``ACKNOWLEDGED``).
    ``type`` | Yes | ``string`` | Set explicitly to ``acknowledgement``.

Acknowledgement checks expect the **message identifier** of the message in question. The check for the acknowledgement
is made by polling the Domibus backend web service API until the polling threshold elapses, or until a final
acknowledgement status is received. The ``successState`` and ``failureState`` inputs make it possible to adapt the
step's behaviour, by considering for example failed status values as a success state when checking for
a message that should not have been accepted. Polling steps are logged in the test session log to allow
monitoring of the step's status.

.. code-block:: none

  ...
  Polling for acknowledgement [message-123456789]...
  Acknowledgement [message-123456789] not found after 1 attempt...
  Acknowledgement [message-123456789] not found after 2 attempts...
  Successful acknowledgement [ACKNOWLEDGED] for message [message-123456789].

Once the :ref:`receive <tdl-step-receive>` step completes, the resulting report will include the following data:

.. csv-table::
    :header: "Output name", "Always present?", "Type", "Description"
    :delim: |

    ``messageIdentifier`` | Yes | ``string`` | The message's identifier.
    ``status`` | Yes | ``string`` | The final reported status.

The following example shows how to use this handler to check for a message's acknowledgement. The message identifier to
check for is the resulting from sending a message through Domibus.

.. code-block:: xml

    <!--
        Send a message.
    -->
    <send id="messageData" desc="Send message" handler="DomibusMessaging">
        <input name="header">$header</input>
        <input name="payload">$payloads</input>
        <input name="backendAddress">$DOMAIN{domibusBackend}</input>
    </send>
    <!--
        Wait to receive the acknowledgement.
    -->
    <receive id="ackData" desc="Receive message" handler="DomibusMessaging">
        <input name="type">"acknowledgement"</input>
        <input name="messageIdentifier">$messageData{messageIdentifier}</input>
        <input name="backendAddress">$DOMAIN{domibusBackend}</input>
        <!--
            The 'successState' and 'failureState' inputs could also have been
            skipped as they are set with their default values.
        -->
        <input name="successState">"ACKNOWLEDGED"</input>
        <input name="failureState">"NOT_FOUND,SEND_FAILURE"</input>
        <!--
            Make 10 total attempts separated by 5 second intervals.
        -->
        <input name="maximumPollAttempts">10</input>
        <input name="pollInterval">5000</input>
    </receive>