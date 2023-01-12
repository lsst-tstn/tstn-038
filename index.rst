:tocdepth: 1

.. sectnum::

Abstract
========

This documents describes the sequence of events that led to the network failure during the 202212A run in auxtel.

Description of the Incident
===========================

This incident initiated between 8:00-8:30 CLT/3:00-3:30PST December, 6 2022, during the first night of the 2022A run on the Auxiliary Telescope.
We experienced a catastrophic network failure that took down the network in the entire observatory.
The failure extended to both the internal and external network, both over the WiFi and the cable connections.
All WiFi connections were dropped and we could no longer connect to the network.

Up until the issue started we were preparing to go on sky with the Auxiliary Telescope.
The TMA was experiencing power issues and would not be able to operate during the night for the soak and start tracker testing. 
Alysha Shugart and Tiago Ribeiro were both in the telescope console at the summit executing some dome movement tests with the remote help of Bruno Quint.

At one point around 8:30 PM local time, Tiago Ribeiro went to update LOVE in the love02 node with the latest version of the logging system for testing during the night.
The love02 node is a "development" node, normally used to test unreleased features at the summit, while keeping the production deployment running (on love01).
At that time the LOVE system was completely shutdown on love02, no LOVE system or ospl daemon running.
Shortly after initializing the ospl daemon on love02, the network outage started.
We were able to communicate with La Serena over Slack using a poor cellular connection and after being able to raise IT support it was established that it was impossible to recover from the issue remotely.

At that point the crew decided to call in the night.
The Auxiliary Telescope dome was manually closed and the telescope was left in the position it was in.
The mirror covers were closed and all equipment was left in a safe state.

The next morning IT arrived at the summit and determined that the issue was caused because two hosts were broadcasting using the same IP, which created an internal loop, causing the ACI to completely shutdown the network.
They were able to determined that the IP responsible for the incident was 139.229.170.30, from the main ospl daemon, running on azar01, and the ospl daemon that was started on love02, initiating the episode.

Mitigation Strategies
=====================

In summary, the key factors behind this incident are:

*   CISCO ACI network.

*   Brittle manual configuration required by DDS + docker.

    The current docker compose configuration, used to deploy the systems in the standalone nodes (e.g. those systems not managed by kubernetes), exposes a bridge network to the container.
    Unfortunately this configuration is not capable of reaching out to the network name server to request an IP number, requiring IP addresses to be manually hard-coded in the configuration file.
    As we mentioned above, it was this manual setup that triggered the incident.

In terms of mitigation strategies, it is worth noting that the CISCO ACI is already scheduled to be deprecated in early 2023.

It is also worth noting that we are planning to retire DDS in favor of Kafka in the first quarter of 2023.
Kafka does not use multicast traffic and requires a much simpler setup, with no need for manual IP configurations or daemons on each node.

Altogether, the two main risk factors involved in this incident are already planned to be deprecated in a short time frame.

Nevertheless, an additional mitigation could be performed in case the Kafka adoption is delayed.
In some nodes, we have successfully deployed the ospl daemon using the host network.
The migration of using the host network in all the azar and love nodes would require some help from IT in setting up the correct network interface.
This would eliminate the need to manually specify the network address, removing the possibility of further collisions in the future.
Given the current timeline for adoption of Kafka, we are not sure this task is worth the effort.

.. Make in-text citations with: :cite:`bibkey`.
.. Uncomment to use citations
.. .. rubric:: References
.. 
.. .. bibliography:: local.bib lsstbib/books.bib lsstbib/lsst.bib lsstbib/lsst-dm.bib lsstbib/refs.bib lsstbib/refs_ads.bib
..    :style: lsst_aa
