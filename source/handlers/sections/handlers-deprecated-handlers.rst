.. warning::

    The built-in messaging handlers listed here have been deprecated as they cannot be effectively customised and require the direct exposure of the internal
    test engine to systems under test.

Each following section defines a table with the information expected by each messaging handler. The meaning of this information is as follows:

* **Input:** These are the inputs provided for the send step.
* **Output:** These are the outputs returned from the receive step.
* **Actor configuration:** These are configuration properties that will be automatically set for simulated actors using this handler.
* **Receive configuration:** These are configuration properties expected by the receive step.
* **Send configuration:** These are configuration properties expected by the send step.
* **Transaction configuration:** These are configuration properties defined in the btxn or bptxn step.

The title of each section corresponds to the name of the handler that needs to be configured in the relevant step's ``handler`` attribute (in
:ref:`send <tdl-step-send>`, :ref:`receive <tdl-step-receive>` or :ref:`btxn <tdl-step-btxn>` steps).

.. index:: HttpMessaging
.. index:: http_headers (HttpMessaging)
.. index:: http_body (HttpMessaging)
.. index:: http_parts (HttpMessaging)
.. index:: http_method (HttpMessaging)
.. index:: http_version (HttpMessaging)
.. index:: http_path (HttpMessaging)
.. index:: http_status (HttpMessaging)
.. index:: network.host (HttpMessaging)
.. index:: network.port (HttpMessaging)
.. index:: http.uri (HttpMessaging)
.. index:: http.uri.extension (HttpMessaging)
.. index:: http.ssl (HttpMessaging)
.. index:: status.code (HttpMessaging)
.. index:: http.method (HttpMessaging)

.. _handlers-httpmessaging:

HttpMessaging
+++++++++++++

.. warning::
  This messaging handler is **deprecated**. Use instead the :ref:`handlers-httpmessagingv2` handler or a
  :ref:`custom messaging handler<handlers-custom-handlers>` if you have complex messaging needs.

Used to send or receive content over HTTP.

.. csv-table::
    :header: "Element name", "Element type", "Required?", "Type", "Description"

    ``http_body``, Input, No, ``binary``, The HTTP request body's bytes.
    ``http_body``, Output, No, ``binary``, The bytes of the received body.
    ``http_headers``, Input, No, ``map``, The ``map`` of HTTP headers to send.
    ``http_headers``, Output, No, ``map``, The ``map`` of received headers.
    ``http_method``, Output, No, ``string``, The HTTP method.
    ``http_parts``, Input, No, ``map``, A ``map`` including the definition of the parts (see description below).
    ``http_parts``, Output, No, ``map``, A ``map`` including the received parts (see description below).
    ``http_path``, Output, No, ``string``, The HTTP request path.
    ``http_status``, Output, No, ``string``, The HTTP status code from the received response.
    ``http_version``, Input, No, ``string``, The HTTP version to consider.
    ``http_version``, Output, No, ``string``, The HTTP version.
    ``http.method``, Send configuration, Yes, ``string``, The HTTP method to use when sending.
    ``http.ssl``, Transaction configuration, No, ``boolean``, Whether or not connections should be over HTTP (default) or HTTPS.
    ``http.uri``, Actor configuration, No, ``string``, The request path for the request.
    ``http.uri``, Send configuration, No, ``string``, The request path URI to send to.
    ``http.uri.extension``, Send configuration, No, ``string``, HTTP URI extension for the address.
    ``network.host``, Actor configuration, Yes, ``string``, The host of the actor.
    ``network.port``, Actor configuration, Yes, ``number``, The listen port for the actor.
    ``status.code``, Receive configuration, No, ``string``, The status code for responses.
    ``status.code``, Send configuration, No, ``string``, Status for responses.

