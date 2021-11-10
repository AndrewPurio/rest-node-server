rest-node-server
================

Contains the flows for the REST node Node-Red server

<br />

# About

This is your project's README.md file. It helps users understand what your
project does, how to use it and anything else they may need to know.

This is the REST Node server repository which contains the flows

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
        onpayload: {
            ...NIGHT_LIGHT,
            state: 'FADE_ON' // Can be one of the following values: ON, FADE_ON
        },
        offpayload: {
            ...NIGHT_LIGHT,
            state: 'FADE_OFF' // Can be one of the following values: OFF, FADE_OFF
        },
        days_selected
    },
    sound: {
        onoffset: 0, // The on time offset in minutes e.g. the sound will start playing at the same time (0* minutes) that was set
        offoffset: 420, // The off time offset in minutes e.g. the sound will stop playing after 420* minutes (7 hours) of the time set
        onpayload: {
            ...NIGHT_SOUND,
            state: 'PLAYING'
        },
        offpayload: {
            ...NIGHT_SOUND,
            state: 'STOPPED'
        },
        days_selected
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

