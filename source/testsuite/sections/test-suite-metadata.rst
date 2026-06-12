The purpose of the ``metadata`` element is to provide basic information about the test suite. This information is used both by administrators to better
manage existing test suites but also for end users to understand the test suite's purpose. The structure of the ``metadata`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    authors, no, A string to indicate the test suite's authors.
    dependencies, no, One or more ``dependency`` elements to record information on the test suite's dependencies.
    description, no, A string to provide a user-friendly description of the test suite that is displayed to users.
    documentation, no, Rich text content that provides further information on the current test suite.
    lastModified, no, A string acting as an indication of the last modification time for the test suite.
    name, yes, The name of the test suite that is used to identify it to users.
    published, no, A string acting as an indication of the test suite's publishing time.
    scopes, no, One or more ``scope`` elements to record information on the test suite's intended scope.
    type, no, Either "CONFORMANCE" (the default) or "INTEROPERABILITY". "INTEROPERABILITY" is used when multiple systems under test are considered in the test suite's test cases.
    update, no, Instructions determining the default choices when an update of this test suite is taking place.
    version, no, A string that indicates the test suite's version.

.. note::
    **GITB software support:** The test suite's ``id`` attribute is used to uniquely identify the test suite within a specification so ensure that it's unique 
    within a given specification. An uploaded test suite whose ``id`` matches that of an existing test suite will result in the existing test suite
    being updated. Furthermore, the ``version`` value is used only for display purposes whereas the ``authors``, ``dependencies``, ``published``, ``scopes`` and ``lastModified``
    values are recorded but never used or displayed. Finally, the "INTEROPERABILITY" ``type`` (defined at test suite level) is currently ignored.

.. index:: dependencies (Test suite)
.. index:: dependency (Test suite)
.. index:: identifier (Test suite)
.. index:: description (Test suite)
.. index:: uri (Test suite)
.. _test-suite-metadata-dependencies:

dependencies
++++++++++++

The ``dependencies`` element serves to include metadata on possible dependencies that this test suite might mave, such as the existence of
test services or other supporting software. It contains one or more ``dependency`` elements with the following structure:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"
    :delim: |

    description | no | A free text describing the nature of this dependency.
    identifier | no | A unique identifier for the dependency (if applicable).
    uri | no | A remote URI reference with further information on the dependency.

The following example illustrates how dependency metadata could be included in a test suite:

.. code-block:: xml
    :emphasize-lines: 4-10

    <metadata>
      <gitb:name>Test suite 1</gitb:name>
      <gitb:description>Test suite description.</gitb:description>
      <gitb:dependencies>
        <gitb:dependency>
          <gitb:identifier>DEP_1</gitb:identifier>
          <gitb:description>Description for dependency.</gitb:description>
          <gitb:uri>https://wiki.test.org/dep1</gitb:uri>
        </gitb:dependency>
      </gitb:dependencies>
    </metadata>

Dependency metadata is not currently leveraged in the Test Bed, and is only recorded as metadata in the test suite descriptor.

.. index:: documentation (Test suite)
.. index:: import (Test suite documentation)
.. index:: from (Test suite documentation)
.. index:: encoding (Test suite documentation)
.. _test-suite-metadata-documentation:

documentation
+++++++++++++

The ``documentation`` element complements the test suite's ``description`` by allowing the test suite's author to include extended rich text documentation as HTML. The structure of this element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    encoding, no, In case an ``import`` reference is defined this can be used to specify the file's encoding. If not provided ``UTF-8`` is considered.
    from, no, The identifier of a test suite from which the ``import`` file will be loaded. If unspecified the current test suite is assumed.
    import, no, A reference to a separate file within the test suite archive that defines the documentation content.

Using the above attributes to specify a reference to a separate file is not mandatory. The documentation's content can also be provided as the element's text content,
typically enclosed within a CDATA section if this includes HTML elements (in which case the ``from``, ``import`` and ``encoding`` attributes are omitted).
When loading documentation from a separate file, it is also possible to lookup this file from another test suite. This is
done by specifying as the value of the ``from`` attribute the ``id`` of the target test suite. This is used to lookup the
target test suite as follows:

#. Look for the test suite in the same **specification** as the current test case.
#. If not found in the same specification, look for the test suite in the other specifications of the test case's **domain**.
   If across specifications multiple matching test suites are found, one of them will be arbitrarily picked. To avoid such
   a scenario it is obvious that you should ensure test suites used to load shared resources can be uniquely identified.

This documentation can provide further information on the context of the test suite, diagrams or reference information that are useful to understand how it is to be completed or its purpose within the
overall specification. The content supplied supports several HTML features:

* Structure elements (e.g. headings, text blocks, lists).
* In-line styling.
* Tables.
* Links.
* Images.

The simplest way to provide such information is to enclose the HTML content in a CDATA section to ensure the XML remains well-formed. The
following sample provides an example of this approach:

.. code-block:: xml

    <testsuite id="TS1" xmlns="http://www.gitb.com/tdl/v1/" xmlns:gitb="http://www.gitb.com/core/v1/">
        <metadata>
            <gitb:name>TS1</gitb:name>
            <gitb:version>1.0</gitb:version>
            <gitb:description>A short description of the test suite to offer a short summary of its purpose.</gitb:description>
            <gitb:documentation><![CDATA[
                <p>Extended documentation for test suite <b>TS1</b></p>
                <p>This is an example to support the <a href="https://www.itb.ec.europa.eu/docs/tdl/latest">GITB TDL docs</a>.</p>
            ]]></gitb:documentation>
        </metadata>    
        ...
    </testsuite>

