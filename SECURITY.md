# Secure Connect security notes

This page explains, in plain language, what Secure Connect protects you
against, what it does not, and the known weak spots. It describes the
program as it is today (an early release), not plans.

> Secure Connect is new and has **not had an independent security review**.
> Several parts have only been tested with simulated computers (see the
> user guide's "What has been tested"). Do not use it to protect anything
> where a failure would be serious.

## The short version

- Everything between the two computers (picture, mouse, keyboard, password
  check) is **encrypted end to end**. The relay servers that help computers
  find each other cannot read it.
- A host only lets someone in if they know its **password**, or if a person
  at the host presses **Accept**. If a password is set, the correct password
  is enough and nobody has to press Accept; Accept prompts are used only
  when no password is set.
- Your **ID is a public key**, not a secret. People who know it can see that
  your computer is online and can try to connect.
- Whoever is let in can do **anything you could do** at that computer with
  the mouse and keyboard. Only let in people you trust.
- **Clipboard sync and file access are ON by default.** The person in
  control can also read and write the clipboard text and every file your
  user account can reach, directly, without going through the screen. See
  [Clipboard and files](#clipboard-and-files); each can be switched off.
- Both computers must run the same protocol version. This version (protocol
  3) does not connect to older builds (v0.1.0, or earlier test builds with
  protocol 2), in either direction.

## How a connection works

1. Each copy of Secure Connect has a secret key stored on the computer
   (`node.key`). Its matching public key is the computer's **ID**.
2. To connect, the viewer looks up the host's ID. The host publishes a small
   record "this ID can be reached through this relay server" to a public
   lookup service (`dns.iroh.link`, run by number 0, the makers of the iroh
   networking library).
3. The computers first talk through a public **relay server** (also run by
   number 0), then try to switch to a **direct** connection between them when
   the networks allow it. No router port forwarding is needed.
4. The connection uses QUIC with TLS 1.3 encryption. Each side proves it
   holds the secret key for its ID, so you are talking to the computer whose
   ID you entered, and the relay only passes along encrypted data.
5. The host then asks for the password (or, if no password is set and
   Accept prompts are on, the Accept prompt) before
   showing the screen or accepting any input.

## What is protected

**Eavesdropping and tampering on the network.** Picture, mouse and keyboard
data, and the password check, travel encrypted from one Secure Connect to
the other. The relay servers, your internet provider and anyone else on the
network see only encrypted data. Video frames are compressed and sent; the
host never saves a picture of its screen to disk. The viewer keeps one small
preview per saved computer unless that is turned off (see "Card previews"
below).

**Impersonating a host.** The viewer checks that the computer it reaches
holds the secret key that belongs to the ID you typed. Someone who does not
have that key cannot pretend to be that computer.

**"Harvest now, decrypt later".** By default the key exchange uses a hybrid
of classical X25519 and post-quantum ML-KEM-768 whenever both sides support
it (two copies of Secure Connect always should). This is meant to protect
recorded traffic even against a future quantum computer. With the setting
**Require post-quantum encryption** (takes effect after a restart), Secure
Connect refuses, in both directions, any connection to another computer that
did not use the hybrid key exchange.

**Your password.** Secure Connect does not store the password. It stores a
check value made with Argon2id (a deliberately slow, memory-hard function)
in the file `password.verifier`, readable only by your user account. The
password itself is never sent over the network; the viewer proves it knows
it.

**Remembered connection passwords (viewer side).** Remembering a host's
password for one-click connecting is off unless you tick it for that host.
The password is then stored **only** in the operating system's secret
store: the Secret Service (GNOME Keyring, KWallet, …; reached over D-Bus)
on Linux, the Windows Credential Manager (kept on this PC, not roamed) on
Windows. Secure Connect never writes it to a file, and there is no file
fallback: if no secret store is reachable, the option is disabled and the
reason is shown. `hosts.txt` records only *that* a password is remembered.
The password is never printed or logged, and the app wipes its own copies
from memory when it is done with them (copies made by the GUI toolkit while
you type it, or by the store library, may remain until the memory is
reused). **Forget password** and **Forget this computer** delete it from the
store.

**Password guessing.** Lockouts slow guessing down: after 5 wrong passwords
from one computer it must wait 60 seconds, and an overall limit for all
callers together (40 per minute, 60 per hour) triggers longer and longer
pauses of up to one hour.

**Accidental Accept.** The Accept button only works after the prompt has
been visible for one second, and the prompt expires after 60 seconds.

**A connected person changing your access.** The security settings
(Accept prompts, post-quantum setting, password, the clipboard and
file-access switches) are locked while anyone is
connected, so a viewer controlling your computer cannot use the Secure
Connect window to change who may get in.

**Visibility.** While anyone is connected, a coloured bar in the main window
shows how many viewers there are, with an **End all** button.

## What is not protected (honest limits)

**Anyone you let in has full control.** The first connected viewer can use
the mouse and keyboard as if they were sitting at the computer, as your user
account. Secure Connect cannot limit what they do.

**Your ID reveals that you are online.** The ID-to-relay record is public.
Anyone who knows your ID can find out whether your computer is currently
running Secure Connect, and can attempt to connect.

**A patient attacker can lock out password logins.** Because of the overall
guessing limits, someone who knows your ID can keep sending wrong passwords
and keep password logins locked for up to an hour at a time, so you
cannot get in with the password either. Accept prompts are not affected.

**Weak or reused passwords.** Lockouts slow guessing but cannot save a weak
password. Also, the password check is **not tied to the IDs of the two
computers**. This means that if you use the same password on several hosts,
a malicious host you connect to could relay your login to another host with
the same password. And any computer you send a password login to receives
an answer from which it could try to guess the password offline (slowly,
because of Argon2id). So only type a host's password when connecting to
that host's correct ID, and **use a different, long password (12+
characters) for every computer.**

**The password check file is a key.** Anyone who can read
`password.verifier` (for example someone with access to your user account,
an administrator, or a backup of your files) can log in to that host
without knowing the password. Protect it like the password itself. Likewise,
anyone who copies `node.key` can pretend to be your computer (take over its
ID).

**Someone with access to your computer.** Secure Connect does not protect
against malware or other people already using your computer or your user
account. They can read its files, change its settings, or watch your screen
by other means.

**IDs are not post-quantum.** The keys that prove "this is computer X"
(Ed25519) are classical. A future quantum computer could, in principle, be
used to impersonate a computer. The post-quantum part protects the secrecy
of the traffic, not who you are talking to.

**Relay and lookup servers.** By default Secure Connect uses the public
relay and lookup servers of number 0. They cannot read your sessions, but
they see which IDs connect, when, from which IP addresses, and how much data
is sent. They are rate-limited and are not meant for production use; they
may be slow or unavailable. The connections to these servers themselves use
ordinary (not post-quantum) TLS. There is currently no setting in the app to
use your own relay.

**Router changes.** To make direct connections work, the networking library
may ask your router to open a port automatically (UPnP / NAT-PMP), if your
router allows that.

**View-only viewers are not told.** If a second person connects while
someone else is in control, they can watch but their input is silently
ignored. The host's list shows them as "(view only)"; the viewer gets no
message.

**What the viewer sees.** On Linux with X11, the whole screen area of the
session is captured. On Windows, only the main monitor. The viewer sees
whatever is on that screen, including notifications and private windows.

**Windows secure desktop.** On Windows, Secure Connect cannot see or control
the lock screen, UAC ("allow changes") prompts or programs running as
administrator. This also means a viewer cannot approve those prompts.

**Linux Wayland.** Input goes through the desktop's RemoteDesktop portal
(`org.freedesktop.portal.RemoteDesktop`). The controlling viewer's session is
ONE portal session that combines screen sharing with keyboard and pointer
access, so the approval dialog grants **both**: whoever can sign in to this
computer through Secure Connect can then control all windows, not only see
them. The first approval needs a person at the host; it is then remembered in
the file `remotedesktop.token` (owner-only, 0600), so later controlling
sessions start without asking. Treat that file like a key: deleting it
revokes the remembered approval (the next controlling session asks again).
Watch-only viewers (anyone connecting while someone else controls) get a
separate plain screen-sharing session (remembered in `screencast.token`, also
0600) and never a keyboard/pointer grant. When the controlling session ends,
every key and button it still holds is released through the portal before
the portal session is closed. If the portal is missing, or the owner turned
off "remote interaction" in the dialog, input falls back to XTEST, which only
reaches XWayland windows; the host window says so. A cancelled or denied
dialog ends the connection without asking again.
Keys (Enter, arrows, shortcuts, and every key in "Match key positions") are
sent as physical key positions (evdev key codes), so on a non-US host layout
such a key does what that key does in the host's layout, and the host's own
Caps Lock state applies to letters. Typed characters ("Match typed
characters", the default) are sent as keyboard symbols
(`NotifyKeyboardKeysym`, one press and release per character), through the
same queue, time limits and "never replay a stale press, always send a
release" rules as every other portal call. The pointer is aimed at the
shared monitor only; with a wrong `SECURE_CONNECT_PORTAL_POINTER` override on
a scaled display, positions can fall outside it, onto an adjacent monitor
that is not shared (the viewer then clicks there without seeing it).

**Card previews (viewer side).** By default, when a session ends the viewer
saves a small (at most 320 x 180) PNG of the last picture it showed of the
other computer's screen, for that computer's card. It is only taken for
computers you saved (or are being asked to save), and it can contain
whatever was on that screen (documents, messages). It is stored only on the
viewer computer, as an owner-only (0600) file in an owner-only (0700)
`previews` folder in the local (non-roaming) app folder, under a file name
derived from a hash of the ID. It is never uploaded, sent to the host or
logged. Turning off **Save previews of remote screens** deletes all of them
and stops new ones; **Clear all previews** and the per-card **Clear
preview** delete them. Someone with access to your user account (or your
backups) can see them, as with any of your files.

**Password-store requests can wait.** All password-store requests run one
after another on a single background thread. If the store shows an unlock
prompt that is never answered, later requests (including "Forget password")
wait behind it until Secure Connect is restarted; a forgotten computer's
password deletion is retried at the next start.

**A remembered password is one click away.** Anyone who can use your
unlocked user session can connect to a host whose password you let Secure
Connect remember, just as they could use any other remembered password.
Your system password store is usually unlocked when you sign in.

**No audit log.** Apart from the failed-attempt counter shown in the
window, Secure Connect does not keep a record of who connected and when.
The host's **Activity** list (recent sessions and file operations, at most
200 lines) exists only in memory and disappears when Secure Connect closes;
it is not a log you can rely on afterwards.

**Untested parts.** These have not been tested in real conditions yet: two
computers over the real internet, the post-quantum key exchange through the
public relays, real mouse and keyboard control, and anything on Windows
(including the Windows file-system protections described below). The file
manager window has never been looked at on a real screen, no file has been
copied between two real computers, and clipboard sync has only run against a
simulated clipboard. Bugs in these areas are likely to be found.

**Unsigned downloads.** The programs are not code-signed. Check the
SHA-256 checksums published with each release (see the README) to make sure
your download was not altered.

## Clipboard and files

**What a password holder can now do.** With the default settings, anyone
who gets in as the **controlling** viewer (the correct password, or a
person at the host pressing Accept) can, through the file manager:

- read and copy to their own computer **every file the host's logged-in
  user can read** (documents, browser profiles, SSH keys, password-manager
  databases, mail, …);
- create, overwrite (with **Replace**), rename and **delete** files and
  folders **wherever that user may write**. This can create **lasting
  access**: they can add a Secure Connect file of their own, or write to
  `~/.ssh/authorized_keys`, an autostart entry or a shell start-up file
  (`.bashrc` and similar), and so get back in, or run code, after the
  session has ended. **Accept once is not one-time access** when file
  access is on: the person you accepted may have left something behind.
  Turn file access off (below) when that matters;
- read every text the host user copies while the session runs, and put
  text into the host's clipboard.

The one place the file service refuses is Secure Connect's own settings
folder (the host's identity key, password verifier, saved computers,
settings and tokens): nothing in it, or the folder itself, can be listed,
read, copied, created, renamed or deleted from a viewer, whatever link or
spelling is used to reach it (checked by file identity after resolving
links; if the check cannot be done, the request is refused). Everything
else the user owns is reachable.

Files copied to the host are created private to the user (mode 0600 on
Linux); the executable bit is not carried over.

A viewer that reconnects quickly after you pressed **End** can land
view-only for a few seconds, until the host has finished closing the old
session.

They act **as that user account, with its rights, never elevated**: no
administrator or root rights, no access to other users' private files
(beyond what that account can already read). This is much more direct than
screen control, so treat the host password accordingly. Turn the features
off on the host if you do not need them: **Share clipboard text with the
viewer in control** and **Allow file access (browse, copy, rename,
delete) for the viewer in control**, each on its own. They are locked while
anyone is connected and apply from the next session; what a session may do
is fixed when it starts (the host tells the viewer at the start and refuses
anything else; a viewer that tries a switched-off feature is disconnected).

**Only the controlling viewer.** Extra viewers who connect while someone is
in control watch only: they get no clipboard and no file access, and the
host never opens the channel for them.

**The owner-side guard and what it does not protect.** The computer that
owns the files refuses to delete or rename a drive or root folder, the home
folder of the user Secure Connect runs as, any folder containing it, and
the system folders (and folders containing them). It checks this after
following links in the path and by file identity (device and inode on
Linux, volume and file index on Windows), so another path to the same
folder does not get around it; if it cannot check, it changes nothing. This
guard prevents the worst slips (one click deleting the whole home folder).
It is **not** an access-control boundary:

- everything **inside** home (Documents, Desktop, `.ssh`, …) can still be
  deleted or overwritten, and everything readable can be copied away;
- folders inside system folders are not protected (an ordinary account
  usually cannot change them anyway);
- on Linux the change is made relative to the checked folder, so a folder
  swapped into the path after the check cannot redirect it; but if the
  **last** name itself is swapped by a local process at that moment, a
  plain file or link put there is removed (a folder is not). On Windows
  (and Unix systems other than Linux) the check is path-based, so a local
  process of the same user could race it;
- a bind mount of home on the same file system is caught only by identity
  checks that were tested with simulated values (making one needs root);
- the Windows parts (short names, junctions, file identity) have only been
  unit-tested on Linux, never run on Windows.

**Copies cannot overwrite by accident.** Incoming data is written to a new
`name.part` file that the copy itself created (`O_EXCL`; links and
hard-linked files are never written through; an existing `.part` of another
program is left alone and `name.~N.part` is used instead). Only after the
whole file's BLAKE3 hash matches is it renamed to its final name, with a
"no replace" rename unless the user chose **Replace**. Names are cleaned the
same way on both sides (no path separators, control or direction-override
characters, reserved Windows names, …).

**Abuse limits.** File requests are rate-limited (50/s), at most 4 file
operations and 4 copies run per session, at most 16 blocking file
operations host-wide, and each operation has a 10-second deadline. A copy
slower than 4 KiB/s on average after its first minute is stopped. A
controlling viewer can still fill the host's disk (it has write access as
the user anyway); the error says the disk is full.

