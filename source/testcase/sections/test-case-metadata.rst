The ``metadata`` element is basically the same as the one defined for the test suite :ref:`test-suite-metadata`. Its purpose is to provide basic information 
about the test case to help users understand its purpose. Its structure is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    authors, no, A string to indicate the test case's authors.
    dependencies, no, One or more ``dependency`` elements to record information on the test case's dependencies.
    description, no, A string to provide a user-friendly description of the test case that is displayed to users.
    documentation, no, Rich text content that provides further information on the current test case.
    lastModified, no, A string acting as an indication of the last modification time for the test case.
    name, yes, The name of the test case that is used to identify it to users.
    published, no, A string acting as an indication of the test case's publishing time.
    scopes, no, One or more ``scope`` elements to record information on the test case's intended scope.
    specification, no, Optional information regarding the test case's normative specification reference.
    tags, no, Optional tags used to record additional metadata for the test case and visually highlight its attributes.
    type, no, Either "CONFORMANCE" (the default) or "INTEROPERABILITY". "INTEROPERABILITY" is used when more than one actor are defined as SUTs.
    update, no, Instructions determining the default choices when an update of this test case is taking place.
    version, no, A string that indicates the test case's version.

.. note::
    **GITB software support:** The test case ``type`` must currently be set to "CONFORMANCE" (the default value) as the
    "INTEROPERABILITY" type is not supported. Finally, the ``version``, ``authors``, ``dependencies``, ``published``, ``scopes`` and ``lastModified`` values are recorded but never used or displayed.

.. index:: dependencies (Test case)
.. index:: dependency (Test case)
.. index:: identifier (Test case)
.. index:: description (Test case)
.. index:: uri (Test case)
.. _test-case-metadata-dependencies:

dependencies
++++++++++++

The ``dependencies`` element serves to include metadata on possible dependencies that this test case might mave, such as the existence of
test services or other supporting software. It contains one or more ``dependency`` elements with the following structure:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"
    :delim: |

    description | no | A free text describing the nature of this dependency.
    identifier | no | A unique identifier for the dependency (if applicable).
    uri | no | A remote URI reference with further information on the dependency.

The following example illustrates how dependency metadata could be included in a test case:

.. code-block:: xml
    :emphasize-lines: 4-10

    <metadata>
      <gitb:name>Test case 1</gitb:name>
      <gitb:description>Test case description.</gitb:description>
      <gitb:dependencies>
        <gitb:dependency>
          <gitb:identifier>DEP_1</gitb:identifier>
          <gitb:description>Description for dependency.</gitb:description>
          <gitb:uri>https://wiki.test.org/dep1</gitb:uri>
        </gitb:dependency>
      </gitb:dependencies>
    </metadata>

Dependency metadata is not currently leveraged in the Test Bed, and is only recorded as metadata in the test case descriptor.

.. index:: documentation (Test case)
.. index:: import (Test case documentation)
.. index:: from (Test case documentation)
.. index:: encoding (Test case documentation)
.. _test-case-metadata-documentation:

documentation
+++++++++++++

The ``documentation`` element complements the test case's ``description`` by allowing the author to include extended rich text documentation as HTML. The structure of this element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    import, no, A reference to a separate file within the test suite archive that defines the documentation content.
    from, no, The identifier of a test suite from which the ``import`` file will be loaded. If unspecified the current test suite is assumed.
    encoding, no, In case an ``import`` reference is defined this can be used to specify the file's encoding. If not provided ``UTF-8`` is considered.

Using the above attributes to specify a reference to a separate file is not mandatory. The documentation's content can also be provided as the element's text content,
typically enclosed within a CDATA section if this includes HTML elements (in which case the ``from``, ``import`` and ``encoding`` attributes are omitted).
When loading documentation from a separate file, it is also possible to lookup this file from another test suite. This is
done by specifying as the value of the ``from`` attribute the ``id`` of the target test suite. This is used to lookup the
target test suite as follows:

#. Look for the test suite in the same **specification** as the current test case.
#. If not found in the same specification, look for the test suite in the other specifications of the test case's **domain**.
   If across specifications multiple matching test suites are found, one of them will be arbitrarily picked. To avoid such
   a scenario it is obvious that you should ensure test suites used to load shared resources can be uniquely identified.

