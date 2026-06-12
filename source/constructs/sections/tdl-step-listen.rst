The ``listen`` step is used to instruct the Test Bed to act as a proxy between messages sent to and from two actors defined as SUTs. 
Similar to the ``send`` and ``receive`` steps, this step is expected to take place within a transaction created by ``btxn``, the 
identifier of which it references. The structure of the ``listen`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"
    :delim: |

    @from | no | The ID of the actor (defaulting to the SUT actor) that will be sending the message (see :ref:`test-case-actors`). Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.
    @handler | no | The :ref:`messaging handler<handlers>` to use for this messaging step. If not specified (for transactional messaging) the ``txnid`` attribute is required.
    @hidden | no | A boolean flag determining whether or not the step is displayed to users (default is "false"). Note that within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`. See :ref:`tdl-steps-common-hidesteps` for further details.
    @id | no | The ID for the step. This is also the name of a ``map`` variable in the session context in which output will be stored.
    @invert | no | A boolean flag determining whether the step's result should be inverted (default is "false"). Setting to "true" will expect a communication failure to complete the step as a success.
    @level | no | The severity level to be considered when this step fails. Can be set to ``ERROR`` (the default) or ``WARNING``, or be defined dynamically via :ref:`variable reference<test-case-referring-to-variables>`. See :ref:`tdl-steps-common-level` for further details.
    @reply | no | A boolean flag indicating that this communication should be presented as a reply (default is "false"). Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.
    @skipped | no | A boolean value or variable reference (default being "false") which will result in the step being skipped if "true". See also :ref:`tdl-steps-common-skipped`.
    @stopOnError | no, A boolean flag determining whether the test session should end if this step fails (default is "false"). See also :ref:`tdl-steps-common-stoponerror`.
    @to | no | The ID of the actor (defaulting to the non-SUT actor if one is defined) that will be receiving the message (see :ref:`test-case-actors`). Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.
    @txnid | no | The ID of the transaction this ``listen`` belongs to. If not specified (for non-transactional messaging) the ``handler`` attribute is required.
    config | no | Zero or more elements containing configuration values pertinent to the message exchange.  Each ``config`` element has a ``name`` attribute and a text content or variable reference as value.
    documentation | no | Rich text content that provides further information on the current step.
    input | no | Zero or more elements for for the messaging handler to consider. See :ref:`handlers-inputs-outputs` for details.
    output | no | Zero or more elements for the output values reported back to the test case. See :ref:`handlers-inputs-outputs` for details.
    property | no | Zero or more elements to provide configuration regarding the setup of the messaging handler call that are not passed to the handler. Each ``property`` element has a ``name`` attribute and a text content or variable reference as value.

.. note::
    **GITB software support:** The ``listen`` step is currently not supported. As a general note, 
    interoperability tests involving multiple actors as SUTs are not currently possible.
