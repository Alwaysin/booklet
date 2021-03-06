# Programming for Hosted

We've [shown you how to start with Python](program.md), but if you're trying to access our Hosted platform from within Python you'll need a bit more:

* We use TLS only
* We require authentication
* Note also, that we require you use clientIDs which tell us who you are (e.g. as below: `username-device-xxx`). We do not (yet) enforce adherence that that but may well in future.

The following small program accounts for that. You will need:

* The `ca-bundle.pem` file [which you can download here](https://www.startssl.com/certs/ca-bundle.pem).
* A _Hosted_ _username_, _device_ name, and _token_ which you export into the environment before invoking the program. The variables used are called `OWNTRACKS_USERNAME`, `OWNTRACKS_DEVICE`, and `OWNTRACKS_TOKEN` respectively. 

```python
#!/usr/bin/env python
# Connect to OwnTracks Hosted with TLS

import paho.mqtt.client as mqtt
import ssl
import os
import sys

# Download from https://www.startssl.com/certs/ca-bundle.pem
PEM = 'ca-bundle.pem'

# The callback for when the client successfully connects to the broker
def on_connect(client, userdata, rc):
    ''' We subscribe on_connect() so that if we lose the connection
        and reconnect, subscriptions will be renewed. '''

    client.subscribe("owntracks/+/+")

# The callback for when a PUBLISH message is received from the broker
def on_message(client, userdata, msg):

    topic = msg.topic
    payload = msg.payload

    print topic, payload

# The callback for when Mosquitto logs something

def on_log(mosq, userdata, level, string):
    print(string)


username = os.getenv("OWNTRACKS_USERNAME")
device = os.getenv("OWNTRACKS_DEVICE")
token = os.getenv("OWNTRACKS_TOKEN")

if username is None or device is None or token is None:
    print "I need $OWNTRACKS_USERNAME, $OWNTRACKS_DEVICE, and $OWNTRACKS_TOKEN"
    sys.exit(1)

clientid = "%s-%s-py-%d" % (username, device, os.getpid())
client = mqtt.Client(clientid)
client.on_connect = on_connect
client.on_message = on_message
# client.on_log = on_log    # uncomment to see logging

client.username_pw_set("%s|%s" % (username, device), token)
client.tls_set(PEM, cert_reqs=ssl.CERT_REQUIRED)
client.connect("hosted-mqtt.owntracks.org", 8883, 60)

# Blocking call which processes all network traffic and dispatches
# callbacks (see on_*() above). It also handles reconnecting.

client.loop_forever()
```

In other words:

1. Use TLS with the appropriate CA certificate file.
2. The hostname is `hosted-mqtt.owntracks.org`.
3. The TCP port number is `8883`
4. The MQTT username for authentication is your _Hosted_ user name and your _Hosted_ device name concatenated together, separated by a vertical bar (`|`).
5. The MQTT password is your _Hosted_ device _token_.
