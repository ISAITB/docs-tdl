Used to process collections (maps and lists) in ways not possible otherwise with TDL expressions. This processing handler does not require
a processing transaction to be established. The following operations are supported:

.. csv-table::
    :header: "Operation", "Description", "Input(s)", "Output(s)"
    :delim: |

    ``append`` | Append the elements of one collection to another. | Yes | No.
    ``clear`` | Receive a collection as input and empty it. | Yes | No.
    ``contains`` | Check to see whether a collection contains a given value. | Yes | Yes, a ``boolean`` representing the check result.
    ``entries`` | Extract a map's entries in a list. Each entry is a map with ``key`` and ``value`` elements | Yes | Yes, the list of entries (as maps).
    ``find`` | Search for and return a given value from the collection. | Yes | Yes, the matched item (if found).
    ``keys`` | Extract a map's keys in a list. | Yes | Yes, the list of ``string`` keys.
    ``randomKey`` | Return a random key from a map. | Yes | Yes, one of the map's ``string`` keys.
    ``randomValue`` | Return a random value from a collection. | Yes | Yes, the selected value (type varies depending on the content).
    ``remove`` | Remove an entry from a collection. | Yes | No.
    ``size`` | Receive a collection as input and return the number of elements it contains. | Yes | Yes, a ``number`` named ``output`` in the resulting step's ``map``.
    ``values`` | Extract a map's values in a list. | Yes | Yes, the list of values.

Collection or *container* variables represent flexible means of recording arbitrary sequences of data or hierarchical data structures. In particular
``map`` variables are very common as these are used to store results of :ref:`processing<tdl-step-process>`, :ref:`messaging<tdl-messaging-steps>` and :ref:`validation<tdl-step-verify>` operations.
Adding new elements to collections or replacing existing values is achieved using the :ref:`assign<tdl-step-assign>` step, where the
expressions used may also :ref:`determine collections<test-case-variables-from-expression-output>` that don't previously exist.
The ``CollectionUtils`` processing handler complements such operations by allowing further manipulations that cannot be achieved
through simple :ref:`expressions<test-case-expressions>`.

.. _handlers-CollectionUtils_append:

CollectionUtils - append
^^^^^^^^^^^^^^^^^^^^^^^^

The ``append`` operation allows you to merge two collections, by appending the values of one collection (the "from" collection) to
another one (the "to" collection). The inputs it expects are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``fromList`` | No | The ``list`` to read the values from (if the collection is a :ref:`list<test-case-types-lists>`). Either this or the ``fromMap`` input must be provided.
    ``fromMap`` | No | The ``map`` to read the values from (if the collection is a :ref:`list<test-case-types-maps>`). Either this or the ``fromList`` input must be provided.
    ``ignoreCase`` | No | When ``onlyMissing`` is set to "true", whether the matching of existing items will ignore casing (default is "false").
    ``onlyMissing`` | No | Whether only items missing from the target ``map`` or ``list`` should be added (default is "false"). For a ``list`` the check is made on values whereas for a ``map`` it is on keys.
    ``toList`` | No | The ``list`` to append the values to (if the collection is a :ref:`list<test-case-types-lists>`). Either this or the ``toMap`` input must be provided.
    ``toMap`` | No | The ``map`` to append the values to (if the collection is a :ref:`list<test-case-types-maps>`). Either this or the ``toList`` input must be provided.

The ``append`` operation does not return an output but rather modifies directly the target collection. Appended entries are added to the end
of the collection, maintaining their insertion order. Furthermore, you can leverage the ``onlyMissing`` and ``ignoreCase`` inputs to have only
missing and unique items appended to the target collection. The following examples illustrate the operation's use:

