.. _actions_list:


execute
^^^^^^^^

The ``execute`` action is used to send a command to the device. Keywords `include`
and `exclude` are to be used to verify if specific string exists or do not
exists in the output. You also, have the option to check if a specific
``regex`` exists within the output of the action.

.. code-block:: YAML

    - execute: # ACTION
        # (Either device hostname or device alias)
        device: R1
        # Send show version to the device
        command: show version
        # To execute command with specified connection alias, 'cli' is default
        connection_alias: ssh
        # Can have as many items under include or exclude that you want
        include:
            - '12.9.1'
            - 'CSR1000V'
            # Regular expression can also be provided
            - '\d+'
        exclude:
            - 'Should not be in the output'


include, exclude and connection_alias keywords are optional to use.
Two things needs to be followed to use connection_alias:
    1. New mapping datafile schema.
    2. ``alias`` in mapping datafile should be the ``connection_alias``.
How to use the new schema for different connection can be found at this `link
<https://pubhub.devnetcloud.com/media/genie-docs/docs/cookbooks/harness.html#what-can-you-do-with-the-mapping-datafile>`__.

You can apply additional arguments to ``execute`` command.
List of arguments that can be applied to execute command can be found at this `link
<https://pubhub.devnetcloud.com/media/unicon/docs/user_guide/services/generic_services.html#execute>`__.
Example can be seen below.

.. code-block:: YAML

    # A timeout of 10 second is applied to execute action,
    # Now if the device has not executed the command within 10 seconds, the step will fail.
    - execute:
        command: show version
        device: PE1
        timeout: 10

configure
^^^^^^^^^

The `configure` action is used to configure the device.

.. code-block:: YAML

    - configure: # ACTION
        device: device_name
        # To execute command with specified connection alias, 'cli' is default
        connection_alias: ssh
        command: |
            router bgp 65000
            shutdown

Connection_alias keyword is optional to use. Refer the execute action for
how to use connection_alias.

You can apply additional arguments to ``configure`` command.
List of arguments for the configure command can be found at this `link
<https://pubhub.devnetcloud.com/media/unicon/docs/user_guide/services/generic_services.html#configure>`__.
Example can be seen below.

.. code-block:: YAML

    # A timeout of 10 second is applied to configure action,
    # Now if the device is not configured within 10 seconds, the step will fail.
    - configure:
        command: feature bgp
        device: PE1
        timeout: 10


configure_dual
^^^^^^^^^^^^^^^

Nxos supports dual configuration, where a commit is necessary. In these case, use this action.

.. code-block:: YAML

    - configure_dual: # ACTION
        device: device_name
        # To execute command with specified connection alias, 'cli' is default
        connection_alias: ssh
        command: |
            router bgp
            commit

Connection_alias keyword is optional to use. Refer the execute action for
how to use connection_alias.

parse
^^^^^^

The ``parse`` action use pyATS `Parsers
<https://pubhub.devnetcloud.com/media/genie-feature-browser/docs/#/parsers>`_.
The parsers return structured data in a dictionary format. It allows to verify
if certain key have an expected output, where `execute` verify that it is
somewhere in the output, irrelevant of the structure. You can use the keywords
`include` and `exclude` to *query* the output of your parser. You can learn, how
to use `include/exclude` keywords in a parse action by reading through
this `section
<#verifying-actions-output>`__.

.. code-block:: YAML

    - parse: # ACTION
        device: R2
        command: show version
        # To specify which context to use Eg: cli, yang etc. By default cli.
        context: yang
        # To configure using different connections, by default cli
        connection_alias: ssh
        # Can have as many items under include or exclude that you want
        include:
            - raw("[version][version]")
            - contains("version").value_operator('mem_size' '>=', 1217420)
              # Make sure the memory is greater than 1217420

        ...

Context and Connection_alias keywords are optional to use. Refer the execute action
for how to use connection_alias.

api
^^^^

The ``api`` action use pyATS `Api
<https://pubhub.devnetcloud.com/media/genie-feature-browser/docs/#/apis>`_.

You can learn how to query the results of the apis by taking a look at this `section
<#verifying-actions-output>`__.

.. code-block:: YAML

        - api: # ACTION
            function: get_interface_mtu_config_range
            arguments:
                device: PE1
                interface: GigabitEthernet1
            include:
                - contains('max')
                - get_values('range')
            exclude:
                - contains('min-max')
        ...

If the api is a common utils api that does not have a device as its argument, then it is not required to specify a device value for that api action.
Instead by setting the keyword ``common_api: True`` you can have access to that api. See below example.

.. code-block:: YAML

        - api: # ACTION
            function: get_devices
            common_api: True
            arguments:
                testbed: "%VARIABLES{runtime}"
        ...

tgn
^^^^

The ``tgn`` action now allows you to call `traffic generator` (tgn) apis in addition to the
other existing apis.

