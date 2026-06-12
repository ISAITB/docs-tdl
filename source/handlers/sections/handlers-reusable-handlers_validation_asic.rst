The ASiC validation service allows you to validate `ASiC containers <https://en.wikipedia.org/wiki/Associated_Signature_Containers>`_. The validator
currently supports two validation profiles you can consider in your tests:

* **base**: The base ASiC container definition (considered as the default).
* **etendering**: The rules relevant to the `PEPPOL Business Interoperability Specifications (BIS) <https://peppol.org/learn-more/peppol-interoperability-framework>`_.

You can use this component:

* **Locally**, by pulling the `isaitb/asic-validator <https://hub.docker.com/r/isaitb/asic-validator>`_ Docker image.
* **As a service**, by setting your handler to ``https://www.itb.ec.europa.eu/asic/api/validation?wsdl``.

When used in a :ref:`verify<tdl-step-verify>` step this validator expects the following inputs:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: ~

    ``file`` ~ Yes ~ A ``binary`` input corresponding to the container to validate.
    ``profile`` ~ No ~ A ``string`` identifying the profile to consider that can be "base" (the default) or "etendering".

The following test case sample illustrates how to use the validator to verify an ASiC container using the default profile:

.. code-block:: xml

    <steps>
       <verify id="asicValidation" desc="Validate ASiC container" handler="https://www.itb.ec.europa.eu/asic/api/validation?wsdl">
          <!-- The binary container to validate -->
          <input name="file">$container</name>
          <!-- An optional profile to apply. -->
          <input name="profile">"base"</name>
       </verify>
    </steps>