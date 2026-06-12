Used to pause a test session for a given duration. The following operation is supported:

.. csv-table::
    :header: "Operation", "Description", "Input(s)", "Output(s)"
    :delim: |

    ``delay`` | Delay the test session for a given duration. | Yes | No.

The input parameters expected by the ``delay`` operation are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``duration`` | Yes | A ``number`` representing the duration (expressed in milliseconds) to delay for.

The following examples illustrate how the ``DelayProcessor`` can be used to pause the test session.

.. code-block:: xml

    <!-- Wait for a fixed 5 seconds before proceeding. -->
    <process handler="DelayProcessor" operation="delay" input="5000"/>

    <!-- Wait for a random duration between 5 and 10 seconds. -->
    <process handler="TokenGenerator" operation="random" output="randomDelay">
        <input name="minimum">5000</input>
        <input name="maximum">10000</input>
        <input name="integer">true()</input>
    </process>
    <process handler="DelayProcessor" operation="delay" input="$randomDelay"/>