.. code-block:: xml

    <!-- Create a map -->
    <assign to="map1{a}">'Value A'</assign>
    <assign to="map1{b}">'Value B'</assign>

    <!-- Create a second map -->
    <assign to="map2{x}">'Value X'</assign>
    <assign to="map2{y}">'Value 2'</assign>

    <!-- Append map1 to the end of map2 -->
    <process handler="CollectionUtils" operation="append">
        <input name="fromMap">$map1</input>
        <input name="toMap">$map2</input>
    </process>

    <!-- Prints (the newly appended) 'Value A' -->
    <log>$map2{a}</log>

    <!-- Create two lists -->
    <assign to="list1" append="true">'A'</assign>
    <assign to="list1" append="true">'B'</assign>
    <assign to="list1" append="true">'C'</assign>

    <assign to="list2" append="true">'a'</assign>
    <assign to="list2" append="true">'X'</assign>
    <assign to="list2" append="true">'Z'</assign>

    <!-- Append list1 to list2 without duplicates (case-insensitive) -->
    <process handler="CollectionUtils" operation="append">
        <input name="fromList">$list1</input>
        <input name="toList">$list2</input>
        <input name="onlyMissing">true()</input>
        <input name="ignoreCase">true()</input>
    </process>
    <!-- Prints "a,X,Z,B,C" (having skipped the "A" element of list1) -->
    <log>$list2</log>

.. _handlers-CollectionUtils_clear:

CollectionUtils - clear
^^^^^^^^^^^^^^^^^^^^^^^

The ``clear`` operation allows a test case to empty the contents of a given collection if this becomes necessary.
The inputs it expects are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``list`` | No | The ``list`` to be cleared (if the collection is a :ref:`list<test-case-types-lists>`). Either this or the ``map`` input must be provided.
    ``map`` | No | The ``map`` to be cleared (if the collection is a :ref:`map<test-case-types-maps>`). Either this or the ``list`` input must be provided.

The following examples illustrate how this works for lists and maps:

.. code-block:: xml

    <!-- Create a map with three elements -->
    <assign to="aMap{a}">'Value 1'</assign>
    <assign to="aMap{b}">'Value 2'</assign>
    <assign to="aMap{c}">'Value 3'</assign>
    <!-- Create a list with two elements -->
    <assign to="aList" append="true">'Value 1'</assign>
    <assign to="aList" append="true">'Value 2'</assign>
    <!-- Empty the map -->
    <process handler="CollectionUtils" operation="clear">
        <input name="map">$aMap</input>
    </process>
    <!-- Empty the list -->
    <process handler="CollectionUtils" operation="clear">
        <input name="list">$aList</input>
    </process>

.. _handlers-CollectionUtils_contains:

CollectionUtils - contains
^^^^^^^^^^^^^^^^^^^^^^^^^^

The ``contains`` operation allows checking whether a value is defined within a collection. In case the collection is a ``map``, the lookup is
done on the basis of the entries' keys. Otherwise for a ``list`` the lookup considers the contained elements' values.
The inputs it expects are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``ignoreCase`` | No | Whether the matching of items to determine the operation's result will ignore casing (default is "false").
    ``list`` | No | The ``list`` to be considered (if the collection is a :ref:`list<test-case-types-lists>`). Either this or the ``map`` input must be provided.
    ``map`` | No | The ``map`` to be considered (if the collection is a :ref:`map<test-case-types-maps>`). Either this or the ``list`` input must be provided.
    ``value`` | Yes | The value to look for.

The following examples illustrate the operation's use:

