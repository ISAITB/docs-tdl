Used to send or receive payloads via SOAP web service calls.

.. index:: SoapMessagingV2 (send)
.. index:: SoapMessagingV2 (send - uri)
.. index:: SoapMessagingV2 (send - envelope)
.. index:: SoapMessagingV2 (send - version)
.. index:: SoapMessagingV2 (send - headers)
.. index:: SoapMessagingV2 (send - action)
.. index:: SoapMessagingV2 (send - attachments)
.. index:: SoapMessagingV2 (send - tolerateNonSoapResponse)
.. index:: SoapMessagingV2 (send - showRequestUri)
.. index:: SoapMessagingV2 (send - showRequestBody)
.. index:: SoapMessagingV2 (send - showRequestEnvelope)
.. index:: SoapMessagingV2 (send - showRequestHeaders)
.. index:: SoapMessagingV2 (send - showRequestAttachments)
.. index:: SoapMessagingV2 (send - showResponseBody)
.. index:: SoapMessagingV2 (send - showResponseAttachments)
.. index:: SoapMessagingV2 (send - showResponseEnvelope)
.. index:: SoapMessagingV2 (send - showResponseStatus)
.. index:: SoapMessagingV2 (send - showResponseHeaders)
.. index:: SoapMessagingV2 (send - showResponseError)
.. index:: SoapMessagingV2 (send - requestHeadersToShow)
.. index:: SoapMessagingV2 (send - requestHeadersToHide)
.. index:: SoapMessagingV2 (send - requestAttachmentsToShow)
.. index:: SoapMessagingV2 (send - requestAttachmentsToHide)
.. index:: SoapMessagingV2 (send - responseHeadersToShow)
.. index:: SoapMessagingV2 (send - responseHeadersToHide)
.. index:: SoapMessagingV2 (send - responseAttachmentsToShow)
.. index:: SoapMessagingV2 (send - responseAttachmentsToHide)
.. _handlers-soapmessagingv2-send:

Use in send steps
^^^^^^^^^^^^^^^^^