Note that documentation such as this is also supported for:

* The :ref:`test cases<test-case-metadata>` included in the test suite.
* Individual :ref:`test case steps<tdl-steps-common-documentation>`.

.. index:: dependencies (Test suite)
.. index:: dependency (Test suite)
.. index:: identifier (Test suite)
.. index:: description (Test suite)
.. index:: uri (Test suite)
.. _test-suite-metadata-scopes:

scopes
++++++

The ``scopes`` element serves to include metadata on the test suite's scope, such as functional coverage or legal articles.
It contains one or more ``scope`` elements with the following structure:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"
    :delim: |

    description | no | A free text describing the covered scope.
    identifier | no | A unique identifier for the scope (if applicable).
    uri | no | A remote URI reference with further information on the covered scope.

The following example illustrates how scope metadata could be included in a test suite:

.. code-block:: xml
    :emphasize-lines: 4-10

    <metadata>
      <gitb:name>Test suite 1</gitb:name>
      <gitb:description>Test suite description.</gitb:description>
      <gitb:scopes>
        <gitb:scope>
          <gitb:identifier>Invoicing</gitb:identifier>
          <gitb:description>Description for this scope.</gitb:description>
          <gitb:uri>https://wiki.test.org/invoicing</gitb:uri>
        </gitb:scope>
      </gitb:scopes>
    </metadata>

Scope metadata is not currently leveraged in the Test Bed, and is only recorded as metadata in the test suite descriptor.

.. index:: update (Test suite)
.. index:: updateMetadata (Test suite update)
.. index:: updateSpecification (Test suite update)
.. _test-suite-metadata-update:

update
++++++

The ``update`` element allows the test suite's developer to prescribe what should happen when this test suite is being uploaded and
an existing test suite with the same identifier is found. Through this you can define if the test suite's existing metadata 
(e.g. name, description and documentation) and the existing specification actors should be updated to match the definitions from
the new archive. Note that these choices represent the default selected options during the test suite upload, and can always be verified
and replaced by the Test Bed's operator.

It could be interesting to use the ``update`` element if the test developer is not the one performing the test suite upload. Doing so,
avoids providing detailed instruction to operations staff, by already encoding the relevant choices within the test suite archive itself.

The structure of the ``update`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @updateMetadata, no, A boolean value determining whether the existing test suite's metadata should be updated based on the new archive (default is "false").
    @updateSpecification, no, A boolean value determining whether the existing test suite's actor information should be updated based on the new archive (default is "false").

The following example shows how you can specify that the test suite's metadata should be updated to reflect the new values in the archive
(see attribute ``updateMetadata``). Any existing definitions of actors are left unchanged (see attribute ``updateSpecification``).

.. code-block:: xml
    :emphasize-lines: 6

    <testsuite id="TS1" xmlns="http://www.gitb.com/tdl/v1/" xmlns:gitb="http://www.gitb.com/core/v1/">
        <metadata>
            <gitb:name>TS1</gitb:name>
            <gitb:version>1.0</gitb:version>
            <gitb:description>A short description of the test suite to offer a short summary of its purpose.</gitb:description>
            <gitb:update updateMetadata="true" updateSpecification="false"/>
        </metadata>    
        ...
    </testsuite>

Relevant options to manage updates for existing test cases are possible through a similar ``update`` element of the :ref:`test case <test-case-metadata-update>` definitions.

.. index:: specification (Test suite)
.. index:: reference (Test suite reference)
.. index:: description (Test suite reference)
.. index:: link (Test suite reference)
.. _test-suite-metadata-specification:

specification
+++++++++++++

The ``specification`` element is an optional part of a test suite's metadata, that allows you to record in a structured manner a normative specification
reference for the test suite. Besides being present in the test suite definition, this information will also be rendered appropriately in the test suite's
on-screen display and in reports.

The structure of the ``specification`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    description, no, A text describing the referred specification.
    link, no, A link to allow navigation to the referred specification's online documentation.
    reference, no, The reference identifier or code.

All the above elements are optional, meaning that you can choose to provide any documentation you see fit for the specification. Depending on what is provided,
this information will be displayed accordingly, presenting for example the reference as a link if both are provided, or presenting only a link icon if only the
link is present.

The following example illustrates how this metadata could be used to identify the specification section relevant to the test suite and point to its online
documentation.

.. code-block:: xml
    :emphasize-lines: 6-10

    <testsuite id="TS1" xmlns="http://www.gitb.com/tdl/v1/" xmlns:gitb="http://www.gitb.com/core/v1/">
        <metadata>
            <gitb:name>TS1</gitb:name>
            <gitb:version>1.0</gitb:version>
            <gitb:description>A short description of the test suite to offer a short summary of its purpose.</gitb:description>
            <gitb:specification>
                <gitb:reference>Section-1.2.A</gitb:reference>
                <gitb:description>Security requirements</gitb:description>
                <gitb:link>https://my.spec.wiki.org</gitb:link>
            </gitb:specification> 
        </metadata>    
        ...
    </testsuite>

.. note::
    Similar specification reference information can also be added to individual :ref:`test cases<test-case-metadata-specification>`.
