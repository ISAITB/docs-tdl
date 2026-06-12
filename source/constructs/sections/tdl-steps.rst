Overview
--------

TDL test steps are used to capture a test case's core testing logic. They are used in test cases and
also in :ref:`test-case-scriptlets` to define their sequence of test steps. The available test
steps are described in the sections that follow, organised in four main categories:

* **Messaging steps** used to exchange information between actors.
* **Processing steps** to perform complex arbitrary processing.
* **Flow steps** to manage the execution flow of the test case.
* **Support steps** to introduce support features to test cases.

.. index:: Messaging steps
.. _tdl-messaging-steps:

Messaging steps
---------------

Messaging steps allow the test case to handle the exchange of messages between actors. The actual implementation
allowing content to be sent or received is implemented by a messaging handler (see :ref:`introduction-concepts-messaging-handlers`).
