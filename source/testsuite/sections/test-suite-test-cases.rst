This section is used to reference the test cases contained in the test suite. One or more test case entries must be defined using the 
``testcase`` element whose structure is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @group, no, The ID of the group to which this test case belongs (see :ref:`test-suite-groups`).
    @id, yes, The ID of the test case. This needs to match one defined in the test case's XML file (see :ref:`test-case`).
    option, no, Zero or more elements each defining as text content string values that match an option defined for the actor in the specification.
    prequisite, no, Zero or more elements each defining as text content a test case ID that should be considered as a prerequisite before running this one.

The order with which the test case entries are defined is important as it defines their **execution order**. This ordering would apply when multiple test
cases are selected for execution at once, which typically occurs when launching a complete test suite as opposed to an individual test case. Executing a
complete test suite can be done in two ways:

* **Interactively**, by presenting to the tester all test cases and launching each one in sequence while interacting with the tester and updating
  the test session diagram and log.
* **In the background**, by launching all test cases as a batch, allowing also the tester to choose whether tests are executed in parallel or in
  sequence. Note that test cases that do not :ref:`support parallel execution<test-case>` can be configured as such so that they are always executed
  in isolation (for a given SUT).

.. note::
    **GITB software support:** The ``prequisite`` and ``option`` values are currently ignored.
