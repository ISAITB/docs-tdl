The following sections discuss the purpose and use of each element contained within the ``scriptlet`` definition.

.. index:: metadata (Scriptlets)
.. _scriptlets_elements_metadata:

Metadata
~~~~~~~~

The structure and content of the ``metadata`` element is identical to the one defined for the :ref:`test suite<test-suite-metadata>`
and its :ref:`test cases<test-case-metadata>`. In the case of scriptlets however, no such metadata is displayed to users
meaning that any information provided here is purely for test developers. In contrast to test suites and test cases,
the ``metadata`` block for scriptlets is optional and can be fully skipped.

For further information on this element check the :ref:`test case metadata documentation<test-case-metadata>`.

.. index:: namespaces (Scriptlets)
.. _scriptlets_elements_namespaces:

Namespaces
~~~~~~~~~~

The ``namespaces`` element is identical in structure and purpose to the one defined for :ref:`test cases<test-case-namespaces>`.
It is used to declare one or more namespace mappings (prefix to value), allowing the declared prefixes to be used in the scriptlet's
XPath expressions.

If the scriptlet is called by a test case or another scriptlet that already defines namespaces, these definitions are inherited. In case
the current scriptlet defines a namespace prefix that has already been defined, then the scriptlet's own definition overrides the inherited
one.

For further information on this element check the :ref:`test case namespaces documentation<test-case-namespaces>`.

.. index:: imports (Scriptlets)
.. _scriptlets_elements_imports:

Imports
~~~~~~~

The ``imports`` element allows the use of arbitrary resources from the same or another test suite. The purpose and structure
of this element is identical to the one defined for :ref:`test cases<test-case-imports>`. One notable difference in the case
of scriptlets however, is that imports lacking an explicit ``from`` attribute are loaded from the test suite containing the
scriptlet, which is not necessarily the same as the test suite containing the currently executing test case.

For further information on this element check the :ref:`test case imports documentation<test-case-imports>`.
