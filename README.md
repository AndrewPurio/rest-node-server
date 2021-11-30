rest-node-server
================

Contains the flows for the REST node Node-Red server

<br />

# About
This is the REST Node server repository which contains the flows

<br />

# Raspberry Pi Setup (based on a newly installed RaspberryPi OS in Raspberry Pi 4, already connected to Wifi and updated ```sudo apt update && sudo apt upgrade```)

1) Enable I2C interface and Remote GPIO
    - Go to the Raspberry Pi Terminal (via SSH or the desktop GUI) and type the command ```sudo raspi-config```
    - A menu of options will appear, select **3. Interface Options** (use the arrow keys to select) and then press Enter
    - Select **P2 SSH** and select **Yes** (arrow keys to select) and then press Enter two times
    - Repeat the previous steps but on the third one, select **P8 Remote GPIO** and enable it
    - Select Finish after the previous steps were successfully accomplished

2) Pigpiod Installation
    - Install by entering the following command ```sudo apt install pigpiod```
    - Enable on startup by entering the following command ```sudo systemctl enable pigpiod```

3) Install Avahi Daemon
    - Enter the command ```sudo apt install avahi-daemon```
    - Start the avahi daemon service ```service avahi-daemon start```
    - Enter the following command: ```cd /etc/avahi/services && sudo nano http.service```
    - Copy the following and paste it (right mouse-click), save(press Ctrl + X, then press y):
    ```
    <?xml version="1.0" standalone='no'?><!--*-nxml-*-->
    <!DOCTYPE service-group SYSTEM "avahi-service.dtd">
    <service-group>
    <name replace-wildcards="yes">%h</name>
    <service>
    <type>_http._tcp</type>
    <port>80</port>
    <txt-record>Rest Node by Exist Tribe</txt-record>
    </service>
    </service-group>
    ```
    - Start the avahi daemon by entering the command: ```sudo service avahi-daemon start```
 
4) Change the hostname
    - Enter the following command: ```sudo nano /etc/hostname```
    - Delete the text ("raspberrypi" by default) and replace it with "restnode"
    - Save the configuration by pressing Ctrl + X, then press y
    - Change the available hosts by entering the following command: ```sudo nano /etc/hosts```
    - Replace all the text inside the file with the following:
    ```
    127.0.0.1       localhost
    127.0.0.1       restnode

    ::1             localhost ip6-localhost ip6-loopback
    ff02::1         ip6-allnodes
    ff02::2         ip6-allrouters

    127.0.1.1               restnode
    ```

6) Reboot the Raspberry Pi to apply the system changes by plugging it out then plugging it in or simply enter the following command in the terminal:
    ```sudo reboot```


# Docker Container Installation:

1) Install Docker in your Raspberry Pi
    - Get and install updates if not yet done from the previous procedures: ```sudo apt-get update && sudo apt-get upgrade```
    - Download the script to install Docker: ```curl -fsSL https://get.docker.com -o get-docker.sh```
    - Execute the script: ```sudo sh get-docker.sh```
    - Add current user (e.g. pi) to Docker: ```sudo usermod -aG docker pi```
    - Verify if Docker is installed by checking its version: ```sudo docker version```
    <br />
    You can check this Reference for a more elaborate guide of Docker installation: https://phoenixnap.com/kb/docker-on-raspberry-pi
    <br />
2) Download and run the image by pasting and entering the following command in the terminal
```
sudo docker run -p 80:1880 -v /etc/wpa_supplicant:/etc/wpa_supplicant -v /etc/localtime:/etc/localtime:ro --device=/dev/gpiomem --device=/dev/i2c-1:/dev/i2c-1 --device=/dev/snd:/dev/snd  --name rest-node --restart=always restnode/rest_node
```

# Testing inside the Node-Red flows

## Light Controls

1) Go to the **Hardware Interface** tab
2) In the Lights(Hardware) section, change the GPIO pins of the Night Light and Wake Light inside the pi gpiod nodes
3) Enable the Set Brightness nodes(I2C) and change the bus address depending on the designated I2C pin or bus address.
4) Go to the Node-Red Dashboard (e.g. **http://<raspberry-pi-ip-address>/ui**) and there you can toggle the Night Lights and Wake Lights for testing
    
## Audio Controls
1) Go to the Raspberry Pi terminal (e.g. via ssh or the Raspberry Pi desktop itself)
2) Copy and paste the following command in the terminal:
 ```docker exec -it rest-node sh```
3) This will open a terminal for the rest-node docker container itself. In this terminal, copy and paste the comamnd:
   ```amixer```
4) The result will be all the available audio devices similar to the ones below:
  ```
 Simple mixer control 'Headphone',0
    Capabilities: pvolume pvolume-joined pswitch pswitch-joined
    Playback channels: Mono
    Limits: Playback -10239 - 400
    Mono: Playback -683 [90%] [-6.83dB] [on]
  ```
