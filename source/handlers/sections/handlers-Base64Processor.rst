Used to manipulate Base64-encoded content for use in test cases. This processing handler supports but does not require a processing
transaction to be established. The following operations are supported:

.. csv-table::
    :header: "Operation", "Description", "Input(s)", "Output(s)"

    ``decode``, Receive a ``string`` input that is Base64-encoded and return the ``binary`` output it corresponds to., Yes, A ``binary`` value named ``output`` in the resulting step's ``map``.
    ``encode``, Receive a ``binary`` input and return a ``string`` with its Base64-encoded representation., Yes, A ``string`` named ``output`` in the resulting step's ``map``.

Base64 encoding is a technique often used to represent arbitrary byte sequences as text. Using this processing handler you can work with Base64 encoded texts
that need to be decoded in test cases, but also encode binary content where this is needed. In both the encoding and decoding steps there is support for Base64
content and also data URLs. Data URLs are commonly used in web representations for the inline definition of binary resources. A data URL is essentially the
Base64-encoded bytes prefixed with the content's mime type as ``data:[mime type],base64,[BASE64 encoded string]`` (e.g. ``data:application/xml;base64,YXNoZGl1cXcgaGRva...``).

.. _handlers-Base64Processor_decode:

Base64Processor - decode
^^^^^^^^^^^^^^^^^^^^^^^^

The ``decode`` operation is used to decode a Base64-encoded string to its binary representation. The inputs it expects are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"

    ``input``, Yes, The ``string`` value (expected to be Base64-encoded or formatted as a data URL) that will be processed to return its corresponding ``binary`` value.

The following example illustrates how to use the ``decode`` operation to decode and validate an image:

.. code-block:: xml

    <!--
        Decode a Base64-encoded string representing an image. The input could also be encoded as a data URL.
    -->
    <process id="decodeData" handler="Base64Processor" output="image" operation="decode">
        <input name="input">$myBase64EncodedText</input>
    </process>
    <!--
        Validate the image using an custom validator.
    -->
    <verify desc="Validate image" handler="$DOMAIN{imageValidator}">
        <input name="image">$image</input>
    </verify>

.. _handlers-Base64Processor_encode:

Base64Processor - encode
^^^^^^^^^^^^^^^^^^^^^^^^

The ``encode`` operation is used to encode a Base64-encoded string from binary data. The inputs it expects are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"

    ``dataUrl``, No, A ``boolean`` flag that indicates whether or not the output should be formatted as a data URL (default is ``false``).
    ``input``, Yes, The ``binary`` value that will be encoded as a Base64 string.

The following example illustrates how to use the ``encode`` operation to process binary data:

.. code-block:: xml

    <!--
        Encode the binary variable "aBinaryVariable" and return the encoded string as "data1{output}".
    -->
    <process id="encode1" handler="Base64Processor" output="result" operation="encode">
        <input name="input">$aBinaryVariable</input>
    </process>
    <!--
        Encode the binary variable "aBinaryVariable" and return the encoded string as "data2{output}".
        The result in this case is formatted as a data URL.
    -->
    <process id="encode2" handler="Base64Processor" output="resultAsDataUrl" operation="encode">
        <input name="input">$aBinaryVariable</input>
        <input name="dataUrl">true()</input>
    </process>