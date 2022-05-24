This page just documents a simple issue, but a nevertheless annoying one: 
Sonos speakers won't play a playlist unless that playlist was also added in "favorites". 

More specifically, if you create a named playlist in the mobile or desktop app 
(that playlist will appear under the "Sonos Playlist" item in the "Select a music source" tab),
then an automation step looking like:

``` 
- service: media_player.select_source
    data:
      source: 'Playlist Name'
    entity_id: media_player.bedroom
```
will fail with a UPnP 804 error at runtime.

The error will disappear once you add the same playlist to your "Sonos Favorites".

This is done in the MacOs desktop app by selecting the ">" submenu button next to "Sonos Playlists" 
(in the "Select a music source" tab), 
then the desired playlist, then the "∨" dropdown menu button, 
then the "Add to Sonos Favorites" menu item. 



Sample script:

```
alias: Play white noise now
sequence:
  - service: media_player.volume_set
    data:
      volume_level: '0.25'
    entity_id: media_player.bedroom
  - service: media_player.select_source
    data:
      source: 'White Noise
    entity_id: media_player.bedroom
  - service: media_player.shuffle_set
    data:
      shuffle: true
    entity_id: media_player.bedroom
  - service: media_player.media_play
    entity_id: media_player.bedroom
mode: single
icon: hass:mother-nurse
```