.. code-block:: xml

    <btxn from="Actor1" to="Actor2" txnId="t1" handler="HttpMessaging"/>
    <send id="dataSend" desc="Send data" from="Actor1" to="Actor2" txnId="t1">
        <config name="http.method">"POST"</input>
        <config name="http.uri">"/path/to/service"</input>
        <input name="http_body">$binaryContent</input>
    </send>
    <receive id="dataReceive" desc="Receive data" from="Actor2" to="Actor1" txnId="t1">
        <config name="status.code">"200"</input>
    </receive>
    <etxn txnId="t1"/>

.. note::
    **Isolating communications:** When using a ``HttpMessaging`` handler to receive communication from a SUT, the Test Bed dynamically starts listening on
    a new port for incoming traffic. This port (along with the host) are presented to the Test Bed user upon test initiation so that he/she can configure
    the SUT accordingly. To avoid unwanted communication being received on this port that is unrelated to the test session, the Test Bed will only
    listen to requests originating from the SUT's address, ignoring others originating from other sources. To achieve this, the Test Bed uses the
    ``network.host`` parameter configured for the SUT that needs to be provided by the tester as part of the SUT's configuration before starting a test.

    The value for the ``network.host`` parameter must be set with the **public IP Address** of the SUT endpoint.

Using HTTPS
^^^^^^^^^^^

The ``HttpMessaging`` handler can be used both for HTTP and (one-way) HTTPS connection. The default setting is connection over HTTP. Switching to
HTTPS is done at the level of the handler's enclosing transaction and applies to all subsequent :ref:`tdl-step-send` or :ref:`tdl-step-receive` steps. Enabling HTTPS
is achieved by passing a configuration parameter named "http.ssl" with a value of true or false (case insensitive) as part of the begin transaction
step (step :ref:`tdl-step-btxn`). This must be provided at this point because it is needed when creating the sender and receiver implementation.

The following example illustrates its use:

.. code-block:: xml
    :emphasize-lines: 2

    <btxn from="sender" to="receiver" txnId="t1" handler="HttpMessaging">
        <config name="http.ssl">true</config>
    </btxn>
    <send id="dataSend" desc="Send data" from="sender" to="receiver" txnId="t1">
        <config name="http.method">POST</config>
        <input name="http_body">$content</input>
    </send>

Note that the value "true" in this example could also have been provided as a variable reference (e.g. ``$isHTTPS``) allowing a test case to remain unaffected
if the underlying communication needs to be over HTTP or HTTPS. This could be especially interesting in cases where the ``SoapMessaging`` handler is used to
test SUT endpoints over which the Test Bed has no control over the underlying transport channel. In this case the "http.ssl" parameter could be set as part of
the system's configuration, as in the following example (assuming an endpoint name of "sutInfo" and an endpoint parameter named "isHTTPS"):

.. code-block:: xml
    :emphasize-lines: 2

    <btxn from="sender" to="receiver" txnId="t1" handler="HttpMessaging">
        <config name="http.ssl">$sutInfo{isHTTPS}</config>
    </btxn>

Support for sending and receiving multipart form data
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When **receiving**, a multipart message is detected if the ``ContentType`` header contains a boundary part. The ``http_parts`` output is a ``map`` that contains:

    * ``http_parts{parts}``: A list of all parts in sequence.
    * ``http_parts{parts}{0}{header}``: The part's header as a string.
    * ``http_parts{parts}{0}{content}``: The part's content as a binary.
    * ``http_parts{partsByName}``: A map of parts by name (for easy lookup of named parts):
    * ``http_parts{partsByName}{NAME}{header}``: The part's header as a string.
    * ``http_parts{partsByName}{NAME}{content}``: The part's content as a binary.

When **sending**, if a ``http_body`` input is present this takes precedence. If not, and a ``http_parts`` input is provided, then a multipart request is created. The
``http_parts`` input is a ``list`` of ``maps`` (one map per part). To send a part as a file the ``file_name`` property needs to be passed. Specifically the information
on a part is as follows:

    * ``http_parts{0}{name}``: The name of the part.
    * ``http_parts{0}{content_type}``: The mime type of the part (text/plain for simple text).
    * ``http_parts{0}{file_name}``: The name of the file to set for the part if this is a file/binary p

