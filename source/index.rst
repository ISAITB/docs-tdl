.. _contents:

GITB TDL Documentation
======================

Welcome to the documentation of the **GITB Test Description Language (TDL)**. The GITB TDL is used to define test cases to
realise a specification's conformance testing needs. This documentation provides a reference for all concepts and constructs
involved in the definition of test cases, as well as code snippets and guidelines to use during test development.

Chapter overview
----------------

.. csv-table::
   :class: full-width   
   :header: "Chapter", "Description"
   :delim: |

   :doc:`introduction/index` | Introducing the GITB TDL and its key concepts.
   :doc:`testsuite/index` | Defining test suite descriptors and packaging your test suite archives.
   :doc:`testcase/index` | Defining the executable test cases to drive your tests.
   :doc:`scriptlets/index` | Reusing and customising common sets of test steps across test cases.
   :doc:`types/index` | Understanding the type system used by variables to manage test session state.
   :doc:`expressions/index` | Defining expressions, accessing configuration, and using built-in metadata.
   :doc:`constructs/index` | Defining the test steps to implement test cases and common step concepts.
   :doc:`handlers/index` | Using the built-in test step implementations and custom extensions.
   :doc:`examples/index` | Exploring common scenarios through complete example test suites.
   :doc:`changelog/index` | Listing the highlights of the latest GITB TDL release and changelog.

.. toctree::
   :maxdepth: 1
   :caption: Table of contents
   :hidden:
   :includehidden:

   introduction/index
   testsuite/index
   testcase/index
   scriptlets/index
   types/index
   expressions/index
   constructs/index
   handlers/index
   examples/index
   changelog/index
   genindex

.. note::
   Click **Ask AI** to get interactive help from our **AI assistant** based on the official documentation.

Related documentation
---------------------

Looking for other documentation related to the Interoperability Test Bed? Follow the links below:

.. |case1| raw:: html

   <a href="https://www.itb.ec.europa.eu/docs/services/latest/" target="_blank"><img src="https://www.itb.ec.europa.eu/files/docs-static/images/itb_gitb_services_documentation.png" rel="noopener noreferrer" alt="GITB services documentation"/></a>

.. |case2| raw:: html

   <a href="https://www.itb.ec.europa.eu/docs/guides/latest/" target="_blank"><img src="https://www.itb.ec.europa.eu/files/docs-static/images/itb_guides.png" rel="noopener noreferrer" alt="ITB guides"/></a>

.. |case3| raw:: html

   <a href="https://www.itb.ec.europa.eu/docs/itb-ta/latest/" target="_blank"><img src="https://www.itb.ec.europa.eu/files/docs-static/images/itb_guide_ta.png" rel="noopener noreferrer" alt="User guide (Test Bed administrator)"/></a>

.. csv-table::
    :widths: 33,33,33
    :delim: ~
    :class: image-caption-table

    |case1| ~ |case2| ~ |case3|
    Documentation for the GITB services, used to extend test cases' validation, processing and messaging capabilities. ~ Guides, tutorials and focused documentation on specific topics concerning Test Bed setup, configuration and use. ~ User guide for the Test Bed administrator covering the management and configuration of the GITB Test Bed software.
