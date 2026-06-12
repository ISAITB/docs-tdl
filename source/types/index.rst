.. index:: Types
.. _test-case-types:

Types
-----

The GITB specification foresees a type system that is used to better manage ongoing testing state and facilitate calculations.
The supported types can be split in three main categories:

* **Primitive types:** Simple values.
* **Object types:** XML documents or other complex structures.
* **Container types:** Structures that can contain primitive and object types or embedded container types.

The following table lists the available types. Note that the value under "Type name" is what needs to be used when referring to a given type
in GITB TDL:

.. csv-table::
    :stub-columns: 1
    :header: "Type name", "Type category", "Description"

    ``string``, Primitive, Used for text values
    ``number``, Primitive, Used for numeric values (integers and floating point numbers)
    ``boolean``, Primitive, Boolean values (true/false)
    ``binary``, Primitive, A byte buffer used typically to represent file content
    ``object``, Object, An XML document recorded as a Document Object Model (DOM)
    ``schema``, Object, Identical to an ``object`` but with an additional property for a schemaLocation
    ``map``, Container, A map of key-value pairs (similar to a ``java.util.Map``).
    ``list``, Container, A list of values (similar to a ``java.util.List``)

.. index:: Map (Type)
.. _test-case-types-maps:

Map variables
~~~~~~~~~~~~~

.. include:: /types/sections/test-case-types-maps.rst

.. index:: List (Type)
.. _test-case-types-lists:

List variables
~~~~~~~~~~~~~~

.. include:: /types/sections/test-case-types-lists.rst

.. _test-case-types-type-conversions:

Type conversions
~~~~~~~~~~~~~~~~

.. include:: /types/sections/test-case-types-type-conversions.rst