# discord.js-light

A minimalistic [discord.js](https://discord.js.org) framework that disables the original library's aggressive caching behavior and prevents excessive resource usage.

This library is a rewrite of the old "djs-shenanigans" found in the folders above, dropping most of its experimental features and focusing mostly on anti-caching. It is powered by discord.js v12.0.0dev on a [fixed commit](https://github.com/discordjs/discord.js/tree/21d37ed0cc71b227be0feadc4ffbcd522ca4f00e) basis so it wont be affected by updates until approved.

## Why?

Discord.js has been THE javascript discord library for a long time now, and successfully powers thousands of bots. But as your bot grows larger, you will start noticing a horrendous increase in resource usage, especially memory consumption.

This is due to the fact that discord.js caches EVERYTHING it can in order to avoid hitting the Discord API as much as possible. This behavior can however make the library feel bloated for most developers as many times the library is caching and processing data that your bot will never use.

This issue has been discussed a few times by the community but ultimately it has been decided that the library is too tightly coupled with its caching systems and seperating them would be unfeasible. Thus the task of taming the monster falls back to us bot developers.

Several solutions have been presented so far, such as regular cache sweeping, i have felt however that the existing solutions were not good enough and decided to investigate it myself. This project later became the base boilerplate for all my bots and it does a wonderful job keeping hosting costs and scaling maintenance in check.

## Features

* Provides most of discord.js's basic events without any automatic caching
* Most classes have their structures intact and can be used the same way as the original library
* Partial objects are given when data is missing and can be manually fetched and cached when needed
* Makes use of Discord's new intents system to receive only data that is actually used
* Uses a fraction of memory, cpu and bandwidth compared to the original library (benchmarks can be found in the djs-shenanigans folder)

## Getting Started

Installation:

```npm install timotejroiko/discord.js-light```

optional packages (recommended to improve performance, especially zlib-sync)

```
npm install zlib-sync
npm install bufferutil
npm install discordapp/erlpack
npm install utf-8-validate
```

Simple usage:

```js
const Discord = require("discord.js-light");
const client = new Discord.Client({
	token: "yourbottoken"
});

client.on("ready", () => {
	// do stuff
});

client.on("message", message => {
	// do stuff
});
```

Unlike its predecessor, discord.js-light does not include prefix managers or command handlers, it does however feature auto-login, auto sharding (via internal sharding) and default logging of connection events and errors. PM2 cluster support was also removed to be reworked as a separate feature in the future.

## Caching

This library alters caching behavior by extending and modifying a few of discord.js's original classes as follows:

* Users are not cached. Relevant events contain a temporary User object or partial instead. Users can be manually cached by using `client.users.fetch(id)`. This will not cache the guild member.
* Channels are not cached. Relevant events contain a temporary Channel object or partial instead. Channels can be manually cached by using `client.channels.fetch(id)`. This will not cache the channel in the guild.
* Messages are not cached. Relevant events contain a temporary Message object or partial instead. Messages can be manually cached by fetching its channel and then using `channel.messages.fetch(id)`
* Guilds themselves are always cached and synced, but not its contents. See below
* Guild Emojis are not cached and not synced. They can be cached by using `guild.fetch()`. This will also enable them to sync.
* Guild Roles are not cached and not synced. They can be cached by using `guild.fetch()`. This will also enable them to sync.
* Guild Members are not cached. Relevant events contain a temporary Member object or partial instead. Members can be manually cached by using `guild.members.fetch(id)`. This will not cache the user.
* Guild Channels are not cached. Guild Channels can be cached in the guild by fetching it and then linking it using `guild.channels.cache.set(fetchedChannel.id,fetchedChannel)`.
* Guild Channel Permission Overwrites are not cached and not synced. They can only be cached by fetching the guild to enable role syncing, then fetching the channel and linking it to the guild. This will also enable them to sync. This is required to enable many permission checking functions.

Cached Users, Channels and their Member and GuildChannel counterparts get automatically sweeped after 24 hours of inactivity. To disable sweeping of a User or Channel, give them a custom `noSweep` property set to `true`. This will also disable sweeping of their Member and GuildChannel counterparts.

## Events

Most events should be identical to the originals, with a few exceptions. All partial objects will be full objects if they are cached.

| Event | Description |
| ------------- | ------------- |
| message | This event is fired by both new messages and edited messages. It provides a full Message object with a partial Channel object. The messageUpdate event was merged into the message event in order to make it easy for the bot to reply to edited messages. Edited messages can be identified by checking for the existence of .editedTimestamp |
| messageDelete | Provides a partial Message object with a partial Channel object |
| messageDeleteBulk | Provides a Collection of deleted messages as above |
| messageReactionAdd | Provides full Reaction object with partial Message object, and full User object (partial if DM). Does not provide Reaction count nor Reaction users if not cached |
| messageReactionRemove | Provides full Reaction object with partial Message object, and full User object (partial if DM). Does not provide Reaction count nor Reaction users if not cached |
| messageReactionRemoveAll | Provides a partial Message object |
| messageReactionRemoveEmoji | Provides a full Reaction object with a partial Message object. Does not provide Reaction count nor Reaction users if not cached |
| channelCreate | Provides a full Channel object |
| channelUpdate | Provides a NULL old Channel object if not cached and a full new Channel object |
| channelDelete | Provides a full Channel object |
| channelPinsUpdate | Provides a partial Channel object and a Date object |
| roleCreate | Provides a full Role object |
| roleUpdate | Provides a NULL old Role object if not cached and a full new Role object |
| roleDelete | Provides a partial Role object |
| guildBanAdd | Provides a full Guild object and a full User object |
| guildBanRemove | Provides a full Guild object and a full User object |
| guildCreate | Provides a full Guild object without Roles, Emojis, Channels, Members, Presences and VoiceStates |
| guildUpdate | Provides a full old Guild object and a full new Guild object. Does not contain Roles, Emojis, Channels, Members, Presences and VoiceStates unless previously cached or fetched. |
| guildDelete | Provides a full Guild object |
| userUpdate | This event is only fired if the user is cached and when the updated user is active. It provides a full User object |
| guildMemberUpdate | This event is only fired if the member is cached and when the updated member is active. It provides a full Member object |

guildMemberAdd and guildMemberRemove are disabled due to discord's new restrictions when using Intents. They can be enabled either by disabling Intents in the client options or by enabling the priviledged SERVER MEMBERS Intent in the discord developer portal and passing in a new Intents value in the client options. Disabling Intents will massively increase CPU and bandwidth usage. If Intents are disabled or the priviledged Intent is enabled, the following events will be enabled:

| Event | Description |
| ------------- | ------------- |
| guildMemberAdd | Provides a full Member object |
| guildMemberUpdate | Provides a full Member object. Replaces the activity-based version mentioned above |
| guildMemberRemove | Provides a partial Member object |

All other events, except connection events, are not available. Additionally, there might be bugs or inconsistencies due to being unable to test all possible situations that might occur, so thread carefully.

## Extras

Some extra functionality is included:

| Func/Prop | Returns | Description |
| ------------- | ------------- | ------------- |
| client.getInfo() | promise>object | Gather several statistics about the client such as guild count, user count, sharding information, total active (cached) users and channels, websocket pings, uptimes, cpu usage and memory usage |
| message.eval(string) | promise>anything | An eval function compatible with promises, async/await syntax and complex code. Can access the client via `client` and the message object via `this` (should be locked to owners only) |
| message.reply(content,options) | promise>message | This function does the same as message.channel.send() but adds several improvements: handles promises, objects, falsey values and other non-string types, truncates large strings if no split options are provided, automatically caches the channel, author and messages involved, tracks activity for automatic sweeping, logs response times, adds request-response pairing and if possible sends responses as edits when triggered by editing a previous command |
| message.commandResponse | message | The message object that was sent as a response to this command. Only available if it was sent with message.reply() |
| message.commandMessage | message | The message object that triggered this response. Only available if this response was sent with message.reply() |
| message.commandResponseTime | number | Message response time in milliseconds. Only available in response messages if they were sent with message.reply() |
| channel.lastActive | number | Timestamp of the last time this channel was used by the client |
| channel.noSweep | boolean | Whether this channel should be automatically sweeped |
| user.lastActive | number | Timestamp of the last time this user was replied to by message.reply() |
| user.noSweep | boolean | Whether this user should be automatically sweeped |

## About

This project is somewhat experimental, so there might be bugs and broken features especially in untested scenarios (i have tested only features that my bots need). You are encouraged make your own tests with your specific use cases and post any issues, questions, suggestions or contributions you might find.

You can also find me in my [discord](https://discord.gg/BpeedKh) (Tim#2373)