.. code-block:: YAML

    - tgn: # ACTION
        function: get_traffic_stream_objects
        ...

rest
^^^^

The ``rest`` action allows to make rest call to any endpoint on a device. Rest uses http method to
transfer data. Five http protocols are supported, `get`, `post`, `put`, `patch` and `delete`.

You can find additional information on rest, using this `tutorial
<https://pubhub.devnetcloud.com/media/rest-connector/docs/user_guide/services/index.html>`_.

.. code-block:: YAML

    test_sections:
        - plain_actions:
            - rest:
                method: get
                dn:  '/api/mo/sys/intf/phys-[eth1/1].json'
                device: N93_3
            - rest:
                method: delete
                device: N93_3
                dn: '/api/mo/sys/bgp/inst.json'
            - rest:
                method: put
                dn:  '/api/mo/sys/bgp/inst/dom-default/af-ipv4-mvpn.json'
                device: N93_3
                payload: {
                    "intf-items": {
                      "phys-items": {
                        "PhysIf-list": [
                          {
                            "adminSt": "down",
                            "id": "eth1/2",
                            "userCfgdFlags": "admin_layer,admin_state"
                          }
                        ]
                      }
                    }
                  }
            - rest:
                method: post
                dn:  'api/mo/sys/bgp/inst.json'
                device: N93_3
                payload: {
                  "bgpInst": {
                    "attributes": {
                      "isolate": "disabled",
                      "adminSt": "enabled",
                      "fabricSoo": "unknown:unknown:0:0",
                      "ctrl": "fastExtFallover",
                      "medDampIntvl": "0",
                      "affGrpActv": "0",
                      "disPolBatch": "disabled",
                      "flushRoutes": "disabled"
                     }
                  }
                }
            - rest:
                method: patch
                dn:  '/api/mo/sys/bgp/inst/dom-default/af-ipv4-mvpn.json'
                device: N93_3
                payload: {
                    "intf-items": {
                      "phys-items": {
                        "PhysIf-list": [
                          {
                            "adminSt": "down",
                            "id": "eth1/2",
                            "userCfgdFlags": "admin_layer,admin_state"
                          }
                        ]
                      }
                    }
                  }

sleep
^^^^^

The ``sleep`` action is used to pause the execution for a specified amount of time.

.. code-block:: YAML

    - sleep: # ACTION
        # Sleep for 5 seconds
        sleep_time: 5
        ...

learn
^^^^^^

The ``learn`` action is used to learn a feature on a specific device, returning an
OS agnostic structure.  You also can query the outcome of this action
similar to api action and parse action.

.. code-block:: YAML

    - learn:
        device: R1
        feature: bgp
        include:
            - raw("[info][instance][default][vrf][default][cluster_id]")
        ...

It is also possible to learn the entire running config of the device. Below you can see the example of it.


.. code-block:: YAML

    - learn:
        device: R1
        feature: config
        include:
            - raw("[info][instance][default][vrf][default][cluster_id]")
        ...

print
^^^^^^

``print`` action allows you to print messages, variables and actions output into the console.

.. code-block:: YAML

    - print:
        item:
          value: "%VARIABLES{parse_output}"
        another_item:
          value: "%VARIABLES{parse_output1}"
        ...

yang
^^^^^

The :ref:`yang action<yang action>` is designed to work with different messaging protocols
which include NETCONF, RESTCONF and gNMI.  Changing the connection and protocol determines
the message format.  See :ref:`yang action details<yang action>` for other configurations.

Example of configuration using NETCONF (with automated verification of edit-config on device)

.. code-block:: YAML

    - yang:
        device: uut2
        connection: netconf
        protocol: netconf
        operation: edit-config
        datastore:
          type: candidate  # empty string means type is chosen from device capabilities.
          lock: true
          retry: 40
        banner: YANG EDIT-CONFIG MESSAGE
        content:
          namespace:
            ios-l2vpn: http://cisco.com/ns/yang/Cisco-IOS-XE-l2vpn
          nodes:
          - value: 10.10.10.2
            xpath: /native/l2vpn-config/ios-l2vpn:l2vpn/ios-l2vpn:router-id
            edit-op: merge

bash_console
^^^^^^^^^^^^^

Using this action, now you can run various bash command on the device. You can save output of each command, and apply include/exclude
verification on the output of each command. Below example shows how to use bash_console action.

.. code-block:: YAML

    - verify_config:
          - bash_console:
              device: csr1000v-1
              target: standby
              timeout: 45
              save:
                - variable_name: second_cmd
                  filter: contains('ls')
                - variable_name: everything
              commands:
                - pwd
                - ls
                - |
                  cd ~
                  echo A string of text
              include:
                  - contains('ls')

configure_replace
^^^^^^^^^^^^^^^^^^^^

The ``configure_replace`` action is used to replace the running-config. Users only needs
to provide the location of the saved configuration.