This documentation can provide further information on the context of the test case, diagrams or reference information that are useful to understand how it is to be completed or its purpose within the
overall specification. The content supplied supports several HTML features:

* Structure elements (e.g. headings, text blocks, lists).
* In-line styling.
* Tables.
* Links.
* Images.

The simplest way to provide such information is to enclose the HTML content in a CDATA section to ensure the XML remains well-formed. The
following sample provides an example of this approach:

.. code-block:: xml

    <testcase id="TS1-TC1" xmlns="http://www.gitb.com/tdl/v1/" xmlns:gitb="http://www.gitb.com/core/v1/">
        <metadata>
            <gitb:name>Test case 1</gitb:name>
            <gitb:version>1.0</gitb:version>
            <gitb:description>A short description of the test case to offer a short summary of its purpose.</gitb:description>
            <gitb:documentation><![CDATA[
                <p>Extended documentation for <b>Test case 1</b></p>
                <p>This is an example to support the <a href="https://www.itb.ec.europa.eu/docs/tdl/latest">GITB TDL docs</a>.</p>
            ]]></gitb:documentation>
        </metadata>    
        ...
    </testcase>

Note that documentation such as this is also supported for:

* The overall :ref:`test suite<test-suite-metadata>`.
* Individual :ref:`test case steps<tdl-steps-common-documentation>`.

.. index:: dependencies (Test case)
.. index:: dependency (Test case)
.. index:: identifier (Test case)
.. index:: description (Test case)
.. index:: uri (Test case)
.. _test-case-metadata-scopes:

scopes
++++++

The ``scopes`` element serves to include metadata on the test case's scope, such as functional coverage or legal articles.
It contains one or more ``scope`` elements with the following structure:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"
    :delim: |

    description | no | A free text describing the covered scope.
    identifier | no | A unique identifier for the scope (if applicable).
    uri | no | A remote URI reference with further information on the covered scope.

The following example illustrates how scope metadata could be included in a test case:

.. code-block:: xml
    :emphasize-lines: 4-10

    <metadata>
      <gitb:name>Test case 1</gitb:name>
      <gitb:description>Test case description.</gitb:description>
      <gitb:scopes>
        <gitb:scope>
          <gitb:identifier>Invoicing</gitb:identifier>
          <gitb:description>Description for this scope.</gitb:description>
          <gitb:uri>https://wiki.test.org/invoicing</gitb:uri>
        </gitb:scope>
      </gitb:scopes>
    </metadata>

Scope metadata is not currently leveraged in the Test Bed, and is only recorded as metadata in the test case descriptor.

.. index:: update (Test case)
.. index:: updateMetadata (Test case update)
.. index:: resetTestHistory (Test case update)
.. _test-case-metadata-update:

update
++++++

The ``update`` element allows the test suite's developer to prescribe what should happen when this test case is being uploaded and
an existing test case with the same identifier is found. Through this you can define if the test case's existing metadata 
(e.g. name, description and documentation) should be updated to match the definitions from the new archive. In addition, you can 
specify whether the testing history linked to the test case being updated should be reset. Note that these choices represent the
default selected options during the test suite upload, and can always be verified and replaced by the Test Bed's operator.

It could be interesting to use the ``update`` element if the test developer is not the one performing the test suite upload. Doing so,
avoids providing detailed instruction to operations staff, by already encoding the relevant choices within the test suite archive itself.

