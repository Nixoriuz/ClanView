# Clan Tracker

A self-hosted clan map and shared bank for Old School RuneScape clans. Clan members install the
**Group Ironmen Tracker** plugin from the RuneLite Plugin Hub and point it at your server; their
position, bank, inventory, skills and more then show up on your clan's website.

Built on [Group Ironmen Tracker](https://github.com/christoabrown/group-ironmen) by Christopher
Brown, reworked for a whole clan instead of a 5-player group. No extra RuneLite plugin is needed.

## Features

- **Clan map**: everyone's position, live, on the world map. Players on a Sailing boat get a boat
  marker. Click an entrance circle to jump to where it leads (circles can be hidden per browser).
- **Clan Bank**: all shared banks together, with seed vault, potion storage and Dizana's quiver.
  Each item lists who holds it, biggest holders first.
- **Members**: skills, inventory, equipment, quests, diaries and collection log per player.
- **Graphs**: skill XP over a day, month or year.
- **Players choose what they share** with words in their plugin's Group Token: `NoBank`,
  `NoLocation`, `NoClog` (collection log). Everything else is shared. An admin can override this per
  player (always shared / always hidden), for players who can't get it to work.
- **Access question** before the clan pages (for example "What's the clan password?"). Each browser
  that answers is remembered for a year. Players visiting from the same internet connection their
  plugin syncs from get straight in, and so can your home network.
- **Admin tools** on the Settings page: the plugin address, online time, access question, and
  approving or removing members.
- Members who haven't synced for 90 days (configurable) are removed automatically.

## Install

You need a machine that runs Docker with Compose (a NAS, a home server, a VPS). Download the latest
release zip from this repository's **Releases** page. It holds:

```
server               the server program (Linux x86-64)
public/              the website
docker-compose.yml
```

1. Put `server` and `public` in a folder on the machine, for example `/srv/clan-tracker`.
2. Edit `docker-compose.yml`:
   - Under `volumes`, replace `/path/to/clan-tracker` with that folder.
   - Replace every `CHANGE_ME` (the server won't start until you do). Use the same database
     password in both places.
   - Set `CLAN_NAME` to your clan's name.
3. Start it: `docker compose up -d` in the folder with `docker-compose.yml`.
4. Open `http://<your-server>:8080`. On the first start the server downloads the map tiles and item
   icons (about 150 MB) into `public/` by itself.
5. Open **Settings**, unlock the admin tools with your `ADMIN_PASSWORD` and set the
   **Plugin address**: your public IP address or host name and the port players reach the server
   on, for example `http://203.0.113.10:8080`.
6. On your router, forward that port to the server so clanmates can reach it from the internet.

### TrueNAS SCALE (no command line)

1. Copy `server` and `public` into a dataset or share, for example
   `/mnt/pool/apps/clan-tracker`.
2. **Apps → Discover Apps → Custom App → Install via YAML**, paste `docker-compose.yml`, set the
   folder under `volumes`, replace every `CHANGE_ME`, and **Save**.
3. Continue from step 4 above.

### Updating

Replace `server`, `public/index.html` and `public/app.js` with the ones from the new release and
restart the app. Leave the rest of `public/` (map tiles, icons) as it is. The database updates
itself on start; members, banks and history are kept. Visitors may need to press Ctrl+F5 once.

## For players

The website's **Plugin setup** page explains this too, with copy buttons.

1. Install **Group Ironmen Tracker** from the RuneLite Plugin Hub.
2. In its settings, under **Self Hosted Config**, set **Server base URL override** to the address on
   the Plugin setup page.
3. Fill in **Group Name** and **Group Token**. Both need something in them, or the plugin sends
   nothing. Players in a real group ironman group should write down their group's name and token
   first: they need them to switch back.
4. Everything is shared by default. Words in the Group Token hide things (any order, capitals
   don't matter):

   | Group Token contains | Hides |
   |---|---|
   | `NoBank` | the bank (with seed vault and potion storage) |
   | `NoLocation` | the spot on the map |
   | `NoClog` | the collection log |
   | `NoBank-NoLocation-NoClog` | all three |

5. Log in on a normal world (not seasonal, Deadman, PvP Arena, beta or tournament worlds).

The plugin only sends when something changes and sends nothing when a player logs out, so a player
shows as online for a few minutes (the **Online for** setting) after their last update.

## Settings (docker-compose.yml)

| Variable | Default | What it does |
|---|---|---|
| `CLAN_NAME` | (required) | Clan name shown on the website |
| `ADMIN_PASSWORD` | (none: admin tools off) | Password for the Settings page admin tools |
| `ACCESS_QUESTION` | `What's the clan password?` | Question visitors answer, used on the first start |
| `ACCESS_ANSWER` | (none: question off) | Its answer, used on the first start. Capitals, extra spaces and a final `.` or `!` don't matter |
| `PLUGIN_URL` | (this website's address) | Address shown to players until one is saved on the Settings page |
| `INACTIVE_DAYS` | `90` | Remove players who haven't synced for this many days |
| `ONLINE_MINUTES` | `5` | Minutes a player counts as online after their last update |
| `GROUP_TOKEN` | (any) | If set, every player must use exactly this Group Token (the sharing words then don't work) |
| `GIM_PLUGIN_SHARE_BANK` | `true` | Whether banks show in the Clan Bank unless a player opts out |
| `VIEW_PASSWORD` | (none) | Optional extra password that also opens the clan pages |
| `STATIC_DIR` | (none: API only) | Folder with the website, `/app/public` in the compose file |
| `PG_HOST` `PG_PORT` `PG_USER` `PG_PASSWORD` `PG_DB` | | Database connection (or `DATABASE_URL`) |
| `PORT` | `8080` | Port inside the container |
| `LOG_LEVEL` | `info` | `debug`, `info`, `warn` or `error` |
| `ASSETS_URL` | the original project's archive | Where map tiles and icons are downloaded from |

The question, its answer, the plugin address and the online time can be changed on the Settings
page later; what's saved there wins over the compose file. Changing the answer makes every
remembered browser answer again.

## Security and privacy

- **What the server stores**: what the plugin sends (name, position, items, skills, quests,
  diaries, collection log) for players who use it, and the settings.
- **The "same connection" check**: to let players in without the question, the server keeps a
  salted SHA-256 fingerprint of the internet address each player's plugin syncs from, for 30 days.
  Real addresses are not stored or logged.
- **Guessing limits**: 10 wrong answers to the question, or 20 wrong admin passwords, from one
  address within 10 minutes make that address wait 10 minutes.
- **Behind a proxy**: behind Cloudflare Tunnel or a reverse proxy on your network, the server reads
  the visitor's address from `CF-Connecting-IP`, `X-Real-IP` or `X-Forwarded-For`. It only trusts
  these headers when the connection itself comes from a private address, so they can't be faked
  from the internet. Settings shows the address the server sees for you ("You: ..."): if that is a
  private address for everyone, untick **Let the home network in without the question**.

## Building from source

Only needed if you change the code.

```
cd server && cargo test && cargo build --release    # -> server/target/release/server
cd site && npm install && node build.js            # -> site/public/app.js, site/public/index.html
```

The release `server` is built on Linux and needs glibc 2.34 or newer (Debian bookworm has 2.36).
`site/app.src.js` is the website's source; `build.js` minifies it and inlines it, with the CSS, into
`index.html`. `tools/plugin_simulator.py` sends requests the way the plugin does, for testing.

## Credits and license

- [Group Ironmen Tracker](https://github.com/christoabrown/group-ironmen) (server and website) and
  the [RuneLite plugin](https://github.com/christoabrown/group-ironmen-tracker) by Christopher
  Brown, BSD 2-Clause. This project keeps that license; see `LICENSE`.
- Map tiles and item icons are downloaded from the original project on first start.
- Old School RuneScape is a trademark of Jagex Ltd. This project is not affiliated with Jagex.
