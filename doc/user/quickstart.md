# Quickstart


The module allows you to `publish` observation messages and `retract` them if needed. 
You can also `subscribe` to _alert topics_. The default configuration file using the snews-test kafka servers and a 
_TEST_ detector is stored in [auxiliary/test-config.env](https://github.com/SNEWS2/SNEWS_Publishing_Tools/blob/main/SNEWS_PT/auxiliary/test-config.env)
please **change the configuration** to your needs before using the tools. Once set, `snews_pt` can fetch from this file at each execution. 
Alternatively, custom made configuration file can also be passed to functions.   


Below, we present a quick start for the Supernova Early Warning System communication tools [Python API](#Python-Api) using both [Jupyter Notebooks](#snews-on-jupyter-notebooks), and also the [command line interface](#command-line-interface)(CLI).



**Table of Contents**
1. [Python API](#python-api)
    1. [Publish](#how-to-publish)
    2. [Subscribe](#how-to-subscribe)
    
    
2. [Command Line Interface-(CLI)](#command-line-interface-(cli))
    1. [Subscribe](#subscribing-to-alert-topics)
    2. [Message Schema](#message-schema)
    3. [Publish](#publishing-observation-messages)
    4. [Publish Heartbeat](#publishing-heartbeat-messages)
    7. [Retraction](#retraction-ressages)


## Python-Api
### How to Publish
Keep in mind that currently the topic set under `/auxiliary/test_config.env` is set to a test kafka server. 
This can be changed, new environment file can be passed to tools.

First you need to import Publisher and  your desired Observation class:

````Python
# Import the Publisher class
from SNEWS_PT.snews_pub import Publisher
# Import the constructor for Coincidence Tier
from SNEWS_PT.snews_pub import CoincidenceTier
# Import the constructor for Significance Tier
from SNEWS_PT.snews_pub import SignificanceTier
# Import the constructor for Timing Tier
from SNEWS_PT.snews_pub import TimingTier
````

First let's make a dummy nu time method (optional)

    Note: datetime object will be used to create a dummy nu times

```Python
from datetime import datetime

def nu_t():
    return datetime.utcnow().strftime("%H:%M:%S:%f")
```

Let's define the name of our detector.
```Python
my_detector = 'DS-20K'
```


Finally, to send a message you need initialize the Publisher, construct your message, and send it to Publisher.
```Python
with Publisher() as pub:
    message = CoincidenceTier(detector_name=my_detector, neutrino_time=nu_t(), p_value = 0.98).message()
    pub.send(message)
```
> Notice that each tier accepts a predefined set of keys. See message schema.
> 
See also this [examples notebook](https://github.com/SNEWS2/SNEWS_Publishing_Tools/blob/main/examples.ipynb) for more examples.

### How to Subscribe

In two lines, one can subscribe to the alert topic specified in the default configuration. <br>
This starts a stream, and waits for alert messages to be received.
```python
from SNEWS_PT.snews_sub import Subscriber

Subscriber().subscribe()
```

Should there be an alert message, this will be both displayed on the screen and saved into your local machine as `SNEWS_MSGs/<today fmt="%y_%m_%d">/subscribed_messages.json` and if there are multiple messages in the same day e.g. hype-mode is on and for the same supernova you kept receiving alerts with every coincidence message, these will be appended in this file with the sent time as the first key. An example (partly missing) can be found [here](https://github.com/SNEWS2/SNEWS_Publishing_Tools/blob/main/doc/subscribed_messages.json)


## Command Line Interface (CLI)

It is also possible to interact with `snews_pt` through the command line. <br>
All the commands have their short descriptions accessible via `--help` flag. 
```bash
(venv) User$: snews_pt --help 
```
```bash
Usage: snews_pt [OPTIONS] COMMAND [ARGS]...
  User interface for snews_pt tools

Options:
  --version   Show the version and exit.
  --env TEXT  environment file containing the configurations  [default: (auxiliary/test-config.env)]
  --help      Show this message and exit.

Commands:
  heartbeat       Publish heartbeat messages.
  message-schema  Display the message format for `tier`, default 'all'
  publish         Publish a message using snews_pub
  retract         Retract N latest message
  subscribe       Subscribe to Alert topic 
```
The main command `snews_pt` serves an entry point. It is also possible to set an _environment_ by passing it to this with any other command. 
E.g. `snews_pt --env myenvfile.env subscribe` will set the variables in _myenvfile.env_  and subscribe to the _ALERT_TOPIC_ specified in this file. <br>
By default, it uses the environment file that comes with the package.

---
### Subscribing to Alert Topics
The subscription command can be called without any arguments.
```bash 
(venv) User$: snews_pt subscribe 
```
```bash
> You are subscribing to ALERT 
> Broker:kafka://kafka.scimma.org/snews.alert-test
```
---
### Message Schema
`snews_pt message-schema` can tell you the required contents for each tiers. You can display the contents of a single tier by calling e.g.
```bash
(venv) User$: snews_pt message-schema time
```
In which case it displays the following
```bash
         >The Message Schema for TimeTier 
_id                 :(SNEWS SETS)
detector_name       :(SNEWS SETS)
sent_time           :(SNEWS SETS)
machine_time        :(User Input)
neutrino_time       :(User Input)
timing_series       :(User Input)  
```
or you can simply call `snews_pt message-schema` without any positional arguments in which case it displays all the message schemes. <br>

---

### Publishing Observation Messages
User can publish observation messages to one of the 'CoincidenceTier', 'TimeTier', or 'SigTier'. It is also possible to publish _Heartbeat_ and _Retraction_ messages, see respective section below.

To publish one or more tier user can request arbitrary number of tiers in one line
```bash
(venv) User$: snews_pt publish coincidence time time significance s
```
The `publish` tool takes all the request and queries known aliases e.g. `s` is accepted as `SignificanceTier`, and returns a list of unique requested tiers.<br>
Without any additional _options_ this by default publishes a dummy observation to each of the requested tiers with their required data fields.

For a more realistic case, we would want to submit a message that is saved by our experiment as a _json_ file. This can be passed with a `--file` (or `-f`) flag.
```bash
(venv) User$: snews_pt publish coincidence -f my_coincidence_message.json
```
There are several dummy examples [here](../test/) that can be used as a reference. In principle, SNEWS only accept specific fields (see `snews_pt message-schema`), however the tools does not fail if you provide additional arguments. It kindly warns you about them and publishes the remaining parts.

Try publishing the following file which contains an `extra_key` field.
```bash
(venv) User$: snews_pt publish coincidence -f SNEWS_PT/test/test_coincidence_tier_message.json
```

It should give the following
```bash
Requested tiers are;
                > CoincidenceTier
Publishing to CoincidenceTier;
extra_key not a valid key for CoincidenceTier
---------------------------------------------------------
_id                 :0_CoincidenceTier_22/01/01_20:19:06:356690
detector_name       :TEST
sent_time           :22/01/01 20:19:06
machine_time        :test machine time
neutrino_time       :test nu time
p_value             :test p-values 
```
----

### Publishing Heartbeat messages

`snews_pt heartbeat`  can be used to publish heartbeat messages. It is up to the user to invoke this function with a desired frequency, however it is recommended to publish heartbeats consistently and with couple of minutes intervals.

The `heartbeat` command takes a `status` argument which can either be 'ON' or 'OFF'. 
Additionaly, `machine_time` can be passed using `--machine_time` (`-mt`) flag as a string. Each heartbeat message is appended with a `sent_time`, if machine time is also provided, the latency can be tracked.

```bash
(venv) User$: snews_pt heartbeat ON -mt '22/01/01 20:19:06' --verbose False
```
---

### Retraction Messages

It can happen that user publishes a message by accident or with wrong input. In these cases `snews_pt` allows for retraction messages. <br>
While the specific message id can be passed, it is also possible to publish a retraction message for the last `n` number of messages. This

```bash
(venv) User$: snews_pt retract --tier Coinc -n 3 --reason 'daq failure' 
```
