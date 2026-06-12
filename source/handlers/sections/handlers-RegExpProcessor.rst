Used to process texts using regular expressions, to verify whether they match a specific pattern or to extract data. This processing handler
does not require a processing transaction to be established. The following operations are supported:

.. csv-table::
    :header: "Operation", "Description", "Input(s)", "Output(s)"
    :delim: |

    ``check`` | Check to see if a ``string`` matches an expression. | Yes | Yes, a ``boolean`` named ``output`` in the resulting step's ``map``.
    ``collect`` | Use an expression to collect data from a provided ``string`` based on the expression's capturing groups. | Yes | A ``list`` of ``string`` values, one value per matched group.

Regular expressions offer a very powerful means of describing a text's content and extracting from it certain parts for further processing. They can be used
against any text content, offering a counterpart to the use of XPath in the :ref:`assign<tdl-step-assign>` step that is best adapted, but also limited, to XML structures.
The regular expressions are expected to be provided using the `syntax used by the Java language <https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/regex/Pattern.html>`__.

.. _handlers-RegExpProcessor_check:

RegExpProcessor - check
^^^^^^^^^^^^^^^^^^^^^^^

The ``check`` operation can be used to verify whether a given text matches a specific pattern. This may at first appear similar to the
:ref:`RegExpValidator<handlers-RegExpValidator>`, however there is a subtle difference: using the :ref:`RegExpValidator<handlers-RegExpValidator>`
constitutes an assertion made by the test case which, if failed, would likely mean that the test session itself is considered failed. The ``check``
operation doesn't presume anything for the test session's status, but is rather used as an internal check to e.g. determine
whether an optional set of steps should be followed. The input parameters expected by this operation are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``expression`` | Yes | A ``string`` with the expression that will be used to check the ``input``.
    ``input`` | Yes | The ``string`` to check.

The following example illustrates the ``check`` operation's use:

.. code-block:: xml

    <!-- Check if a given text includes "test" in a case-insensitive manner -->
    <process output="check" handler="RegExpProcessor" operation="check">
        <input name="input">$someTextData</input>
        <!-- Flags are passed in embedded format (e.g. case insensitive match). -->
        <input name="expression">"(?i)test"</input>
    </process>
    <if desc="Optional steps">
        <cond>$check</cond>
        <then>
            ...
        </then>
    </if>

.. _handlers-RegExpProcessor_collect:

RegExpProcessor - collect
^^^^^^^^^^^^^^^^^^^^^^^^^

The ``collect`` operation is used to process a provided text using an expression that defines one or more capturing groups. This
operation can be particularly powerful as it can collect data from both structured and unstructured data. Each matching group
is appended to a ``list`` of ``string`` elements in the sequence with which it was matched, otherwise resulting in an empty ``list``
if no matches were made. The input parameters expected by this operation are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``expression`` | Yes | A ``string`` with the expression to collect data with. The provided expression must define at least one capturing group.
    ``input`` | Yes | The ``string`` to process to collect data.

The following example illustrates the ``collect`` operation's use:

.. code-block:: xml

    <!-- Define a firstname and lastname in an unstructured text block -->
    <assign to="aText">"My firstname is 'John' and my lastname is 'Doe'."</assign>
    <!-- Collect the data using an expression with two capturing groups -->
    <process output="personData" handler="RegExpProcessor" operation="collect">
        <input name="input">$aText</input>
        <input name="expression">".+ firstname is '([\w]+)' .+ lastname is '([\w]+)'"</input>
    </process>
    <!-- Prints "John" -->
    <log>$personData{0}</log>
    <!-- Prints "Doe" -->
    <log>$personData{1}</log>