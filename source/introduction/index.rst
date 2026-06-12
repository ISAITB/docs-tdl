.. _introduction:

Introduction
============

The purpose of the current documentation is to make it possible for you to understand and create test suites
using the GITB TDL. The approach followed herein is to document the available possibilities providing easy to 
follow examples but also to note where appropriate certain considerations on the use of TDL in the GITB Test Bed
software.

.. index:: GITB
.. index:: TDL

What is the GITB TDL?
---------------------

The `GITB project`_ represents a CEN standardisation initiative funded by the European Commission’s DG GROW 
to provide the specifications for a generic interoperability Test Bed and their implementation in the form 
of the GITB Test Bed software. The focus of these specifications and software is not any kind of testing 
(e.g. performance, regression, penetration) but rather conformance and interoperability testing. Simply
put, *conformance testing* verifies that the requirements of a given specification are met, whereas
*interoperability testing* comes as a second step to verify that two or more parties can interact as expected.
Furthermore, the focus here is on technical specifications related to e.g. messaging protocols or content,
whereas the parties previously mentioned would typically be software components.

A key element of the GITB specifications is the **GITB Test Description Language** (GITB TDL) that is used to 
express how individual tests are defined in terms of involved actors and foreseen steps. The goal for GITB TDL
is to provide test descriptions that are executable, i.e. ready to be processed by a testing engine capable of
understanding GITB TDL, but also human readable. To achieve this, GITB TDL is based on XML, defining specific 
elements to match its foreseen testing constructs. 

.. _GITB project: http://www.cen.eu/work/areas/ict/ebusiness/pages/ws-gitb.aspx

Where is it used?
~~~~~~~~~~~~~~~~~

The short answer is that GITB TDL is used in the GITB Test Bed software to define and run tests.

To be more complete however, GITB TDL is not tied to the GITB software per se but can be processed by any Test Bed
implementation that conforms to the GITB specifications. In fact, the GITB specification foresees the concept
of TDL compliance and splits this in *GITB compliant TDL processors* and *GITB TDL compliant producers*. Simply
put, a TDL processor is any software that can read GITB TDL content and understand it, typically being a
Test Bed implementation that can read and execute GITB TDL test cases when requested (without necessarily conforming
to other elements of the overall GITB specification). On the other hand, a TDL producer is software that can 
output TDL content, in which case we would typically be referring to test case editors or converter tools from
other test case representations. Finally, consider that compliance to GITB TDL is not necessarily an "all-or-nothing"
statement; compliance can be partial by supporting only a limited set of constructs. To support this last point, GITB
TDL concepts and constructs are formalised using a `published taxonomy`_ that can be referenced to understand the TDL 
constructs that are used (within a test case) or supported (by a Test Bed).

One of the goals of GITB TDL is also to help reuse existing work by ideally sharing test cases between Test Bed 
implementations. To this effect the `GITB Test Registry and Repository (TRR)`_ aims to act as a reuse portal for 
GITB TDL test cases by facilitating their discovery and listing their expected and used constructs.

.. _published taxonomy: https://www.itb.ec.europa.eu/skos/gitb_tdl_skos.rdf
.. _GITB Test Registry and Repository (TRR): https://joinup.ec.europa.eu/collection/gitb-trr

.. index:: Interoperability Test Bed Action

How is it maintained?
~~~~~~~~~~~~~~~~~~~~~

Following the initial work from the CEN GITB workgroup, the maintenance and evolution of GITB TDL has been taken over 
by the European Commission, specifically the `Interoperability Test Bed Team`_ of DIGIT B.2. DIGIT B.2 foresees the
maintenance of the GITB software and specification based on the needs of the testing community and carries out evolutive
maintenance with regular releases. Regarding GITB TDL in particular, introduced changes are always done in a backwards-compatible manner, adding 
capabilities rather than changing existing ones. Releases of GITB TDL and its version numbers currently follow the 
versioning of the GITB software.

.. _Interoperability Test Bed Team: https://joinup.ec.europa.eu/solution/interoperability-test-bed/about

.. _introduction_core_concept:

Core concepts
-------------

.. include:: /introduction/sections/introduction_core_concept.rst

.. index:: Messaging handlers
.. _introduction-concepts-messaging-handlers:

Messaging handlers
~~~~~~~~~~~~~~~~~~

.. include:: /introduction/sections/introduction-concepts-messaging-handlers.rst

.. index:: Validation handlers
.. _introduction-concepts-validation-handlers:

Validation handlers
~~~~~~~~~~~~~~~~~~~

.. include:: /introduction/sections/introduction-concepts-validation-handlers.rst

.. index:: Processing handlers
.. _introduction-concepts-processing-handlers:

Processing handlers
~~~~~~~~~~~~~~~~~~~

.. include:: /introduction/sections/introduction-concepts-processing-handlers.rst
