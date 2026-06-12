Up to now we have seen the definitions for our :ref:`test case <helloWorld_testCase>` and :ref:`test suite <helloWorld_testSuite>`.
Before we can run the test case we will need to create the test suite archive to deploy to the Test Bed. The resulting archive's
structure will be as follows:

.. code-block:: none

  <archive root>
  ├── testCases
  │   └── testCase1.xml
  └── testSuite.xml

Note that the placement of the ``testSuite.xml`` and ``testCase1.xml`` files does not have to follow this structure. They can be placed
anywhere in the ZIP archive, however the approach above matches the :ref:`proposed best practice <test-suite-deploying>`.

.. note::

    The hello world test suite is also available as part of the published `sample test suites <https://github.com/ISAITB/sample-test-suites>`__
    on GitHub. Click to `view the test suite <https://github.com/ISAITB/sample-test-suites/tree/master/testSuites/helloWorld>`__ or
    `download its archive <https://github.com/ISAITB/sample-test-suites/raw/refs/heads/master/testSuites/helloWorld/testSuite.zip>`__.

With the test suite archive ready you can proceed to :ref:`deploy and use it on the Test Bed <test-suite-deploying>`.
