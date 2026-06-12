Used to verify that a provided ``string`` matches a regular expression.

.. csv-table::
    :header: "Input name", "Required?", "Type", "Description"

    ``expression``, Yes, ``string``, The expression to match.
    ``failureMessage``, No, ``string``, An optional message to display in case the check fails.
    ``input``, Yes, ``string``, The value to check.
    ``successMessage``, No, ``string``, An optional message to display in case the check succeeds.

.. code-block:: xml

    <verify handler="RegExpValidator" desc="Check string">
        <input name="input">$aString</input>
        <input name="expression">'^REF\-\d+$'</input>
        <input name="successMessage">'The provided value is correct.'</input>
        <input name="failureMessage">'The provided value does not match the requirements.'</input>
    </verify>

The regular expression provided for the ``expression`` input is expected to be provided using the `syntax used by the Java language <https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/regex/Pattern.html>`__.
This syntax also supports expression flags provided in an embedded manner, within an expression.

.. code-block:: xml

    <verify handler="RegExpValidator" desc="Check string">
        <input name="input">$aString</input>
        <!-- Same expression but executed in a case insensitive (?i) and multiline (?m) manner. -->
        <input name="expression">'(?im)^REF\-\d+$'</input>
        <input name="successMessage">'The provided value is correct.'</input>
        <input name="failureMessage">'The provided value does not match the requirements.'</input>
    </verify>