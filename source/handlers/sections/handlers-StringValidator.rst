Used to verify that a provided ``string`` matches an expected value.

.. csv-table::
    :header: "Input name", "Required?", "Type", "Description"

    ``actual``, Yes, ``string``, The value to check.
    ``expected``, Yes, ``string``, The expected value.
    ``failureMessage``, No, ``string``, An optional message to display in case the check fails.
    ``successMessage``, No, ``string``, An optional message to display in case the check succeeds.

.. code-block:: xml

    <verify handler="StringValidator" desc="Check string">
        <input name="actual">$aString</input>
        <input name="expected">'expected_string'</input>
        <input name="successMessage">'The provided value is correct.'</input>
        <input name="failureMessage">'The provided value does not match the requirements.'</input>
    </verify>