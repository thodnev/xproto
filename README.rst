Xproto Specification
####################
Xproto is a secure lightweight NAT-safe IoT protocol based on protobuf and blissful imperial intentions

********************
Rationale
********************
Xproto is designed to be:

#. Lightweight — **NO** HTTP transport, XML payload packing or other traffic-heavy stuff. 
   We use `protobuf <https://en.wikipedia.org/wiki/Protocol_Buffers>`_ for payload
   encapsulation, which is known for `near-perfect <https://developers.google.com/protocol-buffers/docs/encoding>`_
   efficiency.
#. IoT-ready — it should fit the constained runtimes such as MCUs, in terms of: small code footprint; small RAM usage;
   low computational power (though nowadays there is no single strict definition what IoT means)
#. Secure — ...
#. NAT-safe — most of times a device node is located behind the NAT, and could only initiate the outgoing
   connections, not accept the incoming ones. ..longpolling, beaconing.. 
   (stuff such as firewalls, dynamic IPs is also adressed using this approach)

| There are plenty of IoT-ready protocols, however they do not fit the specifig goals the Xproto is addressing.
Below is presented the overview of the most obvious drawbacks for some of these protocols:

* `CoAP <https://en.wikipedia.org/wiki/Constrained_Application_Protocol>`_ (Constrained Application Protocol) ...
* `MQTT <https://en.wikipedia.org/wiki/MQTT>`_ (Message Queuing Telemetry Transport) requires a message broker...
  (such as `Mosquitto <https://mosquitto.org>`_) ...
* `XMPP <https://en.wikipedia.org/wiki/XMPP>`_ (Extensible Messaging and Presence Protocol) ...
* `LWM2M <https://en.wikipedia.org/wiki/OMA_LWM2M>`_ (LightWeight Machine-to-Machine) ...
* `DDS <https://en.wikipedia.org/wiki/Data_Distribution_Service>`_ (Data Distribution Service) ...
* `AMQP <https://en.wikipedia.org/wiki/Advanced_Message_Queuing_Protocol>`_ (Advanced Message Queuing Protocol) ...

********************
Overview
********************
Transport
********************
...

Packet format
*******************
...
