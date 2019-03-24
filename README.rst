Xproto Specification
####################
Xproto is a secure lightweight NAT-safe IoT protocol based on protobuf and blissful imperial intentions

********************
Rationale
********************
Xproto is designed to be:

#. Lightweight — no HTTP transport, XML payload packing or other traffic-heavy stuff. 
   We use `protobuf <https://en.wikipedia.org/wiki/Protocol_Buffers>`_ for payload
   encapsulation, which is known for `near-perfect <https://developers.google.com/protocol-buffers/docs/encoding>`_
   efficiency.
#. IoT-ready — it should fit the constrained runtimes such as MCUs, in terms of small code footprint; small RAM usage;
   low computational power (though nowadays there is no single strict definition what IoT means)
#. Secure — we use military-grade encryption algorithms on a per-session basis. Each following session key
   is derived from **all** the previous sessions. This means that even if the attacker somehow manages to find the current 
   session key (using  impossible alien supercomputers), it will be outdated in minutes, as soon as the next session 
   starts.
#. _`NAT-safe` — most of the times a device node is located behind the NAT, and could only initiate the outgoing
   connections, not accept the incoming ones. It is possible to deal with it using the longpolling mechanism,
   similar to the one used with HTTP or in push notifications. The device node regularly sends small requests to 
   server and gets new data in response.
   Stuff such as firewalls, dynamic IPs may also be addressed using the longpolling approach.
#. Transport-agnostic — any underlying transport could be used with Xproto. TCP, UDP, TLS, or even HTTP (if you like 
   *die großen* headers).
#. RPC-ready — Xproto supports the `Remote Procedure Calls <https://en.wikipedia.org/wiki/Remote_procedure_call>`_,
   which allows for seamless device-server integration based on clean concise native APIs. We have taken the best
   approaches from `JSON-RPC <https://en.wikipedia.org/wiki/JSON-RPC>`_ and rethought them to keep Xproto RPC 
   as robust as possible.
   
| There are plenty of IoT-ready protocols, however, they do not fit the specific goals the Xproto is addressing.
Below is presented the overview of the most obvious drawbacks for some of these protocols:

* `CoAP <https://en.wikipedia.org/wiki/Constrained_Application_Protocol>`_ (Constrained Application Protocol)

  #. is asynchronous by design. It means if we need the messages to arrive in some order we should implement it ourselves.
     This makes implementing a control sequence a fairly challenging task.
     *[TURN_ON, GET_TEMP, CHECK_SAFETY, TURN_REACTOR, ...]* could easily become 
     *[TURN_ON, TURN_REACTOR, GET_TEMP, CHECK_SAFETY, ...]*.
     Then the reactor turns before security checks and you got a disaster. Easy
  #. has relatively big memory footprint (``+9.5 KB`` flash / ``+1.3 KB`` RAM according to `this report`_)
  #. message format. There is no message error checking facilities provided. Instead, it relies on UDP/HTTP and the
     corresponding underlying protocols. We know that UDP runs on top of IP and IP runs on top of whatever.
     For UDP running on top of IPv4 the 16-bit CRC checksum is completely optional
     (though usually is used). And IPv4 provides only a *Header Checksum*. This means that in some (untypical, but still
     possible) configurations it could happen that UDP packets are not verified at all. Reliability?
     
     The minimum CoAP header size is 5 bytes when the payload is used. And numerous CoAP packets could not be placed into 
     a single UDP packet. This means that we eventually must carry additional 8 bytes.
     It means high efficiency for long packets and low efficiency for small. 
     When 4 bytes of data is transmitted, the efficiency would be ``4 / (4 + 5 + 8) = 24 %``.
     And the maximum CoAP payload size is limited to UDP packet size, which generally is ``~64 KB`` — probably too much
     to prove the need in 1 CoAP = 1 UDP packet usage.
  #. is tightly bound to UDP or HTTP transport. HTTP could be used as CoAP mapping which means that CoAP messages are
     represented with corresponding HTTP headers and methods. That is a cool feature, but we pretend 
     not to use HTTP as it is being too traffic-heavy.
     
     UDP means fast low-delay, but though unreliable transfers. It does not
     guarantee that the message is being received at all. CoAP tries to deal with this by providing *acknowledgements* and
     the *piggyback* mechanism, but still has nothing to do with out-of-order reception. And you can not simply replace UDP
     with TCP as it will break the standard compatibility.
  #. provides no security means by itself, instead relying on 
     `DTLS <https://en.wikipedia.org/wiki/Datagram_Transport_Layer_Security>`_ underlying protocol or HTTP security
     (when being mapped to HTTP). When DTLS is being used in conjunction, its security features are largely 
     limited to three options: PSK (Pre-Shared Key), asymmetric RPK (Raw Public Key) and X.509 (Certificate).
     Also, the usage of DTLS does not contribute to overall traffic efficiency.
     And we could not use HTTP security as it complicates the stack and enlargens the packet size
  #. ... and many more ... Seriously, enough, that is not an academic comparison of protocols. Of all the alternatives,
     CoAP is probably the best, but it still does not fulfill our needs because of its drawbacks.
