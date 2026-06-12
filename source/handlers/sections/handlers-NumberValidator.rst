Used to verify that a provided ``number`` matches an expected value.

.. csv-table::
    :header: "Input name", "Required?", "Type", "Description"

    ``actual``, Yes, ``number``, The value to check.
    ``expected``, Yes, ``number``, The expected value.
    ``failureMessage``, No, ``string``, An optional message to display in case the check fails.
    ``successMessage``, No, ``string``, An optional message to display in case the check succeeds.

.. code-block:: xml

    <verify handler="NumberValidator" desc="Check number">
        <input name="actual">$aNumber</input>
        <input name="expected">'10'</input>
        <input name="successMessage">'The provided value is correct.'</input>
        <input name="failureMessage">'The provided value does not match the requirements.'</input>
    </verify>