.. code-block:: YAML

    - configure_replace:
        device: my_device
        config: bootflash:/golden_config

        # Iteration and interval is used for a retry mechanism
        iteration: <int> #optional, default is 2
        interval: <int> #optional, default is 30

save_config_snapshot
^^^^^^^^^^^^^^^^^^^^

The ``save_config_snapshot`` action is used to save a snapshot of the current
device configuration. The config can later be used with the
``restore_config_snapshot`` action.

.. code-block:: YAML

    - save_config_snapshot:
        device: my_device

restore_config_snapshot
^^^^^^^^^^^^^^^^^^^^^^^

The ``restore_config_snapshot`` action is used to restore a snapshot taken
from the ``save_config_snapshot`` action. If you want to re-use the same
snapshot you can specify to not delete it. See `example` below.

.. code-block:: YAML

    - restore_config_snapshot:
        device: my_device
        delete_snapshot: False #optional, default is True

run_genie_sdk
^^^^^^^^^^^^^^^

The ``run_genie_sdk`` action is used to run other triggers from within
*Blitz*. All you have to do is to mention the trigger name and its arguments
in your *Blitz* datafile.

.. note::

    You must extend the main trigger_datafile for any of those triggers
    to be accessible. Put this at the top of your trigger_datafile:
    `extends: "%CALLABLE{genie.libs.sdk.genie_yamls.datafile(trigger)}"`

.. code-block:: YAML

    - run_genie_sdk:
        <trigger_name>:
            <any trigger arguments>

        # An example of running TriggerSleep
        TriggerSleep:
            devices: [my_device]

diff
^^^^^

Allow to diff two variables (Dictionary or Ops object).

By default it will just print the difference, but can also fail the section
if they are different with the argument `fail_different=True`.

``command`` or ``feature`` to diff will gather pre-defined exclude list from
the parser or Ops.

``mode`` can be specified only what you want to check. ``mode`` has ``add``,
``remote`` and ``modified``. By default, it will show all the differences,
for the case ``add``, will show only added difference.

.. code-block:: YAML

        - snapshot_pre_configuration:
           - parse:
               device: R3_nx
               command: show interface
               save:
                 - variable_name: pre_snapshot_nxos

        - configure_interface:
            # List of actions
            - configure:
                device: R3_nx
                command: |
                  interface Ethernet1/56
                  no switchport
                  ip address 10.5.5.5 255.255.255.0
                  no shutdown

            - parse:
                device: R3_nx
                command: show interface
                save:
                  - variable_name: post_snapshot_nxos

            - diff:
                pre: "%VARIABLES{pre_snapshot_nxos}"
                post: "%VARIABLES{post_snapshot_nxos}"
                device: R3_nx
                command: show interface
                mode: modified

Example with ``feature``.

.. code-block:: YAML

            - diff:
                pre: "%VARIABLES{pre_interface_ops}"
                post: "%VARIABLES{post_interface_ops}"
                device: R3_nx
                feature: interface
                mode: add

.. note::

    Please find more detail for ``diff`` from below document.
    `Diff <https://pubhub.devnetcloud.com/media/genie-docs/docs/userguide/utils/index.html#diff>`_

compare
^^^^^^^^^

Action ``compare`` allows you to verify the values of the saved variables. Below example shows how you can actually use this action.

.. code-block:: YAML

    # assume you already saved values in the variable bios, os, date_created and bootflash
    - compare:
        items:
        - "'%VARIABLES{os}' == 'NX-OS' and '%VARIABLES{date_created}' == '10/22/2019 10:00:00 [10/22/2019 16:57:31]'"
        - " %VARIABLES{bootflash} >= 290000 or '%VARIABLES{bios}' == '07.33'"

.. note::

    Please note that if each comparison statement provided to compare would fail. The actions results would be set to Failed. 

dialog
^^^^^^

Action ``dialog`` allows you to create a list of sequences to handle multiple interactions within a transaction. 
The following items must be declared in the action dialog:

 * ``device`` the name of the device.
 * ``start`` and ``end`` represent states a device can be in. 
 * ``sequence`` is a list of steps that can be executed to interact with the interface.

In turn, each step in sequences must be declared with the following keywords: 

 * ``step_msg`` message to display when the step is executed.
 * ``action`` represents the action to be executed during a step.
 * ``expect`` is a pattern to check against the output of the interaction.

The example below shows how you can use this action.

.. code-block:: YAML

    - dialog:
        device: switch
        start: show version
        end: "end"
        sequence:
            - step_msg: Wait for prompt
              expect: switch#
            - step_msg: Send new action
              action: sendline(show version)
              expect: switch#
              exclude: NXOS

``include`` or ``exclude`` are optional keywords you can use to verify if a pattern exists in the output.
In the example above, the test will pass if the pattern "NXOS" is not present in the output.
If ``include`` was declared instead of ``exclude``, the test would fail if the output contained "NXOS".