5) Copy the name of the device enclosed in single quotes, in this case **Headphone**. Go to the **Audio Controls** of the Node-Red tab and in the Testing Section, find a node called **Store Audio Device to Context** and replace the value 'Headphone' with the copied value (must be enclosed in single quotes)
6) Click **Download Media File** (Inject) to Download a sample audio from Firebase storage
7) Click **Test API Call for Audio Controls** to set the Wake Sound audio for testing.
8) If the setup was a success, you can test playing, pausing, resuming, or stopping the music via the Inject Buttons (Play, Pause, Resume, Stop) in the Testing Section
9) You can adjust the volume, currently in fix values (Low, Medium, High) while an audio is playing by clicking its respective Inject buttonsin the left of the audio controls

    **As of 11:06 AM, Nov. 19, 2021 (UTC+8), Firebase storage has exceeeded its max quota for today that's why you may get some errors regarding the download
    
<br />

# REST Endpoints:

## Dev Endpoint:
```https://restnode.info```

## mDNS Endpoint: (Raspberry Pi)
```http://restnode.local```

<br />

## Events

### GET
```/restnode/event``` - Gets all the available events that will be triggered on the specified time (except sunrise and sunset which is automatically set to a certain point of time)

```/restnode/event/:type``` - Gets the info from the ```type``` of **event** inputted. ```type``` can be one of the string values that was returned from the event request before. e.g. ```https://restnode.info/restnode/event/bedtime```

### POST
```restnode/event``` - Creates/updates an event specified in the body

<br />

**Example request body:**

```
const request_body = {
    type: 'bedtime', // Should be one of the values: bedtime, waketime, sunrise, sunset
    time: '21:00', // Base time of the on and off offsets
    
    light: {
        onoffset: 0, // integer - The on time offset in minutes e.g. the light will turn on at the same time (0* minutes) that was set
        
        offoffset: 60, // integer - The off time offset in minutes e.g. the light will turn off after 60 minutes of the time set
        
        // onpayload is fired when the time (with the onoffset considered) is reached
        onpayload: {
            light: 'NIGHT_LIGHT', // type of light to trigger
            max_brightness: 100, // total counter/brightness to increment to starting from 0
            tick: 1000, // in ms - determines how long the light brightness should increment from 0(off) to 100(on) during ```FADE_ON``` state [100(on) to 0(off) in ```FADE_OFF```], in this case ```1000ms```
            state: 'FADE_ON' // Can be one of the following values: ON, FADE_ON
        },
        
        // offpayload is fired when the time (with the offoffset considered) is reached
        offpayload: {
            light: 'NIGHT_LIGHT', // type of light to trigger
            max_brightness: 100, // total counter/brightness to increment to starting from 0 (in this case light will start from 0 and count up to 100 during ```FADE_ON```)
            tick: 1000, // in ms - determines how long the light brightness should increment from 0(off) to 100(on) during ```FADE_OFF``` state [100(on) to 0(off) in ```FADE_OFF```], in this case ```1000ms```
            state: 'FADE_ON' // Can be one of the following values: ON, FADE_ON
        },
        
        days_selected: {
            mon: true,
            tue: true,
            wed: true,
            thu: true,
            fri: true,
            sat: true,
            sun: true
        }
    },
    
    sound: {
        onoffset: 0, // The on time offset in minutes e.g. the sound will start playing at the same time (0* minutes) that was set
        
        offoffset: 420, // The off time offset in minutes e.g. the sound will stop playing after 420* minutes (7 hours) of the time set
        
        onpayload: {
            audio_file: null, // url for the audio file - firebase admin storage implementation still in progress
            volume: 50, // initial volume to start during fading
            max_volume: 100, // max volume threshold during initial fading and fade off of the sound (fading happens automatically)
            state: 'PLAYING', // onpayload state can either be: 'PLAYING', 'RESUMED'
            sound: 'NIGHT_SOUND' // Either of the three: "NIGHT_SOUND", "WAKE_SOUND", "RELAXATION_SOUND"
        },
        
        offpayload: {
            audio_file: null, // url for the audio file - firebase admin storage implementation still in progress
            volume: 50, // initial volume to start during fading
            max_volume: 100, // max volume threshold during initial fading and fade off of the sound (fading happens automatically)
            state: 'STOPPED', // offpayload state can either be: 'STOPPED', 'PAUSED',
            sound: 'NIGHT_SOUND' // Either of the three: "NIGHT_SOUND", "WAKE_SOUND", "RELAXATION_SOUND"
        },
        
        days_selected: {
            mon: true,
            tue: true,
            wed: true,
            thu: true,
            fri: true,
            sat: true,
            sun: true
        }
    }
}
```

<br />

## Lights

### GET
```/restnode/light``` - Gets all the available lights that can be controlled

```/restnode/light/:type``` - Gets the info and current state of the ```type``` of light specified

<br />

## Audio

### GET
```/restnode/audio``` - Gets all the available audio types that can be modified

```/restnode/audio/:type``` - Gets the info and current state of the ```type``` of audio specified

