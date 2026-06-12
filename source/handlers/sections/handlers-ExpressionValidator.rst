Used to verify whether a provided :ref:`expression<test-case-expressions>` evaluates to "true". The
``ExpressionValidator`` is the most generic validation handler as it can be used to check any arbitrary
condition.

.. csv-table::
    :header: "Input name", "Required?", "Type", "Description"

    ``expression``, Yes, :ref:`Expression<test-case-expressions>`, The expression to evaluate.
    ``failureMessage``, No, ``string``, An optional message to display in case the check fails.
    ``successMessage``, No, ``string``, An optional message to display in case the check succeeds.

.. code-block:: xml

    <verify handler="ExpressionValidator" desc="Validate UUID">
        <input name="expression">$variable != "unwantedValue"</input>
        <input name="successMessage">'The provided UUID is correct.'</input>
        <input name="failureMessage">'The provided UUID does not match the requirements.'</input>
    </verify>

.. note::
    The ``expression`` input is not presented in the :ref:`verify step's<tdl-step-verify>` validation report as it
    would only ever display a "true" or "false".