To use this handler to **call a SOAP web service** of an external system, set it as the ``handler`` of a :ref:`send <tdl-step-send>` step
(or the step's :ref:`transaction <tdl-step-btxn>`). In this case the supported step **inputs** are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Type", "Description"
    :delim: |

    ``action`` | No | ``string`` | The SOAPAction header to set. This overrides the value set in the HTTP headers (if present).
    ``attachments`` | No | ``list`` | The MTOM attachments to set, provided as a list of maps or a single map. Each map corresponds to an attachment and includes keys *"name"* (required), *"content"* (required), *"contentType"* and *"forceText"* (a boolean to force the attachment's inclusion as text).
    ``envelope`` | Yes | ``object`` | The SOAP envelope to send (as an XML object).
    ``headers`` | No | ``map`` | The HTTP headers to include in the request, provided as a map of key (header name) to comma-separated string values (header values). Note that the Content-Type header is automatically set based on the SOAP version used.
    ``requestAttachmentsToHide`` | No | ``boolean`` | When request SOAP attachments are displayed (see ``showRequestAttachments``), this is the ``list`` of attachments to explicitly hide (can also be provided as a single ``string``).
    ``requestAttachmentsToShow`` | No | ``boolean`` | When request SOAP attachments are displayed (see ``showRequestAttachments``), this is the ``list`` of attachments to explicitly show (can also be provided as a single ``string``).
    ``requestHeadersToHide`` | No | ``boolean`` | When request HTTP headers are displayed (see ``showRequestHeaders``), this is the ``list`` of headers to explicitly hide (can also be provided as a single ``string``).
    ``requestHeadersToShow`` | No | ``boolean`` | When request HTTP headers are displayed (see ``showRequestHeaders``), this is the ``list`` of headers to explicitly show (can also be provided as a single ``string``).
    ``responseAttachmentsToHide`` | No | ``boolean`` | When response SOAP attachments are displayed (see ``showRequestAttachments``), this is the ``list`` of attachments to explicitly hide (can also be provided as a single ``string``).
    ``responseAttachmentsToShow`` | No | ``boolean`` | When response SOAP attachments are displayed (see ``showRequestAttachments``), this is the ``list`` of attachments to explicitly show (can also be provided as a single ``string``).
    ``responseHeadersToHide`` | No | ``boolean`` | When request HTTP headers are displayed (see ``showRequestHeaders``), this is the ``list`` of headers to explicitly hide (can also be provided as a single ``string``).
    ``responseHeadersToShow`` | No | ``boolean`` | When response HTTP headers are displayed (see ``showRequestHeaders``), this is the ``list`` of headers to explicitly show (can also be provided as a single ``string``).
    ``showRequestAttachments`` | No | ``boolean`` | Whether to display the request's SOAP attachments in the step's report (default is true).
    ``showRequestBody`` | No | ``boolean`` | Whether to display the request's SOAP body in the step's report (default is true).
    ``showRequestEnvelope`` | No | ``boolean`` | Whether to display the request's SOAP envelope in the step's report (default is true).
    ``showRequestHeaders`` | No | ``boolean`` | Whether to display the request's HTTP headers in the step's report (default is true).
    ``showRequestUri`` | No | ``boolean`` | Whether to display the request's URI in the step's report (default is true).
    ``showResponseAttachments`` | No | ``boolean`` | Whether to display the response's SOAP attachments in the step's report (default is true).
    ``showResponseBody`` | No | ``boolean`` | Whether to display the response's SOAP body in the step's report (default is true).
    ``showResponseEnvelope`` | No | ``boolean`` | Whether to display the response's SOAP envelope in the step's report (default is true).
    ``showResponseError`` | No | ``boolean`` | Whether to display the response's error fault in the step's report (default is true).
    ``showResponseHeaders`` | No | ``boolean`` | Whether to display the response's HTTP headers in the step's report (default is true).
    ``showResponseStatus`` | No | ``boolean`` | Whether to display the response's HTTP status in the step's report (default is true).
    ``tolerateNonSoapResponse`` | No | ``boolean`` | Whether a non-SOAP response will be tolerated (otherwise the step results in a failure).
    ``uri`` | Yes | ``string`` | The URI of the SOAP endpoint to be called.
    ``version`` | No | ``string`` | The SOAP protocol version (either "1.1" or "1.2"). If not provided this is calculated from the provided Content-Type HTTP header, otherwise "1.1" is used.

Once the :ref:`send <tdl-step-send>` step completes, the resulting report will include two nested maps named ``request`` and ``response``, corresponding
respectively to the submitted request and received response. The ``request`` map includes the following:

.. csv-table::
    :header: "Output name", "Always present?", "Type", "Description"
    :delim: |

    ``attachments`` | No | ``map`` | The MTOM attachments sent with the envelope. The key of each entry is the attachment name, whereas the value is the content.
    ``body`` | Yes | ``object`` | The SOAP body extracted from the envelope.
    ``envelope`` | Yes | ``object`` | The SOAP envelope that was sent.
    ``headers`` | Yes | ``map`` | The HTTP headers added to the request, provided as a map of key (header name) to comma-separated string values (header values).
    ``uri`` | Yes | ``string`` | The URI of the endpoint that was called.

The ``response`` map includes the following:

.. csv-table::
    :header: "Output name", "Always present?", "Type", "Description"
    :delim: |

    ``attachments`` | No | ``map`` | The MTOM attachments sent with the envelope. The key of each entry is the attachment name, whereas the value is the content.
    ``body`` | No | ``object`` | The SOAP body extracted from the response envelope.
    ``envelope`` | No | ``object`` | The SOAP envelope of the response.
    ``error`` | No | ``binary`` | In case the response could not be parsed as a SOAP envelope this property includes the HTTP response's payload.
    ``headers`` | No | ``map`` | The HTTP headers from the response, provided as a map of key (header name) to comma-separated string values (header values).
    ``status`` | Yes | ``number`` | The HTTP status code of the response.

The following GITB TDL snippets provide various examples using the ``SoapMessagingV2`` handler in :ref:`send <tdl-step-send>` steps, as well as accessing
outputs (see inline comments for details per case).

.. code-block:: xml

    <!--
        Make a SOAP 1.1 call to https://my.sut.org/api/service.
    -->
    <send id="send1" desc="Send SOAP message" from="Actor1" to="Actor2" handler="SoapMessagingV2">
        <input name="uri">"https://my.sut.org/api/service"</input>
        <input name="envelope">$send1Envelope</input>
    </send>
    <log>$send1{request}{envelope}</log> <!-- Print the SOAP envelope that was sent. -->
    <log>$send1{response}{body}</log> <!-- Print the response's SOAP body. -->
    <log>$send1{response}{headers}{Content-Type}</log> <!-- Print the response HTTP Content-Type header. -->

    <!--
        Make a SOAP 1.2 call to https://my.sut.org/api/service2 setting the SOAP action.
    -->
    <send id="send2" desc="Send SOAP message" from="Actor1" to="Actor2" handler="SoapMessagingV2">
        <input name="uri">"https://my.sut.org/api/service2"</input>
        <input name="envelope">$send2Envelope</input>
        <input name="action">"http://my.sut.org/MyService/Operation1"</input>
        <input name="version">"1.2"</input>
    </send>
    <log>$send2{request}{envelope}</log> <!-- Print the SOAP envelope that was sent. -->
    <log>$send2{response}{body}</log> <!-- Print the response's SOAP body. -->
    <log>$send2{response}{headers}{Content-Type}</log> <!-- Print the response HTTP Content-Type header. -->

    <!--
        Make a SOAP 1.1 call to https://my.sut.org/api/service3 with MTOM attachments (a binary file and a JSON document).
    -->
    <assign to="send3File1{name}">"id1"</assign>
    <assign to="send3File1{content}">$send3Attachment1</assign>
    <assign to="send3File1{contentType}">"application/zip"</assign>

    <assign to="send3File2{name}">"id2"</assign>
    <assign to="send3File2{content}">$send3Attachment2</assign>
    <assign to="send3File2{contentType}">"application/json"</assign>
    <assign to="send3File2{forceText}">true()</assign> <!-- Add the attachment inline as text rather than Base64. -->

    <assign to="send3Attachments" append="true">$send3File1</assign>
    <assign to="send3Attachments" append="true">$send3File2</assign>

    <send id="send3" desc="Send SOAP message" from="Actor1" to="Actor2" handler="SoapMessagingV2">
        <input name="uri">"https://my.sut.org/api/service3"</input>
        <input name="envelope">$send3Envelope</input>
        <input name="attachments">$send3Attachments</input>
    </send>
    <log>$send3{request}{envelope}</log> <!-- Print the SOAP envelope that was sent. -->
    <log>$send3{request}{attachments}{id2}</log> <!-- Print the JSON file sent as an attachment. -->
    <log>$send3{request}{headers}{Content-Type}</log> <!-- Print the HTTP Content-Type header that was generated for the request. -->
    <log>$send3{response}{status}</log> <!-- Print the response status. -->
    <log>$send3{response}{body}</log> <!-- Print the response SOAP body. -->

.. index:: SoapMessagingV2 (receive)
.. index:: SoapMessagingV2 (receive - uriExtension)
.. index:: SoapMessagingV2 (receive - status)
.. index:: SoapMessagingV2 (receive - envelope)
.. index:: SoapMessagingV2 (receive - version)
.. index:: SoapMessagingV2 (receive - attachments)
.. index:: SoapMessagingV2 (receive - headers)
.. index:: SoapMessagingV2 (receive - showRequestUri)
.. index:: SoapMessagingV2 (receive - showRequestBody)
.. index:: SoapMessagingV2 (receive - showRequestEnvelope)
.. index:: SoapMessagingV2 (receive - showRequestHeaders)
.. index:: SoapMessagingV2 (receive - showRequestAttachments)
.. index:: SoapMessagingV2 (receive - showResponseBody)
.. index:: SoapMessagingV2 (receive - showResponseAttachments)
.. index:: SoapMessagingV2 (receive - showResponseEnvelope)
.. index:: SoapMessagingV2 (receive - showResponseStatus)
.. index:: SoapMessagingV2 (receive - showResponseHeaders)
.. index:: SoapMessagingV2 (receive - showResponseError)
.. index:: SoapMessagingV2 (receive - requestHeadersToShow)
.. index:: SoapMessagingV2 (receive - requestHeadersToHide)
.. index:: SoapMessagingV2 (receive - requestAttachmentsToShow)
.. index:: SoapMessagingV2 (receive - requestAttachmentsToHide)
.. index:: SoapMessagingV2 (receive - responseHeadersToShow)
.. index:: SoapMessagingV2 (receive - responseHeadersToHide)
.. index:: SoapMessagingV2 (receive - responseAttachmentsToShow)
.. index:: SoapMessagingV2 (receive - responseAttachmentsToHide)
.. _handlers-soapmessagingv2-receive:

Use in receive steps
^^^^^^^^^^^^^^^^^^^^

To use this handler to **receive a SOAP web service call** from an external system, set it as the ``handler`` of a :ref:`receive <tdl-step-receive>` step
(or the step's :ref:`transaction <tdl-step-btxn>`). In this case the supported step **inputs** are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Type", "Description"
    :delim: |

    ``attachments`` | No | ``list`` | The MTOM attachments to include in the response, provided as a list of maps or a single map. Each map corresponds to an attachment and includes keys *"name"* (required), *"content"* (required), *"contentType"* and *"forceText"* (a boolean to force the attachment's inclusion as text).
    ``envelope`` | Yes | ``object`` | The SOAP envelope to respond with (as an XML object).
    ``headers`` | No | ``map`` | The HTTP headers to include in the response, provided as a map of key (header name) to comma-separated string values (header values).
    ``requestAttachmentsToHide`` | No | ``boolean`` | When request SOAP attachments are displayed (see ``showRequestAttachments``), this is the ``list`` of attachments to explicitly hide (can also be provided as a single ``string``).
    ``requestAttachmentsToShow`` | No | ``boolean`` | When request SOAP attachments are displayed (see ``showRequestAttachments``), this is the ``list`` of attachments to explicitly show (can also be provided as a single ``string``).
    ``requestHeadersToHide`` | No | ``boolean`` | When request HTTP headers are displayed (see ``showRequestHeaders``), this is the ``list`` of headers to explicitly hide (can also be provided as a single ``string``).
    ``requestHeadersToShow`` | No | ``boolean`` | When request HTTP headers are displayed (see ``showRequestHeaders``), this is the ``list`` of headers to explicitly show (can also be provided as a single ``string``).
    ``responseAttachmentsToHide`` | No | ``boolean`` | When response SOAP attachments are displayed (see ``showRequestAttachments``), this is the ``list`` of attachments to explicitly hide (can also be provided as a single ``string``).
    ``responseAttachmentsToShow`` | No | ``boolean`` | When response SOAP attachments are displayed (see ``showRequestAttachments``), this is the ``list`` of attachments to explicitly show (can also be provided as a single ``string``).
    ``responseHeadersToHide`` | No | ``boolean`` | When request HTTP headers are displayed (see ``showRequestHeaders``), this is the ``list`` of headers to explicitly hide (can also be provided as a single ``string``).
    ``responseHeadersToShow`` | No | ``boolean`` | When response HTTP headers are displayed (see ``showRequestHeaders``), this is the ``list`` of headers to explicitly show (can also be provided as a single ``string``).
    ``status`` | No | ``number`` | The HTTP status code to return (default is 200).
    ``showRequestAttachments`` | No | ``boolean`` | Whether to display the request's SOAP attachments in the step's report (default is true).
    ``showRequestBody`` | No | ``boolean`` | Whether to display the request's SOAP body in the step's report (default is true).
    ``showRequestEnvelope`` | No | ``boolean`` | Whether to display the request's SOAP envelope in the step's report (default is true).
    ``showRequestHeaders`` | No | ``boolean`` | Whether to display the request's HTTP headers in the step's report (default is true).
    ``showRequestUri`` | No | ``boolean`` | Whether to display the request's URI in the step's report (default is true).
    ``showResponseAttachments`` | No | ``boolean`` | Whether to display the response's SOAP attachments in the step's report (default is true).
    ``showResponseBody`` | No | ``boolean`` | Whether to display the response's SOAP body in the step's report (default is true).
    ``showResponseEnvelope`` | No | ``boolean`` | Whether to display the response's SOAP envelope in the step's report (default is true).
    ``showResponseError`` | No | ``boolean`` | Whether to display the response's error fault in the step's report (default is true).
    ``showResponseHeaders`` | No | ``boolean`` | Whether to display the response's HTTP headers in the step's report (default is true).
    ``showResponseStatus`` | No | ``boolean`` | Whether to display the response's HTTP status in the step's report (default is true).
    ``uriExtension`` | No | ``string`` | A URI extension following the base endpoint path at which the request will be expected.
    ``version`` | No | ``string`` | The SOAP protocol version (either "1.1" or "1.2") to construct the response with. If not provided this is calculated from the provided Content-Type HTTP header, otherwise "1.1" is used.

When the :ref:`receive <tdl-step-receive>` step executes, the test session will be suspended until a matching request is received from the system under test.
The listening endpoint where this request will be received is constructed by concatenating in sequence:

1. The test engine's **base messaging URL**.
2. The unique **API key** of the system under test.
3. If defined, the **URI extension** provided as the step's ``uriExtension`` input.

As an example, for a Test Bed installation running locally with default settings, a system with API key "MY_API_KEY", and a provided URI extension of "/test";
the listen endpoint will be:

``http://localhost:8080/itbsrv/api/soap/MY_API_KEY/test``

Received query string parameters are ignored when matching incoming requests, except if a query string has been included in the ``uriExtension`` input.
In any case, once the :ref:`receive <tdl-step-receive>` step executes, the listen endpoint will be added in the test session log.

.. code-block:: none

    ...
    [2024-06-05 18:19:56] INFO  - Waiting to receive SOAP message at [http://localhost:8080/itbsrv/api/soap/7039EC8EX4786X4103XB40FXC7242D11E9B0/service] for step [Receive SOAP message]

.. note::

    **Supporting parallel test sessions:** The listen endpoint for a :ref:`receive <tdl-step-receive>` step determines also whether test sessions can execute in parallel. If no ``uriExtension`` is
    provided, all test sessions for the system (at least the ones with :ref:`receive <tdl-step-receive>` steps) will need to execute sequentially. You can
    :ref:`configure the relevant test cases <test-case>` with ``supportsParallelExecution`` as "false" to enforce this.

    If possible, try to have ``uriExtension`` differ across test cases, and ideally vary per test session.

Once the :ref:`receive <tdl-step-receive>` step completes, the resulting report will include two nested maps named ``request`` and ``response``, corresponding
respectively to the received request and provided response. The ``request`` map includes the following:

.. csv-table::
    :header: "Output name", "Always present?", "Type", "Description"
    :delim: |

    ``attachments`` | No | ``map`` | The MTOM attachments received with the envelope. The key of each entry is the attachment name, whereas the value is the content.
    ``body`` | Yes | ``object`` | The SOAP body extracted from the envelope.
    ``envelope`` | Yes | ``object`` | The SOAP envelope that was received.
    ``headers`` | Yes | ``map`` | The request HTTP headers, provided as a map of key (header name) to comma-separated string values (header values).
    ``uri`` | Yes | ``string`` | The URI of the endpoint that was called.

The ``response`` map includes the following:

.. csv-table::
    :header: "Output name", "Always present?", "Type", "Description"
    :delim: |

    ``attachments`` | No | ``map`` | The MTOM attachments sent with the response envelope. The key of each entry is the attachment name, whereas the value is the content.
    ``body`` | No | ``object`` | The SOAP body extracted from the response envelope.
    ``envelope`` | No | ``object`` | The SOAP envelope provided as the response.
    ``headers`` | No | ``map`` | The response HTTP headers, provided as a map of key (header name) to comma-separated string values (header values).
    ``status`` | Yes | ``number`` | The HTTP status code of the response.

The following GITB TDL snippets provide various examples using the ``SoapMessagingV2`` handler in :ref:`send <tdl-step-send>` steps, as well as accessing
outputs (see inline comments for details per case).

.. code-block:: xml

    <!--
        Receive a SOAP call and respond with a success and specific envelope.
        The step will complete for a received SOAP call to http://localhost:8080/itbsrv/api/soap/MY_API_KEY/.
    -->
    <receive id="receive1" desc="Receive SOAP message" from="Actor1" to="Actor2" handler="SoapMessagingV2">
        <input name="envelope">$receive1Envelope</input>
    </receive>
    <log>$receive1{request}{body}</log> <!-- Print the received SOAP body. -->
    <log>$receive1{response}{envelope}</log> <!-- Print the SOAP envelope sent as the response. -->

    <!--
        Receive a SOAP call at a specific URI extension and respond with a 500 status and SOAP fault using SOAP version 1.2.
        The step will complete for a received SOAP call to http://localhost:8080/itbsrv/api/soap/MY_API_KEY/service.
    -->
    <receive id="receive2" desc="Receive SOAP message" from="Actor1" to="Actor2" handler="SoapMessagingV2">
        <input name="uriExtension">"/service"</input>
        <input name="status">"500"</input>
        <input name="version">"1.2"</input>
        <input name="envelope">$receive2FaultEnvelope</input>
    </receive>
    <log>$receive2{request}{body}</log> <!-- Print the received SOAP body. -->
    <log>$receive2{response}{status}</log> <!-- Print the status of the response. -->

    <!--
        Receive a SOAP call at a specific URI extension and respond with a success and envelope along with two MTOM attachments  (a binary file and a JSON document).
        The step will complete for a received SOAP call to http://localhost:8080/itbsrv/api/soap/MY_API_KEY/service.
    -->
    <assign to="receive3File1{name}">"id1"</assign>
    <assign to="receive3File1{content}">$receive3Attachment1</assign>
    <assign to="receive3File1{contentType}">"application/zip"</assign>

    <assign to="receive3File2{name}">"id2"</assign>
    <assign to="receive3File2{content}">$receive3Attachment2</assign>
    <assign to="receive3File2{contentType}">"application/json"</assign>
    <assign to="receive3File2{forceText}">true()</assign> <!-- Add the attachment inline as text rather than Base64. -->

    <assign to="receive3Attachments" append="true">$receive3File1</assign>
    <assign to="receive3Attachments" append="true">$receive3File2</assign>
    <receive id="receive3" desc="Receive SOAP message" from="Actor1" to="Actor2" handler="SoapMessagingV2">
        <input name="uriExtension">"/service"</input>
        <input name="envelope">$receive3Envelope</input>
        <input name="attachments">$receive3Attachments</input>
    </receive>

.. _handlers-soapmessagingv2-reportdisplay:

Tuning reported data
^^^^^^^^^^^^^^^^^^^^

You can fine-tune the data displayed by the ``SoapMessagingV2`` handler in test step reports through a set inputs
that apply both when used in :ref:`send <handlers-soapmessagingv2-send>` and :ref:`receive <handlers-soapmessagingv2-receive>` steps.
The display of request and response data is addressed by the following ``boolean`` flags:

* Flags ``showRequestAttachments``, ``showRequestBody``, ``showRequestEnvelope``, ``showRequestHeaders`` and ``showRequestUri`` for requests.
* Flags ``showResponseAttachments``, ``showResponseBody``, ``showResponseEnvelope``, ``showResponseError``, ``showResponseHeaders`` and ``showResponseStatus`` for responses.

These flags determine whether reports will display their respective data. They are optional and by default considered as
true, meaning that all data is displayed. Regarding HTTP headers and SOAP attachments in particular further control is enabled by listing
specific headers to be considered, specifically:

* Inputs ``requestAttachmentsToHide``, ``requestAttachmentsToShow``, ``requestHeadersToHide`` and ``requestHeadersToShow`` to manage specific request headers.
* Inputs ``responseAttachmentsToHide``, ``responseAttachmentsToShow``, ``responseHeadersToHide`` and ``responseHeadersToShow`` to manage specific response headers.

These inputs are considered only in case SOAP attachments or HTTP headers are displayed, meaning that flags ``showRequestAttachments``, ``showRequestHeaders``,
``showResponseAttachments``, or ``showResponseHeaders`` are not set, or are set explicitly to true. They are provided with either a ``list`` of header
names or a single attachment or header, and disregard casing differences, meaning that for example, a "Content-Type" value will also
match "content-type" headers. The headers listed in ``requestAttachmentsToShow``, ``responseAttachmentsToShow``, ``requestHeadersToShow`` and ``responseHeadersToShow`` serve as a
**whitelist**, meaning that only the matching ones are displayed. In contrast, ``requestAttachmentsToHide``, ``requestHeadersToHide``,
``responseAttachmentsToHide`` and ``responseHeadersToHide`` serve as a **blacklist**, meaning that the matching entries are hidden. In the uncommon
scenario where an attachment or header is specified both as being explicitly displayed and hidden, the entry will be hidden.

Note that not specifying these inputs means they are full disregarded. In other words, and taking HTTP headers as an example, if you want to display all request
headers you needn't list them explicitly as ``requestHeadersToShow``. Similarly, when hiding all headers you don't need
to explicitly list them as ``requestHeadersToHide``. What you achieve in listing them is that only certain
headers are managed. The following examples show how header display is managed through these inputs:

.. code-block:: xml

    <!--
      Hide all headers.
    -->
    <send id="send1" desc="Send data" handler="SoapMessagingV2">
        ...
        <input name="showRequestHeaders">false()</input>
        <input name="showResponseHeaders">false()</input>
    </send>
    <!--
      Show all headers (the inputs could also be skipped in this case as this is the default).
    -->
    <send id="send2" desc="Send data" handler="SoapMessagingV2">
        ...
        <input name="showRequestHeaders">true()</input>
        <input name="showResponseHeaders">true()</input>
    </send>
    <!--
      Hide a specific response header while showing all others.
    -->
    <send id="send3" desc="Send data" handler="SoapMessagingV2">
        ...
        <input name="responseHeadersToHide">"HEADER-TO-HIDE"</input>
    </send>
    <!--
      Show only a specific response header while hiding all others.
    -->
    <send id="send4" desc="Send data" handler="SoapMessagingV2">
        ...
        <input name="responseHeadersToShow">"HEADER-TO-SHOW"</input>
    </send>

The following example illustrates a more complete scenario where you are making multiple requests to a SUT
and want to ensure that a sensitive API key header is not displayed. Notice here how hidden data can still
be leveraged in subsequent test steps:

.. code-block:: xml

    <!--
      Call the SUT and hide the X-API-KEY header received in the response.
    -->
    <send id="getApiKey" desc="Get API key" handler="SoapMessagingV2">
        <input name="uri">"https://my.sut.org/api/key"</input>
        <input name="envelope">$getEnvelope</input>
        <input name="responseHeadersToHide">"X-API-KEY"</input>
    </send>
    <!--
      Call the SUT and hide only the X-API-KEY header included in the request.
    -->
    <assign to="submitHeaders{X-API-KEY}">$getApiKey{response}{headers}{X-API-KEY}</assign>
    <send id="submit" desc="Submit data" handler="SoapMessagingV2">
        <input name="uri">"https://my.sut.org/api/submit"</input>
        <input name="envelope">$submitEnvelope</input>
        <input name="headers">$submitHeaders</input>
        <!--
          Hide (only) the X-API-KEY header from the request.
        -->
        <input name="requestHeadersToHide">"X-API-KEY"</input>
        <!--
          Hide the response's body, envelope and headers (i.e. show only the status).
        -->
        <input name="showResponseBody">false()</input>
        <input name="showResponseEnvelope">false()</input>
        <input name="showResponseHeaders">false()</input>
    </send>

Showing and hiding request and response data can be useful in removing **superfluous** information that is not needed in
the step's report. It can also be important in not including information that might be **sensitive** in nature, while still
keeping it available for use in further test steps.