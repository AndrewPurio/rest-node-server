rest-node-server
================

Contains the flows for the REST node Node-Red server

<br />

# About
This is the REST Node server repository which contains the flows

<br />

# REST Endpoints:

## Dev Endpoint:
```https://restnode.info```

## mDNS Endpoint: (Raspberry Pi)
```http://restnode.local```

<br />

## Installation:

1) Install Docker in your Raspberry Pi
    <br />
    Reference: https://phoenixnap.com/kb/docker-on-raspberry-pi
    <br />
2) Download and run the image by pasting and entering the following command in the terminal
```
docker run -p 80:1880 -v /etc/wpa_supplicant:/etc/wpa_supplicant -v /etc/localtime:/etc/localtime:ro --device=/dev/gpiomem --device=/dev/i2c-1:/dev/i2c-1 --device=/dev/snd:/dev/snd  --name rest-node --restart=always restnode/rest_node:dev
```

### Testing inside the Node-Red flows

**Light Controls**

1) Go to the **Hardware Interface** tab
2) In the Lights(Hardware) section, change the GPIO pins of the Night Light and Wake Light inside the pi gpiod nodes
3) Enable the Set Brightness nodes(I2C) and change the bus address depending on the designated I2C pin or bus address.

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

