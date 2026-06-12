.. index:: Test suites
.. _test-suite:

Test suites
===========

Overview
--------

The purpose of a test suite is to group test cases into a cohesive set and define the actors
that its test cases involve. In addition, test suites introduce metadata such as a version
number and a description to facilitate their identification and management.

The following is an example test suite that defines a single actor and two included test cases:

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <testsuite id="invoiceValidation" xmlns="http://www.gitb.com/tdl/v1/" xmlns:gitb="http://www.gitb.com/core/v1/">
        <metadata>
            <gitb:name>UBL invoice validation</gitb:name>
            <gitb:version>1.0</gitb:version>
            <gitb:description>A test suite to validate UBL invoices uploaded by a user</gitb:description>
        </metadata>
        <actors>
            <gitb:actor id="User">
                <gitb:name>User</gitb:name>
                <gitb:desc>User to upload a UBL invoice for validation</gitb:desc>
            </gitb:actor>
        </actors>
        <testcase id="testCase1"/>
        <testcase id="testCase2"/>
    </testsuite>

A test suite is defined as the XML file's root element ``testsuite``. The following table defines its attributes and child elements:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @id, yes, A string to uniquely identify the test suite by.
    @order, no, A number used to establish the relative ordering of this test suite compared to other test suites.
    actors, no, The list of actors that relate to the test suite's test cases. If not defined the test suite is assumed to be used only for :ref:`resource sharing<test-suite-sharing>`.
    groups, no, A list of groups to be referenced by test cases. Only a single successful test case is needed from within a group to consider the entire group as successful (see :ref:`test-suite-groups` for details).
    metadata, yes, A block containing the metadata used to describe the test suite.
    testcase, no, A set of one or more test cases that are included in the test suite. If not defined the test suite is assumed to be used only for :ref:`resource sharing<test-suite-sharing>`.

Regarding the test suite's ``order`` attribute, this is set with an integer (negative, zero or positive) defining its relative
order compared to other test suites, and defaulting to zero if unspecified. This is not considered as an absolute value
because test suites may be associated to multiple specifications, either as copies or as shared test suites. Ordering is
established once all applicable test suites are loaded, comparing first their ``order`` value, followed by their name.

Test suite ordering affects the test suite's display in screens and reports, as well as their execution order when
multiple test suites are executed in one batch. You could choose to manage ordering in a fine grained manner, setting
it for each test suite, or use a convention to manage exceptions. For example, you could use a convention whereby test
suites that are to always be displayed first are set with a negative order, leaving others to be naturally sorted based
on their name.

Elements
--------

Here we will see how a test suite breaks down into its individual sections and discuss the purpose of each. 

.. index:: metadata (Test suite)
.. index:: name (Test suite metadata)
.. index:: type (Test suite metadata)
.. index:: version (Test suite metadata)
.. index:: authors (Test suite metadata)
.. index:: description (Test suite metadata)
.. index:: published (Test suite metadata)
.. index:: lastModified (Test suite metadata)
.. index:: documentation (Test suite metadata)
.. index:: update (Test suite metadata)
.. _test-suite-metadata:

Metadata
~~~~~~~~

.. include:: /testsuite/sections/test-suite-metadata.rst

.. index:: actors (Test suite)
.. index:: id (Test suite actors)
.. index:: default (Test suite actors)
.. index:: hidden (Test suite actors)
.. index:: displayOrder (Test suite actors)
.. index:: name (Test suite actors)
.. index:: desc (Test suite actors)
.. index:: endpoint (Test suite actors)
.. _test-suite-actors:

Actors
~~~~~~

.. include:: /testsuite/sections/test-suite-actors.rst

.. index:: groups (Test case groups)
.. index:: group (Test case groups)
.. index:: id (Test case groups)
.. index:: name (Test case groups)
.. index:: desc (Test case groups)
.. _test-suite-groups:

Test case groups
~~~~~~~~~~~~~~~~

.. include:: /testsuite/sections/test-suite-groups.rst

.. index:: testcase (Test suite)
.. index:: id (Test suite testcase)
.. index:: group (Test suite testcase)
.. index:: prequisite (Test suite testcase)
.. index:: option (Test suite testcase)
.. index:: supportsParallelExecution (test case order configuration in test suite)

.. _test-suite-test-cases:

Test cases
~~~~~~~~~~

.. include:: /testsuite/sections/test-suite-test-cases.rst

.. _test-suite-deploying:

Test suite organisation and deployment
--------------------------------------

.. include:: /testsuite/sections/test-suite-deploying.rst

.. _test-suite-sharing:

Sharing resources across test suites
------------------------------------

.. include:: /testsuite/sections/test-suite-sharing.rst

.. _test-suite-sharing-empty:

Test suites as shared resource holders
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. include:: /testsuite/sections/test-suite-sharing-empty.rst
