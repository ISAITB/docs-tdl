**Messaging handlers** are embedded or external components whose purpose is to allow 
messaging for actors, defining how these receive and send messages. How this actually takes place can be
completely arbitrary and is not tied to a specific protocol. What is important is that the Test Bed can signal to the handler that
it needs to send data or similarly signal that it is waiting to receive data. In this latter case, the handler notifies the 
Test Bed by means of a call-back with the relevant received payload.

To enable interaction with the Test Bed, messaging handlers implement the `GITB messaging service API`_. This interface defines the 
methods needed to signal events between the Test Bed and the handler and provide relevant input and output. Implementations can 
either be embedded Test Bed components or external web service endpoints. Input to a handler is provided using the test session's
configuration and context whereas the output of messaging is recorded in the session context for subsequent use.

Having the actual implementation of sending and receiving decoupled in dedicated components means that the Test Bed can be extended 
to handle virtually any needed protocol, requiring only that an appropriate handler exists. If for example the Test Bed needs to send 
and receive email, it does not need to support e.g. SMTP natively, it just needs access to a handler that can send and receive emails on its behalf.

.. note::
    **Messaging handlers as adapters:** Messaging handlers are used when messaging to and from simulated actors but can also
    act as adapters over existing software, translating GITB-specific calls to the actual processes that carry out messaging.

.. _GITB messaging service API: https://www.itb.ec.europa.eu/specs/latest/gitb_ms.wsdl
