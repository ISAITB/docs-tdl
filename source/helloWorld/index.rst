.. _helloWorld:

Hello world
===========

The first step in understanding a new language or framework is typically going through a simple **hello world** example. This
remains as simple as possible while still being valid and showcasing some fundamental concepts in action.

In the GITD TDL, you express the scenarios to test as :ref:`test cases <test-case>`, with each one defining the steps to 
execute. These :ref:`test steps <tdl-steps>` can vary greatly from one test case to another, ranging from user interactions,
to message exchanges between systems, with verifications along the way to check that each step is successfully passed.

To allow test cases to be executed, they are bundled in a **test suite archive**, a ZIP archive basically, which
includes also a :ref:`test suite <test-suite>` definition. Besides things like names and descriptions, this test suite
also lists the contained test cases as well as the specification actors (consider them as roles) that are considered.

In the sections that follow we will consider a **GITB TDL hello world example**, by going over its :ref:`test case <helloWorld_testCase>`,
:ref:`test suite <helloWorld_testSuite>`, and :ref:`packaging <helloWorld_packaging>` before running on the Test Bed.

.. _helloWorld_testCase:

The hello world test case
-------------------------

.. include:: /helloWorld/sections/helloWorld_testCase.rst

.. _helloWorld_testSuite:

The test suite definition
-------------------------

.. include:: /helloWorld/sections/helloWorld_testSuite.rst

.. _helloWorld_packaging:

Packaging the test suite
------------------------

.. include:: /helloWorld/sections/helloWorld_packaging.rst
