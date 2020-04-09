## Adaptive Audio Format v1.1
The Adaptive Audio Format is an audio format for OpenComputers that can adapt to a lot of kinds of sound cards, this supports 255 channels, 65535Hz, ADSR and volume: all while still being compact. All numbers are little endian.

### Header
The header of the file starts with the signature, it is `<DC3>AAF<ETX>` where `<DC3>` is byte 19 and `<ETX>` is byte 3.  
After the signature, there are the capabilities this file request, see Capabilities below.  
Finally all the audio fragments this file is made of are appended one after the other as described by Audio fragment below.

### Capabilities
Thoses are the capabilities requested on the sound card by the current AAF file.  
First are the types of waves being requested, this act as flags stored in a byte with following possible bits:

Flag bits | Wave type
--------- | ------------
xxx1      | Square
xx1x      | Sinusoidal
x1xx      | Triangle
1xxx      | Sawtooth

After that there are other flags (stored in a byte), used for miscelanous requested features, here possible bits:

Flag bits | Feature
--------- | -----------
x1        | ADSR
1x        | VOLUME
And then the requested capabilities end with an unsigned byte containing the number of channels to use.

### Audio fragment
An audio fragment contains multiple channel datas appended to each other depending on the number of channels.
This mean for example, there is channel data for channel 1, then channel 2, then channel 3. The AAF player knows the number of channels (and by extension, of channel data in an audio fragment) via the Capabilities above.

This means for 2 channels, an AAF file is: header then channel 1 data, channel 2 data, channel 1 data, channel 2 data, etc. until the file ends.

This is also described by:
```
- [PER CHANNEL DATA] x (channel number)
```

#### Channel Data
```
- [FREQUENCY] - unsigned short, in Hertz
```
**[FREQUENCY] acts as action type if it is below 3Hz!**

Packets are more data that should be appended just after the frequency when they have a corresponding action type, see below for the different packets:

ASDR body (if action type is 1): 
```
- [ATTACK] - unsigned short, duration in milliseconds
- [DELAY] - unsigned short, duration in milliseconds, equals to 0 if delay must be equal to the frequency of next audio fragments.
- [SUSTAIN] - unsigned short, volume level (like volume packet)
- [RELEASE] - unsigned short, duration in milliseconds
```
Volume body (if action type is 2):
```
- [VOLUME] - unsigned byte, decimal obtained by dividing by 255 (giving 255 volume levels)
```
Wave type body (if action type is 3):
If it's a wave type action (3 as frequency), wave type should be appended, it is an unsigned byte.
For the type packet, wave type can have different values:
Value | Wave Type
------|----------------
0     | Square wave
1     | Sinusoidal wave
2     | Triangle wave
3     | Sawtooth wave

Otherwise, if there is no action type (if the frequency is higher than 3Hz), then you should add:
```
- [DURATION] - unsigned short, duration in milliseconds
```

