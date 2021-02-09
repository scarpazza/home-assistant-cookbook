# Grandfather Clock-style Hourly Chimes using a Sonos Smart Speaker

## Summary 

This tutorial describes how to set up automations that will play hourly chimes on a Sonos speaker.

While my original intention was entirely selfish, it turns out that my 2.5 year old toddler is loving it and it's a useful prop to teach him to count and even an aid to enforce bedtime.

This automation will pause the speaker if it was playing, play the chime, then return the speaker to its previous activity.


## Step 1 - Configure your Sonos speakers
Configure the Speaker from the smartphone app.

Install the Sonos integration into Home Assistant and configure your speakers. 

For the purposes of this tutorial, the speaker we'll use is `media_player.kitchen`

## Step 2
Create a `www` subdirectory under your `.homeassistant` directory, and create another subdirectory `chimes` under it. 

If you run Home Assistant Core or have otherwise access to an SSH shell, this is done with an `mkdir -p ~/.homeassistant/www/chimes` issued as the same user that runs Home Assistant. 

The location you just created is where the static web server component of HASS will serve contents from.

## Step 3 - Samples
Download the 12 hourly chime samples from https://freesound.org/people/joedeshon/packs/7926/.

Be so kind as to send a thank you note via https://freesound.org/home/messages/new/joedeshon/ to Joe DeShon who sampled them from his clock.  Without his samples, this project would not exist.

Copy all these samples to `~/.homeassistant/www/chimes`.

Rename the files to `01.wav`, `02.wav`, ..., `12.wav`, respectively, for brevity.

## Step 4 - Chime script

In your scripts.yaml, add the following script

```
chime:
  alias: Chime
  sequence:
  - service: sonos.snapshot
    data:
      entity_id: media_player.kitchen
  - service: sonos.unjoin
    data:
      entity_id: media_player.kitchen
  - service: media_player.volume_set
    data:
      entity_id: media_player.kitchen
      volume_level: 0.4
    entity_id: media_player.kitchen
  - service: media_player.play_media
    data:
      entity_id: media_player.kitchen
        media_content_id: https://yourdomain.duckdns.org:8123/local/chimes/{{"%02i" % ([12,1,2,3,4,5,6,7,8,9,10,11][strptime(states('sensor.time'), '%H:%M').hour%12])}}.wav   
        media_content_type: music
  - delay: '70'
  - service: sonos.restore
    data:
      entity_id: media_player.kitchen
  mode: single
  icon: hass:clock 
```  
  
If you use https, you have to use the external URL (yourdomain.duckdns.org) as I did, or the https certificates will mismatch at run time.
If you use http, you can replace that with your internal URL e.g., `http://192.168.0.123:8123/local/chimes/...`
These URLs are tell the Sonos speakers where to get the chimes from.

## Step 5 - Hourly automation

In your automations.yaml, add the following entry:
```
- id: hourly_chime 
  alias: Hourly chime from 7am to 10pm  
  description: '' 
  trigger:  
  - platform: time_pattern  
    minutes: '00' 
  condition:  
  - condition: time 
    after: 06:59:00 
    before: 22:01:00  
  action: 
  - service: script.chime 
    data: {}  
  mode: single   
```

I only play chimes from 7 am to 10 pm. You may want to 
Modify the automation according to your needs.

## Step 6 - Tests

From the Configuration > Server Controls page, reload the scripts.

Test manually the `chime` script, by entering the Configuration > Scripts page and clicking on the "play" triangle icon next to the Chime entry. Ensure that the hourly concerto plays, with the appropriate number of chimes. 

If the script works, test the automation as well. 
This is done on the Configuration > Automations page by locating the "Hourly chime from 7am to 10pm" automating and clicking on "execute".
