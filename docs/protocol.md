# mppclone.com Protocol

## Message Format
All messages sent by the client and the server are JSON arrays. Socket messages are strings, not binary. Each array can contain one or more individual message objects. Each individual message object has a "m" property where its value is a string signaling which message type it is.
#### Example socket message:
[{"m":"hi","token":"abcdef"}]

## Important concepts
### Colors
All colors are hexadecimal strings.
#### Example
"#ff8ff9"
### Times
All times are UNIX timestamps (number). All times that are sent and received by clients are the server's time. Clients adjust the times they send and receive to ensure it lines up with the server's time. It figures out how much to adjust by using "t" messages.
#### Example
1627968429598

### String validation
For most messages that get sent to other clients, strings are checked to make sure they don't cause issues. Strings following string validation must not include any of the following characters:
- \n
- \r
- \v
- \0
- \t
- \u202e
- \u2066
- \u2067
- \u202b
- \u200f  
Strings following string validation cannot be made entirely of spaces, and cannot be empty.

### Participant info
In some messages, the server will send a participant info object instead of an id.
#### Properties
- "id": The user's id (string).
- "\_id": The user's id (string). This is identical to the above field but is sent to keep backwards compatibility.
- "name": The user's name.
- "color": The user's color.
- "x": The user's mouse x coordinate (number). This is usually between 0-100 for standard clients, but can be any number if set with scripts. 0 is on the left edge, 100 is on the right edge.
- "y": The user's mouse y coordinate (number). This is usually between 0-100 for standard clients, but can be any number if set with scripts. 0 is on the top edge, 100 is on the bottom edge.
- ?"tag": Optional tag (string). This is usually either "BOT", "MOD", "ADMIN", or "OWNER", but could be any string. If this property is not present, the user does not have a tag.
- ?"vanished": Whether the user is vanished (boolean). Regular users and bots will never see this property, however moderators will receive this if they or another user are vanished. If this property is not present, the user is not vanished.  
Mouse will start as (200, 100) for users who haven't sent a mouse position.
#### Example
{
  "\_id":"514df042c61528f566530313",
  "id":"514df042c61528f566530313",
  "name":"Lapis",
  "color":"#ff8ff9",
  "tag":"OWNER",
  "x":50,
  "y":50
}

### Channel settings
Channel settings are an object with properties for each setting.
#### Properties
- "visible": Whether the channel is visible to normal users in the channel list (boolean).
- "color": The channel's inner background color.
- ?"color2": The channel's outer background color.
- "chat": Whether chat is enabled in this channel (boolean).
- "crownsolo": Whether anyone can play the piano (boolean). If this is false, only the crown holder can play the piano.
- ?"no cussing": Whether no cussing is enabled (boolean). If this is enabled, some things in chat will get replaced with asterisks for users who don't have the crown. If this property isn't present, "no cussing" is disabled.
- "limit": The maximum amount of users that can join this channel (number). This is an integer between 0-99. If this is lowered while more users are in the channel, users won't get kicked. The crown holder and users who already have a participant in the channel bypass this limit.
- ?"minOnlineTime": The minimum amount of time that a user needs to have been online to join this channel (number). It's measured in milliseconds and is between 0 and 86400000. If this field is not present, the restriction does not apply. If a user holds the crown in this channel or if they already have a participant in the channel, they bypass this restriction.
- ?"lobby": Whether the channel is a lobby (boolean). Clients cannot change this property. If this property is not present, the channel is not a lobby.
#### Example
{
  "lobby":true,
  "limit":20,
  "color":"#73b3cc",
  "color2":"#273546",
  "visible":true,
  "chat":true
  "crownsolo":false
}

### Crown
This is an object containing information about the crown in that channel.
#### Properties
- "userId": The user id who has the crown (string).
- ?"participantId": This field is either identical to "userId" or it is not present (string). If this field is not present, the crown is dropped. If it is present, the crown is held.
- "time": The time at which the crown was dropped (number). If the crown is not dropped, this value can be ignored.
- "startPos": An object containing an "x" and "y" property with coordinates (numbers) where the crown's animation should start.
- "endPos": An object containing an "x" and "y" property with coordinates (numbers) where the crown's animation should finish.
#### Example
{
  "userId":"b40df99cc2ca6f503fba77cb",
  "time":1627968456997,
  "startPos":{
    "x":75.8,
    "y":30.6
  },
  "endPos":{
    "x":75.8,
    "y":82.3
  }
}

