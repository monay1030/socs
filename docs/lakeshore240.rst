.. highlight:: rst

.. _lakeshore240:

=============
Lakeshore 240
=============

.. argparse::
    :filename: ../agents/lakeshore240/LS240_agent.py
    :func: make_parser
    :prog: python3 LS240_agent.py


    These options can be included in the site-config entry for the agent,
    or can be specified manually from the command line.

LS240 OCS agent
----------------

The following tasks are registered for the LS240 agent.

.. autoclass:: agents.lakeshore240.LS240_agent.LS240_Agent
    :members: init_lakeshore, set_values, upload_cal_curve, acq


The Lakeshore 240 Agent can (and probably should) be configured to run in a
Docker container. An example configuration is::

  ocs-LSA24MA:
    image: grumpy.physics.yale.edu/ocs-lakeshore240-agent:latest
    depends_on:
      - "sisock-crossbar"
    devices:
      - "/dev/LSA24MA:/dev/LSA24MA"
    hostname: nuc-docker
    volumes:
      - ${OCS_CONFIG_DIR}:/config:ro
    command:
      - "--instance-id=LSA24MA"
      - "--site-hub=ws://sisock-crossbar:8001/ws"
      - "--site-http=http://sisock-crossbar:8001/call"

The serial number will need to be updated in your configuration. The hostname
should also match your configured host in your OCS configuration file. The
site-hub and site-http need to point to your crossbar server, as described in
the OCS documentation.

Out of the box, the Lakeshore 240 channels are not enabled or configured
to correctly measure thermometers. To enable, you can use the ``set_values`` task
of the LS240 agent to configure a particular channel. Below is an example of a
client script that uses the ``ocs.matched_client`` functionality to
set channel 1 of a lakeshore module to read a diode::

    from ocs.matched_client import MatchedClient

    ls_client = MatchedClient("LSA24MA", args=[])

    diode_params = {
        'sensor': 1,
        'autorange': 1,
        'units': 3,
        'enabled': 1,
    }

    ls_client.set_values.start(channel=1, name="CHWP_01", **diode_params)
    ls_client.set_values.wait()


Lakeshore240 Simulator
-------------------------

.. argparse::
    :filename: ../socs/simulators/ls240_simulator.py
    :func: make_parser
    :prog: python3 ls240_simulator



The Lakeshore240 Simulator is a tool that you can use to emulate a Lakeshore 240
in order to test and debug agent functionality if you don't have a real device.
It opens a socket port and interprets commands and queries in a similar manner
to the real device. Not all lakeshore 240 commands actually do something currently,
but you can set and read channel variables (even though they don't change anything)
and read data from channels, which will currently return white noise centered around
zero.

Running ``python3 ls240_simulator.py`` will start the simulator and it will
wait for a connection. To connect to it, you can use the same Lakeshore240.py
module that is used to connect to the real device, but by providing
``port=`tcp::/<address>:<port>'`` instead of the device port.
For instance, if you run ``python3 ls240_simulator.py -p 1000``, you can connect
by providing the Lakeshore240 module with ``port="tcp://localhost:1000"``.
You can specify the port for a LS240 agent in the site-file or through the command line.
Talking to the simulator from an agent inside a docker container hasn't yet been
tried.