The following TDL example illustrates how to populate and send a multipart request with three parts (two file parts and one
test part):

.. code-block:: xml

    <imports>
        <artifact name="file1">testSuite1/artifacts/file1.xml</artifact>
        <artifact name="file2">testSuite1/artifacts/file2.zip</artifact>
    </imports>
    <actors>
        ...
    </actors>
    <steps>
        <!--
            Define first file part.
        -->
        <assign to="filePartInfo1{name}" type="string">"file1"</assign>
        <assign to="filePartInfo1{content_type}" type="string">"text/xml"</assign>
        <assign to="filePartInfo1{file_name}" type="string">"file1.xml"</assign>
        <assign to="filePartInfo1{content}" type="binary">$file1</assign>
        <!--
            Define second file part.
        -->
        <assign to="filePartInfo2{name}" type="string">"file2"</assign>
        <assign to="filePartInfo2{content_type}" type="string">"application/zip"</assign>
        <assign to="filePartInfo2{file_name}" type="string">"file2.zip"</assign>
        <assign to="filePartInfo2{content}" type="binary">$file2</assign>
        <!--
            Define a third text part.
        -->
        <assign to="textPartInfo1{name}" type="string">"text1"</assign>
        <assign to="textPartInfo1{content_type}" type="string">"text/plain"</assign>
        <assign to="textPartInfo1{content}" type="string">"A simple text value"</assign>
        <!--
            Put all parts in a list.
        -->
        <assign to="parts" append="true">$filePartInfo1</assign>
        <assign to="parts" append="true">$filePartInfo2</assign>
        <assign to="parts" append="true">$textPartInfo1</assign>
        <!--
            Send the request.
        -->
        <btxn from="Sender" to="Receiver" txnId="t1" handler="HttpMessaging"/>
        <send desc="Send file" from="Sender" to="Receiver" txnId="t1">
            <config name="http.method">POST</config>
            <input name="http_parts">$parts</input>
        </send>
        <etxn txnId="t1"/>
    </steps>

.. index:: HttpsMessaging
.. index:: http_headers (HttpsMessaging)
.. index:: http_body (HttpsMessaging)
.. index:: http_method (HttpsMessaging)
.. index:: http_version (HttpsMessaging)
.. index:: http_path (HttpsMessaging)
.. index:: http_status (HttpsMessaging)
.. index:: network.host (HttpsMessaging)
.. index:: network.port (HttpsMessaging)
.. index:: http.uri (HttpsMessaging)
.. index:: http.uri.extension (HttpsMessaging)
.. index:: http.ssl (HttpsMessaging)
.. index:: status.code (HttpsMessaging)
.. index:: http.method (HttpsMessaging)

.. _handlers-httpsmessaging:

HttpsMessaging
++++++++++++++

.. warning::
  This messaging handler is **deprecated**. Use instead the :ref:`handlers-httpmessagingv2` handler or a
  :ref:`custom messaging handler<handlers-custom-handlers>` if you have complex messaging needs.

Used to send or receive content over HTTPS.

.. csv-table::
    :header: "Element name", "Element type", "Required?", "Type", "Description"

    ``http_body``, Input, No, ``binary``, The HTTP request body's bytes.
    ``http_body``, Output, No, ``binary``, The bytes of the received body.
    ``http_headers``, Input, No, ``map``, The ``map`` of HTTP headers to send.
    ``http_headers``, Output, No, ``map``, The ``map`` of received headers.
    ``http_method``, Output, No, ``string``, The HTTP method.
    ``http_status``, Output, No, ``string``, The HTTP status code from the received response.
    ``http_uri``, Output, No, ``string``, The HTTP request path.
    ``http_version``, Output, No, ``string``, The HTTP version.
    ``http.method``, Send configuration, Yes, ``string``, The HTTP method to use when sending.
    ``http.uri``, Actor configuration, No, ``string``, The request path for the request.
    ``http.uri.extension``, Send configuration, No, ``string``, HTTP URI extension for the address.
    ``network.host``, Actor configuration, Yes, ``string``, The host of the actor.
    ``network.port``, Actor configuration, Yes, ``number``, The listen port for the actor.
    ``status.code``, Receive configuration, No, ``string``, The status code for responses.
    ``status.code``, Send configuration, No, ``string``, Status for responses.

