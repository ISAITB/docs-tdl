Custom service handlers are meaningful when you have project-specific testing needs that cannot be addressed by the test engine's
:ref:`built-in capabilities<handlers-predefined-handlers>` or the :ref:`existing reusable services<handlers-reusable-handlers>` offered by the Test Bed.
In practice any non-trivial test setup would usually require at least a **custom messaging service implementation** to handle the messaging protocol foreseen
by the project. This holds true even if a seemingly suitable built-in messaging handler is available, as you will most likely need to add customisations
when making or receiving calls, but also adapt the reports produced by your messaging steps.

Custom service handler implementations would be defined in a **custom web application** that complements your test suites. This application would include
implementations (as needed) of the **GITB test service SOAP APIs** that allow it to be orchestrated by the Test Bed. To guide you in the implementation of
these APIs you can refer to the `GITB test services documentation <https://www.itb.ec.europa.eu/docs/services/latest/index.html>`_ for:

* `Validation services <https://www.itb.ec.europa.eu/docs/services/latest/validation/index.html>`_, to validate content.
* `Messaging services <https://www.itb.ec.europa.eu/docs/services/latest/messaging/index.html>`_, to send and receive messages.
* `Processing services <https://www.itb.ec.europa.eu/docs/services/latest/processing/index.html>`_, to implement supporting utility functions.

The starting point for the implementation is the Test Bed's `published template service <https://www.itb.ec.europa.eu/docs/services/latest/templates/index.html>`_.
This is an **executable** template, allowing you to create new services based on existing demo starting implementations. Although simple, the pre-existing
implementations fully cover the GITB service APIs and allow you to replace them with your own logic. Moreover, the documentation also includes a
`sample test case <https://www.itb.ec.europa.eu/docs/services/latest/templates/index.html#example-test-case>`_ that illustrates how the demo service
implementations can be used in test steps. For a guided, **step-by-step tutorial** on how to develop custom test services you can also check out the
`complex test development guide <https://www.itb.ec.europa.eu/docs/guides/latest/developingComplexTests/index.html>`_.

.. index:: Handler authentication
.. index:: HTTP Basic
.. index:: UsernameToken
.. index:: WS-Security
.. index:: auth.basic.username
.. index:: auth.basic.password
.. index:: auth.token.username
.. index:: auth.token.password
.. index:: auth.token.password.type
.. _handlers-authentication:

Authentication for external handlers
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Handlers defined as external service implementations may need to be protected with access control. To support such protected services,
the GITB software foresees the possibility to authenticate as part of each service call.

The authentication possibilities currently supported are:

* **Basic HTTP authentication** for all calls to the service's HTTP/HTTPS endpoint. This is authentication at the transport layer.
* Authentication using the **WS-Security UsernameToken profile** (see `here <https://www.oasis-open.org/committees/download.php/13392/wss-v1.1-spec-pr-UsernameTokenProfile-01.htm>`__), supporting text and digest password transmission with timestamps and nonces. This is authentication at the SOAP application layer.

Note that use of HTTP basic authentication and the UsernameToken are not necessarily exclusive. A case where both are provided would be
where a service protects access to its WSDL using HTTP basic authentication and adds additional protection for SOAP service calls by means
of a UsernameToken. Combining both approaches is rare but possible.

As of release 1.28.0, you can configure authentication for test services from the Test Bed's UI or REST API, by registering your
**test services** under the relevant **domain**. Settings configured as such, will be automatically propagated to all test sessions
and apply whenever a given test service is used (matched by means of the key used to identify the service as a
:ref:`domain configuration parameter <test-case-expressions-domain>`).

As an alternative to this automatic configuration, you can still define authentication information within your test cases.
Authentication configuration is defined through ``property`` elements that are used as part of the handler setup in:

* The :ref:`tdl-step-btxn` step for transactional messaging services.
* The :ref:`tdl-step-send`, :ref:`tdl-step-receive` and :ref:`tdl-step-listen` step for non-transactional messaging services.
* The :ref:`tdl-step-bptxn` step for transactional processing services.
* The :ref:`tdl-step-process` step for non-transactional processing services.
* The :ref:`tdl-step-verify` step for validation services.