### Target
This is an object describing who something should be sent to. Currently this is only used in "custom" messages. If mode is "subscribed", it targets everyone in the channel who is subscribed to custom messages. If mode is "id", it targets a single participant. If mode is "ids", it targets all of the listed participants. Targeting will **never** target the sender.
#### Properties
- "mode": This can either be "subscribed", "id", or "ids".
- "id": If "mode" is "id", this is sent with a string user id.
- "ids": If "mode" is "ids", this is sent with an array of user ids.
#### Example
{
  "mode":"ids",
  "ids":\[
    "4d354eaddf02eedc6211034c",
    "fd4365210b7f25b6cb0ed683"
  ]
}

### Note
Notes can either be a note start, or a note stop. Note starts have a "v" property singaling the velocity of the note, and note stops have an "s" property with the value of 1. It's impossible for a note to have both "v" and "s".
#### Properties
- "n": This is the name of the note (string). Run `Object.keys(MPP.piano.keys)` in console to see a list of possible values.
- ?"d": Delay from the message's time to trigger this note event (number). The delay is in milliseconds and should be no more than 200. If this property is not present, the delay is 0.
- ?"v": The velocity for this note if it's a note start (number). This is between 0 and 1 inclusive.
- ?"s": 1 if the note is a note stop.
#### Example
{
  "n":"b2",
  "d":150,
  "v":0.32
}

## Client -> Server Messages