.. code-block:: xml

    <btxn from="Actor1" to="Actor2" txnId="t1" handler="HttpsMessaging"/>
    <send id="dataSend" desc="Send data" from="Actor1" to="Actor2" txnId="t1">
        <config name="http.method">"POST"</input>
        <config name="http.uri.extension">"/path/to/service"</input>
        <input name="http_body">$binaryContent</input>
    </send>
    <receive id="dataReceive" desc="Receive data" from="Actor2" to="Actor1" txnId="t1">
        <config name="status.code">"200"</input>
    </receive>
    <etxn txnId="t1"/>

.. note::
    **Isolating communications:** Handler ``HttpsMessaging`` builds upon the mechanism of :ref:`handlers-httpmessaging` to isolate test
    session communications when receiving data. Check it's documentation on what is needed to achieve this.

.. index:: HttpProxyMessaging
.. index:: request_data (HttpProxyMessaging)
.. index:: http_method (HttpProxyMessaging)
.. index:: http_version (HttpProxyMessaging)
.. index:: http_path (HttpProxyMessaging)
.. index:: network.host (HttpProxyMessaging)
.. index:: network.port (HttpProxyMessaging)
.. index:: proxy.address (HttpProxyMessaging)

.. _handlers-HttpProxyMessaging:

HttpProxyMessaging
++++++++++++++++++

.. warning::
  This messaging handler is **deprecated**. Use instead the :ref:`handlers-httpmessagingv2` handler or a
  :ref:`custom messaging handler<handlers-custom-handlers>` if you have complex messaging needs.

Used to proxy HTTP requests and responses between two actors.

.. csv-table::
    :header: "Element name", "Element type", "Required?", "Type", "Description"
    :delim: ~

    ``http_method``~ Output~ No~ ``string``~ The HTTP method.
    ``http_path``~ Output~ No~ ``string``~ The HTTP request path.
    ``http_version``~ Output~ No~ ``string``~ The HTTP version.
    ``network.host``~ Actor configuration~ Yes~ ``string``~ The host of the actor.
    ``network.port``~ Actor configuration~ Yes~ ``number``~ The listen port for the actor.
    ``proxy.address``~ Send configuration~ No~ ``string``~ Address of the proxied service.
    ``request_data``~ Input~ No~ ``map``~ The ``map`` of data to consider. Contains the ``http_method``, ``http_path``, ``http_body``, ``http_headers`` inputs from the HttpMessaging handler.

In this case the ``request_data`` input ``map`` is defined as a convenience considering that we will always be receiving
a call that we want to proxy to a final destination. The HTTP-related parameters to send to the destination need to match
the initial parameters received.

.. code-block:: xml

    <btxn from="Actor1" to="Actor2" txnId="t1" handler="HttpProxyMessaging"/>
    <receive id="receiveData" desc="Receive call" from="Actor1" to="Actor2" txnId="t1" />
    <send desc="Send call" from="Actor2" to="Actor1" txnId="t1">
        <config name="proxy.address">http://PROXIED_SERVICE_ADDRESS</config>
        <input name="request_data" source="$receiveData" />
    </send>
    <etxn txnId="t1"/>

