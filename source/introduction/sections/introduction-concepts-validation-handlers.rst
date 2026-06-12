**Validation handlers** are similar in concept to messaging handlers but much simpler. Their purpose is limited to validating content and 
returning a report with the result. Validation is a fully decoupled process in that the content being validated, as well as any other parameters, 
are provided by the Test Bed as input without needing to be aware of how validation actually takes place. The result of a validation handler is 
recorded in the test session context for subsequent use in the test case.

Validation handlers implement the `GITB validation service API`_ that defines the operations needed by the Test Bed to request
validation for specific input and receive the results. Implementations can be either embedded components or external web service endpoints.

.. _GITB validation service API: https://www.itb.ec.europa.eu/specs/latest/gitb_vs.wsdl
