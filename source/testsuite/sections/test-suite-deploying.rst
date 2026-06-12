A test suite is packaged as a **compressed ZIP archive** that contains:

* A single test suite file defining common information and test cases.
* One or more :ref:`test case files <test-case>` for the scenarios to cover (unless this is a :ref:`shared resource holder<test-suite-sharing-empty>`).
* Zero or more :ref:`scriptlet files <scriptlets>` with common testing blocks reused across test cases.
* Any number of arbitrary files used as resources within test cases.
* Documentation files referred to by the test suite, test cases and test steps.

The names of the archive, the test suite and the test case files are not important, neither is the archive's folder structure.
Nonetheless, the following structure represents a **best practice** that achieves clarity and good organisation:

.. code-block:: none

  <archive root>
  ├── docs
  │   └── <HTML files containing documentation for the test suite and its test cases>
  ├── resources
  │   └── <Arbitrary files (nested as you please) used as imports in test cases and scriptlets>
  ├── scriptlets
  │   └── <Scriptlet files (reusable test step blocks) to use across test cases>
  ├── testCases
  │   └── <Test case files>
  └── testSuite.xml

Notice how the archive **does not include a top-level folder**. Considering that resource references are relative to the
archive root, having a root folder would repeat in all path references. In terms of file naming the advised approach is to use
**Camel Case** (e.g. "testCase1.xml" as opposed to "test-case-1.xml" or "test_case_1.xml"), at least for the files under
your control (as opposed to e.g. sets of schema files that you copy into the test suite).

When the test suite is deployed to GITB Test Bed software is will be validated in depth to ensure its overall correctness.
Deploying a test suite can be achieved in two ways:

* Using the `user interface <https://www.itb.ec.europa.eu/docs/itb-ta/latest/domainDashboard/index.html#upload-test-suite>`_.
* Using the `REST API <https://www.itb.ec.europa.eu/docs/itb-ta/latest/api/index.html#deploy>`_ (if enabled).

.. note::

    **Deploying test suites during development:** When deploying test suites to a development Test Bed instance, the fastest way
    is to use the Test Bed's `REST API <https://www.itb.ec.europa.eu/docs/itb-ta/latest/api/index.html#deploy>`_. You can find
    instructions `here <https://www.itb.ec.europa.eu/docs/guides/latest/developingComplexTests/index.html#step-3-prepare-your-workspace>`_
    on enabling the API and creating a script to automate its use.

Deploying a test suite has the following results:

* If it doesn't previously exist, the test suite is recorded along with its test cases and linked to the appropriate specification actors.
* If the test suite does exist, the user selects whether this new version should invalidate previous conformance testing sessions (if the
  change is significant) or not. Choosing to replace the test suite results in the test suite being updated and its test cases being replaced
  with the ones contained in the new version. Matching of the test suite with an existing one is on the basis of their ID within the specification.
* The actors that are defined in the test suite are created if they don't already exist along with their endpoints and endpoint parameters.
* If the user chooses to, actors that already exist in the specification are updated based on the latest provided information. In this case new endpoints and parameters are added
  and existing ones are updated. Note that actors, endpoints and parameters that are not defined in the new test suite are not removed. The matching of
  actors is on the basis of their ID, whereas for endpoints and parameters their name is used.
* If the test suite does not include any test cases it is marked as hidden (See :ref:`test-suite-sharing-empty`).

As previously discussed, the :ref:`test-suite-actors` section serves to define which actors are used within the test suite and to provide their details (their name, endpoints
and endpoint parameters). An alternative approach to avoid defining the complete actor details in the test suite is to simply refer to the actors used
in its test cases without providing their information. Referring to actors is on the basis of their ID and referred actors are assumed and required to be present in the
target specification (resulting in an upload error otherwise).

The following example shows a test suite in which a "User" actor is referred to.

.. code-block:: xml
    :emphasize-lines: 7,8,9

    <?xml version="1.0" encoding="UTF-8"?>
    <testsuite id="UBL_invoice_validation" xmlns="http://www.gitb.com/tdl/v1/" xmlns:gitb="http://www.gitb.com/core/v1/" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.gitb.com/tdl/v1/ ../gitb_tdl.xsd">
        <metadata>
            <gitb:name>UBL_invoice_validation</gitb:name>
            <gitb:version>0.2</gitb:version>
        </metadata>
        <actors>
            <gitb:actor id="User"/>
        </actors>	
        <testcase id="UBL_invoice_validation_test_1"/>
        <testcase id="UBL_invoice_validation_test_2"/>
    </testsuite>

Specifying a test suite's actors in this way could be interesting if you want to manage their information fully through the GITB software's
user interface. The only thing to ensure is that the specification's actors are already defined before uploading test suites and that their 
IDs are correctly referenced.
