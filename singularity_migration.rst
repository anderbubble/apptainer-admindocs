.. _singularity_migration:

############################
 Migrating From Singularity
############################

Since the community decided to `move the project into the Linux Foundation
<https://apptainer.org/news/community-announcement-20211130>`_ with the
constraint of a name change to the project, it has been a goal of the
project to minimize the impact to the user base. If you experience issues making
the move, please reach out to the `community <https://apptainer.org/help>`_ so
we can help you!


When migrating to {Project} from Singularity, administrators that have
modified the system configurations of their installation and want {Project}
to have identical configuration will need to migrate the configurations they
have in place for Singularity to {Project}.
An exception is an RPM update from an already-installed singularity package;
in that case, the RPM will automatically import the old configuration,
but an administrator should verify the conversion.

All singularity system configuration files are typically located under
``/etc/singularity`` or ``/usr/local/etc/singularity``
when installed from a RPM/Deb package or source respectively,
but options passed during source installations could cause this to
be in another location.
{Project} will store its configurations in similar locations, but within
a directory named ``{command}`` instead of ``singularity``.

.. note::

    The ``singularity`` configuration directory at the prefix corresponding to
    your ``{command}`` configuration directory (e.g. ``/etc/singularity`` or
    ``/usr/local/etc/singularity``) needs to be manually removed to prevent
    {Project} from producing a message at runtime about the cleanup being
    incomplete.

All system configuration names, file formats, and parameters for {Project} are
identical to their Singularity counterparts with the exception of
``singularity.conf``, which has been renamed to ``{command}.conf`` as a part of
the project renaming.  It is important to note that comments within 
``{command}.conf`` have been changed and the default contents of
``remote.yaml`` has changed.
So, while you can copy files around, we **recommend** applying the same
configuration changes to the new files instead of simply copying
contents.

.. warning::

    Take care to not wipe out all configuration in the {Project} config
    directory as a part of your migration because it will remove configuration
    files that are new to the project like configurations for
    :ref:`checkpointing <dmtcp_configuration>`.

If you are migrating from an installation with default configuration,
there is no need to perform any configuration migration.

However, a big change from Singularity is that {Project} does not
install a setuid-root component by default.  That means that either
user namespaces needs to be enabled or the setuid-root component needs
to be installed separately.  See the 
:ref:`User Namespaces & Fakeroot <userns>` section,
or to find out about how to install the setuid-root component see the
:ref:`Installation <installation>` section.
An exception is with RPM packaging when updating from the
``singularity`` package: that case installs the ``{command}-suid``
package by default, but since setuid-root is a higher security risk,
consider removing that package after upgrade.

Also see the user guide documentation about `Singularity compatibility
<{userdocs}/singularity_compatibility.html>`__ for information about how the
migration to {Project} will impact users.
In particular the system administrator may want to 
:ref:`restore pre-Apptainer library behavior <no_default_remote>`
for all users.

.. _log_plugin_migration:

****************************************
 Migrating the SingularityCE log plugin
****************************************

{Project} provides a port of the SingularityCE log plugin (see
:ref:`Log plugin config.yml <log_plugin>`). Its configuration and
template files work unchanged, but some details differ:

.. list-table::
   :header-rows: 1

   * - Item
     - SingularityCE
     - {Project}
   * - Syslog tag
     - The invoked command name, e.g. ``singularity[pid]`` or
       ``/usr/bin/singularity[pid]``
     - ``{command}[pid]`` by default; set per output with ``tag``
   * - Syslog facility
     - ``kern`` (none set)
     - ``user`` by default; set per output with ``facility``
   * - Plugin name
     - ``sylabs.io/log-plugin``
     - ``{command}.org/log-plugin``
   * - Package name
     - ``singularity-ce-log-plugin``
     - ``{command}-log-plugin``
   * - Configuration directory
     - ``/etc/singularity/plugins/sylabs.io/log-plugin/``
     - ``/etc/{command}/plugins/{command}.org/log-plugin/``
   * - ``.Executable``
     - ``/usr/bin/singularity``
     - ``/usr/bin/{command}``
   * - ``.Version``
     - ``4.x``
     - The {Project} version
   * - ``.Flags`` names
     - SingularityCE command line
     - {Project} command line (mostly the same; ``--oci`` and other
       SingularityCE-only flags are absent)
   * - Environment variables
     - ``SINGULARITY_*``
     - ``{ENVPREFIX}_*``; ``SINGULARITY_*`` are still accepted, so
       templates that log ``.Env.SINGULARITY_*`` should also log the
       ``{ENVPREFIX}_*`` names

To migrate:

#. Copy ``config.yml`` and the ``*.tpl`` files from
   ``/etc/singularity/plugins/sylabs.io/log-plugin/`` to
   ``/etc/{command}/plugins/{command}.org/log-plugin/``.

#. Update log parsing and SIEM rules for the tag, facility,
   executable and version changes listed above. To keep rules that
   match on the tag, set ``tag: singularity`` on each output. Rules
   that select these messages by the ``kern`` facility need to change
   to the configured facility.

#. Check any template that logs ``.Flags`` against the {Project} flag
   names.