.. code-block:: xml

    <!-- Create a map -->
    <assign to="aMap{a}">'Value 1'</assign>
    <assign to="aMap{b}">'Value 2'</assign>
    <assign to="aMap{c}">'Value 3'</assign>
    <!-- Lookup an existing value -->
    <process handler="CollectionUtils" output="mapCheck1" operation="contains">
        <input name="map">$aMap</input>
        <input name="value">'b'</input>
    </process>
    <!-- Prints "true" -->
    <log>$mapCheck1</log>
    <!-- Lookup an non-existing value -->
    <process handler="CollectionUtils" output="mapCheck2" operation="contains">
        <input name="map">$aMap</input>
        <input name="value">'x'</input>
    </process>
    <!-- Prints "false" -->
    <log>$mapCheck2</log>
    <!-- Lookup an existing value in a non case-sensitive way -->
    <process handler="CollectionUtils" output="mapCheck3" operation="contains">
        <input name="map">$aMap</input>
        <input name="value">'A'</input>
        <input name="ignoreCase">true()</input>
    </process>
    <!-- Prints "true" -->
    <log>$mapCheck3</log>

    <!-- Create a list -->
    <assign to="aList" append="true">'Value 1'</assign>
    <assign to="aList" append="true">'Value 2'</assign>
    <assign to="aList" append="true">'Value 3'</assign>
    <!-- Lookup an existing value -->
    <process handler="CollectionUtils" output="listCheck1" operation="contains">
        <input name="map">$aList</input>
        <input name="value">'Value 1'</input>
    </process>
    <!-- Prints "true" -->
    <log>$listCheck1</log>
    <!-- Lookup an non-existing value -->
    <process handler="CollectionUtils" output="listCheck2" operation="contains">
        <input name="map">$aList</input>
        <input name="value">'Value X'</input>
    </process>
    <!-- Prints "false" -->
    <log>$listCheck2</log>
    <!-- Lookup an existing value in a non case-sensitive way -->
    <process handler="CollectionUtils" output="listCheck3" operation="contains">
        <input name="map">$aList</input>
        <input name="value">'value 1'</input>
        <input name="ignoreCase">true()</input>
    </process>
    <!-- Prints "true" -->
    <log>$listCheck3</log>

.. _handlers-CollectionUtils_entries:

CollectionUtils - entries
^^^^^^^^^^^^^^^^^^^^^^^^^

The ``entries`` operation is used to return the entries contained within a ``map``. The entries are returned as a ``list``
with each entry being a ``map`` with ``key`` and ``value`` elements. In these, the ``key`` is always a ``string`` whereas the
``value`` is the type of the original entry. The inputs expected by this operation are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``map`` | Yes | The ``map`` to process.

The following example illustrates the operation's use:

.. code-block:: xml

    <assign to="myMap{key1}">"Value 1"</assign>
    <assign to="myMap{key2}">"Value 2"</assign>
    <assign to="myMap{key3}">"Value 3"</assign>
    <process handler="CollectionUtils" operation="entries" output="extractedEntries">
       <input name="map">$myMap</input>
    </process>
    <!-- Prints "[key]=[key1],[value]=[Value 1],[key]=[key2],[value]=[Value 2],[key]=[key3],[value]=[Value 3]" -->
    <log>$extractedEntries</log>
    <!-- Prints "Value 1" -->
    <log>$extractedKeys{0}{value}</log>

.. _handlers-CollectionUtils_find:

CollectionUtils - find
^^^^^^^^^^^^^^^^^^^^^^

The ``find`` operation is used to return a value from a collection. In case the collection is a ``map``, the lookup is
done on the basis of the entries' keys. Otherwise for a ``list`` the lookup considers the contained elements' values.
The inputs it expects are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``ignoreCase`` | No | Whether the matching of items to will ignore casing (default is "false").
    ``list`` | No | The ``list`` to be considered (if the collection is a :ref:`list<test-case-types-lists>`). Either this or the ``map`` input must be provided.
    ``map`` | No | The ``map`` to be considered (if the collection is a :ref:`map<test-case-types-maps>`). Either this or the ``list`` input must be provided.
    ``value`` | Yes | The value to look for.

The following examples illustrate the operation's use:

.. code-block:: xml

    <!--
        Map example. Look up a value from a map (case-sensitive lookup based on the key).
    -->
    <assign to="myMap{key1}">"Value1"</assign>
    <assign to="myMap{key2}">"Value2"</assign>
    <process handler="CollectionUtils" output="mapValue" operation="find">
        <input name="map">$myMap</input>
        <input name="value">'key1'</input>
    </process>
    <process handler="VariableUtils" operation="exists" input="mapValue" output="mapValueFound"/>
    <!-- Prints "true" -->
    <log>$mapValueFound</log>
    <!-- Prints "Value1" -->
    <log>$mapValue</log>

    <!--
        List example. Look up a value from a map (case-sensitive lookup).
    -->
    <assign to="myList" append="true">"Value1"</assign>
    <assign to="myList" append="true">"Value2"</assign>
    <process handler="CollectionUtils" output="listValue" operation="find">
        <input name="list">$myList</input>
        <input name="value">'value1'</input>
        <input name="ignoreCase">true()</input>
    </process>
    <process handler="VariableUtils" operation="exists" input="listValue" output="listValueFound"/>
    <!-- Prints "true" -->
    <log>$listValueFound</log>
    <!-- Prints "Value1" -->
    <log>$listValue</log>

