Test suites, and in particular the :ref:`test cases<test-case>` they include, may rely on additional file resources for
their documentation and test content. Such files may be made available to test suites in one of two ways:

* As binary :ref:`external configuration properties<test-case-configuration>`
* As files included in test suite archives.

When provided through configuration properties, files will be available in the test sessions' context from which they
can be used as ``binary`` variables. Although this approach is flexible, it does not cover all cases, cannot support
large files, and is not well adapted for resources with dependencies (e.g. an XML schema with imports). The approach of
providing files within test suite archives is an effective way of addressing such limitations.

The files included within test suites, apart from the test suite and test case definition files, can be:

* :ref:`Scriptlets<scriptlets>`, XML files containing blocks of test steps to be used across multiple test cases.
* Arbitrary files used as imports in :ref:`test cases<test-case-imports>` and :ref:`scriptlets<scriptlets_elements_imports>`.
* Documentation files, containing HTML content that is used to provide extended documentation for :ref:`test suites<test-suite-metadata>`,
  :ref:`test cases<test-case-metadata>`, and individual :ref:`test steps<tdl-steps-common-documentation>`.

Once such files are included in a test suite they can be used within itself but also shared with other test suites. The
approach to share files uses the test suites' ``id`` attribute to identify the test suite from which a resource will be
loaded. The constructs that support this all foresee a ``from`` attribute that is set with the target test suite ``id``,
defaulting to the current test suite if missing. Specifically:

* The ``call`` step to :ref:`call a scriptlet<tdl-step-call>`.
* The ``artifact`` element of a test case or scriptlet's :ref:`imports<test-case-imports>` block.
* The ``documentation`` element for :ref:`test suites<test-suite-metadata>`, :ref:`test cases<test-case-metadata>` and
  :ref:`test steps<tdl-steps-common-documentation>`.

The approach to lookup a test suite using the identifier specified in the ``from`` attribute is as follows:

#. Look for the test suite in the same **specification**.
#. If not found, look for the test suite in the other specifications of the **domain**.
   If across specifications multiple matching test suites are found, one of them will be arbitrarily picked. To avoid such
   a scenario it is obvious that you should ensure test suites used to load shared resources can be uniquely identified.

Once the target test suite has been located, the specific file is loaded using a file path relative to the test suite's
root. The approach to provide this path depends on the test construct in question:

* For ``call`` steps this is the value of the ``path`` attribute.
* For ``artifact`` import elements this is the element's value.
* For ``documentation`` elements this is the value of the ``import`` attribute.

.. note::
    **Resources with dependencies:** When resources such as XML schemas and Schematrons are imported from other test suites,
    resolution of dependent files (e.g. imported schemas) is carried out as expected based on the loaded file's location.