The properties that are supported in the ``property`` elements are listed in the following table:

.. csv-table::
    :header: "Property name", "Value", "Description"

    ``auth.basic.password``, Any ``string``, The password to provide when prompted for basic HTTP authentication.
    ``auth.basic.username``, Any ``string``, The username to provide when prompted for basic HTTP authentication.
    ``auth.token.password``, Any ``string``, The password to include in the SOAP header as the UsernameToken's password.
    ``auth.token.password.type``, 'DIGEST' (the default) or 'TEXT', The way the password is to be serialised in the header. 'DIGEST' includes it as a DIGEST whereas 'TEXT' adds it in plaintext.
    ``auth.token.username``, Any ``string``, The username to include in the SOAP header as the UsernameToken's username.

.. note::
  Authentication settings for test service handlers defined within test cases, will override any settings defined in the Test Bed
  through the UI or REST API.

The following example illustrates using authentication properties when calling test services:

.. code-block:: xml

    <!--
        Transactional messaging service authentication with UsernameToken (DIGEST).
    -->
    <btxn from="Sender" to="Receiver1" txnId="t1" handler="$DOMAIN{messagingServiceURL}">
        <property name="auth.token.username">$DOMAIN{serviceUsername1}</property>
        <property name="auth.token.password">$DOMAIN{servicePassword1}</property>
        <property name="auth.token.password.type">DIGEST</property>
    </btxn>
    <send id="dataSend" desc="Send message" from="Sender" to="Receiver1" txnId="t1"/>
    <etxn txnId="t1"/>
    <!--
        Validation service authentication with UsernameToken (DIGEST - the default) and HTTP basic authentication.
    -->
    <verify handler="$DOMAIN{validationService1}" desc="Validate content">
        <property name="auth.basic.username">$DOMAIN{serviceUsername2}</property>
        <property name="auth.basic.password">$DOMAIN{servicePassword2}</property>
        <property name="auth.token.username">$DOMAIN{serviceUsername3}</property>
        <property name="auth.token.password">$DOMAIN{servicePassword3}</property>
        <input name="content">$contentToValidate</input>
    </verify>
    <!--
        Transactional processing service authentication with HTTP basic authentication.
    -->
    <bptxn txnId="t1" handler="$DOMAIN{processingServiceURL}">
        <property name="auth.basic.username">$DOMAIN{serviceUsername4}</property>
        <property name="auth.basic.password">$DOMAIN{servicePassword4}</property>
    </bptxn>
    <process id="result" txnId="t1">
        <operation>action</operation>
        <input name="anInput">$aValue</input>
    </process>
    <eptxn txnId="t1"/>
    <!--
        Non-transactional processing service authentication with HTTP basic authentication.
    -->
    <process id="result" handler="$DOMAIN{otherProcessingServiceURL}">
        <property name="auth.basic.username">$DOMAIN{serviceUsername5}</property>
        <property name="auth.basic.password">$DOMAIN{servicePassword5}</property>
        <operation>action</operation>
        <input name="anInput">$aValue</input>
    </process>
    <!--
        Validation service authentication with UsernameToken (TEXT) authentication.
    -->
    <verify handler="$DOMAIN{validationService2}" desc="Validate content">
        <property name="auth.token.username">$DOMAIN{serviceUsername6}</property>
        <property name="auth.token.password">$DOMAIN{servicePassword6}</property>
        <property name="auth.token.password.type">TEXT</property>
        <input name="content">$contentToValidate</input>
    </verify>
    <!--
        Non-transactional messaging service with UsernameToken (TEXT) authentication.
    -->
    <send id="dataSend" desc="Send message" from="Sender" to="Receiver" handler="$DOMAIN{messagingServiceURL}">
        <property name="auth.token.username">$DOMAIN{serviceUsername7}</property>
        <property name="auth.token.password">$DOMAIN{servicePassword7}</property>
        <property name="auth.token.password.type">TEXT</property>
        <input name="message">$messageToSend</input>
    </send>
    <etxn txnId="t1"/>