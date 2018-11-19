# Mental Calculation Quiz

Largely inspired by the [times tables quiz](https://github.com/snipsco/snips-skill-times-tables-quiz), this skill poses simple 
arithmetic questions to test basic addition, subtraction, multiplication, and division.

The implementation of this skill is the focus for the workshop given
at LauzHack.

## Important TODOs when creating your own action

- Your action scripts must begin with `action-*`.
- Create a `requirements.txt` and `setup.sh` file as the ones in this repo.
- Make your action script and `setup.sh` executable, i.e. from the 
Terminal (for MacOS and Linux) run the following commands:
```bash
chmod +x action-*.py 
chmod +x setup.sh
```

## Important info when coding your own action

Essentially all scripts should start off like so (with the additonal
libraries that you requires for your application):
```python
#!/usr/bin/env python2
from hermes_python.hermes import Hermes    

MQTT_IP_ADDR = "localhost"
MQTT_PORT = 1883
MQTT_ADDR = "{}:{}".format(MQTT_IP_ADDR, str(MQTT_PORT))
```

It is good practice to create global variables for your different intents,
as is done in this skill:
```python
INTENT_START = "start_mental_calculation"
INTENT_ANSWER = "give_mental_calculation"
INTENT_STOP = "stop_lesson"
INTENT_DOES_NOT_KNOW = "does_not_know_calculation"
```

Typically each intent will have its own function/callback that should be
triggered when the intent is identified. Our action code to "subscribe" 
for the various intents and declara what function/callback should be used.
This should be done when the `action-*.py` script is called as is done in
this example (at the bottom of the script):
```python
with Hermes(MQTT_ADDR) as h:

    h.subscribe_intent(INTENT_START, user_request_quiz) \
        .subscribe_intent(INTENT_STOP, user_quits) \
        .subscribe_intent(INTENT_DOES_NOT_KNOW, user_does_not_know) \
        .subscribe_intent(INTENT_ANSWER, user_gives_answer) \
        .start()
```

Each callback will receive two arguments:

- `hermes`: this object will allow us to continue/terminate interactions with the user.
- `intent_message`: this object will contain useful info, e.g. slot entries, in order for
our application code to respond appropriately.

In general, an intent's function/callback should perform the following:

1. Parse `intent_message` to perform the appropriate action.
2. Either continue `hermes.publish_continue_session` or end `hermes.publish_end_session`
the current session. Examples of both can be seen in the action code for this skill.

## Debugging tips