.. index:: SoapMessaging
.. index:: http_headers (SoapMessaging)
.. index:: soap_message (SoapMessaging)
.. index:: soap_attachments (SoapMessaging)
.. index:: soap_header (SoapMessaging)
.. index:: soap_body (SoapMessaging)
.. index:: soap_message (SoapMessaging)
.. index:: soap_content (SoapMessaging)
.. index:: soap_attachments (SoapMessaging)
.. index:: soap_attachments_size (SoapMessaging)
.. index:: network.host (SoapMessaging)
.. index:: network.port (SoapMessaging)
.. index:: http.uri (SoapMessaging)
.. index:: soap.version (SoapMessaging)
.. index:: soap.encoding (SoapMessaging)
.. index:: http.uri.extension (SoapMessaging)
.. index:: http.ssl (SoapMessaging)
.. index:: http_status (SoapMessaging)

.. _handlers-soapmessaging:

SoapMessaging
+++++++++++++

.. warning::
  This messaging handler is **deprecated**. Use instead the :ref:`handlers-soapmessagingv2` handler or a
  :ref:`custom messaging handler<handlers-custom-handlers>` if you have complex messaging needs.

Used to send or receive payloads via SOAP web service calls.

.. csv-table::
    :header: "Element name", "Element type", "Required?", "Type", "Description"

    ``http_headers``, Input, No, ``map``, A ``map`` of HTTP headers to include.
    ``http_headers``, Output, No, ``map``, The HTTP headers received.
    ``http_status``, Output, No, ``string``, The HTTP status code from the received response.
    ``http.uri``, Actor configuration, No, ``string``, The request path to send the SOAP request to.
    ``http.uri.extension``, Send configuration, No, ``string``, HTTP URI extension for the address.
    ``http.ssl``, Transaction configuration, No, ``boolean``, Whether or not connections should be over HTTP (default) or HTTPS.
    ``network.host``, Actor configuration, Yes, ``string``, The host of the actor.
    ``network.port``, Actor configuration, Yes, ``number``, The listen port for the actor.
    ``soap_attachments``, Input, No, ``map``, A ``map`` of ``binary`` attachments.
    ``soap_attachments``, Output, No, ``map``, A ``map`` of received ``binary`` attachments.
    ``soap_attachments_size``, Output, No, ``number``, The number of attachments received.
    ``soap_body``, Output, Yes, ``object``, The received SOAP body.
    ``soap_message``, Input, Yes, ``object``, The SOAP envelope to send.
    ``soap_header``, Output, Yes, ``object``, The received SOAP header.
    ``soap_message``, Output, Yes, ``object``, The received SOAP envelope.
    ``soap_content``, Output, Yes, ``object``, The XML content of the received SOAP body.
    ``soap.encoding``, Send configuration, No, ``string``, Character set encoding.
    ``soap.version``, Receive configuration, Yes, ``string``, SOAP Version. Can be 1.1 or 1.2.
    ``soap.version``, Send configuration, Yes, ``string``, SOAP Version. Can be 1.1 or 1.2.

.. code-block:: xml

    <btxn from="Actor1" to="Actor2" txnId="t1" handler="SoapMessaging"/>
    <send id="dataSend" desc="Send data" from="Actor1" to="Actor2" txnId="t1">
        <config name="soap.version">1.2</config>
        <input name="soap_message">$soapMessage</input>
    </send>
    <receive id="dataReceive" desc="Receive data" from="Actor2" to="Actor1" txnId="t1">
        <config name="soap.version">1.2</config>
    </receive>
    <etxn txnId="t1"/>

Using HTTPS
^^^^^^^^^^^

The ``SoapMessaging`` handler can be used both over an HTTP and (one-way) HTTPS connection. The default setting is connection over HTTP. Switching to
HTTPS is done at the level of the handler's enclosing transaction and applies to all subsequent :ref:`tdl-step-send` or :ref:`tdl-step-receive` steps. Enabling HTTPS
is achieved by passing a configuration parameter named "http.ssl" with a value of true or false (case insensitive) as part of the begin transaction
step (step :ref:`tdl-step-btxn`). This must be provided at this point because it is needed when creating the sender and receiver implementation.

The following example illustrates its use:

