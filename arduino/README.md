PubNub Arduino Library
======================

This library allows your sketches to communicate with the PubNub cloud
message passing system using an Ethernet shield. Your application can
receive and send messages.


Synopsis
========

	void setup() {
		Serial.begin(9600);
		Ethernet.begin(mac);
		PubNub.begin(pubkey, subkey);
	}

	void loop() {
		EthernetClient *client;
		/* Publish message. */
		client = PubNub.publish(pubchannel, "\"message\"");
		if (client)
			client->stop();

		/* Wait for news. */
		client = PubNub.subscribe(subchannel);
		if (!client) return; // error
		char buffer[64]; size_t buflen = 0;
		while (client->connected()) {
			while (client->connected() && !client->available()) ; // wait
			buffer[buflen++] = client->read();
		}
		buffer[buflen] = 0;
		client->stop();

		/* Print received message. You will want to look at it from
		 * your code instead. */
		Serial.println(buffer);
		delay(10000);
	}

See also included examples.

Library Reference
=================

``bool PubNub.begin(char *publish_key, char *subscribe_key, char *origin)``
---------------------------------------------------------------------------

To start using PubNub, use PubNub.begin().  This should be called after
Ethernet.begin().

Note that the string parameters are not copied; do not overwrite or free the
memory where you stored the keys! (If you are passing string literals, don't
worry about it.) Note that you should run only one of publish, subscribe and
history requests each at once.

The origin parameter is optional, defaulting to "pubsub.pubnub.com".

``EthernetClient *publish(char *channel, char *message)``
---------------------------------------------------------

Send a message (assumed to be well-formed JSON) to a given channel.

Returns NULL in case of error, instead an instance of EthernetClient
that you can use to read the reply to the publish command. If you
don't care about it, call ``client->stop()`` right away.

``PubSubClient *subscribe(char *channel)``
------------------------------------------

Listen for a message on a given channel. The function will block
and return when a message arrives. NULL is returned in case of error.
The return type is PubSubClient, but from user perspective, you can
work with it exactly like with EthernetClient.

Typically, you will run this function from loop() function to keep
listening for messages indefinitely.

As a reply, you will get a JSON array with messages, e.g.:

```
	["msg1",{msg2:"x"}]
```

and so on. Empty reply [] is also normal and your code must be
able to handle that. Note that the reply specifically does not
include the time token present in the raw reply from PubNub;
no need to worry about that.

``EthernetClient *history(char *channel, int limit)``
-----------------------------------------------------

Receive list of the last messages published on the given channel.
The limit argument is optional and defaults to 10.

Installation
============

Move the contents of the ``pubnub-api/arduino/`` directory to
``~/sketchbook/libraries/PubNub/`` and restart your Arduino IDE.
Try out the examples!

Notes
=====

* There is no SSL support on Arduino, it is unfeasible with
Arduino Uno or even Arduino Mega's computing power and memory limits.
All the traffic goes on the wire unencrypted and unsigned.

* We re-resolve the origin server IP address before each request.
This means some slow-down for intensive communication, but we rather
expect light traffic and very long-running sketches (days, months),
where refreshing the IP address is quite desirable.

* We let the users read replies at their leisure instead of
returning an already preloaded string so that (a) they can do that
in loop() code while taking care of other things as well (b) we don't
waste precious RAM by pre-allocating buffers that are never needed.

* If you are having problems connecting, maybe you have hit
a bug in Debian's version of Arduino pertaining the DNS code. Try using
an IP address as origin and/or upgrading your Arduino package.

* We assume that server replies always use Transfer-encoding: chunked;
adding auto-detection would be straightforward if that ever changes.
Adding support for multiple chunks is going to be possible, not so
trivial though if we are to shield the user application from chunked
encoding. Note that /history still uses non-chunked encoding.
