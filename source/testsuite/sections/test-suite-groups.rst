Test case groups allow defining a set of test cases, of which only a single successful result is needed to consider the entire group
as successful. In such a case, any failures or incomplete test cases are ignored for the overall conformance status. The group is considered
to be failed if there is at least one failure and no successful tests. Note that in terms of display and execution sequence, all test cases
in a group are considered sequentially.

To use test case groups two steps are needed:

* Define the groups as part of the test suite using the ``groups`` element.
* Reference the groups when listing test cases using the ``testcase`` element(s).

The optional ``groups`` element, used to define the groups you will refer to, contains one or more ``group`` elements for each of the
test case groups. The structure of each ``group`` is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @id, yes, The ID of the group. This must be unique and is used to refer to the group from its test cases (see :ref:`test-case`).
    desc, no, A description for the group that will be presented as additional information when viewing the group's details.
    name, no, A short name for the group to be presented as a label when listing the group's test cases.

The following example shows the definition of three groups, the first one with only an identifier, the second one with a name, and the third one with a
name and a description. These groups are then referred to by their respective test cases.

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <testsuite id="invoiceValidation" xmlns="http://www.gitb.com/tdl/v1/" xmlns:gitb="http://www.gitb.com/core/v1/">
        <metadata>
            ...
        </metadata>
        <actors>
            ...
        </actors>
        <groups>
            <group id="group1"/>
            <group id="group2">
                <name>Security tests</name>
            </group>
            <group id="group3">
                <name>Connection options</name>
                <desc>Test cases covering alternative connection options</desc>
            </group>
        </groups>
        <testcase id="testCase1a" group="group1"/>
        <testcase id="testCase1b" group="group1"/>
        <testcase id="testCase1c" group="group1"/>
        <testcase id="testCase2"/>
        <testcase id="testCase3a" group="group2"/>
        <testcase id="testCase3b" group="group2"/>
        <testcase id="testCase2"/>
        <testcase id="testCase4a" group="group3"/>
        <testcase id="testCase4b" group="group3"/>
        <testcase id="testCase4c" group="group3"/>
        <testcase id="testCase5"/>
        <testcase id="testCase6"/>
    </testsuite>
