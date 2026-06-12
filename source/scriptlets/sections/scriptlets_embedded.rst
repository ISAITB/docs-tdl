Scriptlets are typically defined as separate XML documents within a test suite that can be used across its test cases or
externally from other test suites. It is also possible however, to define scriptlets within a specific test case in its
:ref:`scriptlets<test-case-scriptlets>` element. Such embedded scriptlets are processed in the same way but have certain
key differences:

* **Private access:** Embedded scriptlets are only usable from the test case that contains them and from its other embedded
  scriptlets.
* **Identification:** Embedded scriptlets rely on their ``id`` attribute to be identified and referenced in :ref:`call<tdl-step-call>`
  steps. As such, this must be unique within the scope of their containing test case.

For more details on defining embedded scriptlets check the documentation of the test case's :ref:`scriptlets<test-case-scriptlets>`
element. For information on how a scriptlet, embedded or not, is called check the documentation of the :ref:`call<tdl-step-call>`
step.
