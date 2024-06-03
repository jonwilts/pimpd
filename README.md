# pimpd
**P**impD **i**s a **M**usic **P**layer **D**aemon - a music player for command line people.

PimpD is a music player with a very simple premise - it reads a playlist file, and then plays the tracks in the file sequentially. That might sound like the most underwhelming thing ever, but bear with me... 

PimpD can run in the foreground in a terminal, or it can fork off and run as a daemon, or it can be started by systemd, initd, etc. In it's default configuration it won't start playing until told to, so it can run happily in the background.

PimpD monitors the playlist file for changes, and attempts to handle changes to that file in a "natural" (but configurable) way. So, whatever you want to listen to, you just need to make changes to that playlist file and PimpD will handle them in real time. Some examples might help here. Assume your playlist is in `~/.config/pimpd/playlist.m3u`:

- To shuffle the current playlist:
  
```
% cat ~/.config/pimpd/playlist.m3u | shuf > ~/.config/pimpd/playlist.m3u.new
% mv ~/.config/pimpd/playlist.m3u.new ~/.config/pimpd/playlist.m3u
```

- To wipe the current playlist:

```
% echo "" > ~/.config/pimpd/playlist.m3u
```

- To add an album to the end of the current playlist:

```
% cd ~/Music/artist/album
% find "${PWD}" -type "*.mp3" >> ~/.config/pimpd/playlist.m3u
```

- To back up a playlist, and then listen to a previously backed-up playlist:

```
% cp ~/.config/pimpd/playlist.m3u ~/Playlists/SundayMorning.m3u
% cp ~/Playlists/Workout.m3u ~/.config/pimpd/playlist.m3u
```

- And you can really go to town. Here's adding files to the playlist with fzf and kitty terminal:

https://github.com/jonwilts/pimpd/assets/9073828/15a1f8c0-64b7-45fe-aaa3-46629348348e

The point is, the playlist is just a text file, and any way you want to add/update/remove tracks from that playlist is up you.

PimpD ships with a number of example commands that can be accessed using the `pimp` command. As of the time of writing the following commands are shipped:
- **add** adds music from an index file using fzf, as shown in the video above.
- **back** go back a track (or go back to the start of this track. Configurable).
- **end** shuts down PimpD.
- **next** skip to the next track.
- **pause** pauses playback. *Play* resumes from the same point.
- **play** starts/resumes playback.
- **random** adds random tracks from your collection to the playlist.
- **reindex** rebuilds an index of your music; makes *add* & *random* faster.
- **shuffle** shuffles the current playlist.
- **stop** stops playback. *Play* will resume from the start of the track.
- **wipe** clears the playlist.

Note that these are really meant as examples, though they may well work for you. Tweak them / rewrite them to suit your setup.

I said that PimpD attempts to handle changes to the playlist file "naturally". What I mean by that is, whatever changes you make to the file won't interrupt the currently playing track. If the currently playing track is still in the playlist after any changes, PimpD plays the new playlist from the track after the current one (this is configurable). If the currently playing track is not in the updated playlist, then PimpD will start at the beginning of the playlist, and will not play that track again. When PimpD reaches the end of the playlist, it loops back to the beginnging (configurable).

Also, for those of you whose desktop environment utilizes DBus (Gnome, KDE, XFCE, maybe others?), PimpD implements the [base & Player MPRIS interfaces](https://specifications.freedesktop.org/mpris-spec/latest/), which means desktop integration should Just Work. PimpD works best in these environments.

## Under the Covers

PimpD uses libmpv for playback, and most of the logic in PimpD involves keeping libmpv's own internal playlist in sync with PimpD's. This means PimpD can play any audio format that libmpv supports, which is a lot. PimpD works best with DBus; without DBus, controlling playback requires communicating with libmpv directly using fifos, and PimpD then has to figure out what's changed by listening to signals issued by libmpv. With DBus, playback control is handled by PimpD directly.

PimpD is written in Perl, and makes use of a number of Perl modules; see **Installation** below.

## Installation

### Dependencies

PimpD relies on the following software:
- **Perl 5** a recent build.
- **libmpv** normally installed as part of the **mpv** package.
- **dbus-send** if you're in a DBus environment. Usually installed as part of DBus.
- **socat** if you're note in a DBus environment. Simplifies communicating with libmpv via fifo.
- Optional: **fzf** for selecting albums/tracks from a list or from the output of find.

Perl Libraries: These can all be installed via `cpan` with one exception.
- **AppConfig**
- **File::HomeDir**
- **File::Monitor**
- **Log::Log4Perl**
- **MPV::Simple**
- **Proc::Daemon**
- Optional: **Net::DBus** *if using a DBus environment*
- Optional: **Log::JournalD** *Add JournalD support to Log4Perl. Note that the cpan version is old, so install by: `cpanm https://github.com/lkundrak/perl-Log-Journald/tarball/master`






