===================================
swupdate: API for external programs
===================================

Overview
========

swupdate contains an integrated webserver to allow remote updating.
However, which protocols are involved during an update is project
specific and differs significantly. Some projects can decide
to use FTP to load an image from an external server, or using
even a proprietary protocol.
The integrated webserver uses this interface.

swupdate has a simple interface to let external programs
to communicate with the installer. Clients can start an upgrade
and stream an image to the installer, querying then for the status
and the final result. The API is at the moment very simple, but it can
easy be extended in the future if new use cases will arise.

API Description
---------------

The communication runs via UDS (Unix Domain Socket). The socket is created
at the startup by swupdate in /tmp/sockinstdata.

The exchanged packets are described in network_ipc.h

::

	typedef struct {
		int magic;
		int type;
		msgdata data;
	} ipc_message;


Where the fields have the meaning:

- magic : a magic number as simple proof of the packet
- type : one of REQ_INSTALL, ACK, NACK, GET_STATUS
- msgdata : a buffer used by the client to send the image
  or by swupdate to report back notifications and status.

The client sends a REQ_INSTALL packet and waits for an answer.
swupdate sends back ACK or NACK, if for example an update is already in progress.

After the ACK, the client sends the whole image as a stream. swupdate
expectes that all bytes after the ACK are part of the image to be installed.
swupdate recognizes the size of the image from the CPIO header.
Any error lets swupdate to leave the update state, and further packets
will be ignored until a new REQ_INSTALL will be received.

.. image:: images/API.png