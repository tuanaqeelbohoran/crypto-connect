# Secure Connect user guide

Secure Connect lets you see and control another computer's screen over the
internet, or let someone else see and control yours. The connection is
encrypted from one computer to the other.

> **Early release.** Secure Connect is new and has had limited real-world
> testing. In particular, the Windows version has **never been run on a real
> Windows PC** yet, and a full test between two computers over the internet
> has not been done. Please try it on computers that do not matter before you
> rely on it. The section [What has been tested](#what-has-been-tested) lists
> exactly what is known to work.

## Contents

1. [What you need](#what-you-need)
2. [Starting Secure Connect](#starting-secure-connect)
3. [The main window](#the-main-window)
4. [Your ID](#your-id)
5. [Deciding who may connect](#deciding-who-may-connect)
6. [Connecting to another computer](#connecting-to-another-computer)
7. [Controlling the other computer](#controlling-the-other-computer)
8. [Keyboard layouts](#keyboard-layouts)
9. [Ending sessions](#ending-sessions)
10. [Settings](#settings)
11. [Linux with Wayland: approving screen sharing](#linux-with-wayland-approving-screen-sharing)
12. [Where Secure Connect keeps its files](#where-secure-connect-keeps-its-files)
13. [Troubleshooting](#troubleshooting)
14. [What Secure Connect does not do (yet)](#what-secure-connect-does-not-do-yet)
15. [What has been tested](#what-has-been-tested)
16. [Licence](#licence)

Two words used throughout:

- The **host** is the computer being viewed and controlled.
- The **viewer** is the computer you sit at to look at and control the host.

Every copy of Secure Connect can be both at once: it is always ready to be a
host, and you can use it to connect to other computers at the same time.

## What you need

**Linux**

- A 64-bit (x86_64) PC running Ubuntu 24.04 or newer, or another Linux with
  **glibc 2.38 or newer** (for example Debian 13 or Fedora 39 and later).
  Ubuntu 22.04 and older are **not** supported: the program will not start
  there.
- A normal desktop (X11 or GNOME on Wayland). Two small libraries are
  needed, and a standard Ubuntu desktop already has them: the PipeWire
  library (`libpipewire-0.3-0t64` on Ubuntu 24.04) and `libxkbcommon0`.
- For Wayland screen sharing, PipeWire and xdg-desktop-portal must be
  running (they are, on a standard Ubuntu desktop).

**Windows**

- A 64-bit Windows 10 or Windows 11 PC.
- Nothing else to install: the program is a single file.

**Both**

- An internet connection. You do **not** need to change any settings on your
  router (no "port forwarding").
- **The same generation of Secure Connect on both computers.** This version
  (with clipboard sync, the file manager and keyboard layouts) speaks
  connection protocol version 3. Copies built before it, including the
  published v0.1.0 downloads and earlier test builds (protocol 2), cannot
  connect to it and it cannot connect to them: the attempt fails with
  "Connection failed: unsupported protocol version …". Update **both**
  computers.

## Starting Secure Connect

There is no installer. You download one file and run it.

### Linux

1. Download `secure-connect-linux-x86_64` from the project's Releases page.
2. Optionally, check that the download is genuine (see the README's
   "Verify your download" section).
3. Make it runnable, then start it. In a terminal, in the folder where you
   saved it:

   ```sh
   chmod +x secure-connect-linux-x86_64
   ./secure-connect-linux-x86_64
   ```

   You can also make it runnable in your file manager (Properties, then
   "Allow executing file as program") and double-click it.

### Windows

1. Download `secure-connect-windows-x86_64.exe` from the project's Releases
   page.
2. Optionally, check that the download is genuine (see the README).
3. Double-click it. Because the program is new and not signed, Windows
   SmartScreen may warn you about an unrecognised app. If you trust where
   you downloaded it from, choose "More info", then "Run anyway".

### Headless commands

For scripts or a quick check you can run it from a terminal with:

- `--version`: prints the version and exits.
- `--print-id`: prints this computer's ID and exits (it creates the ID first
  if there is none yet).

On Windows these print into the Command Prompt or PowerShell window you
started them from. (This Windows behaviour has not been tested yet.)

## The main window

Secure Connect has one main window with two parts:

- **This computer** (the top part): your ID, your access settings, any
  warnings, and the list of people connected to you.
- **Connect to another computer** (the bottom part): cards for the computers
  you have saved (one click connects), and a box where you type the ID of a
  computer you want to reach.

Each computer you connect to opens in its own extra window.

## Your ID

Next to **Your ID:** you will see a long code of letters and digits, split
into groups by dashes (it starts like `abcd-efgh-…` and has 52 letters and
digits). This is the address of your computer. Press **Copy** (or **Copy my
ID** next to the **Connect** button) to copy it, and send it to the person
who should connect to you.

Things to know about the ID:

- It stays the same every time you start Secure Connect.
- It is not secret in the strict sense, but treat it with care: anyone who
  knows it can see whether your computer is online, and can try to connect
  (they still need your password or your approval).
- It is tied to a key file on your computer. If that file is deleted, you get
  a new ID and everyone must be given the new one. See
  [Where Secure Connect keeps its files](#where-secure-connect-keeps-its-files).

While Secure Connect starts you may briefly see "Starting the network…".

## Deciding who may connect

Nobody can connect to your computer until you choose at least one of the two
ways below. Until then the window says *"Nobody can connect: set a password
or turn on Accept prompts."*

### Attended: "Ask me to accept each connection"

Tick **Ask me to accept each connection**. When someone tries to connect, an
**Incoming connection** box pops up showing the ID of the computer that
wants to view and control yours, with **Accept** and **Deny** buttons.
This only happens when no password is set (see "Both at once" below).

- The Accept button only works after the box has been visible for about one
  second. This stops you accepting by accident with a double-click.
- If nobody answers within 60 seconds, the connection is refused.
- Use this when someone is sitting at the computer, for example when you help
  a friend.

### Unattended: a password

Type a password next to **Unattended password:** and press **Set** (or
Enter). Anyone who knows your ID **and** this password can then connect,
even when nobody is at the computer.

- The password must be at least 8 characters. Use **12 or more**; the app
  warns you about short ones.
- **Use a different password for every computer.** Do not reuse a password
  from another host or from any other account. (The [security
  notes](SECURITY.md) explain why.)
- A new password applies to new connections; people already connected stay
  connected.
- Press **Clear** to remove the password. Then nobody can connect with a
  password any more.
- Secure Connect never stores your password itself, only a scrambled check
  value. But anyone who can read that file on your computer could use it to
  log in, so treat the file like the password itself.

### Both at once

If you set a password **and** tick "Ask me to accept", the right password is
enough: a person who knows it connects without you pressing anything. The
Accept box is only shown when **no password** is set. A wrong password is
refused and never shows the box. To be asked every time, clear the password.

### Wrong passwords and lockouts

To stop people guessing your password, Secure Connect slows them down:

- After 5 wrong passwords from the same computer, that computer must wait 60
  seconds.
- There is also an overall limit for everyone together (40 tries a minute and
  60 an hour). If it is reached, password logins are paused for a while, up
  to one hour.
- The window shows how many sign-in attempts failed, when the last one was,
  and from which ID.

Downside: someone who knows your ID can keep your password logins locked by
guessing on purpose. Accept prompts still work during a lockout.

### How many people at once

Up to 5 people can be connected to one computer at the same time.

## Connecting to another computer

1. In **Connect to another computer**, paste the other computer's ID into
   **ID:** (the **Paste** button next to the field puts the clipboard text
   there).
2. If that computer uses a password, type it into **Password:**. Otherwise
   leave it empty.
3. Tick **Remember this ID** if you want Secure Connect to fill it in next
   time.
4. Press **Connect**.

A new window opens. You will see "Connecting…", then "Connected. Waiting for
the first picture…", then the other computer's screen. If the other side
uses Accept prompts, someone there has to press Accept first (they have 60
seconds).

The main window lists each window you opened as
"Window to <ID>: connecting… / connected / failed / disconnected".

### Saved computers (cards)

The first time a connection to a new computer works, a small box appears in
the bottom-right corner: **Save this computer?** Give it a name (it suggests
"Computer 1", "Computer 2", …) and press **Save**, or press **Not now** (it
will not ask again until you restart Secure Connect). Up to 200 computers can
be saved.

Each saved computer is a card under **Saved computers**, most recently used
first. A card shows a small picture of that computer's screen from your last
session (or a coloured square with its initials), its name, a short form of
its ID (first 6 and last 4 characters; hover over the card for the full ID)
and when you last connected. A 🔑 means its password is remembered.

**Click a card to connect.**

- If that computer uses Accept prompts only, it connects at once.
- If it needs a password and you let Secure Connect remember it, it connects
  at once with the remembered password (your system may first ask you to
  unlock its password store).
- Otherwise a small **Password needed** box asks for it. Tick **Remember
  password for this computer** if you want the next click to connect
  straight away. If the remembered password stops working (it was changed on
  the other computer), the box appears again so you can type the new one.

**Right-click a card** for: **Connect**, **Rename…**, **Forget password**
(deletes the remembered password, keeps the card), **Clear preview**
(deletes its picture) and **Forget this computer** (deletes the card, its
picture and its remembered password).

**Remembered passwords** are kept only in your system's password store: the
login keyring (Secret Service, for example GNOME Keyring or KWallet) on
Linux, the Windows Credential Manager on Windows. They are never written to
a file by Secure Connect. Remembering is off unless you tick it for a
computer. If no password store is available (for example a Linux desktop
without a keyring), the checkbox is greyed out with the reason, and you type
the password each time.

If your password store is locked, the system shows its own unlock dialog.
Until you answer it, the card shows "Unlocking password…" and other
remembered-password actions (saving, forgetting) wait behind it; if the
dialog is dismissed or never appears, they can stay waiting until you
restart Secure Connect. Typing the password in the dialog still works.

**Previews.** When a session ends (or you close its window, or quit), a small
picture (at most 320 x 180 pixels) of the last thing you saw of the other
screen is saved for that computer's card. It can show whatever was on that
screen, so:

- untick **Save previews of remote screens** (under the cards) to stop
  saving them; this also deletes all existing previews;
- press **Clear all previews** to delete them all, or use **Clear preview**
  on one card.

Previews are kept only on this computer, readable only by your user account,
and are never sent anywhere.

## Controlling the other computer

Click inside the remote picture and use your mouse and keyboard as usual. The
picture is scaled to fit the window.

What works and what doesn't:

- **Mouse**: moving, left, right and middle buttons, and the scroll wheel.
- **Keyboard**: normal typing. By default what you type appears on the other
  computer even if the two computers use different keyboard layouts; see
  [Keyboard layouts](#keyboard-layouts) for how this works and its limits.
  - The **Windows key**, **Alt+Tab** and **Ctrl+Alt+Del** are **not** sent to
    the other computer.
  - With **Match typed characters**, holding a letter or other character key
    repeats it as your computer repeats it. Other keys (arrows, Backspace,
    …) are pressed and released once; whether holding them repeats depends
    on the other computer, so press them several times if it does not.
  - Ctrl+C, Ctrl+X and Ctrl+V are sent as key presses, so they copy and paste
    **inside** the other computer. Copied **text** also travels between the
    two computers automatically; see [Clipboard sync](#clipboard-sync).
- **Only the first person connected can control.** If more people connect at
  the same time, they can watch but their mouse and keyboard do nothing.
  They are **not** told this on their side; on the host the list shows them
  as "(view only)".

The picture is sent at up to 30 frames per second. On a slow connection it
may look blurry or skip; Secure Connect drops old frames rather than falling
behind.

## Keyboard layouts

Your keyboard uses your layout (for example German, French or US), and the
other computer uses its own. The **Keyboard** setting decides what happens
in between. You find it in the connect box of the main window and at the top
of every remote window; it is saved and applies at once, also in open
windows.

**Match typed characters** (the default). What you type is what appears on
the other computer, whatever its layout. Type `z` on a German keyboard and a
`z` appears, even if the other computer has a US layout; `é`, `ü`, `ß`, `€`
and characters you make with **AltGr** (for example `@` with AltGr+Q on a
German keyboard) arrive as those characters.

- **Shortcuts are still shortcuts.** Anything typed with **Ctrl**, **Alt** or
  **Cmd** held goes as a key press, named by your keyboard: Ctrl+Z means "undo"
  even when the Z key sits somewhere else on your layout.
- **Enter, Tab, Backspace, Esc, Delete, the arrows, Home/End, Page Up/Down
  and the F-keys** are sent as keys, as before. **Shift, Ctrl and Alt**
  reach the other computer together with the keys, clicks and wheel turns
  they belong to (and on their own when you press and release one alone),
  but never together with typed characters.
- If the other computer cannot type a character (it has no way to make it),
  the remote window says "N characters could not be typed on the other
  computer". It never says which characters.
- Programs that read raw keys instead of text (many games, some remote or
  virtual-machine windows) may not see characters sent this way. Use **Match
  key positions** for them.

**Match key positions.** Every key acts like the key in the **same place**
on a US keyboard, and the other computer's own layout decides what it types.
With different layouts, letters and symbols may then come out differently
(on a German keyboard, the key labelled Z types `y` on a US computer). Use
this for games (W/A/S/D stay where your fingers are) and for shortcuts that
depend on where a key is.

**Not supported** (in either mode, honestly):

- **Dead keys** (accent keys such as `´` or `^` that change the next letter):
  they are not supported. In "Match typed characters" the accent key itself
  is not sent, and the finished letter (`é`) usually arrives if your
  computer composes it, but this has not been checked on real computers.
- **Input methods (IME)** for Chinese, Japanese, Korean and similar
  composition: not supported; composed text is not sent.
- The extra key next to left Shift on many European keyboards (`<` `>` `|`)
  works for typing in "Match typed characters" but has no position in
  "Match key positions".
- On a Linux (Wayland) host, that computer's **Caps Lock** state applies to
  keys sent as keys. On a Linux host, whether every character can be typed
  depends on the desktop; GNOME may silently skip characters that its
  current layout cannot make (then no notice appears). This has not been
  tested on a real desktop yet.

## Ending sessions

- **As the viewer**: close the window that shows the other computer.
- **As the host**: while anyone is connected, a coloured bar at the top of
  the main window says how many viewers are connected. Press **End** next to
  a person in **Connected viewers** to disconnect them, or **End all** in the
  bar to disconnect everyone.
- Closing Secure Connect on either side ends its connections.

## Settings

All settings are in the **This computer** part of the main window:

| Setting | What it does |
| --- | --- |
| **Ask me to accept each connection** | Shows an Accept/Deny box for incoming connections. Used only when no password is set; with a password, the correct password is enough. |
| **Require post-quantum encryption (refuse peers that don't support it)** | See below. Takes effect after you restart Secure Connect. |
| **Unattended password** with **Set** / **Clear** | Lets people connect with a password while nobody is at the computer. |
| **Share clipboard text with the viewer in control** | Host side, on by default. Lets the person controlling this computer exchange copied text with it. See [Clipboard sync](#clipboard-sync). Locked while someone is connected; applies to new sessions. |
| **Allow file access (browse, copy, rename, delete) for the viewer in control** | Host side, on by default. Lets the person controlling this computer use the file manager on it. See [File manager](#file-manager). Locked while someone is connected; applies to new sessions. |
| **Remember this ID** (in the connect box) | Saves the last ID you connected to, so it is filled in next time. |
| **Sync clipboard** (in the connect box) | Viewer side, on by default. Exchange copied text with the computers you connect to (if they allow it). Turning it off stops it at once, also in open windows. |
| **Keyboard** (in the connect box and in each remote window) | Viewer side. **Match typed characters** (default): what you type appears on the other computer whatever its layout. **Match key positions**: keys act like the key in the same place on a US keyboard. See [Keyboard layouts](#keyboard-layouts). |
| **Save previews of remote screens** (under the cards) | Keeps a small picture of each saved computer's screen from your last session. On by default; turning it off deletes them. See [Saved computers](#saved-computers-cards). |

**Security settings are locked while someone is connected.** This stops a
person who is controlling your computer from changing its access settings.
End all sessions first, then change them.

### Clipboard and file access: on by default

When someone controls your computer, two extra features are **on unless
you switch them off**:

- **Clipboard sync**: copied text travels between the two computers.
- **File access**: the person in control can open a file manager on your
  computer and browse, copy files to and from it, make folders, rename and
  delete, everywhere your user account can (see [File manager](#file-manager)).

This means **anyone who gets in** (with your password, or because you
pressed Accept) and is in control can read every file your user account can
read, and change or delete every file your account can change. They act as
your user account, not as an administrator. Only give the password to, and
only accept, people you would trust with your files.

The host panel always shows a line "Clipboard and file access are ON"
(or OFF, or which one is on). How to switch them off:

- **On the host** (the computer being controlled): untick **Share clipboard
  text with the viewer in control** and/or **Allow file access (browse,
  copy, rename, delete) for the viewer in control**. Each switch works on
  its own. They are security settings: locked while someone is connected,
  saved, and used for the next session.
- **On the viewer** (your side): untick **Sync clipboard** in the connect box
  to stop clipboard sync with every computer you connect to. It works at
  once, also in open windows. (The viewer has no file-access switch: the
  file manager only does what you click.)

Two rules always apply:

- **Only the person in control** gets clipboard sync and the remote file
  manager. Extra viewers who connect while someone else is in control only
  watch: they get **neither**, whatever the switches say.
- **What a session may do is fixed when it starts.** Changing a switch on
  the host affects the next session, not the one that is running. To
  apply it, end the session and let the viewer reconnect.

### Clipboard sync

While you control another computer, copied **text** travels both ways by
itself: copy on one computer, paste on the other. Only text (no pictures or
files), up to 1 MiB per copy; a bigger copy is not sent and a short note
says so. What is already on the clipboard when the session starts is not
sent, only what you copy afterwards.

- Only the person **in control** gets it. Extra viewers who can only watch
  never see or change the host's clipboard.
- The **host** can switch it off with **Share clipboard text with the viewer
  in control** (for new sessions). The **viewer** can switch it off for their
  own computer with **Sync clipboard** (at once). Both must be on.
- The clipboard may hold passwords. Secure Connect never writes clipboard
  text to disk or to its logs, but anything you copy while connected goes to
  the other computer. Switch it off if that worries you.
- **Two remote windows open at once:** each window syncs with your local
  clipboard, so text copied on one remote computer also reaches the other
  one (it passes through your own clipboard). That other computer may belong
  to someone else. Close, or turn off **Sync clipboard** for, the windows you
  do not want to share with.
- **Linux without a clipboard manager:** text received during a session may
  disappear from the clipboard when the session ends, because on X11 the
  program that set it has to stay alive to hand it over. Paste what you need
  before you disconnect.
- **Linux with Wayland (e.g. GNOME):** Secure Connect reads the clipboard
  the X11 way. On GNOME this may only see a new copy while a Secure Connect
  window (or another X11 program) has focus, so text you copy in other
  programs on the host may not reach the viewer until you click a Secure
  Connect window. The host panel shows this note. To find out how your own
  desktop behaves, run the clipboard test tool described in
  [BUILD.md](BUILD.md#clipboard-test-tool-clipboard-spike).
- If this computer's clipboard cannot be opened at all, the host panel says
  so and viewers get no clipboard sync.

### File manager

In a remote window, press **Files** (top left) to open the file manager: a
window with **This computer** on the left and the **Remote computer** on the
right.

- **Moving around:** each side has a path bar (type a folder and press
  Enter), **Up**, **Home** and **Roots** (`/`, or the drive letters on
  Windows), and **Refresh**. Click the column titles to sort by name, size
  or modified time (shown in UTC); folders always come first. Double-click a
  folder to open it. Click selects one item, Ctrl+click adds or removes
  one, Shift+click selects a range, **Select all** selects everything.
- **Copying:** select files or folders and press **Copy >>** (this computer
  to the remote one) or **<< Copy** (the other way). They go into the folder
  shown on the other side. At most 4 copies run at once; the rest wait in
  the **Transfers** list, which shows progress, speed, **Cancel** and, for a
  stopped or failed copy, **Resume**. A copied folder is one line ("n of m
  files") with its own Cancel and Resume; cancelling it starts none of its
  files, and the message names the folders already created. Links to
  folders are not copied, and links inside a copied folder are skipped.
- **Name already there:** you are asked. **Keep both** (the default) saves
  the copy as `name (1).ext`; **Skip** leaves the existing one; **Replace**
  overwrites it (a warning says so; folders are never replaced, their files
  are copied into the existing folder).
- **Resume:** a copy that stops leaves `name.part` next to where it was
  going. Copy the same file into the same folder again (or press
  **Resume**) and you are offered to continue where it stopped. It only
  continues if the original has not changed; otherwise it starts over (see
  [Copying in more detail](#copying-in-more-detail)).
- **New folder, Rename:** names follow the same rules on both sides (no
  `/ \ < > : " | ? *`, no control characters, no reserved Windows names
  such as `CON`, no dot or space at the end, at most 255 bytes); the dialog
  says what is wrong with a name.
- **Delete** asks first; deleting a folder, or more than 10 items, needs
  you to type `DELETE`. Protected folders (roots, home, the folders
  containing it, system folders) are refused. See
  [Deleting and protected folders](#deleting-and-protected-folders).
- **What a session may do is fixed when it starts:** if the host's owner
  changes the file-access switch, it applies to the next session.
- **Only the viewer in control** gets the remote side. A window that can
  only watch, or a host whose owner turned file access off, shows the
  reason instead. Closing the remote window cancels its copies.
- **On the host,** the main window lists running copies under **File
  transfers** with a **Stop** button each (it stops only that copy; the
  viewer stays connected), and the **Activity** list shows what was copied,
  made, renamed or deleted (kept in memory only).

#### Copying in more detail

- **Progress, cancel, resume.** Each line shows how much has arrived, the
  speed, and its state (waiting, running, done, skipped, failed,
  cancelled). **Cancel** stops one copy; **Cancel all** stops all of them
  and any folder that is still being read. A stopped or failed copy keeps
  its partial file `name.part` in the destination folder, and **Resume**
  (or copying the same file into the same folder again) offers to continue
  from there. The partial file is only continued if the original file still
  has the same modification time and the partial file still matches its
  beginning; otherwise the copy starts from the beginning. Every finished copy
  is checked end to end (a BLAKE3 hash) before it gets its real name, so a
  damaged copy never appears under the final name.
- **A copy never overwrites by accident.** Without **Replace**, an existing
  file is never touched: Keep both picks the next free name (`name (2).ext`
  …), Skip leaves it alone. A folder is never replaced by a file.
- **Folders.** A copied folder is read on the sending side first, its
  folders are made on the receiving side, then its files are queued (one
  line, "n of m files"). Limits: at most 10,000 files and 32 levels deep
  per copied folder; links inside are skipped and counted. If the copy
  stops (error or Cancel), the folders already made stay and the message
  lists them; delete them yourself if you do not want them.
- **Copied files are private.** A file copied to a computer is created
  readable and writable by its user only (mode 0600 on Linux). The
  executable bit is not preserved: make a copied program executable again
  yourself.
- **Secure Connect's own settings folder is off limits.** The file
  manager on a host never lists, reads or changes the folder where Secure
  Connect keeps its key, password check, saved computers and settings.
- **File access can leave lasting access behind.** Someone with file access
  can write files that give them a way back in (for example
  `~/.ssh/authorized_keys`, an autostart entry or a shell start-up file).
  **Accept once is not one-time access** while file access is on. Turn
  **Allow file access** off in Settings when that matters.
- **Quick reconnect after End.** A viewer that reconnects right after you
  ended its session can land view-only for a few seconds.
- **Mouse and keyboard during a big copy:** copies share the connection
  with the picture and your input. Pictures and your input are sent ahead
  of copy data, so on a slow link the copy slows down first, but the
  picture may still slow too; this has not been measured yet. In automated tests on one machine, mouse
  moves still arrived within a few milliseconds during a 128 MiB copy; over
  a real internet connection this has not been measured yet.

#### Deleting and protected folders

- Delete always asks. The box says **LOCAL computer (this one)** or
  **REMOTE computer**, how many items, the first five full paths, whether
  folders with everything inside are included, and that nothing goes to a
  trash or recycle bin.
- For a folder, or for more than 10 items, you must type `DELETE` (capital
  letters) before the Delete button works. **Cancel** has the keyboard
  focus, so Enter or Escape cancels.
- **Protected folders cannot be deleted or renamed from the file manager**,
  not on the remote computer and not on your own: a drive or root folder
  (`/`, `C:\`), the home folder of the user Secure Connect runs as, every
  folder that contains it (for example `/home`, `C:\Users`), and the
  system folders (Linux: `/etc`, `/usr`, `/bin`, `/sbin`, `/lib*`, `/boot`,
  `/var`, `/proc`, `/sys`, `/dev`, `/root`, `/opt`, `/srv`, `/run`,
  `/home`, `/snap`, `/mnt`, `/media`, `/tmp`; Windows: the Windows folder,
  Program Files (both), ProgramData and Users) together with every folder
  containing one of them. The computer that owns the files decides this,
  after following links in the path, so a link or a different spelling
  (another path to the same folder, a Windows short name such as
  `PROGRA~1`) does not get around it. The answer is "This folder is
  protected and cannot be deleted or renamed from here". If the check
  itself cannot be done, nothing is changed.
- Protection covers those folders **themselves only**. Everything inside
  them, for example your Documents folder or `/etc/something`, can still be
  deleted on purpose, and an ordinary user account cannot delete inside
  system folders anyway. To delete a protected folder, do it at that
  computer, not through Secure Connect.
- A recursive delete stops after about 10 seconds of work and says that
  part of the folder was deleted; ask again to continue. On Linux it also
  stops at the border of another file system or btrfs subvolume inside the
  folder.

#### Pins

Each pane has a **Pins** list next to **Roots**. Open a folder, then press
**Pin this folder** to add it; press **Unpin** (shown instead while the
folder is pinned) to remove it. Choosing a pin opens that folder. Pins show
the folder's name; hold the mouse over one to see the full path. Up to 50
folders can be pinned per list.

This computer's pins are one list. The remote pane's pins are kept per
remote computer (by its ID), so each computer has its own. They are kept in
`pins.txt` next to `hosts.txt`. A pin is only a remembered path: choosing it
asks for that folder like typing the path would, and the usual file
protections still apply. A damaged `pins.txt` is renamed to `pins.txt.bad`
and a new list is started.

#### Limits of the file manager

- **Slow connections:** a copy that averages less than **4 KiB/s** (32
  kbit/s) after its first minute is stopped as "too slow" (with 4 copies at
  once, that is a link of about 128 kbit/s). It can be resumed.
- **Very large folders:** a folder with more than **20,000 entries** is
  only partly listed (the status line says so); the rest cannot be reached
  in the file manager, and such a folder cannot be copied as a whole.
- **Names that are not valid UTF-8** (possible on Linux) are shown with a
  replacement character and cannot be opened, copied, renamed or deleted.
- **Times** in the Modified column are shown in **UTC**, not your local
  time.
- **Leftover partial files.** A stopped copy leaves `name.part`; if that
  name was already taken (for example by a browser's own download, which is
  never touched), the copy uses `name.~1.part`, `name.~2.part`, …. Those
  `.~N.part` files are **not** found by Resume, and a retry that starts over
  does not remove an older partial file. Delete leftover `.part` files
  yourself.
- **Windows:** everything Windows-specific (drive letters, `C:\` paths,
  junctions, short names, the protection of Windows folders, the
  never-overwrite rename) is built and unit-tested but has **never been run
  on a real Windows PC**.

### Require post-quantum encryption

All connections are encrypted. By default Secure Connect also *tries* to use
an extra-strong, "post-quantum" key exchange (designed to resist future
quantum computers), and uses it whenever both sides support it.

If you tick **Require post-quantum encryption**, Secure Connect refuses to
talk to any computer that cannot use it, in both directions. Two copies of
Secure Connect should always be able to use it, so normally you will not
notice a difference. After changing this setting you must **restart Secure
Connect** (the window reminds you).

Note: this protects the encryption of what you send. The IDs themselves still
use ordinary (not post-quantum) keys. The [security notes](SECURITY.md)
explain this.

## Linux with Wayland: approving screen sharing

On Ubuntu's default desktop (GNOME on Wayland), programs are not allowed to
see the screen or move the mouse without your permission. The first time
someone connects to your computer and takes control, a **system "Remote
Desktop" dialog** asks you to approve screen sharing **and remote
interaction** (keyboard and mouse). Choose the screen, leave "Allow remote
interaction" switched on, and confirm. Secure Connect remembers the approval,
so later connections do not ask again.

What your answer means:

- **Approve with remote interaction on**: the viewer can see the screen and
  control every window, including native Wayland apps such as Settings,
  Files and Terminal.
- **Approve but switch remote interaction off**: the viewer sees the screen,
  but can only control older-style "XWayland" windows. The main window says
  so in a line starting with "Input:".
- **Cancel or deny**: the connection ends. Nothing else is asked.

If your desktop has no Remote Desktop portal, Secure Connect falls back to
the plain screen-sharing dialog, and only XWayland windows can be controlled
(the "Input:" line says so).

A second viewer who connects while someone is in control only watches; for
them the plain screen-sharing dialog is used (and remembered separately), and
no keyboard or mouse access is ever requested for them.

Important for unattended use: **someone must be at the computer the first
time** to approve this dialog. If nobody approves it, the connection ends.
Do this once before you leave the computer alone, for example by connecting
to it yourself from another computer.

If the approval was cancelled or failed, the main window shows a red warning
such as "Screen capture failed last time: …".

## Where Secure Connect keeps its files

Secure Connect keeps a few small files, readable only by your user account.
It never saves a picture of **your own** screen to disk. Small previews of
**other** computers' screens are saved for their cards unless you turn that
off (see [Saved computers](#saved-computers-cards)).

| File | What it is | Linux | Windows |
| --- | --- | --- | --- |
| `node.key` | Your computer's secret key; it defines your ID | `~/.config/secure-connect/` | `%LOCALAPPDATA%\secure-connect\` |
| `password.verifier` | The check value for your unattended password (not the password itself) | `~/.config/secure-connect/` | `%LOCALAPPDATA%\secure-connect\` |
| `settings.txt` | Accept-prompt choice, post-quantum choice, remembered ID, the clipboard and file-access switches, **Sync clipboard** | `~/.config/secure-connect/` | `%APPDATA%\secure-connect\` |
| `remotedesktop.token` | Remembers your Wayland approval for screen sharing **with remote control** | `~/.config/secure-connect/` | (not used) |
| `screencast.token` | Remembers your Wayland screen-sharing approval for watch-only viewers | `~/.config/secure-connect/` | (not used) |
| `hosts.txt` | Your saved computers: ID, name, last connection, colour, and whether a password is remembered (never the password) | `~/.config/secure-connect/` | `%LOCALAPPDATA%\secure-connect\` |
| `pins.txt` | Pinned folders in the file manager | `~/.config/secure-connect/` | `%LOCALAPPDATA%\secure-connect\` |
| `previews\*.png` | Small pictures of saved computers' screens (file names are scrambled IDs) | `~/.config/secure-connect/previews/` | `%LOCALAPPDATA%\secure-connect\previews\` |

On Linux, if you have set the `XDG_CONFIG_HOME` environment variable, the
folder is `$XDG_CONFIG_HOME/secure-connect/` instead.

Useful to know:

- **Deleting `node.key` gives you a new ID.** Everyone who connects to you
  will need the new one.
- Deleting `password.verifier` removes the password (same as **Clear**).
- Deleting `remotedesktop.token` (or `screencast.token`) makes the Wayland
  approval dialog appear again. To take back the remembered remote-control
  approval, delete `remotedesktop.token`. (Some desktops may also offer a
  way to revoke it in their own settings; this has not been checked.)
- `remotedesktop.token` lets Secure Connect start a session that controls your
  keyboard and mouse without asking again, for anyone who has signed in to
  your computer through Secure Connect. Keep it private.
- Keep these files private. Anyone who copies `node.key` can pretend to be
  your computer; anyone who can read `password.verifier` can log in to it.
- Deleting `hosts.txt` removes all cards. A damaged `hosts.txt` is renamed
  to `hosts.txt.bad` (kept, not deleted) and a new, empty list is started. Remembered passwords stay in your system password store under
  the name "secure-connect" until you remove them there; use **Forget
  password** first if you want them gone. Deleting the `previews` folder
  removes all previews.

## Running it in the background on Linux

Secure Connect is a window app: it needs your logged-in desktop and cannot
run without a screen. Closing its window quits it (there is no tray icon
yet), so to keep it running, minimise the window.

**Start it detached from the terminal** (closing the terminal will not stop
it):

```sh
setsid nohup ./secure-connect >/dev/null 2>&1 &
```

**Start it automatically when you log in:**

```sh
mkdir -p ~/.config/autostart
cat > ~/.config/autostart/secure-connect.desktop <<'EOF'
[Desktop Entry]
Type=Application
Name=Secure Connect
Exec=/full/path/to/secure-connect-linux-x86_64
X-GNOME-Autostart-enabled=true
EOF
```

Replace `/full/path/to/` with where you saved the program, and keep it
there (update `Exec=` if you move it). Log out and in again to check.

**Check or stop it:**

```sh
pgrep -a secure-connect
pkill secure-connect
```

Limits:

- After a reboot nobody can connect until you have logged in. Automatic
  login in your desktop settings avoids that, but lets anyone at the
  keyboard in.
- While the screen is locked, the remote picture and control do not work.
- On GNOME Wayland the screen-sharing approval should be remembered after
  the first time; if it asks again after a reboot, approve it again.

## Troubleshooting

**"Could not reach that ID. Check the ID and that Secure Connect is running there."**
The other computer could not be found or reached. Check that:
- the ID is complete and correct (copy and paste it rather than typing it);
- Secure Connect is running on the other computer and that computer is
  online;
- both computers have a working internet connection. Very strict networks
  (some offices, schools or hotels) may block the connection.

**"That is not a valid ID."**
The ID is too short or contains wrong characters. A valid ID has 52 letters
and digits, in groups separated by dashes.

**"That is this computer's own ID."**
You pasted your own ID. Use the other computer's ID.

**"Wrong password."**
Check the password (it is case-sensitive). Remember that each computer
should have its own password.

**"Too many wrong passwords: the other computer is locked. Try again in N seconds."**
Wait for the time shown. If it keeps happening even though you use the
right password, someone else may be guessing; the host can use Accept prompts
instead (they are not affected by lockouts).

**"The other computer refused the connection (declined, or it accepts no connections)."**
Someone pressed Deny, the 60-second Accept prompt ran out, or the other
computer has neither a password nor Accept prompts turned on.

**"The other computer did not answer in time."**
The other computer may be very slow or its connection may be poor. Try again.

**"This host does not support post-quantum encryption, which you required."**
You ticked "Require post-quantum encryption" and the other side could not
use it. Untick it (and restart) if you need to reach that computer.

**"Nobody can connect: set a password or turn on Accept prompts."**
Your computer currently refuses everyone. See
[Deciding who may connect](#deciding-who-may-connect).

**"Security settings are locked while someone is connected."**
End all sessions, then change the settings.

**The screen stays black, or the session ends right away (Linux)**
On Wayland, someone must approve the screen-sharing dialog the first time
(see above). Also check that PipeWire and xdg-desktop-portal are installed
and running. The main window shows a red message if screen capture is not
available.

**My mouse and keyboard do nothing on a Linux (Wayland) computer**
Look at the host's main window for a line starting with "Input:".

- "remote control is active through the Remote Desktop portal": control
  should work in every window. If the pointer lands in the wrong place on a
  scaled (HiDPI) screen, see the note below.
- "remote interaction … was not allowed": the Remote Desktop dialog was
  approved with remote interaction switched off. Delete
  `remotedesktop.token` (see [files](#where-secure-connect-keeps-its-files)),
  connect again and leave remote interaction on.
- "the Remote Desktop portal is not available": your desktop cannot grant
  keyboard and mouse access, so only older-style "XWayland" windows can be
  controlled. Workaround: if the login screen offers "Ubuntu on Xorg" (an
  X11 session; under the gear icon), sign in with that instead.

Pointer in the wrong place on a scaled screen: GNOME expects pointer
positions in screen pixels, other desktops in "logical" (scaled) units.
Secure Connect picks GNOME's way on GNOME and the logical way elsewhere. To
override, start the host with the environment variable
`SECURE_CONNECT_PORTAL_POINTER=logical` (or `=pixels`). With the wrong
setting the pointer can land outside the shared screen, for example on a
neighbouring monitor that the viewer cannot see, so clicks may go to windows
there. Only use this override while watching the host screen, and remove it
if the pointer does not end up exactly where the viewer points.

**My mouse and keyboard do nothing, and I am the second viewer**
Only the first connected person can control. Ask them to disconnect, then
reconnect.

**Windows: I see a black or frozen picture at the lock screen or a security prompt**
Secure Connect cannot see or control the Windows lock screen, the
"Do you want to allow this app to make changes" (UAC) prompts, or programs
running as administrator. Someone at the computer has to deal with these.
On Windows only the **main monitor** is shown.

**Linux: the program does not start and mentions `GLIBC_2.38`**
Your Linux is too old (for example Ubuntu 22.04). You need Ubuntu 24.04 or
newer, or another Linux with glibc 2.38 or newer.

**Linux: "error while loading shared libraries: libpipewire-0.3.so.0" (or libxkbcommon)**
Install the missing library: on Ubuntu 24.04,
`sudo apt install libpipewire-0.3-0t64 libxkbcommon0`.

**Windows: the program window does not appear / SmartScreen blocks it**
See [Starting Secure Connect](#windows). The Windows version is untested,
so problems are possible; please report them.

## What Secure Connect does not do (yet)

- Only text is shared through the clipboard (no pictures).
- No sound.
- No separate desktops for several users on one Linux computer; viewers see
  the session that is on the screen.
- No way in the app to choose your own relay server.
- No hardware video encoding and no automatic quality adjustment for slow
  connections (it always aims for about 3 Mbit/s).
- No log of who connected when. The host's **Activity** list shows the
  recent sessions and file operations (the last 200 entries) but is kept in
  memory only and is gone when Secure Connect closes; apart from that there
  is only the failed-attempt counter.
- No copy and paste of files through the clipboard: use the file manager.
- Extra viewers are not told that they can only watch.
- Only the main monitor is shown on Windows; multi-monitor and very
  high-resolution (HiDPI) screens have not been tested.

## What has been tested

Being honest about where things stand:

**Checked**

- Automated tests of the connection, encryption, password checks, lockouts,
  Accept prompts and video, with simulated screens and input.
- Wayland screen sharing on a real GNOME desktop on Linux: worked at 30
  frames per second, and the approval was remembered.

**Not yet checked**

- Two computers connected over the real internet.
- The post-quantum key exchange through the public relay servers.
- Real mouse and keyboard control of a computer (only simulated so far).
  This includes control through the Wayland Remote Desktop portal, which has
  been built and tested only against a simulated portal.
- Keyboard layouts: "Match typed characters" and "Match key positions" were
  tested only with simulated key events (German, French and US layouts,
  AltGr, shortcuts), never with real keyboards and real computers.
- The remote picture shown at full size in the app window.
- Anything at all on Windows (it has been built, but never run on a real
  Windows PC).
- High-resolution (HiDPI) screens and computers with several monitors.
- Saved-computer cards in the real window, and remembering passwords in a
  real system password store (GNOME Keyring, KWallet, Windows Credential
  Manager): the logic is tested only with a simulated store and simulated
  files.
- **Clipboard sync and the file manager on real computers.** They are
  covered by automated tests (two copies of Secure Connect on one machine,
  temporary folders, a simulated clipboard), including a 128 MiB copy while
  mouse moves are measured. But the file manager window has **never been
  seen on a screen**, no file has been copied between two real computers,
  the real system clipboard has not been used, and whether GNOME on Wayland
  lets the host notice copies made in other programs is unknown until the
  clipboard test tool is run (see [BUILD.md](BUILD.md#clipboard-test-tool-clipboard-spike)).
- The file manager and clipboard on Windows (never run there).

Please report problems on the project's GitHub page. Never post your
password, your `node.key` or `password.verifier` files, or anything else
secret in a public report.

## Licence

Secure Connect is © Tuan Aqeel Bohoran and licensed under the **PolyForm
Noncommercial License 1.0.0** (see the `LICENSE` file). In plain words: free
for personal use, study, research, and for charities, schools and other
non-profit organisations. **Commercial use**, including use inside a
company, needs a separate licence: contact aqeelbohoran@gmail.com. The
third-party parts keep their own licences (`THIRD-PARTY-LICENSES.txt`).
