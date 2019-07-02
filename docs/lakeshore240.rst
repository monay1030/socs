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


Out of the box, the Lakeshore240 Modules do not