.. _handlers-CollectionUtils_keys:

CollectionUtils - keys
^^^^^^^^^^^^^^^^^^^^^^

The ``keys`` operation is used to return the keys contained within a ``map``, as a ``list`` of ``string``.
The inputs it expects are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``map`` | Yes | The ``map`` to process.

The following example illustrates the operation's use:

.. code-block:: xml

    <assign to="myMap{key1}">"Value 1"</assign>
    <assign to="myMap{key2}">"Value 2"</assign>
    <assign to="myMap{key3}">"Value 3"</assign>
    <process handler="CollectionUtils" operation="keys" output="extractedKeys">
       <input name="map">$myMap</input>
    </process>
    <!-- Prints "key1,key2,key3" -->
    <log>$extractedKeys</log>
    <!-- Prints "key1" -->
    <log>$extractedKeys{0}</log>

.. _handlers-CollectionUtils_randomKey:

CollectionUtils - randomKey
^^^^^^^^^^^^^^^^^^^^^^^^^^^

The ``randomKey`` operation works specifically with ``map`` variables and is used to randomly retrieve one of its keys.
The inputs it expects are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``map`` | Yes | The ``map`` to be considered.

The following example illustrates the operation's use:

.. code-block:: xml

    <!-- Create a map -->
    <assign to="aMap{a}">'Value 1'</assign>
    <assign to="aMap{b}">'Value 2'</assign>
    <!-- Get one of the map's keys -->
    <process handler="CollectionUtils" output="value1" operation="randomKey">
        <input name="map">$aMap</input>
    </process>
    <!-- Prints either "a" or "b" -->
    <log>$value1</log>

.. _handlers-CollectionUtils_randomValue:

CollectionUtils - randomValue
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The ``randomValue`` operation works is used to randomly retrieve randomly one of the provided collection's values.
The inputs it expects are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``list`` | No | The ``list`` to be considered (if the collection is a :ref:`list<test-case-types-lists>`). Either this or the ``map`` input must be provided.
    ``map`` | No | The ``map`` to be considered (if the collection is a :ref:`map<test-case-types-maps>`). Either this or the ``list`` input must be provided.

The following example illustrates the operation's use:

.. code-block:: xml

    <!-- Create a map -->
    <assign to="aMap{a}">'Value 1'</assign>
    <assign to="aMap{b}">'Value 2'</assign>
    <!-- Get one of the map's values -->
    <process handler="CollectionUtils" output="value1" operation="randomValue">
        <input name="map">$aMap</input>
    </process>
    <!-- Prints either "Value 1" or "Value 2" -->
    <log>$value1</log>

    <!-- Create a list -->
    <assign to="aList" append="true">'Value 1'</assign>
    <assign to="aList" append="true">'Value 2'</assign>
    <!-- Get one of the list's values -->
    <process handler="CollectionUtils" output="value2" operation="randomValue">
        <input name="list">$aList</input>
    </process>
    <!-- Prints either "Value 1" or "Value 2" -->
    <log>$value2</log>

.. _handlers-CollectionUtils_remove:

CollectionUtils - remove
^^^^^^^^^^^^^^^^^^^^^^^^

