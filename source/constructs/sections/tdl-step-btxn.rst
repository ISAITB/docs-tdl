The ``btxn`` step stands for "Begin transaction". Its purpose is to define a scope around a set of messaging
steps that have a logical relation to each other. This scope remains active until a ``etxn`` element is
encountered to end it. Apart from wrapping together related messaging steps, the key purpose of the ``btxn`` step
is to declare the :ref:`messaging handler<handlers>` that is used and the involved actors.

Use of a messaging transaction is not necessary as you can also define the handler and involved actors on the individual
messaging steps. It could nonetheless still be interesting to use transactions to avoid repeating the handler's definition.

The structure of the ``btxn`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @from, no, The ID of the actor that acts as the messaging source (see :ref:`test-case-actors`). Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.
    @handler, yes, A string value or variable reference identifying the messaging handler to use for the transaction (see :ref:`handlers-implementation`).
    @handlerTimeout, no, A number or variable reference with the maximum time (in milliseconds) to wait for the handler service call to complete (in case of an external test service being used as a handler). See also :ref:`tdl-steps-common-handlerTimeouts`.
    @skipped, no, A boolean value or variable reference (default being "false") which will result in the step being skipped if "true". See also :ref:`tdl-steps-common-skipped`.
    @stopOnError, no, A boolean flag determining whether the test session should end if this step fails (default is "false"). See also :ref:`tdl-steps-common-stoponerror`.
    @to, no, The ID of the actor that acts as the messaging target (see :ref:`test-case-actors`). Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.
    @txnid, yes, A string ID for the transaction.
    config, no, Zero or more elements to provide configuration when creating the transaction. Each ``config`` element has a ``name`` attribute and a text content or variable reference as value.
    property, no, Zero or more elements to provide configuration regarding the setup of the messaging handler call that are not passed to the handler. Each ``property`` element has a ``name`` attribute and a text content or variable reference as value.

Executing the ``btxn`` step results in a call to the messaging handler specified by the ``handler`` attribute. This gives it an 
opportunity to take any actions needed for the upcoming transaction and apply specific configurations for its related ``send``
and ``receive`` calls.

.. code-block:: xml
    :emphasize-lines: 1

    <btxn txnId="t1" handler="HttpMessagingV2"/>
    <send id="dataSend" desc="Send data" txnId="t1">
        <input name="uri">"https://my.sut.org/api/get"</input>
        <input name="method">"GET"</input>
    </send>
    <receive id="dataReceive" desc="Receive data" txnId="t1">
        <input name="method">"GET"</input>
    </receive>
    <etxn txnId="t1"/>

Note that ``btxn`` steps are not presented to the user.
