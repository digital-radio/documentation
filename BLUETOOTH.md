BLUETOOTH
===

[Tutorial](https://forum.armbian.com/topic/6480-bluealsa-bluetooth-audio-using-alsa-not-pulseaudio/)

### Manage bluetooth connection
```bash
bluetoothctl
```

### Play music
```bash
aplay -D bluealsa:DEV=78:44:05:D5:8E:D9,PROFILE=a2dp file_example_WAV_2MG.wav
```

### Play radio
Create file
```
~/.asoundrc
```

With the following content
```
defaults.bluealsa {
    interface "hci0"
    device 78:44:05:D5:8E:D9
    profile "a2dp"
}

pcm.btspeaker {
 type plug
  slave {
    pcm {
      type bluealsa
      interface hci0
      device 78:44:05:D5:8E:D9
      profile "a2dp"
    }
  }
  hint {
    show on
    description "Bluetooth Speaker"
  }
}

pcm.!default {
    type plug
    slave.pcm "btspeaker"
}
ctl.!default {
    type hw
    card 0
}
```

Play audio from url
```bash
mplayer -ao alsa:device=bluealsa "https://n17.rcs.revma.com/ypqt40u0x1zuv?rj-ttl=5&rj-tok=AAABc0AqKSQAbHkNwFtzGzebBA"
```