### A
"a" messages are sent to chat in the current channel.
#### Properties
- "message": String to send in chat for everyone in your channel. Must be less 512 characters or less and must follow [string validation](https://github.com/aeiou879/mppclone/blob/main/docs/protocol.md#string-validation).
#### Example
{
  "m":"a",
  "message":"Hello :D"
}
### Bye
A "bye" message can be sent to close the client's socket. No more messages will be handled after the server receives "bye". Standard browser clients don't send this.
#### Example
{
  "m":"bye"
}
### Ch
A "ch" message can be sent to attempt to change the client's channel. If the specified channel does not exist, it will be created.
#### Properties
- "\_id": Channel name. Must be less than 512 characters and must follow [string validation](https://github.com/aeiou879/mppclone/blob/main/docs/protocol.md#string-validation).
- ?"set": Optional settings to initialize the channel with if it doesn't exist. See channel settings. If a property isn't sent in this object, the server will use the default value.
#### Example
{
  "m":"ch",
  "\_id":"My new room",
  "set":{
    "visible":false
  }
}
### Chown
Clients can send chown messages to drop the crown or give it to someone else.
#### Properties
- ?"id": The user id who should receive the crown (string). If this property is not present or if the id is invalid, the participant will drop the crown.
#### Example
{
  "m":"chown",
  "id":"f46132453478f0a8679e1584"
}
### Chset
Clients can send this to change a channel's settings if they have the crown.
#### Properties
- "set": Channel settings to change. See channel settings. If a property isn't sent, it will stay as it is. The only exception is "color2" which gets deleted from the channel's settings if the property is not sent.
#### Example
{
  "m":"chset",
  "set":{
    "color":"#0066ff",
    "color2":"#ff9900",
    "chat":"false"
  }
}
### Custom
Clients can send custom data using this message. This is meant for developers to create addons for multiple people but being restricted to the standard protocol. A participant can only send 16384 bytes of custom data per 10 seconds. This is measured by the stringified value in the "data" property. If the "data" property is not present, it is treated as null.
#### Properties
- "data": Data to send to other clients. This can be any valid JSON. It could be an array, an object, a string, a number, or boolean, or null. Object nesting is acceptable to any depth (within the data quota).
- "target": Object representing who this should get sent to. See target in Important Concepts.
#### Example
{
  "m":"custom",
  "data":{
    "xyz":"abc",
    "def":"ghi",
    "jkl":\[
      "mno",
      123,
      456,
      null,
      "hi",
      {
        "idk"
      },
      true
    ]
  },
  target: {
    "mode":"subscribed"
  }
}

### Devices
Browser clients send a list of connected midi inputs and outputs with this. Bots don't need to send this.
#### Properties
- "list": An array of device infos.
#### Example
{
  "m":"devices",
  "list":\[
    {
      "type": "input",
      "manufacturer": "",
      "name": "loopMIDI Port",
      "version": "1.0",
      "enabled": true,
      "volume": 1
    },
    {
      "type": "output",
      "manufacturer": "",
      "name": "OmniMIDI",
      "version": "0.6",
      "volume": 1
    }
  ]
}

### Dm
"dm" messages are sent to direct message another participant in the channel.
#### Properties
- "message": String to send in chat to the target user. Must be 512 characters or less and must follow [string validation](https://github.com/aeiou879/mppclone/blob/main/docs/protocol.md#string-validation).
- "\_id": User id to send the message to (string).
#### Example
{
  "m":"dm",
  "message":"hi there",
  "\_id":"a8c86bb6e74c9ec8900e061a"
}

### Hi
A "hi" message is sent when the client first connects to the server. This must be done before doing anything else.
#### Properties
- ?"token": A token to use (string). Valid tokens will assign the client a specific user when they join. If this property is not present, the client will get the default user for their IP address.
- ?"code": Response generated by the anti-bot system. Bots can ignore this, valid bot tokens will bypass the anti-bot system.
#### Example
{
  "m":"hi",
  "token":"this is not a valid token"
}

### Kickban
This is sent to ban a user from the channel.
#### Properties
- "\_id": The user id to ban (string).
- "ms": The amount of milliseconds to ban the user for. Between 0 and 3600000.
#### Example
{
  "m":"kickban",
  "\_id":"a4ea42f1d9770e671f938e8c",
  "ms":300000
}

### M
This is sent to move the participant's mouse.
#### Properties
- "x": x position to move to (number). Can be any valid JSON number except NaN, even if it's out of the normal range.
- "y": y position to move to (number). Can be any valid JSON number except NaN, even if it's out of the normal range.
#### Example
{
  "m":25,
  "y":63.5
}

### -custom
This is sent to unsubscribe from custom messages sent with the "subscribed" target.
#### Example
{
  "m":"-custom"
}

### -ls
This is sent to unsubscribe from channel list updates.
#### Example
{
  "m":"-ls"
}

### N
This sends notes to other clients in the channel.
#### Properties
- "t": The time at which the notes should play.
- "n": An array of notes. See [note](https://github.com/aeiou879/mppclone/blob/main/docs/protocol.md#note).
#### Example
{
  "m":"n",
  "t":1627971516894,
  "n":\[
    {
      "n":"c3",
      "v":0.75
    },
    {
      "n":"c3",
      "d":100,
      "s":1
    }
  ]
}

### +custom
This is sent to subscribe to custom messages sent with the "subscribed" target.
#### Example
{
  "m":"+custom"
}

### +ls
This is sent to subscribe to channel list updates.
#### Example
{
  "m":"+ls"
}

### T
"t" is for pinging. This must be sent every 20 seconds, because the server will disconnect your client if it's not sent for more than 30 seconds.
#### Properties
- ?"e": The client's time.
#### Example
{
  "m":"t",
  "e":1627972519126
}

### Unban
"unban" is sent to unban a user from the client's channel.
#### Properties
- "\_id": The user id to unban.
#### Example
{
  "m":"unban",
  "\_id":"0c49bea6c2a328101eee40b7"
}

### Userset
"userset" changes the name or color of the client's user.
#### Properties
- "set": An object containing "name" and or "color" properties. "color" must be a valid color, and "name" must be 40 characters or less and fit [string validation](https://github.com/aeiou879/mppclone/blob/main/docs/protocol.md#string-validation).
#### Example
{
  "m":"userset",
  "set":{
    "name":"Lapis",
    "color:"#ff8ff9"
  }
}

## Server -> Client Messages (coming soon)