.. code-block:: xml
    :emphasize-lines: 2

    <btxn from="sender" to="receiver" txnId="t1" handler="SoapMessaging">
        <config name="http.ssl">true</config>
    </btxn>
    <send id="dataSend" desc="Send data" from="sender" to="receiver" txnId="t1">
        <config name="soap.version">$soapVersion</config>
        <input name="soap_message">$soapMessage</input>
    </send>

Note that the value "true" in this example could also have been provided as a variable reference (e.g. ``$isHTTPS``) allowing a test case to remain unaffected
if the underlying communication needs to be over HTTP or HTTPS. This could be especially interesting in cases where the ``SoapMessaging`` handler is used to
test SUT endpoints over which the Test Bed has no control over the underlying transport channel. In this case the "http.ssl" parameter could be set as part of
the system's configuration, as in the following example (assuming an endpoint name of "sutInfo" and an endpoint parameter named "isHTTPS"):

.. code-block:: xml
    :emphasize-lines: 2

    <btxn from="sender" to="receiver" txnId="t1" handler="SoapMessaging">
        <config name="http.ssl">$sutInfo{isHTTPS}</config>
    </btxn>

.. note::
    **Isolating communications:** Handler ``HttpsMessaging`` builds upon the mechanism of :ref:`handlers-httpmessaging` to isolate test
    session communications when receiving data. Check it's documentation on what is needed to achieve this.

.. index:: TCPMessaging
.. index:: content (TCPMessaging)
.. index:: network.host (TCPMessaging)
.. index:: network.port (TCPMessaging)

TCPMessaging
++++++++++++

.. warning::
  This messaging handler is **deprecated**. Define instead a :ref:`custom messaging handler<handlers-custom-handlers>` to cover your test case messaging needs.

Used to send or receive an arbitrary byte stream over TCP.

.. csv-table::
    :header: "Element name", "Element type", "Required?", "Type", "Description"

    ``content``, Input, Yes, ``binary``, The stream of bytes to send.
    ``content``, Output, Yes, ``binary``, The stream of bytes received.
    ``network.host``, Actor configuration, Yes, ``string``, The host of the actor.
    ``network.port``, Actor configuration, Yes, ``number``, The listen port for the actor.

.. code-block:: xml

    <btxn from="Actor1" to="Actor2" txnId="t1" handler="TCPMessaging"/>
    <send id="dataSend" desc="Send data" from="Actor1" to="Actor2" txnId="t1">
        <input name="content">$binaryContent</input>
    </send>
    <receive id="dataReceive" desc="Receive data" from="Actor2" to="Actor1" txnId="t1"/>
    <etxn txnId="t1"/>

.. index:: UDPMessaging
.. index:: content (UDPMessaging)
.. index:: network.host (UDPMessaging)
.. index:: network.port (UDPMessaging)
.. _handlers-udpmessaging:

UDPMessaging
++++++++++++

.. warning::

  This messaging handler is **deprecated**. Define instead a :ref:`custom messaging handler<handlers-custom-handlers>` to cover your test case messaging needs.

Used to send or receive arbitrary bytes over UDP.

.. csv-table::
    :header: "Element name", "Element type", "Required?", "Type", "Description"

    ``content``, Input, Yes, ``binary``, The stream of bytes to send.
    ``content``, Output, Yes, ``binary``, The stream of bytes received.
    ``network.host``, Actor configuration, Yes, ``string``, The host of the actor.
    ``network.port``, Actor configuration, Yes, ``number``, The listen port for the actor.

.. code-block:: xml

    <btxn from="Actor1" to="Actor2" txnId="t1" handler="UDPMessaging"/>
    <send id="dataSend" desc="Send data" from="Actor1" to="Actor2" txnId="t1">
        <input name="content">$binaryContent</input>
    </send>
    <receive id="dataReceive" desc="Receive data" from="Actor2" to="Actor1" txnId="t1"/>
    <etxn txnId="t1"/>