The ``eptxn`` step (the name stands for "End processing transaction") is the counterpath of the ``bptxn`` step and is used to
close a transaction the ID of which it references. The structure of the ``eptxn`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @txnid, yes, A string identifier for the processing transaction to end.
    @skipped, no, A boolean value or variable reference (default being "false") which will result in the step being skipped if "true". See also :ref:`tdl-steps-common-skipped`.
    @stopOnError, no, A boolean flag determining whether the test session should end if this step fails (default is "false"). See also :ref:`tdl-steps-common-stoponerror`.

The ``eptxn`` step results in a call to the transaction's processing handler to signal that it should consider the transaction as
completed and proceed with any needed actions such as resource clean-up.

.. code-block:: xml
    :emphasize-lines: 6

    <bptxn txnId="t1" handler="https://PROCESSING_SERVICE?wsdl"/>
    <process id="result" txnId="t1">
        <operation>action</operation>
        <input name="anInput">$aValue</input>
    </process>
    <eptxn txnId="t1"/>
