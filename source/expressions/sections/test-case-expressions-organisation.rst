Organisation parameters relate to an organisation as a whole and apply to all its systems and their configured conformance statements.
An example case of using such a parameter would be to define a Member State ISO country code in a community where its participating
organisations are the individual EU Member States. To avoid specifying this in each test case an organisation-level configuration
parameter could be defined to provide this only once.

Organisation parameters are made accessible in test cases through a ``map`` named **ORGANISATION**. This contains key-value pairs for
each configured parameter that can be used in expressions in exactly the same way other variables and configuration entries are used.

Apart from custom properties, the **ORGANISATION** map also contains certain predefined values, specifically:

    * ``shortName``: The ``string`` value of the organisation's name (short form).
    * ``fullName``: The ``string`` value of the organisation's name (full form).

The following example illustrates use of this map to pass a Member State country code (key ``msCode``) to a validator to apply
country-specific validation rules:

.. code-block:: xml
    :emphasize-lines: 4

    <!-- Validation service URL configured at domain level as "handlerURL". -->
    <verify desc="Validate content" handler="$DOMAIN{handlerURL}">
        <!-- Country code retrieved from organisation-level as "msCode". -->
        <input name="country">$ORGANISATION{msCode}</input>
        <input name="content">$theContentToValidate</input>
    </verify>

.. note::
    **GITB software support:** Use of the **ORGANISATION** map is specific to the GITB software. If running on different 
    software it would need to be defined and populated within the test case itself.
