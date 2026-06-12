The ``foreach`` step allows you to execute a sequence of steps for a specific number of iterations or to iterate over a set of items. Its structure is as follows:

.. csv-table::
    :stub-columns: 1
    :delim: |
    :header: "Name", "Required?", "Description"

    @collapsed | no | A boolean flag determining whether or not the step is displayed as initially collapsed (default is "false"). See also :ref:`tdl-steps-common-collapsed`.
    @counter | no | A name for the variable through which to expose the iteration counter (default is "i").
    @desc | no | A description for this loop to display to the user and to include in the test session log. Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.
    @end | no | A number considered as the maximum iteration count plus 1, provided as a constant or as a variable reference. If the ``of`` attribute is not present, the `end` attribute becomes mandatory.
    @hidden | no | A boolean flag determining whether or not the step is displayed to users (default is "false"). Note that within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`. See :ref:`tdl-steps-common-hidesteps` for further details.
    @item | no | The name of a variable that will hold the current iteration's item. This applies only when iterating over a collection provided through the ``of`` attribute.
    @of | no | A variable reference for a collection (map or list) to iterate over. If this is not provided, the ``end`` attribute becomes mandatory.
    @skipped | no | A boolean value or variable reference (default being "false") which will result in the step being skipped if "true". See also :ref:`tdl-steps-common-skipped`.
    @start | no | A number to initialise the zero-based iteration index with, provided as a constant or as a variable reference.
    @stopOnError | no | A boolean flag determining whether the test session should end if this step fails (default is "false"). See also :ref:`tdl-steps-common-stoponerror`.
    @title | no | A short title to display for this step (default is "loop"). Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.
    do | yes | Contains as children any sequence of steps to execute for a loop iteration.
    documentation | no | Rich text content that provides further information on the current step.

The ``foreach`` step allows you to run a specific number of iterations, as defined by the ``start`` and ``end`` values.
When used in this way, iterations will continue as long as ``start`` is less than ``end`` with ``start`` getting incremented by one at the end of each iteration.
If not specified, ``start`` is assumed to be zero.

.. code-block:: xml

    <!--
        The loop will execute 2 times (start must be less than end). The currentIndex variable will be 5 in the first
        iteration and then 6. Note that referring to this is done as a variable reference (if not specified the variable
        would be named "i" and referred to as "$i").
    -->
    <foreach desc="Do iteration" counter="currentIndex" start="5" end="7">
        <do>
            <interact desc="Message to user">
                <instruct desc="Iteration">"Iteration " || $currentIndex</instruct>
            </interact>
        </do>
    </foreach>
    <!-- In the following case the loop's boundaries are set dynamically. -->
    <assign to="start" type="number">5</assign>
    <assign to="end" type="number">$start + 2</assign>
    <foreach desc="Do iteration" counter="currentIndex" start="$start" end="$end">
        <do>
            <interact desc="Message to user">
                <instruct desc="Iteration">"Iteration " || $currentIndex</instruct>
            </interact>
        </do>
    </foreach>

Another way of using the ``foreach`` step is as an iterator over a map or a list. In this case ``of`` becomes the key attribute
to consider, that is set with the reference to the target map or list. With this approach you would typically also
define the ``item`` attribute with the name of a variable to hold the current iteration item. When iterating over a map,
the iterated items are the map's entries, each being exposed as a map with ``key`` and ``value`` entries.

.. code-block:: xml

    <!--
      Prepare a map.
    -->
    <assign to="myMap{key1}">"value1"</assign>
    <assign to="myMap{key2}">"value2"</assign>
    <assign to="myMap{key3}">"value3"</assign>
    <!--
      Iterate over the map's entries
    -->
    <foreach item="entry" of="$myMap" counter="index" desc="Iterate entries">
       <do>
          <!--
            Exposing and using the index counter is not needed, but we can use it if useful.
          -->
          <log>"In foreach of entries (" || $index || "): " || $entry{key} || ":" || $entry{value}</log>
       </do>
    </foreach>
    <!--
      Prepare a list.
    -->
    <assign to="myList" append="true">"value1"</assign>
    <assign to="myList" append="true">"value2"</assign>
    <assign to="myList" append="true">"value3"</assign>
    <!--
      Iterate over the list.
    -->
    <foreach item="item" of="$myList" desc="Iterate values">
       <do>
          <log>$item</log>
       </do>
    </foreach>

When using the ``item`` and ``of`` attributes to iterate over a map or list, you can still make use of the ``start`` and ``end`` attributes.
In this case, when ``start`` is present it defines the zero-based index to start the iteration from, whereas ``end``
defines the end index. The ``start`` and ``end`` attributes can be used together, one at a time, or altogether skipped.