**Nothing new before sign-in.** The clipboard/file channel and every copy
stream are opened **by the host**, and only after sign-in. The host still
lets an unauthenticated caller open only the two streams it always allowed,
so the amount of data a caller can make the host hold before sign-in is
unchanged (at most 2 MiB per pending connection, 32 MiB across the 16
pending connections allowed).

**A malicious host can make the viewer buffer data.** The viewer accepts up
to 5 + 5 streams from the host it chose to connect to, so a hostile host can
make the viewer hold up to about 10 MiB per connection (16 MiB connection
window). A host can also offer files with any content or name (names are
cleaned and never leave the destination folder) and can see whatever you
copy to it.

**Clipboard may hold passwords.** Anything you copy while a session with
clipboard sync is open goes to the other computer, including passwords
copied from a password manager. Text already on the clipboard when the
session starts is not sent. Clipboard text is limited to 1 MiB, never
written to disk or logs, and hidden in debug output. Switch clipboard sync
off (host or viewer side) if you copy secrets during sessions.

**Two-window relay.** If you have two remote windows open with clipboard
sync, text copied on host A lands in your clipboard and is then sent on to
host B (as with other remote-desktop tools). Close, or turn off **Sync
clipboard** for, windows you do not want to share with.

**Typed text.** In "Match typed characters" mode the viewer sends what you
type as characters (at most 16 per message, no control characters). They
are typed on the host exactly like key presses would be, and only for the
session in control: the host drops all input from view-only sessions before
it reaches the keyboard. Typed characters are never written to a log or to
disk and are hidden in debug output (the latency log only counts events).
When the host cannot type some characters it tells the viewer only how
many, never which. Anything you type in the remote window goes to the other
computer, including passwords; that is the point of remote control.

