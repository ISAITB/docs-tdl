Used for common operations on variables. This processing handler supports but does not require a processing
transaction to be established. The following operations are supported:

.. csv-table::
    :header: "Operation", "Description", "Input(s)", "Output(s)"

    ``exists``, Check to see if a variable with a given name is defined in the session context., Yes, A ``boolean`` named ``output`` in the resulting step's ``map``.
    ``type``, Return the type of a variable with a given name., Yes, A ``string`` named ``output`` in the resulting step's ``map``. This will be empty if no variable was found for the provided name.

Besides using as part of your testing logic, the ``VariableUtils`` operations can also be quite useful while developing tests to debug
problems related to variables.

.. _handlers-VariableUtils_exists:

VariableUtils - exists
^^^^^^^^^^^^^^^^^^^^^^

The ``exists`` operation is used to check whether a variable exists in the current scope. An example of when this would be
interesting is to check within a :ref:`scriptlet <scriptlets>` whether an :ref:`optional parameter <scriptlets_elements_params>`
has been set. The expected input parameter is as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``name`` | Yes | The name of the variable to check.

The following example illustrates how the ``exists`` operation could be used:

.. code-block:: xml

    <!-- Create a variable named "aVariable". -->
    <assign to="aVariable">"A value"</assign>
    <!-- Check to see if it exists. -->
    <process handler="VariableUtils" operation="exists" input="aVariable" output="variableExists"/>
    <!-- Prints "true". -->
    <log>$variableExists</log>

    <!-- Check to see if a map is defined (e.g. within a scriptlet) and do something with it. -->
    <process handler="VariableUtils" operation="exists" input="optionalMap" output="mapExists"/>
    <if>
        <cond>$mapExists</cond>
        <then>
            <!-- Do something with the map. -->
        </then>
    </if>

.. _handlers-VariableUtils_type:

VariableUtils - type
^^^^^^^^^^^^^^^^^^^^

The ``type`` operation on the other hand could be used in situations when you are not certain of a variable's type. This could
be the case for an element in a ``map`` that can take values of varying types. The expected input parameter is as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``name`` | Yes | The name of the variable to check.

The following example illustrates how the ``type`` operation could be used:

.. code-block:: xml

    <!-- Create a variable named "aVariable". -->
    <assign to="aVariable">"A value"</assign>
    <!-- Check the variable's type. -->
    <process handler="VariableUtils" operation="type" input="aVariable" output="variableType"/>
    <!-- Prints "string". -->
    <log>$variableType</log>