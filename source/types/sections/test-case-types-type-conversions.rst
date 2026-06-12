It is often the case when executing a test case that we need to convert a session context variable of one type to another. An an example
consider a ``string`` variable containing XML content that we want to validate using a validator that only expects ``binary`` or ``object``
input. Conversions between types can be done explicitly using the ``assign`` step and the ``type`` attribute:

.. code-block:: xml

    <assign to="toVariable" source="$fromVariable" type="object"/>

In this example, the ``fromVariable`` is converted to the ``toVariable`` as an ``object``. Explicit conversions like this can however pollute
the testing logic and can in most cases happen implicitly. An implicit conversion takes place when we attempt to use a source variable of a
given type as a variable of another type. The supported conversions and specific conversion assumptions between types are provided in the
following table:

.. csv-table::
    :stub-columns: 1
    :header: "", ``string``, ``number``, ``boolean``, ``binary``, ``object``, ``schema``, ``map``, ``list``

    ``binary``, Yes - result is the string representation of the bytes, No, No, Yes, Yes - result is the XML representation of the bytes, Yes - same as with ``object``, No,  Yes - a single item list containing the value
    ``boolean``, Yes - result as string (e.g. "true"), Yes - result is 1 if "true" otherwise 0, Yes, Yes - bytes of the ``boolean``'s ``string`` representation, No, No, No,  Yes - a single item list containing the value
    ``list``, Yes - result is the comma-separated values, No, No, No, No, No, Yes - result is a map of the elements with keys their zero-based list index as ``string`` values, Yes
    ``map``, Yes - result is the comma-separated key-value pairs, No, No, No, No, No, Yes, Yes - a single item list containing the value
    ``number``, Yes - result as string (e.g. "10"), Yes, Yes - result is "true" if 1 otherwise "false", Yes - bytes of the ``number``'s ``string`` representation, No, No, No,  Yes - a single item list containing the value
    ``object``, Yes - result is the serialised XML, No, No, Yes - result is the XML's bytes, Yes, Yes, No,  Yes - a single item list containing the value
    ``schema``, Yes - result is the serialised XSD, No, No, Yes - result is the XSD's bytes, Yes, Yes, No,  Yes - a single item list containing the value
    ``string``, Yes, Yes - parsed as double (e.g. "10" or "5.03"), Yes - parsed as "true" or "false" (ignoring case), Yes - result is the ``string``'s bytes, Yes - considered as serialised XML, Yes - same as with ``object``, No, Yes - a single item list containing the value

.. note::
    **GITB software support:** Implicit conversions between types occurs when the expression referencing the source variable
    is a pure variable reference. For example given a ``binary`` variable ``myVariable``, referencing ``$myVariable`` when
    assigning to a ``string`` works whereas but ``concat('My value is ', $myVariable)`` does not. This is because the second
    case is a full XPath expression whereas the first case only identifies the variable.