**What is logged and what is kept.** No file name, path, clipboard text or
file content is written to a log or to disk by these features. The host's
log lines stay as before (session start and end, with the viewer's ID). The
host keeps the **Activity** list in memory only: up to 200 lines such as
"upload finished: /home/you/report.pdf" or "files: deleted /home/you/old",
showing paths (shortened to their last 160 characters) but never
contents. It is gone when Secure Connect closes.

## Good practice

- Prefer **Accept prompts** (with no password set) when someone is at the
  host. A password always lets people in without a prompt.
- Switch off **file access** (and clipboard sharing) on hosts where the
  people who connect do not need them.
- For unattended access, use a **long, unique password** per computer, and
  clear it when you no longer need it.
- Share your ID only with people who need it.
- Keep the files in the Secure Connect folder private, and do not copy
  `node.key` or `password.verifier` to other computers.
- Only let Secure Connect remember passwords for hosts on a computer that
  only you use, and turn off previews if the other screens may show private
  things.
- Watch the "failed sign-in attempts" line. Many failures mean someone is
  guessing; clear the password and use Accept prompts for a while.
- Turn on **Require post-quantum encryption** if you want to be sure every
  connection uses it.
- Close Secure Connect when you do not need it: nobody can connect then.

## Reporting a security problem

Please report security problems privately to the owner through the contact
details on the project's GitHub page, rather than in a public issue. Never
include passwords, `node.key` or `password.verifier` files in any report.
