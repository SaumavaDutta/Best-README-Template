<div align="center">
 <img src="https://raw.githubusercontent.com/ftrapture/blue.ts/main/assets/banner.png" alt="Blue.ts Banner" width="800">
 <h1>Blue.ts</h1>
 <p>A powerful, simple, and effective stable Lavalink client developed in TypeScript.</p>
 <p>
   <a href="#features">Features</a>
   •
   <a href="#requirements">Requirements</a>
   •
   <a href="#installation">Installation</a>
   •
   <a href="#quickstart">Quick Start</a>
   •
   <a href="#documentation">Documentation</a>
   •
   <a href="#issues">Issues</a>
   •
   <a href="#license">License</a>
 </p>
 <br>
</div>

<div>
 <h2 id="features">✨ Features</h2>
 <ul>
   <li>💿 Audio playback with fully-featured controls</li>
   <li>🎚️ Filters support</li>
   <li>🔍 Best search engines: YouTube, Spotify, SoundCloud</li>
   <li>🔄 Autoplay feature</li>
   <li>⚡ Faster, simple, and stable client</li>
   <li>🆙 Supports the latest Lavalink version: 4.0.4</li>
   <li>🌐 Compatible with discord.js, eris, oceanicjs</li>
   <li>🔌 Plugins support</li>
 </ul>
</div>

<div>
 <h2 id="requirements">⚙️ Requirements</h2>
 <ul>
   <li><a href="https://nodejs.org/en/download">Node.js</a> Version: >= 16.9.0</li>
   <li>A Lavalink server, here are some free Lavalink servers: <a href="https://lavalink.darrennathanael.com/">Click Me</a></li>
   <li>Discord Bot <a href="https://discord.com/developers/applications">token</a> to get started</li>
 </ul>
</div>

<div>
 <h2 id="installation">📥 Installation</h2>
 <pre><code>
# npm install
npm install blue.ts

# yarn install
yarn add blue.ts
 </code></pre>
</div>

## Quick Start With Djs 13

```javascript
const { Client, Intents } = require("discord.js");
const { Blue, Events, Types, Library } = require("blue.ts");

const client = new Client({
  failIfNotExists: false,
  allowedMentions: {
    parse: ['roles', 'users'],
    repliedUser: false,
  },
  partials: [
    'MESSAGE',
    'CHANNEL',
    'REACTION',
    'GUILD_MEMBER',
    'USER'
  ],
  intents: [
    Intents.FLAGS.GUILDS,
    Intents.FLAGS.GUILD_MEMBERS,
    Intents.FLAGS.GUILD_BANS,
    Intents.FLAGS.GUILD_EMOJIS_AND_STICKERS,
    Intents.FLAGS.GUILD_INTEGRATIONS,
    Intents.FLAGS.GUILD_WEBHOOKS,
    Intents.FLAGS.GUILD_INVITES,
    Intents.FLAGS.GUILD_VOICE_STATES,
    Intents.FLAGS.GUILD_MESSAGES,
    Intents.FLAGS.GUILD_MESSAGE_REACTIONS,
    Intents.FLAGS.GUILD_MESSAGE_TYPING,
    Intents.FLAGS.DIRECT_MESSAGES,
    Intents.FLAGS.DIRECT_MESSAGE_REACTIONS,
  ],
  presence: {
    activities: [
      {
        name: 'Blue.js',
        type: "LISTENING",
      },
    ],
    status: 'online',
  }
});

const nodes = [
  {
    host: "localhost",
    port: 2333,
    password: "youshallnotpass",
    secure: false
  }
];

const options = {
  spotify: {
    client_id: "CLIENT_ID",  //spotify client ID
    client_secret: "CLIENT_SECRET" //spotify client Secret
  },
  autoplay: true,
  version: "v4",
  library: Library.DiscordJs
};
client.manager = new Blue(nodes, options);

client.on("ready", async () => {
  console.log("Client is ready!");
  client.manager.init(client);
});

client.manager.on(Events.nodeConnect, (a, b) => {
  console.log(b);
});

client.manager.on(Events.nodeDisconnect, (a, b) => {
  console.log(b);
});

client.manager.on(Events.trackStart, async(a, b) => {
   const guild = await client.guilds.fetch(a.guildId).catch(() => null);
   if(!guild) return;
   const channel = await guild.channels.fetch(a.textChannel).catch(() => null);
   if(!channel) return;
   return channel.send(`Track Started: [${b.title}](${b.uri})`);
});

//logging
/*
client.manager.on(Events.api, (data) => {
  console.log(data);
});*/

client.on("messageCreate", async (message) => {
  if (message.author.bot || !message.guild || !message.channel) return;

  const prefix = ">";
  let player = client.manager.players.get(message.guild.id);

  if (!message.content.toLowerCase().startsWith(prefix)) return;

  const args = message.content.slice(prefix.length).trim().split(/ +/);
  const cmd = args.shift()?.toLowerCase();

  if (cmd == "play") {
    if (!message.member?.voice.channel) return message.reply("you must be in a voice channel");

    const query = args.slice(0).join(" ");

    if (!query) return message.reply("provide the query");

    if (!player)
      player = await client.manager.create({
        voiceChannel: message.member.voice.channel.id,
        textChannel: message.channel.id,
        guildId: message.guild.id,
        selfDeaf: true,
        selfMute: false
      });

    const res = await client.manager.search({ query: query }, message.author).catch(() => null);
    if (!res) return message.reply("song not found");
    if (res.loadType == Types.LOAD_SP_ALBUMS || res.loadType == Types.LOAD_SP_PLAYLISTS) {
      player.queue.add(...res.items);
    } else {
      player.queue.add(res.tracks[0]);
    }
    if (!player.queue?.current)
      player.play();
    return message.reply("queued song");
  }

  if (cmd == "skip") {
    if (!player || !player.isConnected) return message.reply("player not initialized yet.");
    if (player.queue.size() < 1 && !player.playing) {
      player.disconnect();
      return message.reply("there's no song to skip.");
    }
    player.stop();
    return message.reply("skipped to the next song.");
  }

  if (cmd == "stop") {
    if (!player || !player.isConnected) return message.reply("player not initialized yet.");
    player.disconnect();
    return message.reply("stopped the song, and left the vc");
  }

  if (cmd == "replay") {
    if (!player || !player.queue.current) return message.reply("Nothing playing rn.");
    player.seek(0);
    return message.reply("alr playing from the beginning.");
  }

  if (cmd == "seek") {
    if (!args[0]) return message.reply("provide the position");
    if (!player || !player.queue.current) return message.reply("Nothing playing rn.");
    player.seek(args.slice(0).join(" "));
    return message.reply("alr player position sets to " + player.position);
  }

  if (cmd == "8d") {
    if (!player || !player.queue.current) return message.reply("Nothing playing rn.");
    if(player.filter.is8D)
      player.filter.set8D(false);
    else
      player.filter.set8D(true);
    return message.reply(`filter has been added`);
  }
  
  if (cmd == "clear") {
    if (!player || !player.queue.current) return message.reply("Nothing playing rn.");
    player.filter.clearFilters();
    return message.reply(`filters has been cleared`);
  }
});

client.login("TOKEN");
```

## Documentation

Check out the [documentation](https://github.com/ftrapture/blue.ts/wiki) for detailed usage instructions and examples.

## Issues

For any inquiries or issues, feel free to [open an issue](https://github.com/ftrapture/blue.ts/issues) on GitHub.

## License

This project is licensed under the ISC License.

By [Rapture](https://github.com/ftrapture)
