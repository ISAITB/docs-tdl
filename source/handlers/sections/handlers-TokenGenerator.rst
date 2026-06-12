Used to generate tokens that can be used as data in test cases. This processing handler supports but does not require a processing
transaction to be established. The following operations are supported:

.. csv-table::
    :header: "Operation", "Description", "Input(s)", "Output(s)"

    ``random``, Generate a random integer of double precision number between optional minimum and maximum bounds., Yes, A ``number`` named ``value`` in the resulting step's ``map``.
    ``string``, Generate a text token with potentially fixed and/or random parts to match a provided regular expression., Yes, A ``string`` named ``value`` in the resulting step's ``map``.
    ``timestamp``, Generate a timestamp for the current or a provided time based on a format string., Yes, A ``string`` named ``value`` in the resulting step's ``map``.
    ``uuid``, Generate a random UUID text value matching a Java UUID (e.g. "123e4567-e89b-12d3-a456-556642440000"). This is a value that can be considered as unique for test purposes., No, A ``string`` named ``value`` in the resulting step's ``map``.

A typical use case for the ``TokenGenerator`` is to generate text tokens that can be used in test cases either as input parameters to
e.g. messaging calls (see :ref:`handlers-inputs-outputs`) or as values to replace in loaded text templates (see :ref:`test-case-expressions-template-files`).

.. _handlers-TokenGenerator_random:

TokenGenerator - random
^^^^^^^^^^^^^^^^^^^^^^^

The ``random`` operation is used to generate random numbers that can be used as-is, or help in selecting random elements from a ``list``, or
as part of XPath expressions. The expected inputs are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``integer`` | No | A ``boolean`` determining whether the produced value will be an integer (when "true") or a double-precision number (when ``false``). By default a double-precision number is produced.
    ``maximum`` | No | A ``number`` representing the maximum bound (exclusive) for the value to produce.
    ``minimum`` | No | A ``number`` representing the minimum bound (inclusive) for the value to produce. If not provided the default considered is zero.

The following example illustrates the operation's usage:

.. code-block:: xml

    <!--
        Generate a random integer between 1 and 10 (exclusive).
    -->
    <process output="numberRandom" handler="TokenGenerator" operation="random">
        <operation>random</operation>
        <input name="minimum">1</input>
        <input name="maximum">10</input>
        <input name="integer">true()</input>
    </process>

.. _handlers-TokenGenerator_string:

TokenGenerator - string
^^^^^^^^^^^^^^^^^^^^^^^

The ``string`` operation can be used to generate any kind of text token with both fixed and random parts following a pattern based on a provided regular expression.
The expected input is as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``format`` | Yes | A regular expression acting as a template to determine the generated token's format.

The following examples illustrate the operation's usage:

.. code-block:: xml

    <!--
        Generate a random string with 2 characters followed by 10 digits.
        Example output would be "cD6723820231".
    -->
    <process output="stringRandom" handler="TokenGenerator" operation="string">
        <input name="format">"[a-zA-Z]{2}\d{10}"</input>
    </process>
    <!--
        Generate a random string:
        - Starting with "PREFIX" and ending with "POSTFIX".
        - With random parts of (a) 5 digits, (b) 5 occurences of 'a', 'b' or 'c', and (c) 2 digits.
        - With hyphens between all fixed and random parts.
        Example output would be "PREFIX-32145-abcaa-02-POSTFIX".
    -->
    <process output="stringRandomAndFixed" handler="TokenGenerator" operation="string">
        <input name="format">"PREFIX-\d{5}-[abc]{5}-\d{2}-POSTFIX"</input>
    </process>

.. _handlers-TokenGenerator_timestamp:

TokenGenerator - timestamp
^^^^^^^^^^^^^^^^^^^^^^^^^^

The ``timestamp`` operation generates a timestamp string that includes date/time values but can also have fixed parts (e.g. if you need to generate
a text token with a fixed part and a variable part based on the current date/time). The inputs expected are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``date`` |  No | A ``string`` representing a date/time to use as the value to format. If specified along with ``time``, the ``time`` input takes precedence.
    ``diff`` | No | A ``number`` representing the milliseconds to consider as a diff from the considered ``time`` or ``date``. This value (default 0) is added to the considered ``time`` or ``date`` before formatting (i.e. a negative value signals an earlier time).
    ``format`` | No | The formatting pattern to apply provided as a ``string`` matching the Java date/time formatting specifications (see `Formatting configuration`_). If unspecified the current Epoch milliseconds are returned.
    ``inputFormat`` |  No | The formatting pattern to use to interpret the ``date`` input (if provided), matching the Java date/time formatting specifications (see `Formatting configuration`_).
    ``time`` |  No | A ``number`` representing the Epoch milliseconds to use as the date/time to format. If unspecified the current date/time is used.
    ``zone`` | No | The timezone to consider when generating a formatted timestamp provided as a ``string``. Expected values are those defined by Java (see `Timezone codes`_). If unspecified the default consider is ``UTC``.

If a ``date`` is provided without an ``inputFormat``, the pattern of ``dd/MM/yyyy'T'HH:mm:ss.SSSZ`` is assumed by default.
Moreover, all parts are considered optional allowing you to specify only parts of a date, making use of the following defaults
for those that are missing:

* Day of year (``dd``): The 1st day of the month.
* Month (``MM``): The 1st month of the year (January).
* Year (``yyyy``): The current year.
* Time elements (``HH``, ``mm``, ``ss`` and ``SSS``): A value of zero.
* Time zone (``Z``): UTC.

The following examples illustrate the operation's usage:

.. code-block:: xml

    <!--
        Generate a timestamp for the current time without specifying formatting.
        Example output would be "1560238501040".
    -->
    <process output="currentTimestamp" handler="TokenGenerator" operation="timestamp"/>
    <!--
        Generate a timestamp for the current time with provided formatting.
        Example output would be "DATE[2019-05-22] TIME[11:48:06]".
    -->
    <process output="formattedTimestampNow" handler="TokenGenerator" operation="timestamp">
        <input name="format">"'DATE['yyyy-MM-dd'] TIME['HH:mm:ss']'"</input>
    </process>
    <!--
        Generate an XML timestamp for the current time.
    -->
    <process output="currentXmlTimestamp" handler="TokenGenerator" operation="timestamp">
        <input name="format">"yyyy-MM-dd'T'HH:mm:ss.SSSXXX"</input>
    </process>
    <!--
        Generate an XML timestamp for the current time but expressed in the GMT+2 timezone.
    -->
    <process output="currentXmlTimestampInTimezone" handler="TokenGenerator" operation="timestamp">
        <input name="format">"yyyy-MM-dd'T'HH:mm:ss.SSSXXX"</input>
        <input name="zone">"GMT+2"</input>
    </process>
    <!--
        Generate a timestamp for the provided time and formatting.
        The output would be "2014-05-11".
    -->
    <process output="formattedTimestamp" handler="TokenGenerator" operation="timestamp">
        <input name="time">'1399792366000'</input>
        <input name="format">"yyyy-MM-dd"</input>
    </process>
    <!--
        Generate a timestamp for the current time minus one minute (600000 milliseconds) using the provided formatting.
        Example output would be "2019-06-11 10:23:10".
    -->
    <process output="formattedTimestampMinusOneMinute" handler="TokenGenerator" operation="timestamp">
        <input name="diff">-600000</input>
        <input name="format">"yyyy-MM-dd HH:mm:ss"</input>
    </process>
    <!--
        Obtain the current time (T) and then generate two timestamps:
        - T minus one hour.
        - T plus one hour.
    -->
    <process output="now" handler="TokenGenerator" operation="timestamp"/>
    <process output="nowMinusOneHour" handler="TokenGenerator" operation="timestamp">
        <input name="time">$now</input>
        <input name="diff">-3600000</input>
        <input name="format">"yyyy-MM-dd HH:mm:ss"</input>
    </process>
    <process output="nowPlusOneHour" handler="TokenGenerator" operation="timestamp">
        <input name="time">$now{value}</input>
        <input name="diff">3600000</input>
        <input name="format">"yyyy-MM-dd HH:mm:ss"</input>
    </process>
    <!--
        Generate a timestamp based on an existing date/time string plus one hour
    -->
    <process output="timestampFromFormattedString1" handler="TokenGenerator" operation="timestamp">
        <input name="date">'20-10-2021 13:30:00'</input>
        <input name="inputFormat">'dd-MM-yyyy HH:mm:ss'</input> <!-- Assumes UTC -->
        <input name="diff">3600000</input>
    </process>
    <!--
        Generate a timestamp based on an existing date/time string plus one hour (default formatting)
    -->
    <process output="timestampFromFormattedString2" handler="TokenGenerator" operation="timestamp">
        <input name="date">'20/10'</input> <!-- Assumes the current year, midnight, and a UTC timezone -->
        <input name="diff">3600000</input>
    </process>

.. note::
    **Timestamps for use in XML content:** Formatted timestamps generated for use in XML content should match the formatting
    of the ISO 8601 version of the W3C XML Schema dateTime definition. The pattern to apply to get a XSD-valid timestamp is:
    ``yyyy-MM-dd'T'HH:mm:ss.SSSXXX``.

.. _Formatting configuration: https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html
.. _Timezone codes: https://docs.oracle.com/javase/8/docs/api/java/time/ZoneId.html#of-java.lang.String-

.. _handlers-TokenGenerator_uuid:

TokenGenerator - uuid
^^^^^^^^^^^^^^^^^^^^^

The ``uuid`` operation provides a random and unique identifier where special formatting is not required (apart from an optional prefix and postfix).
The inputs expected are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``postfix`` | No | An optional ``string`` to add as a postfix to the generated part of the UUID.
    ``prefix`` | No | An optional ``string`` to add as a prefix to the generated part of the UUID.

The following examples illustrate the operation's usage:

.. code-block:: xml

    <!--
        Generate a UUID (e.g. "971b4df9-4351-4cb8-9ba5-1f6373717ae0").
    -->
    <process output="uuid1" handler="TokenGenerator" operation="uuid"/>
    <!--
        Generate a UUID with a prefix and postfix (e.g. "message-971b4df9-4351-4cb8-9ba5-1f6373717ae0@my.org").
    -->
    <process output="uuid2" handler="TokenGenerator" operation="uuid">
        <input name="prefix">"message-"</input>
        <input name="postfix">"@my.org"</input>
    </process>