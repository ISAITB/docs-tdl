The ``etxn`` step stands for "End transaction" and acts as the counterpart to a ``btxn`` element by referencing its transaction
ID. It is structured as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @skipped, no, A boolean value or variable reference (default being "false") which will result in the step being skipped if "true". See also :ref:`tdl-steps-common-skipped`.
    @stopOnError, no, A boolean flag determining whether the test session should end if this step fails (default is "false"). See also :ref:`tdl-steps-common-stoponerror`.
    @txnid, yes, The identifier of the transaction to end.

Executing the ``etxn`` results in a call to the transaction's messaging handler to take necessary actions such as resource clean-up.

.. code-block:: xml
    :emphasize-lines: 9

    <btxn from="Actor1" to="Actor2" txnId="t1" handler="HttpMessagingV2"/>
    <send id="dataSend" desc="Send data" from="Actor1" to="Actor2" txnId="t1">
        <input name="uri">"https://my.sut.org/api/get"</input>
        <input name="method">"GET"</input>
    </send>
    <receive id="dataReceive" desc="Receive data" from="Actor2" to="Actor1" txnId="t1">
        <input name="method">"GET"</input>
    </receive>
    <etxn txnId="t1"/>

Note that ``etxn`` steps are not presented to the user.
