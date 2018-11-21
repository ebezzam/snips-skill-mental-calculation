# Mental Calculation Quiz Workshop

Largely inspired by the [times tables quiz](https://github.com/snipsco/snips-skill-times-tables-quiz), this skill poses simple 
arithmetic questions to test basic addition, subtraction, multiplication, and division.

The implementation of this App was developed for a workshop given at LauzHack Days. In the workshop,
we complete [`INCOMPLETE_action-mental-calculation.py`](INCOMPLETE_action-mental-calculation.py) in order to obtain the
working code in [`action-mental-calculation.py`](action-mental-calculation.py).

This action code will hopefully serve as a useful reference for others to create their own Apps! We first describe how you
can deploy this App to a Raspberry Pi. Further below we describe some key ingredients when writing your own action code.

## Installing an Assistant and Apps

Before installing your App, you need to create an Assistant. An Assistant is (typically) a collection of Apps, but it 
can also consist of a single App, as is done in this workshop. 

For installing an Assistant, it is recommended to use [SAM](https://snips.gitbook.io/getting-started/installation) so that you can avoid SSH'ing
into your board frequently and copying files back and forth.


#### Installing the Assistant for this workshop

The assistant for this workshop can be downloaded [here](https://drive.google.com/open?id=1QAw0ORDti716hzjnm_hTqcuNUDVxVW_s).
The steps for creating this assistant can be found on [these slides](https://drive.google.com/open?id=12ocdhbtRjezviWVz_5yW6eiNuWgcr7GI7AYnykSi290).
Whether or not you have SAM, you can find instructions for deploying the assistant on the Snips [official documentation](https://snips.gitbook.io/documentation/console/deploy-your-assistant).
For completeness, we provide the steps for installing the assistant _if you do not have SAM_:

1. Go to where the ZIP file was downloaded and copy it to the Raspberry Pi (change `<pi-name>` with the name of your
Raspberry Pi):
    ```bash
    $ scp assistant_proj_o8AM7O4ormk.zip pi@<pi-name>.local:~
    ```
2. SSH into your board. Default password is `raspberry`:
    ```bash
    $ ssh pi@<pi-name>.local
    ```
    **From now on, the specified commands should be executed on the Raspberry Pi.**
3. Delete the previous assistant if there is one.
    ```bash
    $ sudo rm -rf /usr/share/snips/assistant/
    ```
4. Unzip the new Assistant in the following folder:
    ```bash
    $ sudo unzip assistant_proj_o8AM7O4ormk.zip -d /usr/share/snips/
    ```
5. The Assistant is now installed. Relaunch the various Snips components with the following command:
    ```bash
    $ sudo systemctl restart 'snips-*'
    ```
    
#### Installing the Action code for our App

Finally, we need to add the action code for our Mental Calculations App in the Assistant, in particular for the our Mental Calculation App. 
[Here](https://snips.gitbook.io/documentation/console/deploying-your-skills) you can find steps for deploying 
your Apps with or without SAM. If you don't have SAM, you will need two Snips program (`snips-template` and 
`snips-skill-server`) and `virtualenv` on your board as described [here](https://snips.gitbook.io/documentation/console/deploying-your-skills#prerequisite).


For the purpose of this workshop, the instruction below for installing the App assume you don't have SAM (but meet the 
above prerequisites). This workflow is presented as we have found it convenient when developing and debugging your 
application code.

1. Create a remote repository (like this one) with your application code.
2. SSH onto the board, e.g.
    ```bash
    $ ssh pi@<pi-name>.local
    ```
3. Navigate to the following directory.
    ```bash
    $ cd /var/lib/snips/skills
    ```
4. Clone the repository to this folder.
    ```bash
    $ git clone https://github.com/ebezzam/snips-skill-mental-calculation.git
    ```
4. Enter the repository and setup the virtual environment:
    ```bash
    $ cd snips-skill-mental-calculation
    $ ./setup.sh
    ```
5. Restart `snips-skill-server` to launch the new App:
    ```bash
    $ sudo systemctl restart snips-skill-server
    ```
6. Run `snips-watch -vvv` and test out the App!


#### Debugging

If you see the following message in the `snips-watch -vvv` output:
```bash
The session was ended because one of the component didn't respond in a timely manner.
```
something is definitely wrong: either there is no action code for the detected intent or 
the action code is "erroring out" due to possible bugs in the Python code. Other common 
errors include not typing the correct intent names (which means you aren't subscribing to the 
intents you want) and similarly not including the correct `username` in the intent names.

## Essentials when creating your own Action

- Your action scripts must begin with `action-*`.
- Create a `requirements.txt` and `setup.sh` file as the ones in this repo.
- Make your action script `action-*.py` and `setup.sh` executable, i.e. from the Terminal (for MacOS and Linux) run the following commands:
```bash
chmod +x action-*.py 
chmod +x setup.sh
```

## Good practices when coding your own Action

Essentially all scripts should start off like so (with the additonal libraries that you requires for your 
application):
```python
#!/usr/bin/env python2
from hermes_python.hermes import Hermes    

MQTT_IP_ADDR = "localhost"
MQTT_PORT = 1883
MQTT_ADDR = "{}:{}".format(MQTT_IP_ADDR, str(MQTT_PORT))
```

It is good practice to create global variables for your different intents, as is done in this skill:
```python
INTENT_START = "bezzam:start_mental_calculations"
INTENT_ANSWER = "bezzam:give_mental_calculation"
INTENT_STOP = "bezzam:stop_lesson"
INTENT_DOES_NOT_KNOW = "bezzam:does_not_know_calculation"
```
where `bezzam` should be replaced by your username on the Snips Console if you create your own intents.

Typically each intent will have its own function/callback that should be triggered when the intent is 
identified. Our action code should "subscribe" to the various intents and declare what function/callback 
should be used. This should be done in the `action-*.py` script, as is done in this example (at the bottom 
of the script):
```python
with Hermes(MQTT_ADDR) as h:

    h.subscribe_intent(INTENT_START, user_request_quiz) \
        .subscribe_intent(INTENT_STOP, user_quits) \
        .subscribe_intent(INTENT_DOES_NOT_KNOW, user_does_not_know) \
        .subscribe_intent(INTENT_ANSWER, user_gives_answer) \
        .start()
```

The above callbacks, e.g. `user_request_quiz`, have two arguments:

- `hermes`: this object will allow us to continue/terminate interactions with the user.
- `intent_message`: this object will contain useful info, e.g. slot entries, in order for
our application code to respond appropriately.

In general, an intent's function/callback should perform the following:

1. Parse `intent_message` to perform the appropriate action based on the various slot values. The various
slots can be read from `intent_message` like a dictionary. For instance, in this action code we can 
extract the number of questions the user would like as such:
    ```python
    intent_message.slots.n_questions.first().value
    ```
    We use `first()` to extract the first slot value that was identified. Alternatively, we can get a list of all 
    identified slot values as such:
    ```python
    intent_message.slots.n_questions.all()
    ```
    It is very important that the member, `n_questions` in this case, matches the slot value as defined in the Console.
2. After performing the necessary action for the intent, we can either continue the interaction/session with the user
with `hermes.publish_continue_session` or end the current interaction/session with `hermes.publish_end_session`. Both
take as arguments:
    - The session ID (`hermes.sesssion_id`).
    - A string to respond to the user.
    
    `hermes.publish_continue_session` could also take a list of potential intents that could follow after the given 
    detected intent. In this example, after starting the lesson (`INTENT_START`), the session could continue to one of 
    three possibilities:
    1. `INTENT_ANSWER`
    2. `INTENT_DOES_NOT_KNOW`
    3. `INTENT_STOP`
    
    For this reason, we pass a list of these three intents to `hermes.publish_continue_session` (Line 106).
 
When coding interactions that involve storing some sort of status, it is useful to define a dictionary, e.g. 
`SessionsStates` in this example, where the key is the session ID. Remember to delete entries when you no longer need
to keep track of a session's status!


## Setting up Snips on your Pi

Here are some pointers, if you are not working with a pre-assembled Maker Kit, as would be provided at a workshop.

The overall steps, as described in more detail [here](https://snips.gitbook.io/documentation/installing-snips/on-a-raspberry-pi)
are:

1. Flash an SD card with Raspbian Stretch Lite, e.g. using [Etcher](https://www.raspberrypi.org/documentation/installation/installing-images/)
2. Enable SSH by adding an empty `ssh` file at the root of the `boot` volume on your SD card.
3. Configure network access: you can simply connect with an Ethernet cable or for WiFi edit the `etc/wpa_supplicant/wpa_supplicant.conf` file (after SSH'ing). More details below.
4. Install the Snips Platform by running [these commands](https://www.raspberrypi.org/documentation/installation/installing-images/).
5. [Configure audio](https://www.raspberrypi.org/documentation/installation/installing-images/). This can also be done
with SAM: `sam setup audio`.

#### More on connecting to local network

See [here](https://www.raspberrypi.org/documentation/configuration/wireless/wireless-cli.md) for instructions.
The easiest way is to use the command line raspi-config tool:
```bash
$ sudo raspi-config
```

For getting on a WPA2 Enterprise network like `eduroam`, you will have to dig a bit deeper. Edit the following file:
```bash
sudo nano /etc/wpa_supplicant/wpa_supplicant.conf
```

And add the following content (e.g. for EPFL):
```
ctrl_interface=/var/run/wpa_supplicant
network={
    ssid="eduroam"      # or "epfl"
    key_mgmt=WPA-EAP
    proto=WPA2
    eap=PEAP
    identity="gaspar@epfl.ch"
    password="my_password"
    anonymous_identity="anonymous@epfl.ch"
    phase2="auth=MSCHAPV2"
    ca_cert="/etc/ssl/certs/thawte_Primary_Root_CA.pem"
    subject_match="CN=radius.epfl.ch"
    priority=8
}
```

Protect your file with the rights 600. (read/write for root only)
```bash
$ chmod 600 /etc/wpa_supplicant/wpa_supplicant.conf
```

The following command starts the WPA supplicant (check the name of the interface)
```bash
$ wpa_supplicant -c /etc/wpa_supplicant/wpa_supplicant.conf -i wlan0
```

The following command starts the dhcp
```bash
$ dhclient wlan0
```

For more info, see [here](https://epnet.epfl.ch/files/content/sites/network/files/Download/WIFI/linux_en.pdf).