The ``remove`` operation is used to remove specific entries from a collection. When using a ``map`` the removed entry is matched
based on its key. For a ``list``, the entry to remove is identified by its zero-based index. The inputs it expects are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``list`` | No | The ``list`` to be considered (if the collection is a :ref:`list<test-case-types-lists>`). Either this or the ``map`` input must be provided.
    ``map`` | No | The ``map`` to be considered (if the collection is a :ref:`map<test-case-types-maps>`). Either this or the ``list`` input must be provided.
    ``item`` | Yes | In case of a ``list`` this is a ``number`` set with the zero-based index of the element to remove. For a ``map`` this is the ``string`` key of the entry to be removed.

The following examples illustrate the operation's use:

.. code-block:: xml

    <!-- Create a map -->
    <assign to="aMap{a}">'Value 1'</assign>
    <assign to="aMap{b}">'Value 2'</assign>
    <!-- Remove the entry with key "a" -->
    <process handler="CollectionUtils" operation="remove">
        <input name="map">$aMap</input>
        <input name="item">'a'</input>
    </process>

    <!-- Create a list -->
    <assign to="aList" append="true">'Value 1'</assign>
    <assign to="aList" append="true">'Value 2'</assign>
    <!-- Remove the second entry from the list -->
    <process handler="CollectionUtils" operation="remove">
        <input name="list">$aList</input>
        <!-- Providing "1" given that indexes are zero-based -->
        <input name="item">1</input>
    </process>

.. _handlers-CollectionUtils_size:

CollectionUtils - size
^^^^^^^^^^^^^^^^^^^^^^

The ``size`` operation allows a test case to determine a collection's size. This can be particularly useful in the case of
operations that return an arbitrary number of data items as a ``list`` which we need to iterate over.
The inputs it expects are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``list`` | No | The ``list`` of which the elements are to be counted (if the collection is a :ref:`list<test-case-types-lists>`). Either this or the ``map`` input must be provided.
    ``map`` | No | The ``map`` of which the elements are to be counted (if the collection is a :ref:`map<test-case-types-maps>`). Either this or the ``list`` input must be provided.

The following examples illustrate how this operation can be used:

.. code-block:: xml

    <!-- Create a map with three elements -->
    <assign to="aMap{a}">'Value 1'</assign>
    <assign to="aMap{b}">'Value 2'</assign>
    <assign to="aMap{c}">'Value 3'</assign>
    <!-- Create a list with two elements -->
    <assign to="aList" append="true">'Value 1'</assign>
    <assign to="aList" append="true">'Value 2'</assign>
    <!-- Calculate the size of the map -->
    <process id="aMapSize" handler="CollectionUtils" operation="size">
        <input name="map">$aMap</input>
    </process>
    <!-- Prints "3" -->
    <log>$aMapSize{output}</log>
    <!-- Calculate the size of the list -->
    <process id="aListSize" handler="CollectionUtils" operation="size">
        <input name="list">$aList</input>
    </process>
    <!-- Prints "2" -->
    <log>$aListSize{output}</log>
    <!-- Print each list element. -->
    <foreach desc="Iterate list" counter="index" start="0" end="$aListSize{output}">
        <do>
            <!-- Prints "Value 1" and then "Value 2" -->
            <log>aList{$index}</log>
        </do>
    </foreach>

.. note::
    **Nested collections:** If a collection structure contains itself further collection structures as elements, the
    ``size`` operation will only count the collection's top level elements.

.. _handlers-CollectionUtils_values:

CollectionUtils - values
^^^^^^^^^^^^^^^^^^^^^^^^

The ``values`` operation is used to return the values contained within a ``map``. The inputs it expects are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``map`` | Yes | The ``map`` to process.

The following example illustrates the operation's use:

.. code-block:: xml

    <assign to="myMap{key1}">"Value 1"</assign>
    <assign to="myMap{key2}">"Value 2"</assign>
    <assign to="myMap{key3}">"Value 3"</assign>
    <process handler="CollectionUtils" operation="values" output="extractedValues">
       <input name="map">$myMap</input>
    </process>
    <!-- Prints "Value 1,Value 2,Value 3" -->
    <log>$extractedValues</log>
    <!-- Prints "Value 1" -->
    <log>$extractedValues{0}</log>