* `MQTT <https://en.wikipedia.org/wiki/MQTT>`_ (Message Queuing Telemetry Transport)

  #. requires a message broker (such as `Mosquitto <https://mosquitto.org>`_) which acts as an intermediate component of
     the system.
  #. does not provide message encryption facilities. If needed, the encryption should be managed by the application itself.
     The underlying transport is TCP or TLS.
  #. has a publisher/subscriber architecture which largely limits the universal applicability. Having numerous clients
     connected to single server maps to ``[N publishers + 1 subsriber]`` for device node sends plus 
     ``[N subscribers + 1 publisher]`` for the reception, either to one single topic, or to N topics for each device.
     This approach does not seem to prove helpful dealing with the `initial requirements <#nat-safe>`_.
  #. has *(almost)* nothing to do with NAT. The client can receive messages as long as the connection is being established.
     If the connection was dropped, the client should reconnect to keep receiving the data.
* `XMPP <https://en.wikipedia.org/wiki/XMPP>`_ (Extensible Messaging and Presence Protocol) ...
* `LWM2M <https://en.wikipedia.org/wiki/OMA_LWM2M>`_ (LightWeight Machine-to-Machine)

  #. is typically implemented on top of CoAP transport, in this case inheriting all of it drawbacks
  #. has overcomplicated specification. This means that the vast majority of libs will substantially contibute to 
     the resulting application size (about ``+8.6 KB`` flash / ``+0.8 KB`` RAM according to `this report`_)
* `DDS <https://en.wikipedia.org/wiki/Data_Distribution_Service>`_ (Data Distribution Service) ...
* `AMQP <https://en.wikipedia.org/wiki/Advanced_Message_Queuing_Protocol>`_ (Advanced Message Queuing Protocol) ...

.. _this report: https://theinternetofthings.report/Resources/Whitepapers/e4d4fb3c-b9cd-4c22-b490-3c6a3bd17d30_10.pdf

Conclusion:
  Any of the previously described protocols could be used as underlying transport for Xproto (if and when needed),
  but none of them could replace it.

********************
Overview
********************
Xproto optionals and extensions
===============================


Transport
********************
...

Packet format
*******************
Inside a session packets have the following format:

+------------------------+-----------------+
| |       PAYLOAD        | CHECKSUM **\*** |
| | (encrypted protobuf) |                 |
+------------------------+-----------------+
\* |RI| uses 1-byte CRC8-Maxim checksum. Other checksums are possible via |ext|. It is possible to have no checksum in |SP| (via |ext|) given that either:
    a. error checks are performed by the underlying transport (with exception to TCP)
    b. error check is performed at the end of session, and only if the whole session dropping is allowed

.. |RI| replace:: Xproto Reference Implementation
.. |SP| replace:: Session Packet
.. |ext| replace:: `Xproto Extensions <#xproto-optionals-and-extensions>`__
