Similar to :ref:`tdl-messaging-steps`, processing occurs in the context of a transaction that acts as a grouping mechanism
over related operations. The ``bptxn`` step (the name stands for "Begin processing transaction") is the construct used to
signal that a processing transaction should be considered as started and is assigned an identifier. Subsequent relevant 
operations will be accompanied by this transaction ID to allow their processing handler to carry them out accordingly.

Use of a processing transaction is not always required. For processing steps that are simple in nature and don't require
state to be maintained across calls, you may skip the definition of a transaction and simply refer to the processing handler
from the ``process`` step itself (see :ref:`tdl-step-process` for details). Whether or not skipping a transaction's definition is 
supported depends on the specific processing handler; typically however, even if a processing handler doesn't require a transaction
and is signalled to create one this will simply be ignored. In terms of whether you need or not to define a processing transaction 
you can consider this rule of thumb:

* **Transaction needed:** When the processing handler is expected to maintain state across individual ``process`` calls and eventually 
  perform some clean up operations.
* **Transaction not needed:** When the processing handler is stateless.

The structure of the ``bptxn`` element (defined when a processing transaction is needed) is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @handler, yes, A string value or variable reference identifying the the processing handler for the transaction (see :ref:`handlers-implementation`).
    @handlerTimeout, no, A number or variable reference with the maximum time (in milliseconds) to wait for the handler service call to complete (in case of an external test service being used as a handler). See also :ref:`tdl-steps-common-handlerTimeouts`.
    @skipped, no, A boolean value or variable reference (default being "false") which will result in the step being skipped if "true". See also :ref:`tdl-steps-common-skipped`.
    @stopOnError, no, A boolean flag determining whether the test session should end if this step fails (default is "false"). See also :ref:`tdl-steps-common-stoponerror`.
    @txnid, yes, A string identifier for the transaction.
    config, no, Zero or more elements to provide configuration when creating the transaction. Each ``config`` element has a ``name`` attribute and a text content or variable reference as value.
    property, no, Zero or more elements to provide configuration regarding the setup of the processing handler call that are not passed to the handler. Each ``property`` element has a ``name`` attribute and a text content or variable reference as value.

The ``bptxn`` step results in a call to the configured processing handler to signal that a new transaction is going to 
start.

.. code-block:: xml
    :emphasize-lines: 1

    <bptxn txnId="t1" handler="https://PROCESSING_SERVICE?wsdl"/>
    <process id="result" txnId="t1">
        <operation>action</operation>
        <input name="anInput">$aValue</input>
    </process>
    <eptxn txnId="t1"/>
