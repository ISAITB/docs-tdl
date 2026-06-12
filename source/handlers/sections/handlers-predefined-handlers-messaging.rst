.. note::

    **Custom vs built-in messaging handlers:** Built-in messaging handlers offer simple, out-of-the-box support for sending and receiving content
    to and from external systems. However, for complex messaging or testing needs you may be better suited by a :ref:`custom implementation<handlers-custom-handlers>`
    that would allow you to:

    * Use **non-HTTP** based protocols.
    * Manage **security aspects** including authentication and use of client certificates.
    * Produce **customised step reports** rather than a listing of request and response data.
    * Manage **large payloads** not supported directly by the test engine.
    * For :ref:`receive <tdl-step-receive>` steps, control fully **exposed APIs** and dynamically **adapt responses** based on received request data.

    To start developing a custom messaging service you can use the `published template <https://www.itb.ec.europa.eu/docs/services/latest/templates/index.html>`_
    and follow the `step-by-step development guide <https://www.itb.ec.europa.eu/docs/guides/latest/developingComplexTests/index.html>`_.

The title of each section corresponds to the name of the handler that needs to be configured in the relevant step's ``handler`` attribute (in
:ref:`send <tdl-step-send>`, :ref:`receive <tdl-step-receive>` or :ref:`btxn <tdl-step-btxn>` steps).