The structure of the ``update`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @resetTestHistory, no, A boolean value determining whether any previously executed test sessions for the test case being updated should be considered as obsolete (default is "false").
    @updateMetadata, no, A boolean value determining whether the existing test case's metadata should be updated based on the new archive (default is "false").

The following example shows how you can specify that the test case's metadata should be updated to reflect the new values in the archive
(see attribute ``updateMetadata``). Also we specify here that any existing test sessions should be considered obsolete, forcing users to 
re-execute their tests for the updated version (see attribute ``resetTestHistory``).

.. code-block:: xml
    :emphasize-lines: 6

    <testcase id="TS1-TC1" xmlns="http://www.gitb.com/tdl/v1/" xmlns:gitb="http://www.gitb.com/core/v1/">
        <metadata>
            <gitb:name>Test case 1</gitb:name>
            <gitb:version>1.0</gitb:version>
            <gitb:description>A short description of the test case to offer a short summary of its purpose.</gitb:description>
            <gitb:update updateMetadata="true" resetTestHistory="true"/>
        </metadata>    
        ...
    </testcase>

Relevant options to manage updates at test suite level are possible through a similar ``update`` element of the :ref:`test suite <test-suite-metadata-update>` definition.

.. index:: tags (Test case)
.. index:: tag (Test case)
.. index:: foreground (Test case tag)
.. index:: background (Test case tag)
.. index:: name (Test case tag)
.. _test-case-metadata-tags:

tags
++++

The ``tags`` element allows you to specify arbitrary additional metadata for the test case that may help better distinguish it for users. You
may also use the same tags across multiple test cases as a means of classifying them based on common traits. The meaning you provide to such
tags is fully up to you, but typical examples would be to highlight test cases linked to "security" or test cases focusing on "unhappy flow"
scenarios. Other than highlighting certain traits, tags have no impact on a test case's execution or result.

The structure of the ``tags`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    tag, yes, One or more elements representing the test case's tags.

Each ``tag`` element has the following structure:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @background, no, A string for the `hexadecimal code <https://en.wikipedia.org/wiki/Web_colors>`_ of the tag's background colour. If not specified a generic default colour will be selected by the Test Bed.
    @foreground, no, A string for the `hexadecimal code <https://en.wikipedia.org/wiki/Web_colors>`_ of the tag's foreground colour (its text). If not specified a generic default colour will be selected by the Test Bed.
    @name, yes, A string for the name of the tag to be displayed. Although no restrictions are applied this is expected to be concise.

The ``tag`` element can also have an optional text content. If this is provided it is considered as a explanation over the meaning of this tag and 
is treated depending on how the tag is viewed. If through the Test Bed's user interface this would be a tooltip, otherwise in a PDF report
this would be a description added in a legend section.

The following example illustrates the definition of two tags to be displayed for a test case. The first one highlights that it relates to
security features and is expressed as a white-on-red "security" label. The second one specifies a white-on-black tag to 
highlight that this is a new test case in "version 2.0".

.. code-block:: xml
    :emphasize-lines: 6-9

    <testcase id="TS1-TC1" xmlns="http://www.gitb.com/tdl/v1/" xmlns:gitb="http://www.gitb.com/core/v1/">
        <metadata>
            <gitb:name>Test case 1</gitb:name>
            <gitb:version>1.0</gitb:version>
            <gitb:description>A short description of the test case to offer a short summary of its purpose.</gitb:description>
            <gitb:tags>
                <gitb:tag foreground="#FFFFFF" background="#FF2E00" name="security">Test case related to security features.</gitb:tag>
                <gitb:tag foreground="#FFFFFF" background="#000000" name="version 2.0">New test case added in version 2.0.</gitb:tag>
            </gitb:tags>	            
        </metadata>    
        ...
    </testcase>

.. index:: specification (Test case)
.. index:: reference (Test case reference)
.. index:: description (Test case reference)
.. index:: link (Test case reference)
.. _test-case-metadata-specification:

specification
+++++++++++++

The ``specification`` element is an optional part of a test case's metadata, that allows you to record in a structured manner a normative specification
reference for the test case. Besides being present in the test case definition, this information will also be rendered appropriately in the test case's
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

The following example illustrates how this metadata could be used to identify the specification section relevant to the test case and point to its online
documentation.

.. code-block:: xml
    :emphasize-lines: 6-10

    <testcase id="TS1-TC1" xmlns="http://www.gitb.com/tdl/v1/" xmlns:gitb="http://www.gitb.com/core/v1/">
        <metadata>
            <gitb:name>Test case 1</gitb:name>
            <gitb:version>1.0</gitb:version>
            <gitb:description>A short description of the test case to offer a short summary of its purpose.</gitb:description>
            <gitb:specification>
                <gitb:reference>Section-1.2.A</gitb:reference>
                <gitb:description>Security requirements</gitb:description>
                <gitb:link>https://my.spec.wiki.org</gitb:link>
            </gitb:specification> 
        </metadata>    
        ...
    </testcase>

.. note::
    Similar specification reference information can also be added to :ref:`test suites<test-suite-metadata-specification>`.
