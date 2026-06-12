The ``ZipProcessor`` is a built-in processing handler allowing you to process ZIP (or ZIP-like) archives, to read and
extract their contents. It supports the following operations:

.. csv-table::
    :header: "Operation", "Description", "Input(s)", "Output(s)"
    :delim: |

    ``extract`` | Extract one or more entries from the archive (loaded through the ``initialize`` operation). | Yes | Yes, the requested entries.
    ``initialize`` | Load a ZIP archive and return information on its entries. | Yes | Yes, the map listing the archive's entries.

Using the ``ZipProcessor`` should always be done in the context of a :ref:`processing transaction <tdl-step-bptxn>`. Doing
so allows you to load an archive once and extract content without needing to reload it every time.

.. _handlers-ZipProcessor_extract:

ZipProcessor - extract
^^^^^^^^^^^^^^^^^^^^^^

The ``extract`` operation is used to extract one or more entries from a ZIP archive. It is expected to be called within
a :ref:`processing transaction <tdl-step-bptxn>`, after the :ref:`initialize operation <handlers-ZipProcessor_initialize>`
has been called to load the archive in question. The inputs expected by this operation are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``case`` | No | A boolean flag determining whether path matching should be case sensitive. The default is false.
    ``match`` | No | A string value determining how the provided ``path`` is matched with archive entries. This can be ``exact`` or ``regexp`` (the default), the latter considering it as a regular expression that can match multiple entries.
    ``path`` | Yes | The path for the entry to extract. This can be provided as a string or as a list of strings in case we want to extract multiple entries.

Once the operation completes, it returns a map with the following entries:

.. csv-table::
    :header: "Output name", "Always present?", "Type", "Description"
    :delim: |

    ``entries`` | Yes | ``number`` | The number of entries that were matched and returned.
    ``matched`` | Yes | ``boolean`` | Whether or not any entries were matched.
    ``entry`` | No | ``list[map]`` | A list of the matched entries that can be missing if none were matched. Each item of the list is a map, with keys ``path`` (for the entry path in the archive provided as a string value) and ``content`` (for the entry's data provided as a binary value).

The following example illustrates the operation's use:

.. code-block:: xml

    <!--
      Create a processing transaction.
    -->
    <bptxn txnId="t1" handler="ZipProcessor"/>
    <!--
      Load the archive.
    -->
    <process desc="Initialise archive" txnId="t1">
        <operation>initialize</operation>
        <input name="zip">$archive</input>
    </process>
    <!--
      Prepare the paths to match. If this was a single path it could also have
      been provided as a string (inline or as a variable reference).
      Note how the first path is provided as a regular expression meaning
      that multiple entries may be matched.
    -->
    <assign to="pathsToMatch" append="true">"resources/invoice/.+"</assign>
    <assign to="pathsToMatch" append="true">"resources/summary/notes.txt"</assign>
    <!--
      Extract the entries.
    -->
    <process desc="Extract entries" txnId="t1" hidden="false" output="data">
        <operation>extract</operation>
        <input name="path">$pathsToMatch</input>
        <!--
          The"regexp" (regular expression) match type could have been omitted
          as it is the default.
        -->
        <input name="match">"regexp"</input>
        <!--
          Case-insensitive matching. Could also have been skipped as 'false' is
          the default.
        -->
        <input name="case">false()</input>
    </process>
    <!--
      Log the entries that were retrieved.
    -->
    <log>"Matched " || $data{entries} || " entries"</log>
    <!--
      If we have matches, iterate over them.
    -->
    <assign to="skipLoop">not($data{matched})</assign>
    <foreach skipped="$skipLoop" item="entry" of="$data{entry}">
        <do>
            <!--
              Log the matched entry's path. The extracted content is available
              as $entry{content}.
            -->
            <log>$entry{path}</log>
        </do>
    </foreach>
    <!--
      Close the transaction when done.
    -->
    <eptxn txnId="t1"/>

.. _handlers-ZipProcessor_initialize:

ZipProcessor - initialize
^^^^^^^^^^^^^^^^^^^^^^^^^

The ``initialize`` operation is used to load a ZIP archive for further processing and return a description of its
entries. It is expected to be called within a :ref:`processing transaction <tdl-step-bptxn>`, potentially followed
by subsequent ``extract`` operations. The inputs expected by this operation are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``zip`` | Yes | A binary variable holding the archive to load.

Once the operation completes, it returns a map with the following entries:

.. csv-table::
    :header: "Output name", "Always present?", "Type", "Description"
    :delim: |

    ``entries`` | Yes | ``number`` | The number of entries included in the archive.
    ``entryPaths`` | Yes | ``list[string]`` | A list with the entry paths included in the archive.

The following example illustrates the operation's use:

.. code-block:: xml

    <!--
      Create a processing transaction.
    -->
    <bptxn txnId="t1" handler="ZipProcessor"/>
    <!--
      Load the archive.
    -->
    <process desc="Initialise archive" txnId="t1" output="archiveInfo">
        <operation>initialize</operation>
        <input name="zip">$archive</input>
    </process>
    <!--
      Log the entries that were retrieved.
    -->
    <log>"Archive contains " || $archiveInfo{entries} || " entries"</log>
    <foreach item="entry" of="$archiveInfo{entryPaths}">
        <do>
            <!--
              Log the entry path.
            -->
            <log>$entry</log>
        </do>
    </foreach>
    <!--
      Close the transaction when done.
    -->
    <eptxn txnId="t1"/>