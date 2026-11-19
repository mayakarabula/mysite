+++
title = "📹 19.11.2026 - diary room"
date = "2023-12-13"
description = "📹 19.11.2026 - diary room"
author = { name = "Maya" }

[extra]
image = "/images/diary.png"
+++

## A week to go

On the 5th of November I got a message from my friend Bela, asking if I would like to collaborate on an art installation on a big party - [tracey](https://www.instagram.com/tracey.club/) next week. I said yes, of course.

It was supposed to be a small diary room. When somebody was entering, a motion sensor should pick up the movement and start a video on a TV with questions, as well as a recording on a camera. A person would sit on a chair and answer some questions about the party. It was inspired by the Big Brother diary room, but quite chill.

So it started, we had a couple days to do everything.

{{ img(id="/images/diary-map.jpg", alt="diary room map", height="320px") }}

And I haven't really done such things before. I only had a temperature sensor connected to my raspberry pi (rpi), so I didn't really know how the motion sensors work. But it sounded fun. 

My first idea was that maybe we can connect it to an rpi, or to Bluetooth, or wifi, but I had no idea. Thankfully I asked my friend and colleague from work, Abhi, who has more knowledge about sensors, and he recommended a [Zigbee](https://en.wikipedia.org/wiki/Zigbee) sensor. Zigbee is a communication protocol dedicated for such devices. It's rather reliable, but it runs at 2.4GHz, the same frequency as standard wifi.

So Zigbee it was.

We got a green light for the project on Monday the 9th.

## 3 days to go

Below a photo of the zigbee sensor, and Bela checking the camera position during scouting in the club on Wednesday, 3 days before the party.

{{ img(id="/images/diary-sensor-room.png", alt="diary sensor and room") }}

Now Bela had to prepare the video to run on the TV, and I had to figure out how to start the camera recording and the video. 

My idea how the flow should look was as follows:

```
Motion sensor -> Home Assistant -> Camera + TV
```

For the camera recording I was in luck, there was an application to control the camera from Sony. I could have used direct communication via the SDK and learned more, but time was limited. At the same time I was working on a project for a public demo at a conference.

I used [AutoHotkey](https://www.autohotkey.com/) script to focus a window and run Super+R to record.

```
#Requires AutoHotkey v2.0

if WinExist("ahk_exe Record.exe") {
    WinActivate()
    WinWaitActive()
    Send("#r")
}
```

The TV setup was quite easy. I could just use [OBS](https://obsproject.com/), software used to manage streaming and video, often used by gamers and other streamers. It has a nice API with which one can run a video.

```js
import OBSWebSocket from "obs-websocket-js";

const obs = new OBSWebSocket();
await obs.connect("ws://127.0.0.1:4455", "password");

await obs.call("TriggerMediaInputAction", {
  inputName: "Diary video",
  mediaAction: "OBS_WEBSOCKET_MEDIA_INPUT_ACTION_RESTART",
});
```

Now for the sensor, I have connected it to [Home Assistant](https://www.home-assistant.io/), it's software to manage devices like sensors or lights. There are also libraries to handle the communication with HA and handle the events. Would I like to write the communication directly via Zigbee myself? Yes. Did I have time for it? Sadly not. 

```js
import {
  createConnection,
  createLongLivedTokenAuth,
} from "home-assistant-js-websocket";

const auth = createLongLivedTokenAuth(
  "http://homeassistant.local:8123",
  "long-lived-token"
);
const connection = await createConnection({ auth });

connection.subscribeEvents((event) => {
  const { entity_id, new_state } = event.data;
  if (entity_id === "binary_sensor.diary_motion" && new_state.state === "on") {
    startOBS();
    startRecording();
  }
}, "state_changed");
```

## Saturday - Party day

Now, Saturday, the day of the party, 5 pm. Final test at 6 pm. Party starts at 11 pm.

Home Assistant is booting up, we are setting up the room.

{{ img(id="/images/diary-tv-ha-setup.png", alt="Home Assistant starting up and room setup", height="450px") }}

Everything is working well, omg. It was the first time we were testing the whole setup.

The sensor is picking up the motion. HA is sending events to my script, and it starts the recording and the TV.

5:50 pm, ten minutes to the final test, everything is set up. The sensor suddenly stops working.

I tried to debug the sensor for about 5 minutes, but it wouldn't work. Then Bela asked for 5 extra minutes before the test.

I decided on plan B, no sensor, buttons in Home Assistant, one to turn it ON and one to turn it OFF.

```js
connection.subscribeEvents((event) => {
  const { domain, service, service_data } = event.data;
  if (domain !== "input_button" || service !== "press") return;

  if (service_data.entity_id === "input_button.diary_on") {
    startOBS();
    startRecording();
  }
  if (service_data.entity_id === "input_button.diary_off") {
    stopRecording();
  }
}, "call_service");
```

It worked. I had to set up the HA app on the phone of the host, and they actually liked the manual control of the flow more.

We were ready to start.

{{ img(id="/images/diary-ready.png", alt="tracey logo and diary room posters") }}

Before the party we went to have some food at Bela's and try to catch some sleep.

From 11 pm the crowd was huge. I don't know how many people exactly, but easily a thousand, with four dance floors and two art installations, including ours.

Some photos from the party on the [tracey instagram](https://www.instagram.com/p/Df6wcdfDqK8/?img_index=14).

We got more interest than we expected, in the first shift of 4 hours we got 50 recordings.

Then after 4 am we went to have some sleep. People were still queueing up. We started the day with a coffee and a salad.

{{ img(id="/images/diary-morning.png", alt="tracey queue and morning") }}

## Sunday - the second day of the party

We came back at around 11 am.

And started the Diary Room again. Thankfully everything worked. We had small technical problems with the wall behind the TV falling and messing up the setup, but not many issues otherwise.

We danced quite a bit with friends. I was very thankful for access to backstage, to rest on couches in peace. We got around 50 more recordings in the second round.

Overall it was a success, a bit stressful, and in a rush, but very fun.

I can't share the recordings yet, as they will be used by tracey for promotion. When they publish some, I will share them :)

{{ img(id="/images/diary-party.png", alt="tracey party and diary room post") }}
