Test cases can leverage externally provided configuration parameters as part of the test session scope. Doing so 
adds a level of customisation and parameterisation that can accommodate various external input at different levels.
Such input is typically provided by testers but can also be supplied by administrators or externally scripted
tooling.

GITB TDL, as implemented in the GITB Test Bed software, offers four levels of such configuration, with a 
progressively narrower scope to address all potential needs:

.. figure:: images/configuration_scopes.png
  :align: center

The following table provides a summary overview of the available configuration levels.

.. csv-table::
    :header: "Level", "Description", "Provided by", "Map variable name", "Predefined map entries"
    :stub-columns: 1
    :delim: |

    :ref:`Domain<test-case-expressions-domain>`| Relates to a complete domain and applies to any and all test cases. Such values are typically treated as high-level constant configuration values to ensure portable test cases. | Administrators.| ``DOMAIN`` | 
    :ref:`Organisation<test-case-expressions-organisation>`| Relates to an organisation as a whole and applies to all its systems and their conformance statements. | Users and administrators.| ``ORGANISATION`` | ``shortName``, ``fullName``
    :ref:`System<test-case-expressions-system>`| Relates to a system as a whole and applies to all test cases defined for it in its linked conformance statements. | Users and administrators.| ``SYSTEM`` | ``shortName``, ``fullName``, ``version``
    :ref:`Actor<test-case-expressions-actor>`| Relates to a specific conformance statement, i.e. a specific system testing as an actor of a selected specification. This is most fine-grained level of configuration.| Users and administrators. |  | 

The type of value defined by each parameter, regardless of its level, can be:

    * Simple texts as ``string`` values.
    * Secret texts as ``string`` values (not communicated to users or client systems).
    * Files as ``binary`` values. 

.. note::
    **Administrator-provided values:** Configuration parameters can also be set to be editable only by administrators
    by defining them as "admin only". In the case of actor-level configuration this is done via the ``adminOnly`` attribute 
    (see :ref:`here<test-suite-actors>` for details).
