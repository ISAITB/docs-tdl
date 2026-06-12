System parameters relate to a system and apply to all test cases defined in its linked conformance statements. An example case
of a system parameter would be one where an organisation's systems are mapped to distinct IT services that are expected to be individually tested.
Each such service may have an API endpoint address that would be used within all test cases to make API calls. Defining such a
configuration property at this level avoids repeating and redefining it in every linked conformance statement.

System parameters are made accessible in test cases through a ``map`` named **SYSTEM**. This contains key-value pairs for
each configured parameter that can be used in expressions in exactly the same way other variables and configuration entries are used.

Apart from custom properties, the **SYSTEM** map also contains certain predefined values, specifically:

    * ``shortName``: The ``string`` value of the system's name (short form).
    * ``fullName``: The ``string`` value of the system's name (full form).
    * ``version``: The ``string`` value of the system's version.
    * ``apiKey``: The ``string`` value of the system's unique API key.

The following example illustrates use of this map to pass a API endpoint address (key ``endpointAddress``), specific to the system, 
as part of a messaging step (see :ref:`handlers` and :ref:`tdl-step-send`). This can then be used by the messaging service to define the destination of its outgoing calls:

.. code-block:: xml
    :emphasize-lines: 4

    <!-- Messaging service URL configured at domain level as "handlerURL". -->
    <send id="sendStep" desc="Send message" handler="$DOMAIN{handlerURL}">
        <!-- Endpoint address retrieved from system-level as "endpointAddress". -->
        <input name="destination">$SYSTEM{endpointAddress}</input>
        <input name="content">$theContentToSend</input>
    </send>

.. note::
    **GITB software support:** Use of the **SYSTEM** map is specific to the GITB software. If running on different 
    software it would need to be defined and populated within the test case itself.
