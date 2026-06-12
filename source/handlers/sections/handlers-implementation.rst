Handlers are defined in the following steps:

* :ref:`tdl-step-btxn`: When beginning a messaging transaction.
* :ref:`tdl-step-send`: When sending a message outside of a messaging transaction.
* :ref:`tdl-step-receive`: When receiving a message outside of a messaging transaction.
* :ref:`tdl-step-listen`: When proxying a call to another service outside of a messaging transaction.
* :ref:`tdl-step-bptxn`: When beginning a processing transaction.
* :ref:`tdl-step-process`: When making a processing operation outside of a processing transaction.
* :ref:`tdl-step-verify`: When validating content.
* :ref:`tdl-step-interact`: When triggering an alternative to a UI-based user interaction.

The element corresponding to each of these steps defines a ``handler`` attribute to identify the handler implementation.
In case an built-in handler is to be used the value specified here is the name of the handler (see :ref:`handlers-predefined-handlers`). Using an external
handler implementation is achieved by specifying as the ``handler`` value the address where the service's WSDL file is
located. The Test Bed will automatically detect in this case that the handler is external and will internally replace local method
invocations with web service calls.

The value provided for the ``handler`` attribute can also be provided with a pure variable reference (see :ref:`test-case-referring-to-variables`)
allowing the actual value to be determined from configuration or even dynamically based on the test session context. In such a case the variable
reference is first evaluated to a ``string`` that is then considered to determine whether the handler is a remote or built-in one.

The following example shows three validation steps taking place, the first one using the built-in :ref:`handlers-XmlValidator`, the second one using
an external validation service, and the third one using an external validation service whose address is configurable:

.. code-block:: xml

    <!--
        Call a local, built-in validation handler called "XmlValidator"
    -->
    <verify handler="XmlValidator" desc="Validate content local">
        <input name="xml">$docToValidate</input>
        <input name="xsd">$schemaFile</input>
    </verify>
    <!--
        Call a remote validation service handler
    -->
    <verify handler="https://serviceaddress?wsdl" desc="Validate content remote">
        <input name="xml">$docToValidate</input>
        <input name="schema">$schemaFile</input>
    </verify>
    <!--
        Call a remote validation service handler (address in configuration)
    -->
    <verify handler="$DOMAIN{validationHandlerAddress}" desc="Validate content remote">
        <input name="xml">$docToValidate</input>
        <input name="schema">$schemaFile</input>
    </verify>
