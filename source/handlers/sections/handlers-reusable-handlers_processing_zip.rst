.. note::
  The built-in :ref:`ZipProcessor <handlers-ZipProcessor>` allows ZIP archive processing without needing an external service.

The ZIP processing service allows you to extract files from received ZIP archives, or ZIP-like archives such as ASiC containers. Using this service you can:

* Obtain the **table of contents** of a provided archive.
* **Extract** one or more files from the archive based on provided search criteria.

This component functions as a **stateful processing service**, with extraction operations carried out within the scope of a
:ref:`processing transaction<tdl-step-bptxn>`. This allows the ZIP archive to be provided once to the service and then maintained as state across calls to
efficiently carry out several extraction operations. The archive in question is removed once the processing transaction or the overall test session ends.
You can use this component:

* **Locally**, by pulling the `isaitb/zip-processing <https://hub.docker.com/r/isaitb/zip-processing>`_ Docker image.
* **As a service**, by setting your handler to ``https://www.itb.ec.europa.eu/zip/api/processing?wsdl``.

The operations supported by the service are listed in the following table:

.. csv-table::
    :header: "Operation", "Description", "Input(s)", "Output(s)"
    :delim: ~

    ``extract``~ Extract one or more files from the archive. ~ Yes ~ A ``map`` containing three entries (``matched``, a ``boolean`` representing if matches were made; ``entries``, a ``number`` representing the count of entries that were matched; ``entry``, a ``list`` with one item per matched entry). Each item in the ``entry`` list (corresponding to a matched entry) is a ``map`` with two further fields (``path``, a ``string`` with the file's precise path; ``content``, the ``binary`` content of the file).
    ``initialize`` ~ Provide the ZIP archive to the service for subsequent extraction operations. ~ Yes ~ A ``map`` with two elements (``entries``, a ``number`` representing the count of entries included in the archive; ``entryPaths``, a ``string`` including a summary of the included paths, listing them one by one in square brackets).

The input parameters expected by the different operations are as follows:

.. csv-table::
    :header: "Operation", "Input name", "Required?", "Description"
    :delim: ~

    ``extract`` ~ ``case`` ~ No ~ A ``string`` set as "true" or "false" (the default) specifying whether the path matching should be case sensitive.
    ``extract`` ~ ``match`` ~ No ~ A ``string`` set as "exact" or "regexp" (the default) specifying whether the path should be considered for an exact match or as a regular expression.
    ``extract`` ~ ``path`` ~ Yes ~ A ``string`` with the path of the archive's entry (or entries) to return.
    ``initialize`` ~ ``zip`` ~ Yes ~ A ``binary`` input corresponding to the archive to process.

The following test case sample illustrates how to use the service to extract a file from a ZIP archive:

.. code-block:: xml

    <steps>
       <!--
          As a first step create processing transaction pointing to the service
       -->
       <bptxn txnId="t1" handler="https://www.itb.ec.europa.eu/zip/api/processing?wsdl"/>
       <!--
          Call the 'initialize' operation to pass the binary archiveContent as an input named 'zip'
       -->
       <process id="toc" txnId="t1" operation="initialize">
          <input name="zip">$archiveContent</input>
       </process>
       <!--
          Call the 'extract' operation to retrieve a file with an exact but not case-sensitive match
       -->
       <process output="zip" txnId="t1" operation="extract">
          <input name="path">'META-INF/manifest.xml'</input>
          <input name="match">'exact'</input>
          <input name="case">'false'</input>
       </process>
       <!--
          Use if needed the number of returned entries
       -->
       <log>"Extracted " || $zip{entries} || " file(s)"</log>
       <if hidden="true">
          <cond>$zip{matched}</cond>
          <then>
            <!--
              Use the extracted file (first match)
            -->
            <log>"Processing file " || $zip{entry}{0}{path} || "..."</log>
            <assign to="file">$zip{entry}{0}{content}</assign>
          </then>
       </if>
       <assign to="file">$zip{entry}{0}{content}</assign>
       <!--
          Close the processing transaction to release the processed archive.
       -->
       <eptxn txnId="t1"/>
    </steps>