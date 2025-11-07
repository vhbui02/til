# Post-installation documentation

## Introduction

This is the document that records **every** changes I've made to the system. Making the desktop ergonomically sane and usable is very important to me

### Guidelines

- Answers **WHAT** questions, not **HOW** question. Provide a link to the solution.

- Avoid content switching. It's best if everything stays inside this docs.

- Every solution must be implemented and tested using `sh/bash/zsh/fish/...` script or _Ansible_.

- `~/.dotfiles` best-practices:

  - Favors **Application-based** categorical strategy instead of **Mirrored Directory** (`~/.dotfiles` mirrors the stucture of `/`).
  - Use **hardlinks** to maintain inter-partition config files.
  - Use **symlink** (prefered, required testing) or **cronjob + inotifywait + rsync** to maintain cross-partition config files.

- Global Keyboard Shortcuts convention:
  - `Ctrl + Alt + [KEY]`
  - `Alt + F[1-12]`
  - `Alt + UIJKLO`
- Application Keyboard Shortcuts convention:

  - VSCodium (built-in + extensions): `Alt + [KEY]`, `Ctrl + [KEY]`, `Alt + Shift + [KEY]`
  - Firefox (extension only): `Alt + Shift + [KEY]`

- Every playground projects must specify the link to the blog post so I can trace back to where I learned it.

- If you're going to rewrite everything what you're reading, DONT. Either copy-paste or paraphrasing it.

- Only 1 editor should be used to edit this file. DO NOT USE BOTH VSCODE AND NANO TO EDIT THIS FILE

### Technical debt

- WHAT and HOW still intertwined with each other, there isn't a clear boundary.
- Required version control system.

### Future

### Tag system

> NOTE: Added into VSCodium's User Markdown snippet.

#### 1. OS

- [#fedorakde40]() : implemented inside Linux distro Fedora KDE 40.
- [#ubuntu2204]() : same as [#fedorakde40](), but for Ubuntu 22.04 LTS.

#### 2. Type of config files inside `~/.dotfiles`

- [#desktop]() : custom user-level `.desktop` files, its `Exec=` entry is customized, sometimes applied special variables. Stored at `~/.local/share/applications`:
  - %h - User's home directory
  - %k - URI or file path of the desktop file
  - %c - Translated application name
  - %f - Single file path
  - %F - List of files
  - %u - Single URL
  - %U - List of URLs
- [#systemd-root]() : custom system-wide systemd's unit files. They can be `.service`, `.timer` or `.sock` files. Stored at `/usr/local/lib/systemd/system`.
- [#systemd-user]() : same as [#systemd-root](), but for user-level, and stored at `~/.config/systemd/user`.
- [#config](): general configuration file (`.conf`, `.toml`, ...)

#### 3. Config files' Maintainance Methods

- [#symlink]() : config files' shortcut are created from `~/.dotfiles` and put inside original directory using `-s` option of `ln` command.
- [#hardlink]() : config files are "instant moved" (atomic moved) from `~/.dotfiles` to original directory using `ln` command.
- [#synclink]() : config files remain inside their original directory. They are being monitored by `inotifywait` command, synced to `~/.dotfiles` using `rsync` whenever a change is made.

#### 4. Package Manager

- [#apt](): **Debian-family** package manager.
- [#dnf](): **Red Hat-family** package manager.

#### 5. Package Manager's Repository

- [#ppa](): a **Debian-family** repository, added via [#apt]() package manager.
- [#rpmfusion](): Official, patent-free **Red Hat-family** repository, added via **DNF** package manager.
- [#copr](): Non-official **Red Hat-family**, added via **DNF** package manager.

#### 6. Package Format

- [#deb](): package whose format can be installed using [#apt]()
- [#rpm](): package whose format can be installed using [#dnf]()

#### 7. Cross-platform Installation Methods

- [#snap](): (not recommended) **Ubuntu**'s proprietary package manager.
- [#flatpak](): (recommemded) the future of apps on Linux.
- [#appimage](): Linux apps' executable that run anywhere.
- [#built-in](): defaultly shipped when installed distro (CLI tools, kernel modules, ...)
- [#saas](): Software-as-a-service, accessed via browsers.
- [#prebuilt-binary](): applications that are portable executable files.
- [#selfbuilt-binary](): applications that are cloned from version controls' repositories and built executable manually. Normally are stored at `~/FOSS` or `/opt` (see [Directory structure](#directory-structure)).

#### 8. Automation

- [#cronjob](): best for non-system applications.
- [#xdg-autostart](): best for GUI applications.
- [#systemd-root](): (recommended) most reliable
- [#systemd-user](): (recommended) most reliable

#### 9. Misc

- [#considered](): known solutions, but haven't been implemented.
- [#uninstalled](): applications that are no longer existed inside system.
- [#untested](): applications that are known but not used and tested yet.

### Credits

#### 1. Bill Dietrich

[Bill Dietrich's Computer > Linux > Using Linux](https://www.billdietrich.me/UsingLinux.html)

#### 2. Willi Mutschler

[Mutschler's "Fedora Workstation 33: installation guide with btrfs-luks full disk encryption (optionally including /boot) and auto snapshots with Timeshift"](https://mutschler.dev/linux/fedora-btrfs-33/)

[Mutschler's "Fedora Workstation 35 with automatic btrfs snapshots and backups using BTRBK"](https://mutschler.dev/linux/fedora-btrfs-35/)

[Mutschler's "Fedora Workstation: Things to do after installation (Apps, Settings, and Tweaks)"](https://mutschler.dev/linux/fedora-post-install/)

#### 3. M.Hanny Sabbagh

[M.Hanny Sabbagh's "Things To Do After Installing Fedora 40"](https://fosspost.org/things-to-do-after-installing-fedora)

[M.Hanny Sabbagh's "Enable zRAM on Linux For Better System Performance"](https://fosspost.org/enable-zram-on-linux-better-system-performance)

#### 4. Dedoimedo

[Dedoimedo's "Make Fedora 30 fun and productive after installation"](https://www.dedoimedo.com/computers/fedora-30-after-install.html)

[Dedoimedo's "Fedora 32 essential post-install tweaks"](https://www.dedoimedo.com/computers/fedora-32-essential-tweaks.html)

[Dedoimedo's "Fedora 33 essential post-install tweaks"](https://www.dedoimedo.com/computers/fedora-33-essential-tweaks.html)

[Dedoimedo's "Fedora 34 KDE - Modern but not polished"](https://www.dedoimedo.com/computers/fedora-34-kde.html)

[Dedoimedo's "Fedora 36 Workstation review - Yeah naah"](https://www.dedoimedo.com/computers/fedora-36.html)

[Dedoimedo's "Fedora 37 Workstation quick review"](https://www.dedoimedo.com/computers/fedora-37.html)

#### 5. Pjotr

[Pjotr's "Avoid 10 Fatal Mistakes in Linux Mint and Ubuntu"](https://easylinuxtipsproject.blogspot.com/p/fatal-mistakes.html)

[Pjotr's "Speed Up your Mint!"](https://easylinuxtipsproject.blogspot.com/p/speed-mint.html)

#### 99. Misc

- [SoloSaravanan/Ansible-fedora](https://github.com/SoloSaravanan/Ansible-fedora)
- [divestedcg/brace](https://github.com/divestedcg/Brace) (use with cautions, they don't document what they do)
- [daveschudel/Fedora-KDE](https://github.com/daveschudel/Fedora-KDE)
- [radbirb/Short-FedoraKDE-Tips](https://github.com/radbirb/Short-FedoraKDE-Tips)

## Table of Contents

> NOTE: enumeration items are sorted by alphabetical order.

### 1. [Introduction](#introduction)

### 2. [To-Do List](#to-do-list)

### 3. [Settings and Tweaks](#settings-and-tweaks)

- [Automation](#automation)
- [BIOS/UEFI](#bios-uefi)
- [Bootloader](#bootloader)
- [Brightness](#brightness)
- [Clipboard](#clipboard)
- [CPU optimization](#cpu-optimization)
- [Disk visualization](#disk-visualization)
- [Driver](#driver)
- [Encryption](#encryption)
- [Filesystem](#filesystem)
- [File searching](#file-searching)
- [Firewall](#firewall)
- [Kernel module](#kernel-module)
- [Monitoring Tools](#monitoring-tools)
- [Multimedia](#multimedia)
- [Network](#network)
- [OCR tool](#ocr-tool)
- [Out-of-memory Killer](#out-of-memory-killer)
- [Power/Battery](#power-battery)
- [RAM Optimization](#ram-optimization)
- [Screen Locking](#screen-locking)
- [Settings](#settings)
- [Sound](#sound)
- [SSD Optimization](#ssd-optimization)
- [Swap strategy](#swap-strategy)
- [Wake up](#wake-up)

### 4. [3rd-party Applications](#3rd-party-applications)

- [Antivirus](#antivirus)
- [Audio editor](#audio-editor)
- [Authentication](#authentication)
- [Backup](#backup)
- [BitTorrent client](#bittorrent-client)
- [Browser](#browser)
- [Cloud API client](#cloud-api-client)
- [Containerization](#containerization)
- [Debloating](#debloating)
- [Desktop Environment tool](#desktop-environment-tool)
- [Documentation](#documentation)
- [Directory structure](#directory-structure)
- [Download Manager](#download-manager)
- [Email service](#email-service)
- [Email client](#email-client)
- [File explorer](#file-explorer)
- [File monitor](#file-monitor)
- [File sharing](#file-sharing)
- [Financial management](#financial-management)
- [Flash OS image](#flash-os-image)
- [Fonts and icons](#fonts-and-icons)
- [Gaming](#gaming)
- [Image editor](#image-editor)
- [Image viewer](#image-viewer)
- [Input method](#input-method)
- [Keyboard autoinput](#keyboard-autoinput)
- [Keyboard input remapper](#keyboard-input-remapper)
- [Language Learning](#language-learning)
- [Linux to Phone](#linux-to-phone)
- [Manufacturer software](#manufacturer-software)
- [Measurement](#measurement)
- [Mouse programming](#mouse-programming)
- [Office suite](#office-suite)
- [Package manager, format and repository](#package-manager-format-and-repository)
- [Password manager and OS Keyring](#password-manager-and-os-keyring)
- [PDF editor](#pdf-editor)
- [Personal Media Server (Servarr stack)](#personal-media-server-servarr-stack)
- [Personal Knowledge Management (PKM)](#personal-knowledge-management-pkm)
- [Pomodoro timer](#pomodoro-timer)
- [Remote access desktop](#remote-access-desktop)
- [RSS feed](#rss-feed)
- [Sandbox](#sandbox)
- [Screenshot/Screen Recoder](#screenshot-screen-recoder)
- [Shell](#shell)
- [Social Media](#social-media)
- [Spaced Repetition System (SRS)](#spaced-repetition-system-srs)
- [Streaming service](#streaming-service)
- [Terminal](#terminal)
- [Text editor](#text-editor)
- [Text hooker](#text-hooker)
- [Text processor](#text-processor)
- [To-do App](#to-do-app)
- [Touchpad](#touchpad)
- [Video conference](#video-conference)
- [Video editor](#video-editor)
- [Video player](#video-player)
- [Virtualization](#virtualization)
- [VPN](#vpn)

### 5. [Miscellaneous](#miscellaneous)

## To-Do List

### After reboot

- [ ] Test Wake On LAN by connecting to Shutdown Start Remote.
- [ ] Test `updatedb` exclude directory.
- [ ] Test blacklisting `hid_sensor_als` kernel module to disable Zero Touch Login on Linux side.
- [ ] Test `sddm` auto suspend scripts by waiting 120s after wake up.
- [ ] Test the stability of Servarr, Docker in specific and system in general after switching some directories into BTRFS subvolumes. After some time without issues, remove the backup directory.
- [ ] Test system and user cronjobs.
- [ ] Test system and user systemd units.
- [ ] Test `ip_tables` persistency by checking **WireGuard** docker logs.
- [x] Test `rtcwake`, `evremap` and `wg` service's persistancy.
- [x] Test `fcitx5` autostart on boot.
- [ ] Test wireless chipset's Power Management using `nmcli connection show 4ff7792b-f508-4050-ae8a-3ae731337113 | grep save`
- [ ] Test closing laptop lid => suspend laptop by creating `/etc/systemd/logind.conf.d/99-laptop-server.conf` [Espionage724's comment on Fedora forum](https://discussion.fedoraproject.org/t/prevent-suspend-when-lid-close-in-fedora-40/114278/7)
- [ ] Test Wifi speed after turn off Wifi power management: [Someone's on Unix StackExchange](https://unix.stackexchange.com/a/315400/607715)
- [ ] Test wireless Intel wireless chipset speed up.

### Long-term

- Finish `~/.dotfiles/0-automation-script/sync-desktop-file.bash` and `~/.dotfiles/0-automation-script/sync-root-and-home.sh`.
- Auto test Prowlarr indexers connection and test Radarr, Sonarr connection to indexers from Prowlarr.
- Remove film from Sonarr/Radarr also removes from qBittorrent.
- Move Firefox's profile data to tmpfs partition on RAM. Can also be done on Thunderbird. Remember to sync to disk by reading this [Anything-sync-daemon](https://wiki.archlinux.org/title/Anything-sync-daemon) and
- Reduce Firefox's the time to update session store (15s => 5m)
- Reduce Firefox's CPU consumption for background tabs: `dom.min_background_timeout_value = 12000`
  [Bill Dietrich's "Using Linux > Browser"](https://www.billdietrich.me/UsingLinux.html?expandall=1#Browser)
- Try Brave instead of Google Chrome.
- Auto update FOSS that are installed not by package managers by fetching GitHub Repository and rebuilding.
  => Work in progress: `~/.dotfiles/desktop-file/sync-github-repo.desktop`, currently supports `/opt` and `~/FOSS`

- Test embedded sub issue, test opensubtitles plugin auto download sub, test MPV external player, test Kodi instead of Jellyfin for Android TV.
- Make a contribution on Servarr's Github Repo by correct Radarr's Example 2 on Delay Profile, it's 01.15am.
- Check `python3` Problem reporting and consider opening a Bugzilla ticket.
- Setup Syncthing and/or Watchman.
- Tweak shotcut's UI so it resembles Adobe Premiere Pro.
- Categorize Facebook's Saved posts and Firefox's bookmarks. Save online tools.
- Add RSS feed (for new chapters of my favorite comics) Liferea or Akregator
- Try Notion.
- Try KVM linux.
- Instead of cloning and symlinking separate `.desktop` file, which can fcked up after update, I'm thinking of a more dynamic approach. Autocheck differences between local `.desktop` and system `.desktop` file after appli
  cation updates. Project: `~/.dotfiles/0-automation-script/desktop-file-watcher/`.
- Update Firefox `user.js` via Arkenfox GUI wiki and `TreeStyleTab` via its wiki.
- Switch between windows of an application behavior in Fedora KDE is not good for me, but in GNOME it's very good.
- Setup Financial Management app.
- Try K-9 Mail on Android and OpenKeychain to do PGP.
- Setup KDE Connect.
- Ricing GIMP: [Les Pounder's "How To Make GIMP Look and Feel Like Photoshop"](https://www.tomshardware.com/how-to/make-gimp-look-and-feel-like-photoshop).
- Install add-on GIMP Paint Studio.
- Fork `xdman`, fix system tray + open 2nd time bug + add regex support when providing exception URLs.
- Check if these instructions can solve `copyq` bugginess in GNOME Wayland [Reddit #1](https://www.reddit.com/r/pop_os/comments/mvhtox/copyq_not_working_with_wayland/k3a9lqu/)
- Clean `~/.local/share/` old data.
- Backup using `btrfs`, rather than DejaDup or Timeshift.
- Auto remove Firefox `user.js` depreciated settings. Read Arkenfox/user.js `prefsCleaner` script.
- Writing an Ansible script to automate all this is the final goal. Ref: https://github.com/SoloSaravanan/Ansible-fedora/tree/main

### Seems impossible

- Find a way to .
- Install GPS tracker inside a computer.

## Settings and Tweaks

### Automation

#### 0. Ansible playbook

Contains every tweaks.

#### 1. `crontab`

> [!TIP]
> System-wide level: `sudo nano /etc/crontab`
> Normal user level: `[EDITOR=nano] crontab -e`
> Root user level: `sudo [EDITOR=nano] crontab -e`
> Log: `sudo cat /var/log/cron`

> [!CAUTION]
> Percent-sign `%` changed into newline characters unless escaped with backslash `\`. Write `\%`.

- Normal user list of cronjobs:
  - Start Servarr stack at 8pm every day and inhibit suspension till 12.00am.
  - Sync Recyclarr config file.

```sh
# Auto-start Servarr stack and inhibit suspension
1 20 * * * /usr/bin/systemd-inhibit --what=idle sleep 14400
2 20 * * * /usr/bin/docker compose -f "/opt/servarr/docker-compose.yml" start

# Auto sync files `recyclarr`
# Don't have to do this if Servarr stack was implemented in home subvolume
@reboot /usr/bin/sleep 60; while inotifywait -m -e modify /opt/servarr/appdata/recyclarr/config/recyclarr.yml; do rsync -avhzP --checksum --backup --suffix=.bak "$file_path" "/home/$USER/.dotfiles/servarr/recyclarr.yml"; done
```

#### 2. `systemd`

> TL;DR
>
> - It's better to migrate from `cron` to `systemd` timers.
> - Check unit file content: `systemctl cat [SERVICE_NAME].[timer|service|socket|...]`
> - Avoiding errors by:
>   - Checking `ExecStart=` runs correctly.
>   - Checking the syntax: `systemd-analyze verify [FILE]`. This is crucial since it can detect errors that get silently ignored.
>   - Check exe times of calender entries: `systemd-analyze calendar [CALENDAR_ENTRY]`
>   - For timer, catching up on missed runs by adding `[Timer] Persistent=true`.
> - When debugging, check inside system journal by using `journalctl -u [TIMER] -u [SERVICE]`

> [!NOTE] > `service` command will redirect to `systemctl` command that handles `systemd` units.

> [!TIP]
> Use `After=` and `Requires=` appropriately. If `After=` specified system default service, you don't have to include them inside `Requires=` anymore. Read [The differences between _After=_ and _Requires=_ by Sufiyan Ghori](https://serverfault.com/a/931383/1138548)
> Use appropriate `Type=` [What is the differences between `Type=simple` and `Type=oneshot`](https://stackoverflow.com/a/39050387)
> Timer can take multiple `OnCalendar=` entries. that's why in order to override it, you must clear it by setting `OnCalendar=` to an empty value. Cre: [filbranden's post](https://unix.stackexchange.com/a/479745/607715)

> [!CAUTION]
> Percent-sign `%` denotes a template specifier. To escape it, write `%%`.
> Command/Variable substitution is not supported. Wrap commands inside `[sh|bash] -c`.
> Too many nested command substitions is not supported. Break them down into local env vars.
> Using symlink from .dotfiles, which failed, then workarounded by disabling SELinux via running `sudo setenforce 0` will make it not survive reboot. So either hardlink (user unit files) or rsync (system unit files, remember to change the owner from root to user by running `rsync -og --chown=$USER:$USER $SRC $DEST`)
> Any service that run after `sleep.target` must be placed at system-level.
> `Type=oneshot` with `RemainsAfterExit=true` keep service files active all the time, therefore timers can't restart them. Thanks to [Lev Levinsky's post](https://superuser.com/a/1560669/2206521).
> `status=203/EXEC` might be caused by SELinux. You can toggle it to start unit, but it will fail to start after reboot.

- System unit files:

  - Best place: `/usr/local/lib/systemd/system` (intended for unit files that are part of "software" not from a package manager)
  - Second best place: `/etc/systemd/system`
  - List:
    - `evremap@.service` (tested): Start keyboard input remapper
    - `rtcwake.service` (unreliable, disable): Auto turn on machine at 8pm.
    - `wg@.service` (tested): Setup custom WireGuard configuration
    - `wg-add-route@.timer` and `wg-add-route@.service` (depends on `wg@.service`, tested): Add routes that needed to be tunnelled through WireGuard network interface
    - `btrbk.service.d/99-override.conf`: Only snapshot btrfs volumes.
    - `btrbk.timer.d/99-override.conf`: Only run `btrbk.service`"hourly".

- User unit files:

  - Best place: `~/.config/systemd/usr`
  - Second best place: `/etc/systemd/user`
  - List:
    - `sync-systemd-system-to-dotfiles.service`: Sync custom systemd system file to dotfiles
    - `user-auto-update.timer` and `user-auto-update.service`: Update everything that only required user permission
    - `revive-hotspot.service`: Auto turn on Wifi Hotspot when system starts up during 12pm to 3am.
    - `speech-dispatcher.service`: Fix Firefox error about speech dispatcher.

- Alternatives places:
  - `/lib/systemd/system-sleep` and `/usr/lib/systemd/system-sleep`: scripts file that define behaviors before and after suspending on RAM.
  - Script must be owned by root and have executable permission. `sudo chmod 744 [SCRIPT_FILE]`
  - Display log: `journalctl -b -u systemd-suspend.service`
  - List:
    - `hotspot-revive.sh` - a script to automatically start Wifi Hotspot between 0am and 3am didn't work.
  - Template:

```sh
#!/bin/sh
export DISPLAY=:0

case $1/$2 in
  pre/*)
    echo "Before going to $2..."
    # write your command here...
    ;;
  post/*)
    echo "Waking up from $2..."
    # write your command here...
    # NOTE: I've seen a person use `sudo -u [USERNAME] [DISPLAY=:0] XDG_RUNTIME_DIR=/run/user/$(id -u [USERNAME]) [COMMAND]` here and it works for their use case
    # Why this works: grawity's post https://unix.stackexchange.com/a/776951/607715
    ;;
esac
```

[Jonathan Komar's answer about different `systemd` location](https://unix.stackexchange.com/a/367237/607715)

- Service file on Ubuntu:

  - `toggle-dock.service`: Toggle Ubuntu Dock Intellisense to fix top window instance collapse with Ubuntu Dock by:
    - (legacy) use `dbus-monitor` according to [Max's answer on AskUbuntu](https://askubuntu.com/a/1504136). This worked inconsistently.
    - Utilizing [gogama/lockheed](https://github.com/gogama/lockheed).
  - `toggle-hotspot.service`: Restart Hotspot when machine wakeup on suspend between 0am and 3am.

[Linux Handbook's "Create systemd services"](https://linuxhandbook.com/create-systemd-services/)  
[openSUSE's Docs "Working with systemd Timers"](https://documentation.suse.com/smart/systems-management/html/systemd-working-with-timers/index.html)

#### 3. KDE Plasma Autostart Manager

> [!IMPORTANT]
> Only supported in distro that utilizes KDE Plasma desktop environment.

`~/.config/plasma-workspace/env`

- Run first.
- For pre-startup scripts, such as set environment variables:
  - `im.sh`: configure `fcitx5`'s env vars.
  - `env.sh`: configure custom environment varibles.

> [!IMPORTANT]
> These scripts must have `0644` permission.

`/etc/xdg/autostart` (system) and `~/.config/autostart` (user)

- Run after pre-startup scripts.
- For applications and login scripts. Login scripts and Applications are different. Login scripts must have `X-KDE-AutostartScript=true` entry.

> [!TIP]
> To disable a system-level service, create a file with the same name inside user-level and add `Hidden=true` entry.

> [!CAUTION]
> It's not recommended to create custom `.desktop` file manually. Use System Settings instead.

- Custom services: [#hardlink]()

  - `alert-high-resource-usage.desktop`: Display a critical notification when either CPU, RAM or Disk usage is too high.
  - `check-conservation-mode.desktop`: Display a critical notification if Conservation Mode is somehow disabled.
  - `org.fcitx.Fcitx5.desktop`: Autostart Fcitx5
  - `org.keepassxc.KeePassXC.desktop`: Autostart KeePassXC Flatpak

- Disable default services: [#hardlink]()
  - `geoclue-demo-agent.desktop`: Prevent Linux knowing my location.
  - `gnome-keyring-pkcs11.desktop`: Prevent using GNOME Keyring on startup.
  - `gnome-keyring-secrets.desktop`: Prevent using GNOME Keyring on startup.
  - `gnome-keyring-ssh.desktop`: Prevent using GNOME Keyring on startup.
  - `org.kde.discover.notifier.desktop`: Prevent displaying KDE Discover pop-up.
  - `pam_kwallet_init.desktop`: Prevent using KWallet for PAM.

`~/.config/plasma-workspace/shutdown`<br />

- For scripts that needs to be run when logged out.

> [!CAUTION]
> It's not recommended to create custom `.desktop` file manually. Use System Settings instead.

#### 4. `System Settings > System > Autostart` (Front-end for KDE Plasma Autostart Manager)

[KDE's Documentation about Autostart](https://docs.kde.org/stable5/en/plasma-workspace/kcontrol/autostart/index.html)

#### 5. `X-KDE-autostart-*` inside `.desktop` file

[#fedorakde40]() [#builtin]()

- This might be the most simple solution that can be used to autostart applications.

```desktop
[Desktop Entry]
...

# only one instance of the app can run
X-DBUS-StartupType=Unique

# 0 - Essential services
# 1 - Basic services
# 2 - Regular applications
X-KDE-autostart-phase=2

# conditional autostart
# format: configfile:group:key:value
# interpretation: checks if `AutoStart` in `[General]` section of `rsibreakrc` is `false`
X-KDE-autostart-condition=rsibreakrc:General:AutoStart:false

# Defines startup order dependency to ensure proper UI initialization sequence
# Application will start after the panel is loaded
X-KDE-autostart-after=panel

...

```

#### 6. Profile file

- Run script on login during startup. Not on per-login.

> [!TIP]
> Default system-wide profile: `/etc/profile`
> System-wide custom directory profile: `/etc/profile.d`
> User-level profile: `~/.profile`

- Script inside `/etc/profile.d` must be written in `.sh` extension and allowed executable permission `chmod +x [MY_SCRIPT].sh`.
- `~/.profile` won't run if you're using Bash as default command interpreter for login shells.

### BIOS/UEFI

- Secure Boot: enabled. Check: `sudo mokutil --sb-state``
- Updated via Lenovo Vantage (Windows).
- Prevent Vention Hub from draining power from USB port after the pc is shutdown/sleep. USB charging disabled can solve the problem while the machine is shutdowned, but not sleep.
- Set up new administrator password for BIOS settings. Without this Secure Boot can be turned off.
- Set UMU Frame Buffer Size to 1G to meet minimal GPU VRAM requirements from Ollama ROCm support. Currently set to 512M after offloading Ollama model to my older brother's machine.

### Bootloader

1. GRUB2

- Edit `/etc/default/grub` and modify the existing `GRUB_CMDLINE_LINUX` line to include the following key-pairs: `cgroup_enable=memory swapaccount=1` to enable [kernel cgroup swap limit capabilities](https://docs.docker.com/engine/daemon/troubleshoot/#kernel-cgroup-swap-limit-capabilities). [Update GRUB inside Fedora](https://fedoraproject.org/wiki/GRUB_2#Updating_the_GRUB_configuration_file) by running `sudo grub2-mkconfig -o /etc/grub2.cfg`.

- [Very good comment from ElvisVan007 on Reddit about the UEFI system](https://www.reddit.com/r/Fedora/comments/12z3vuu/comment/jhr42y9)

### Brightness

1. `ddcutil`

- A software approach to change brightness automatically.

**Bugs:**<br />

- Auto reset to 0 after wake up from suspend on RAM or reboot. Using mechanical button on screen works.

### Clipboard

1. `wl-copy` and `wl-paste` from `wl-clipboard` package [#ubuntu2204]() [#apt]() [#fedorakde40]() [#dnf]()

- Wayland support.

2. **CopyQ** [#ubuntu2204]() [#ppa]()

- Super buggy, unusable. My assumption is lack of Wayland support.

3. **KDE Clipboard** [#fedorakde40]() [#builtin]()

- Pin feature not supported.

### CPU Optimization

1. (discouraged) Set CPU to Performance mode
2. (discouraged) Blacklisting Bluetooth, Webcam, Ethernet on laptop
3. Set to "performance" in "/sys/devices/system/cpu/cpu\*/cpufreq/scaling_governor" does not affect anything. Instead, go inside BIOS and change "Performance Profile" from Silent to Balanced.
4. (forbidden) Overclock CPU.
5. (discouraged) Disabling Spectre/Meltdown and other side-channel attack mitigations by running `sudo grubby --update-kernel=ALL --args="mitigations=off"`

### Disk visualization

1. **Filelight** [#fedorakde40]() [#flatpak]()

- Not all files are visible.

### Driver

- Update via Lenovo Vantage on Windows.
- Unnecessary install Nvidia packages since my machine is AMD.

### Encryption

1. Full-disk encryption with LUKS

2. (optional) `/boot` partition encryption: https://mutschler.dev/linux/fedora-btrfs-33/#step-5-optional-full-disk-encryption-including-boot

### Filesystem

1. **ext4** [#ubuntu2204]() [#builtin]()

- Some people still are skeptical about the maturity of `btrfs`. For them, best practice is to use in combination with _LVM (Logical Volume Manager)_ to achieve dynamic resizing.

**Q&A:**
Q: No space left on device although there are still a lot of spaces left.  
=> A: This can be happened if you have a lot of small files inside your disk. `ext4` has fixed number of inodes and has already ran out of them, before data blocks (that's why you still see plentiful of storage left). Check by running `sudo df -i /` or `sudo df -i /home`

To create a partition from unallocated disk space in Linux using LVM2:

```sh
# identify the physical disk with unallocated space
lsblk

# create a new partition on the disk, e.g. export PARTITION_NAME=/dev/sdX
sudo fdisk "${PARTITION_NAME}"

# create physical volume on new partition, e.g. export PHYSICAL_VOLUME_NAME=/dev/sdXn
sudo pvcreate "${PHYSICAL_VOLUME_NAME}"

# add physical volume to volume group
sudo vgextend "${VOLUME_GROUP_NAME}" "${PHYSICAL_VOLUME_NAME}"

# create logical volume from disk space
sudo lvcreate -l 100%FREE -n "${LOGICAL_VOLUME_NAME}" "${VOLUME_GROUP_NAME}"

# format and mount the logical volume
sudo mkfs.ext4 "/dev/${VOLUME_GROUP_NAME}/${LOGICAL_VOLUME_NAME}"
sudo mkdir /mnt/data # prefer mounting directly to /home/$USER
sudo mount "/dev/${VOLUME_GROUP_NAME}/${LOGICAL_VOLUME_NAME}" /mnt/data
```

2. **btrfs** [#fedorakde40]() [#builtin]()

> [!TIP]
> Put label `@` onto the partition that Fedora would installed into to create subvolume automatically. Therefore can avoid below work.
> List subvolumes: `sudo btrfs subvolume list "$SRC_DIR"`
> Create subvolume: `sudo btrfs subvolume create "$SRC_DIR"`
> Delete subvolume: `sudo btrfs subvolume delete "$SRC_DIR"`
> Create snapshot: `sudo btrfs subvolume snapshot "$SRC_DIR" "$DEST_DIR"`. Add `-r` option to create _read-only_.

- Mount `/mnt/btrfs` as the "true root" from outside to inside view.
- Mount `btrfs` top-level root filesystem to two subvolumes: `@` and `@home`.
- Also mount "true root" at `/mnt/btrfs` to create snapshot for backup.
- List of top-level subvolumes and their nested subvolumes:
  - `@`:
    - `/opt/servarr` and its backup `/opt/servarr_original`: my Servarr stack.
    - `/var/lib/docker` and its backup `/var/lib/docker.bk` (710): my Docker containers, images, volumes, ...
      [Docker's Docs "Configure Docker to use the btrfs storage driver"](https://docs.docker.com/engine/storage/drivers/btrfs-driver/)
    - `/var/tmp` and its backup `/var/tmp.bak` (755)
    - `/var/cache` and its backup `/var/cache.bak` (755)
    - `~/.cache`
    - (will update more)
  - `@home`

[Converting a root BTRFS install to a subvolume install in 4 easy steps](https://www.kubuntuforums.net/forum/general/miscellaneous/btrfs/680895-converting-a-root-btrfs-install-to-a-subvolume-install-in-4-easy-steps) (note that GRUB has `($root)` variable so you can skip adding `@` in GRUB editing at Step 2)

### File searching

1. `locate`/`plocate`

- Inverted trigram index is interesting.

**Tweaks:**

- CPU and MEM high usage frequently, check `htop` shows that `updatedb` is the culprit.
  => Reason: `updatedb` indexes top-level subvolumes and snapshot folders.
  => Workaround: add excluded paths inside `/etc/updatedb.conf` file.

> NOTE: Why I didn't see anyone talking about this problem? It's a real issue.

### Firewall

1. **ufw** [#ubuntu2204]() [#builtin]()

- Configuration for various applications and services

```sh
ufw allow 5522/tcp comment 'SSH'
ufw deny Apache
#ufw allow from 192.168.31.0/24 to any port 137,138 proto udp comment 'Samba for my home'
#ufw allow from 192.168.31.0/24 to any port 139,445 proto tcp comment 'Samba for my home'
ufw route allow in on wlp1s0 out on enx00e04c68097a comment 'Wifi Hotspot'
ufw allow in on wlp1s0 comment 'Wifi Hotspot'
ufw allow from 192.168.31.0/24 to any port 31694 proto tcp comment 'LAN Parsec'
ufw allow from 192.168.31.0/24 to any port 31694 proto udp comment 'LAN Parsec'
ufw route allow in on wlp1s0 out on CloudflareWARP comment 'Use Hotspot with WARP'
ufw allow 8081/tcp
ufw allow from fe80::/64 to any port 137,138 proto udp comment 'Samba (v6)'
ufw allow from fe80::/64 to any port 139,445 proto tcp comment 'Samba (v6)'
```

2. **firewalld**, `firewall-cmd` [#fedorakde40]() [#builtin]()

> [!IMPORTANT]
> It's best to keep firewall logs for later troubleshoot.

- Configuration for `iamhung` Wifi hotspot.

```sh
firewall-cmd --permanent --add-masquerade
firewall-cmd --permanent --direct --add-rule ipv4 filter FORWARD 0 -i $WIFI_INTERFACE_DEVICE -o $ETHERNET_INTERFACE_DEVICE -j ACCEPT
firewall-cmd --permanent --direct --add-rule ipv4 filter FORWARD 0 -i $ETHERNET_INTERFACE_DEVICE -o $WIFI_INTERFACE_DEVICE -j ACCEPT
firewall-cmd --permanent --add-interface=$ETHERNET_INTERFACE_DEVICE
firewall-cmd --permanent --zone=trusted --add-interface=$WIFI_INTERFACE_DEVICE
firewall-cmd --reload
```

### Kernel module

[#fedorakde40]() [#builtin]()

> [!TIP]
> Check modules loaded or not: `lsmod | grep [MODULE_NAME]`
> Enable module non-persistent: `sudo modprobe [MODULE_NAME]`
> Make module persistent after reboot: adding `/etc/modules-load.d/[MODULE_NAME].conf`

- Enable and persists `ip_tables` and `uinput`
- Use LUKS encryption => Enable `dm-crypt`

### Monitoring Tools

1. [#builtin]()

- GNOME System Monitor.
- KDE System Monitor.
- `vmstat`
- `w`: List the current users that are logged in and display their current tasks.
- `free`: Shows different metrics of memory.
- `ss` (a.k.a socket stats): Dump socket statistics.
- `uptime`: Shows how much time the system has been active.
- `iostat`: Shows and collects data about the IO performance of a system's storage devices.
- `ps`: Display active-processes infos.
  - Display all running processes: `ps -a`.
  - Display Top 10 memory consuming processes: `ps -auxf | sort -nr -k 4 | head -10`
  - Display Top 10 cpu consuming processes: `ps -auxf | sort -nr -k 3 | head -10`
- `netstat`: monitor incoming/outgoing network packets
  - See all TCP and UDP: `netstat -a`. TCP only: `netstat -at`. UDP only `netstat -au`.
  - See all listening connections available in the machine: `netstat -l`
  - See route information on all network interface: `netstat -rn`, similar to `route -n`
- `pmap`: looks up address space of a specific process.

2. [#dnf]() [#apt]()

- [clbr/radeontop](https://github.com/clbr/radeontop)
- `htop`
- `neofetch`
- `sysstat`: CPU, Network, Disk, Memory IO, Battery performance

3. Consideration

- `collectl`: combination of `ps, vmstat, top`, <0.1% CPU usage.
- `powertop`: CLI, per-application electrical power usage. It must be used when the machine is not connected to a power source.
- `dstat`: replaces `iostat, ifstat, vmstat, netstat`. Shows CPU, Disk, Network, Paging and System stats.
- `dtrace`: showing detailed views of programs and system internals.
- `monit`: monitor processes on server.
- `munin`: GUI, disk i/o and networks,
- `ganglia`: GUI, CPU, Network. Distributed and Scalable.
- `iptraf`: GUI network monitoring tools
- `iftop`: CLI trach network connections and bandwidth usage
- `whowatch`: similar to `w`.
- `nethogs`: diagnose real-time network bandwidth traffic.
- `iotop`: disk i/o.
- `jnettop`: network statistics and bandwidth consumed.
- `lttng`: tracing utility designed to troubleshoot hard-to-debug issues on production systems.
- `net-snmpd`: use and deploy SNMP which is used for monitoring network-connected devices (switches, routers, servers, ...)
- `pcp` (Performance CoPilot): visualize, monitor, record, diagnose and control the activity, status and performance of computers, networks, servers and applications.
- `wireshark`: monitor and troubleshoot network traffic, analyze network packets (including dropped ones) and can help identify malicious activity.
- `icinga2`: monitor entire data centers, from single server to multiple servers. Check availability of system updates, networks, services. Alerted when there is power outages, network failure, ...
- `conky`: CPU, RAM, Disk, Network, processes.
- `Linux Dash`: Web UI, CPU, RAM, processes, filesystem, users, network, ...
- `iftop`: bandwidth usage only.
- `lm-sensors`: CPU temperature only.
- `hddtemp`: HDD temperature only.
- `nagios`: processes, CPU, Memory, SMTP, POP3, ...
- `psensor`: best GUI-based temperature monitoring tool.

[Nikolaus Oosterhof's "Linux Ubuntu/Debian monitoring tools guide for system administrators"](https://net2.com/ubuntu-debian-monitoring-tools-guide-for-system-administrators/)

### Multimedia

- Swap `dnf` patent `ffmpeg-free` with full `ffmpeg` package. [#fedorakde40]() [#rpmfusion]()
- Additional codec: Install the complements multimedia packages needed by gstreamer enabled applications. [#fedorakde40]() [#rpmfusion]()
- Install `libva-utils` to support `vainfo` command. [#fedorakde40]() [#dnf]()
- Mesa VA-API support with OSS AMD graphics driver. [#fedorakde40]() [#rpmfusion]()
  - Since support for H.264, H.265, VC1 hardware decoding from Fedora 37 and later has been removed.  
    => Swap `dnf` patent `mesa-va-drivers` with free `mesa-va-drivers-freeworld` package.  
    => Swap `dnf` patent `mesa-vdpau-drivers` with free `mesa-vdpau-drivers-freeworld`package.

> [!NOTE]
> According to Jellyfin, rebuilding the Mesa driver with option to include all hardware codecs is a non 3rd-party package involved solution. So it's better.

- `jellyfin-ffmpeg` [#fedorakde40]() [#docker]()
  - There aren't official package on `fedora` or `RPMFusion` repository, therefore it's best to use Docker.

[Jellyfin's Docs "HWA Tutorial On AMD GPU"](https://jellyfin.org/docs/general/administration/hardware-acceleration/amd/#configure-with-linux-virtualization)  
[RPMFusion's Docs "Multimedia on Fedora"](https://rpmfusion.org/Howto/Multimedia)

### Network

[#fedorakde40]() [#ubuntu2204]() [#builtin]()<br />

> [!TIP]
> Network tools: `traceroute`, `ping`, `whois`, `curl`, `wget`, `net-tools`, `nmap`, `macchanger`

- Toggle Wifi Hotspot [#cronjob]()

```sh
...
# Auto-toggle Wifi Hotspot
# Sleep early, don't enable inhibit suspension !
7 0 * * * /usr/bin/nmcli radio wifi on; sleep 5; /usr/bin/nmcli connection up iamhung
0 3 * * * /usr/bin/nmcli connection down iamhung; sleep 5; /usr/bin/nmcli radio wifi off
...
```

- `/usr/lib/NetworkManager/dispatcher.d/98-revive-hotspot`: Autostart Wifi Hotspot after wakeup from suspend from 12pm to 3am.

> **NOTE:** somehow the network interface of the Wifi Hotspot keeps being revived even after being disabled. I had to remove its execution permission `sudo chmod -x 98-revive-hotspot`

- Auto create hotspot on boot: `nmcli con mod [NAME] connection.autoconnect yes`.
- Resolve through `systemd-resolved` service, configure via `/etc/systemd/resolved.conf` file.

  - Set Cloudflare and Google DNS servers as global DNS resolvers.

  > [!CAUTION]
  > libc only supports 3 DNS servers at max so you MUST add 3 DNS servers MAX: 1 Cloudflare, 1 Google and 1 Gateway as the fallback DNS.
  > This is to fix `systemd-resolved.service: Failed with result 'start-limit-hit'` error that prevent me from resolving DNS on boot.

  - Fallback to home router.
  - [Enable DNS over TLS using public DNS resolver](https://fedoramagazine.org/use-dns-over-tls/)
  - Disable `DNSSEC` since many services don't support it.

- Set static hostname to `IAMHUNG` (There are some people said leave it `localhost` which corresponds to `127.0.0.1` entry in `/etc/hosts` is better?)
- Enables smart MAC randomization and IPv6 privacy extensions and disables connectivity checks at `/etc/NetworkManager/conf.d/30-nm-privacy.conf`
- Increase Wi-Fi performance for `iwlwifi` at `/etc/modprobe.d/wireless-perf.conf`
- Config `iamhung` hotspot, disable password prompt when enabling (you could auto prompt using the following command `echo '802-11-wireless-security.psk:[PASSWORD]' | nmcli connection up id [HOTSPOT_NAME] passwd-file /dev/fd/0`)

[Fedora Docs "Networking/CLI"](https://fedoraproject.org/wiki/Networking/CLI)

### OCR tool

1. [tesseract-ocr/tesseract](https://github.com/tesseract-ocr/tesseract) [#untested]()

- Fast variant of the best models, trade little accuracy for much faster speed: [tesseract-ocr/tessdata](https://github.com/tesseract-ocr/tessdata)
- Best models: [https://github.com/tesseract-ocr/tessdata_best](https://github.com/https://github.com/tesseract-ocr/tessdata_best).

2. [mingShiba's Visual Novel OCR](https://www.patreon.com/mingshiba) [#untested](): not FOSS, Windows-only.

3. [Capture2Text](https://capture2text.sourceforge.net) [#untested](): Windows-only.

4. [ShareX's OCR](https://getsharex.com) [#untested](): Windows-only.

5. [Kanjitomo](https://www.kanjitomo.net) [#untested](): Java-based. Not working properly under Wayland. (last update was 2020)

6. [**kha-white/manga-ocr**](https://github.com/kha-white/manga-ocr)

- Cross-platform, high accuracy
- Setup details:

  - Create Python virtual environment: `python3 -m venv ~/FOSS/py-manga-ocr`, activate virtual environment: `source ~/FOSS/py-manga-ocr/bin/activate[.fish]` (if you're using Fish, remove `[]`)
  - Install `manga-ocr` PIP package: `pip install manga-ocr`.
  - Symlink `~/FOSS/py-manga-ocr/bin/manga_ocr` to `~/.local/bin`.

- DO NOT INSTALL CUSTOM PYTORCH ON AMD DEVICES, MANGA OCR WILL RUN CUDA AND CRASH THE MACHINE.

### Out-of-memory Killer

> [!NOTE]
> OOM-killer is per-service, per-process, but NOT per-app.
> `systemd-oomd` monitored entire `cgroup`, when it exceeds its limit, whole `cgroup` is killed and not just the most resource-intensive process.
> `systemd-oomd` is different from Kernel OOM. The latter tends to be much slower and only kicked in to protect kernel. It doesn't prevent desktop freezing.

> [!TIP]
> Use `free` tool and run `free -m` to diagnose RAM usage.
> Use `ps` tool and run `ps aux` to find out which apps use the most amount of memory.
> Use `sysstat` to collect report information about CPU, Disk usage, Network, Battery, I/O, ... Remember to mark `ENABLED=true` inside `/etc/default/sysstat`.
>
> If a task is terminated in order to save memory, it will be logged into log files which are stored at `/var/log`. To search for _out-of-memory_ messages: `sudo grep -i -r 'out of memory' /var/log/`
> Investigate further for suspicious activities:
>
> - High access from 1 IP address.
> - High access to unavailable resources.
> - High incoming number of reqs from 1 type (e.g. HTTP POST).
> - High number of failed login requests attempts.

After getting my VSCodium Flatpak killed numerous times, I've made `systemd-oomd` less "murderours" by modifying some threshold:

```conf
# Make systemd-oomd less "murderous"
[OOM]
SwapUsedLimit=95% # def=90%
DefaultMemoryPressureLimit=80% # def=60%
DefaultMemoryPressureDurationSec=60s # def=30s
```

**Prevent crucial processes (e.g. for web apps) are terminated due to OOM situation by:**

1. Disallow processes from _overcommitting memory_, a.k.a give them a lot of _virtual memory_ to use with no guarantee that the _physical memory_ can afford it when the time comes:

- Check `overcommit_memory` status: `cat /proc/sys/vm/overcommit_memory`. To disable it, run `echo 2 > /proc/sys/vm/overcommit_memory`.
- Check `overcommit_ratio`: `cat /proc/sys/vm/overcommit_ratio`. Only accounted for when `overcommit_memory = 2`. Indicates the amount of physical RAM is used. Swap space goes on top of that. Increase the value: `echo 85 > /proc/sys/vm/overcommit_ratio`.  
  Or:
- Edit `vm.oom-kill=0`, modify `vm.overcommit_memory=2` and `vm.overcommit_ratio=85`
  Or:
- Using CLI commands: `sudo sysctl vm.overcommit_memory=2 && sudo sysctl vm.overcommit_ratio=100`  
  Finally, restart the machine to take effect.

2. Increase the amount of memory (not happening, my laptop's RAM can't be upgraded).

3. (Not recommended) Eliminate `systemd-oomd` service completely by masking it `sudo systemctl mask systemd-oomd`.

**Bugs:**<br />

- System crash too frequently. RAM usage 100%. OOM Killer did not work. [#ubuntu2204]()  
  => Install `Auto Tab Discard` Firefox extensions to save memory.

[voretaq7's answer on Serverfault](https://serverfault.com/a/142003/1138548)  
[Amin Nahdy's "How to fix high memory usage in Linux"](https://net2.com/how-to-fix-high-memory-usage-in-linux/)
[phemmer's answer on Serverfault](https://serverfault.com/a/362625/1138548)

### Power/Battery

> [!TIP]
> Check battery health on Windows: `powercfg /batteryreport`
> Check battery health on Linux in general: `upower -i /org/freedesktop/UPower/devices/battery_BAT0`

- Check if Conservation Mode is enabled on boot [#xdgautostart]().

1. `power-profiles-daemon` [#fedorakde40]() [#builtin]() [#uninstalled]()

- Due to conflict with `tlp`, also it's said to be less optimized than `tlp`.

2. `tlp` and `tlp-stat` [#fedorakde40]() [#dnf]()

- Configure `/etc/tlp.d/00-brace.conf` by following [diversedcg/Brace](https://github.com/divestedcg/Brace), allow for better power savings on AC.
- Configure Wifi Power Management: https://gist.github.com/jcberthon/ea8cfe278998968ba7c5a95344bc8b55

[TLP docs "Optimizing"](https://linrunner.de/tlp/support/optimizing.html)  
[Michael Wu's comment on optimizing battery life using tlp](https://community.frame.work/t/tracking-linux-battery-life-tuning/6665/204)

### RAM Optimization

1. Put `/tmp` on a `tmpfs` partition [#fedorakde40]() [#builtin]()

### Screen Locking

1. [gogama/lockheed](https://github.com/gogama/lockheed/) [#ubuntu2204]()

2. `sddm` [#fedorakde40]() [#builtin]()

- Disable second monitor while login by:

  - Disable the unwanted monitors.
  - `System Settings > Colors & Themes > Global Theme > Login Screen (SDDM)`: Click _Apply Plasma Settings_.
  - Enable the unwanted monitors.
  - Tested: only boot login screen shows 1 monitor, wakeup from suspend login screen still shows 2 monitor.

- If there is no users logged in, `sddm` not suspend itself after some time.
  => Workaround: [Follow kelvie's comment](https://github.com/sddm/sddm/issues/1302#issuecomment-2376015039)

[Arch's docs about setup Wayland for SDDM](https://wiki.archlinux.org/title/SDDM#Wayland).  
[Reddit comment about disabling second monitor while logging](https://www.reddit.com/r/Fedora/comments/13u6nd6/comment/jm6z9ax)

3. `i3lock` [#untested]()

- Wake from suspend: [Raymo111/i3lock on wake from sleep or hibernate.md](https://gist.github.com/Raymo111/91ffd256b7aca6a85e8a99d6331d3b7b)

### Settings

1. `dconf-editor` [#ubuntu2204]() [#apt]()

> [!TIP]
> Add custom values, rather than using limited choice from pre-defined values.

- **Privacy > Screen > Blank Screen Delay** === `/org/gnome/desktop/session/idle-delay`
- **Privacy > Screen > Automatic Screen Lock Delay** === `/org/gnome/desktop/screensaver/lock-delay`

> [!CAUTION]
> DO NOT USE FRACTIONAL SCALE!

- Set `text-scaling-factor` to `1.25` for external monitor, `2.0` for laptop monitor,

2. `gsettings` [#ubuntu2204]() [#builtin]()

3. System Settings or `systemsettings6` [#fedorakde40]() [#builtin]()

- _Volume balance_: Right lounder, Left small
- Keyboard macro:
  - Clear all active notifications: `Ctrl + Alt + C`
  - Planify Quick Add Task: `Ctrl + Alt + A`
  - Anki Quick Add Card: `Ctrl + Shift + Alt + A`
  - Toggle Night Light: `Ctrl + Alt + M`
  - Set default brightness: `Ctrl + Alt + B`
  - Toggle FPS: `Ctrl + Alt + P`.
  - Increase brightness: `Ctrl Alt +`
  - Decrease brightness: `Ctrl Alt -`
  - Turn off monitor: `Ctrl + Alt + ~`
- Task Switcher:
  - Doesn't have the glowing blue box when switching between windows like Ubuntu.

**Bugs**:

- Since the update of Plasma 6, I've been unable to add new shortcuts.
  => Debug: setup keyboard shortcuts via System Settings GUI that's opened via terminal: `kcmshell6 kcm_keys`

Turns out it's bad to use the absolute path. When a new shortcut is created, a `.desktop` file whose name follows the pattern `net.local.[COMMAND_NAME]`, but the command name parsing logic is taking a string that's "anything from the first character to the character before the first encountered space character" and parsing the command. If the string is not ideal, the parsing could be terrible

I've seen a file whose name is `net.local.CURRENT=$(ddcutil.desktop`, because my command was `CURRENT=$(ddcutil getvcp 10 --display 1 | cut -d'=' -f2- | cut -d',' -f1 | awk '{$1=$1};1'); ddcutil setvcp 10 $((CURRENT + 1)) --display 1`. If your command starts with something like `/home/$USER/.local/bin/...`, `/` is used in file name, results in folder creation:

- `net.local./`
- `net.local./home`
- `net.local./home/$USER`
- `net.local./home/$USER/.local` so on and so forth.

=> Workaround: Don't use absolute path! Wrap your command in `sh -c`.

> [!CAUTION]
> DO NOT GO INTO TOUCHPAD SETTING FIRST [#bug]() [#untested]()

4. Misc

- Setup Desktop panel:
  - Favorite apps
  - Widgets (CPU, RAM, Disk, Network, ... usage)
  - Icons
  - Date and Time.
- `qt-qdbusviewer` (not support Qt5 or Qt6)
- `qdbus-qt6` and `kwriteconfig6`:
  - Legacy way, due to `reconfigure` not working after updating to Plasma 6: `kwriteconfig6 --file ~/.config/kwinrc --group Plugins --key [PLUGIN_NAME] "[VALUES]" && qdbus-qt6 org.kde.KWin /KWin reconfigure`.
    [diodiodiosaur's answer on Reddit](https://www.reddit.com/r/kde/comments/115d9ih/comment/j917ly6)
  - New way: [dashmeshsingh98's answer on Reddit](https://www.reddit.com/r/kde/comments/1bajci4/how_do_i_reload_kwin_from_terminal_plasma_601). This is used to toggle Desktop Effects using keyboard macro.
- `gdbus`
- `gsettings` (weird, why is this get installed in a KDE environment?)
- somehow `/usr/bin/xdg-mime` called to `qtpaths` instead of `qtpaths6` when setting up default web browser, create a symbolic link: `sudo ln -s $(which qtpaths6) /usr/bin/qtpaths`

### Sound

1. "Wake-up" speaker [#fedorakde40]() [#builtin]()

- Only method that required playing a little bit of music is working.

### SSD optimization

1. Edit `/etc/fstab` mount options [#fedorakde40]()

[Mutschler's "Fedora Workstation: Things to do after installation (Apps, Settings, and Tweaks) > btrfs filesystem optimizations"](https://mutschler.dev/linux/fedora-post-install/#btrfs-filesystem-optimizations)

2. TRIM [#fedorakde40]()

> [!TIP]
> TRIM only needed if you're doing a lot of deletion day-after-day.

- Using `discard=async` mount option.
- Manual: `sudo fstrim -av`

- Discouraged TRIM by `rc.local`
- Discouraged TRIM by `discard` mount option
  => Nope, this statement is deduced before `discard=async` mount option is implemented. It's a `discard` that doesn't do TRIM immediately, ASAP, but spread out over maybe a few miutes to balance I/O.

[Atemu12's comment](https://www.reddit.com/r/btrfs/comments/12xyvk6/comment/jhmneq9)

3. (unnecessary) SSD alignment and over-provisioning

- For Ubuntu, Debian, Linux Mint: [Pjotr's "Unnecessary: Alignment"](https://easylinuxtipsproject.blogspot.com/p/ssd.html#ID14.1)
- [Pjotr's "Unnecessary: Over-Provisioning"](https://easylinuxtipsproject.blogspot.com/p/ssd.html#ID14.2)

### Swap strategy

1. Swap on Disk [#ubuntu2204]() [#builtin]()

- Create a `/swapfile`, 8GB.

2. `zram` and `zram-generator` [#fedorakde40]() [#builtin]()

[Different zRAM tuning settings](https://www.reddit.com/r/Fedora/comments/mzun99/new_zram_tuning_benchmarks/)

```
# Default value
# Actual disk swap is an order of magnitude slower and the kernel was made to work acceptably well with that,
# Docs: https://www.kernel.org/doc/html/latest/admin-guide/sysctl/vm.html

vm.overcommit_memory = 0
vm.overcommit_ratio = 50
# best according to the formula if reading from swap is 10x faster than reading from disk
vm.swappiness = 180
vm.vfs_cache_pressure = 100
# Prevents uncompressing any more than absolutely have to
vm.page-cluster = 0
vm.dirty_ratio = 20
vm.dirty_background_ratio = 10
```

> [!TIP]
> Default system-level config file: `/usr/lib/systemd/zram-generator.conf`
> Custom system-level config files: either `/etc/systemd/zram-generator.conf` to override it or `/etc/systemd/zram-generator.conf.d/*.conf` drop-ins to augment it.
>
> Check zRAM statistics: `zramctl`
> Check if zRAM is enabled: `cat /proc/swaps`
> Check available compression algorithm, the default one will be contained inside brackets: `cat /sys/block/zram0/comp_algorithm`
> Check swappiness level: `cat /proc/sys/vm/swappiness`. Modify it by calling: `sysctl vm.swappiness=200` > `zstd` and `lz4` are two of the best compression algorithms.

**When you should use zRAM:**<br />

- Low disk space, slow HDD
- 1 program that uses more than half of RAM actively (games, blender, ...)
- Many little programs (firefox with a lot of tabs)

- Custom zram config file: `/etc/systemd/zram-generator.conf`
  - Set swap device using `zstd` (fastest, according to [this benchmark](https://github.com/facebook/zstd?tab=readme-ov-file#benchmarks))
  - Swap device takes half of the entire available RAM.
  - (default) compression ratio is 3-to-1.
- Custom `systctl` config file: `/etc/sysctl.d/99-vm-zram-parameters.conf`, change _swappiness_ to `180` to prefer using zram all the time.

[Arch Wiki's "zram"](https://wiki.archlinux.org/title/Zram)

3. swap on `zswap`

- Haven't tried.

### Wake up

1. `rtcwake` [#fedorakde40]() [#builtin]()

2. `/sys/class/rtc/rtc0/wakealarm` and `/proc/driver/rtc` [#fedorakde40]() [#builtin]()
   > [!CAUTION]
   > Not recommended.

- The manual way.
- Might have to convert your timezone to `GMT+0` (undocumented?)

## 3rd-party Applications

### Antivirus

1. Virustotal [#fedorakde40]() [#browserext]()

### Audio editor

1. Audacity [#fedorakde40]() [#appimage]() [#failed]()

- Unable to run AppImage file, waiting for newer version.

### Authentication

1. [boltgolt/howdy](https://github.com/boltgolt/howdy) [#ubuntu2204]() [#ppa]() [#fedorakde40]() [#copr]()

> [!CAUTION]
> DO NOT RELY ON AUTOMATED SCRIPT TO SET UP COPR.

- COPR setup on Fedora.
  [howdy's step-by-step setup on COPR](https://copr.fedorainfracloud.org/coprs/principis/howdy/)

- Clone its config file to `.dotfiles`.
  > [!TIP]
  > Config file location:
  >
  > - Ubuntu: `/usr/lib/secujrity/howdy/config.ini`
  > - Fedora: `/usr/lib64/security/howdy/config.ini`

**Bugs:**

- IR emitter not working in Lenovo Yoga Slim 7 Pro [howdy's GitHub Issue #269](https://github.com/boltgolt/howdy/issues/269)
  => Workaround: Install [EmixamPP's support for infrared cameras](https://github.com/EmixamPP/linux-enable-ir-emitter

- Option `--user` not working.
  => Haven't tested yet.

- CPU consumption bug when used with `vsftpd`
  => Assumption: Howdy can't be used for FTP server auth. [howdy's GitHub Issue #895](https://github.com/boltgolt/howdy/issues/895)

### Backup

Both on-site and off-site.

1. `timeshift` [#ubuntu2204]() [#apt]() [#fedorakde40]() [#dnf]()

- Haven't tried yet. Will use when I need to reinstall Ubuntu.

2. `deja-dup` [#ubuntu2204]() [#builtin]() [#fedorakde40]() [#flatpak]() [#uninstalled]()
   > [CAUTION]
   > For Flatpak version, you need to set proper permissions inside Flatseal or System Settings (KDE)

**Bugs**:

- Have permission issues, some files are missed. Some files aren't recoverable.

4. [digint/btrbk](https://github.com/digint/btrbk) [#fedorakde40]() [#dnf]()

> [!TIP]
> To exclude directories from snapshot and backup, you must convert them into nested subvolumes. [According to Massimo-B's comment](https://github.com/digint/btrbk/issues/258#issuecomment-440170385)
>
> - Step 1: Create backup: `mv "$ORI_DIR" "$BACKUP_DIR"`
> - Step 2: Create subvolume: `sudo btrfs subvolume create "$ORI_DIR"`
> - Step 3: Transfer methods
>   - `mv ...`
>   - `cp --archive --one-file-system --reflink=always ...` (recommended)
>   - `cp --archive --update ...`
> - Step 4 (after testing): Remove backup: `rm -rf [--one-file-system] "$BACKUP_DIR"`

> [!CAUTION]
> The documentation is HORRIBLE! It's one of the worst docs I've read.
> Luckily, [Aiyomoo's Reddit comment](https://www.reddit.com/r/btrfs/comments/1e4lyi6/comment/ldg0cbk/) grant me insights that could have been achieved if the author put more work on the docs.
>
> Backup reflects snapshot, so if a directory has been changed to nexted subvolumes yet still get included inside your backup, that means the snapshot still held the directory. You have to expire the current snapshot (override snapshot retention policy to ASAP) and create the latext one. Also if you planned to "remove the directory now nested subvolume by creating a 2nd backup", make sure that the 1st backup is expired by overriding the backup retention policy to ASAP.

- Maintain `/etc/btrbk/btrbk.conf` [#symlink]()
- Create config file: `/etc/btrbk/btrbk.conf`
  - Snapshot directory: `/mnt/btrfs/_btrfs_snap`
  - Backup subvolume (inside external SSD): `/run/media/$USER/Fedora-Backup`
- Modify `/lib/systemd/system/btrbk.timer`.
  - Create override: `/lib/systemd/system/btrbk.timer.d/00-override.conf`, basically run `btrbk` hourly.
- Modify `/lib/systemd/system/btrbk.service`.
  - Change override: `/lib/systemd/system/btrbk.service.d/00-override.conf`, basically run _snapshot_ only. Leave _backup_ for manual operation.

### BitTorrent client

1. qBittorrent [#ubuntu2204]() [#ppa]() [#fedorakde40]() [#flatpak]() [#nas]() [#docket]()

**Q: Why I switched from Flatpak to Docker image?**  
=> Answer #1: Feeling unease that `qbittorrent` Flatpak need `filesystems=host` permission although it shouldn't.
[Opened GitHub Issue #121](https://github.com/flathub/org.qbittorrent.qBittorrent/issues/121).  
=> Work in progress: Changing `--filesystem=host` to `--filesystem=xdg-download` could work, but it's not officially documented and could (unlikely) lead to breakage. [qBittorrent Flatpak's GitHue Issue #104](https://github.com/flathub/org.qbittorrent.qBittorrent/issues/104).  
Similar problems happened with `Transmission` Flatpak [Transmission Flatpak's GitHub Issue #31](https://github.com/flathub/com.transmissionbt.Transmission/issues/31)

=> Answer #2: Seamless integration with Servarr stack implemented on Docker (love the internal DNS resolver feature).

[Differences between **Usenet** and **BitTorrent**](https://www.reddit.com/r/usenet/comments/opiibl/comment/h67xzgn)

### Browser

1. Mozilla Firefox [#ubuntu2204]() [#prebuilt-binary]() [#standard-edition]() [#fedorakde40]() [#flatpak]()

To be able to access `about:config` on Firefox Android, access it at `chrome://geckoview/content/config.xhtml`

```js
user_pref("network.trr.uri", "https://1.1.1.1/dns-query"); // Cloudflare DNS
user_pref("network.trr.mode", 2); // Enable DoH
```

**There are 6 different versions:**<br/>

- Rapid Release: major updates every 4 weeks, minor updates (crash fixes, security patch, policy updates) are shipped during those 4 weeks:

  - Firefox: standard.
  - Firefox Beta: sneak peek at the latest features, before they get released in standard. Sends data to devs for feature improvements so not really private. 2-3 updates/week.
  - Firefox Developer Edition: Built on top of Beta, what changes in Beta are reflected in Developer Edition. More DevTool features. 2-3 updates/week.
  - Firefox Nightly: fastest to ship features, but bug-prone and crash-prone. Use this if you want to be a testing guinea pig. 2 updates/day.

- Firefox Extended Support Release (Firefox ESR): stability in exchange for latest tech, built for Enterprise.
  Major updates every 42 weeks. Minor updates every 4 weeks.
- Firefox Android.
- Firefox Android Beta.
- Firefox Android Nightly.
- Firefox iOS / Firefox TestFlight.

**UI Customization**<br />
[Dedoimedo's "How to customize Firefox UI - step-by-step tutorial"](https://www.dedoimedo.com/computers/firefox-change-ui-tutorial.html)  
[Cascade style](https://github.com/cascadefox/cascade)  
[Waterfall style](https://github.com/crambaud/waterfall)

**Site Isolation:**

[Madaidan's Insecurities' "Firefox and Chromium"](https://madaidans-insecurities.github.io/firefox-chromium.html)

> [!NOTE]
> On Ubuntu, Snap and APT version are one (Ubuntu is at fault here). However, if you add Mozilla's PPA, `apt` will install a different version.
> Firefox blocks about 80 ports by default.
> IO read/write are not Disk read/write. IO includes Disk. IO also includes Sockets, IPC, other data transferring methods, and Firefox use a lot of IPC so it won't be shown in Disk read/write.

> [!TIP]
> Flatpak profile location: `/home/silverbullet069/.var/app/org.mozilla.firefox/.mozilla/firefox/`
> Maximum privacy: after install a browser, disconnect from internet, launch the browser then turn off telemetry, quit, connect to internet, then launch the browser again.
> Using Firefox ergonomically [Navigational Instrument](https://exple.tive.org/blarg/2020/10/25/navigational-instruments/)

> [!WARNING]
> Flatpak package `org.freedesktop.Platform.ffmpeg-full` that is needed for Firefox Flatpak, can't be auto-update.

- Custom `.desktop` file [#desktop]() [#symlink]()
- Restore backup using Firefox Sync.
- RAM Cache for anything.

- Custom `user.js`, `userChrome.css` and `userContent.css` [#hardlink]()
- Extension list:

  - AI Grammar Checker & Paraphraser – LanguageTool (optimized)
  - Auto Tab Discard (very complex)
  - Bitwarden Password Manager (optimized)
    - Autofill: `Ctrl + Shift + F`
    - Lock: `Alt + Z`
  - Dark Reader (default is good enough)
  - Redirector (optimized)
  - Search by Image (default is good enough)
  - SponsorBlock for YouTube - Skip Sponsorships (optimized)
  - To Google Translate (disabled)
  - Tree Style Tab (very complex)
  - TWP - Translate Web Pages (optimized)
  - uBlock Origin (default is good enough)
    - Conflict with Privacy Badger. And no longer using heuristic algorithm. Can be replaced by Total Cookie Protection that's enabled inside Strict Mode.
  - VT4Browsers (optimized)
  - Yomitan (fork of depreciated **Yomichan**) (optimized)
    - Provides a text
    - JMDict, JMnedict, KANJIDIC for starter.
    - Kanjium for pitch accent.
  - YouTube NonStop (default is good enough)

- I don't move profile data to `tmpfs` partition on RAM since it's not working with Flatpak version. [Speed up firefox with RAM cache and tmpfs in Linux](https://www.pcsuggest.com/speed-up-firefox-with-ram-cache-and-tmpfs-linux/)

[Pjotr's "Firefox: optimize its settings"](https://easylinuxtipsproject.blogspot.com/p/firefox.html)

**What could slow down Firefox:**<br />

- Accessibility Services.
- Slow SQLite DB.
  => `about:support > Places Database > Verify Integrity` to optimize `sqlite` database called Places.
- No DNS cache causes slow access time.
- Frequent disk access, less RAM access.

**Bugs:**<br/>

- Find a way to disable resizing Firefox Flatpak by pressing its window's edge when the window is snapped, since I always misclick.
  => Workaround: ignore, since using vertical tabs means no need to put cursor near window's edge.
- "You can't use speech synthesis since Speech Dispatcher won't open"
  => Workaround: Follow [Garrett LeSage's comment](https://discussion.fedoraproject.org/t/speech-synthesis-firefox-flatpak/112226/11) to create a user systemd unit file and run `/usr/bin/speech-dispatcher -t 0` whenever the system is boot up.
- If Mozilla Firefox Flatpak does not have devices permission, webcam can't be used.

2. LibreWolf [#fedorakde40]() [#dnf]()

I'm completely ignorant about the synchronization logic of Firefox Sync: If a new device sign-in, will its settings are taken from the sync server, or its existing settings is pushed into the sync server?

According to [socai223's answer](https://support.mozilla.org/en-US/questions/1157664#answer-962372), just sign-out all devices and reset the password. The first device to be sign-in will have its settings to be the SSOT.

After [mozilla/bedrock's commit d459add](https://github.com/mozilla/bedrock/commit/d459addab846d8144b61939b7f4310eb80c5470e) was pushed upstream, contributors have found out that:

- User now have to agree to a Terms of Service to use Mozilla Firefox.
- Firefox have removed any quotes that prevent Firefox from selling user data [change #1](https://github.com/mozilla/bedrock/commit/d459addab846d8144b61939b7f4310eb80c5470e#diff-5c93e7e7cbfacf0d6a8b3bc6d46b345019653051089e00d6fe5e09a531a79442R70), [change #2](https://github.com/mozilla/bedrock/commit/d459addab846d8144b61939b7f4310eb80c5470e#diff-a24e74e4595fa85440a2f4e7e5dcfe68aba6e1e593aef05a2d35581a91423847). All of this was made to take account to the change of California's CCPA law lately.

Also, around the time of writing this post (2025-03-06):

- Google Chrome finally "sunset" Manifest V2 which rendered Manifest V2 Extensions like uBlock Origin useless.
- Mozilla Foundation's CEO Mitchell Baker was fired in 2025-02-19. Before that, her salary was $3M in 2020, $5 in 2021, $7M in 2022. People suspect the money saving by firing developers go all to executives.

=> That's why I switched to LibreWolf. Perminantly.

3. Microsoft Edge [#ubuntu2204]() [#prebuilt-binary]() [#ppa]() [#fedorakde40]() [#prebuilt-binary](https://discussion.fedoraproject.org/t/install-edge/67186/4)

4. Google Chrome [#ubuntu2204]() [#prebuilt-binary]()

[Easy Linux Tips Project's "Google Chrome and Chromium: improve their settings"](https://easylinuxtipsproject.blogspot.com/p/chrome.html)

### Cloud API client

1. `rclone` [#ubuntu2204]() [#prebuilt-binary]() [#fedorakde40]() [#builtin]()

[blog.rymcg.tech "Continuous immediate file sync with Rclone"](https://blog.rymcg.tech/blog/linux/rclone_sync/)

### Containerization

1. Linuxserver.io's Docker images

- Very active, trustworthy. Most of its images are used in my Servarr stack.

> [!TIP]
> Customizable: [Linuxserver.io's Blog "Customizing our Containers"](https://www.linuxserver.io/blog/2019-09-14-customizing-our-containers#custom-scripts)

2. Hot.io's Docker images [#consideration]()

3. Podman [#consideration]()

- Known as Docker's alternative.

4. Portainer [#consideration]()

- GUI, user-friendly container management tool.

5. Diun [#consideration]()

- Receive notifications when a Docker image is updated on a Docker registry.

### Debloating

[#fedorakde40]()<br />

> [!IMPORTANT]
> Next time, use **Fedora Everything** to ignore bloated package groups that you don't need.

```sh
# DeAkonadisation
sudo dnf remove akregator kamoso mediawriter elisa-player kmag kgpg qt5-qdbusviewer kcharselect kcolorchooser dragon kmines kmahjongg kpat kruler kmousetool kmouth kolourpaint konversation krdc kfind kaddressbook kmail kontact korganizer ktnef kf5-akonadi-*

# DeMisc
sudo dnf remove im-chooser khelpcenter krfb kwalletmanager5 plasma-welcome libreoffice-* orca qemu-guest-agent spice-vdagent ibus-anthy ibus-typing-booster ibus-hangul ibus-libpinyin ibus-libzhuyin ibus-unikey

# DeConsideration
sudo dnf remove abrt plasma-thunderbolt
sudo dnf autoremove

# Reinstall some apps using Flatpak
flatpak install flathub --user \
  org.kde.kamoso \
  org.kde.akregator \
  org.fedoraproject.MediaWriter \
  org.kde.gwenview \
  org.kde.okular \
  org.kde.filelight \
  org.libreoffice.LibreOffice\
```

[boredsquirrel's post on Fedora Forum](https://discussion.fedoraproject.org/t/fedora-kde-39-40-upgrade-wayland-only-debloat-tips/117880?replies_to_post_number=1)<br />
[The two ways of "debloating" Fedora KDE](https://github.com/radbirb/Short-FedoraKDE-Tips?tab=readme-ov-file#the-two-ways-of-debloating-fedora-kde) <br />

### Desktop Environment tool

1. Extension Manager [#ubuntu2204]() [#flatpak]()

- [MartinPL/Tray-Icons-Reloaded](https://github.com/MartinPL/Tray-Icons-Reloaded)
- [ubuntu/gnome-shell-extension-appindicator](https://github.com/ubuntu/gnome-shell-extension-appindicator)
- [lucaswerkmeister/activate-window-by-title](https://github.com/lucaswerkmeister/activate-window-by-title). add tntegration with `ydotool`.
- [home-sweet-gnome/dash-to-panel](https://github.com/home-sweet-gnome/dash-to-panel):
  - No more two clicks to open your minimized apps.
  - There is bug that will make full-screen windows collapses with Ubuntu Dock after waking up from suspend
    => Workaround: create a custom service called `screen_watcher.service` [#service]() [#synclink]()
- [eliapasquali/power-profile-switcher](https://github.com/eliapasquali/power-profile-switcher)
- [fflewddur/tophat](https://github.com/fflewddur/tophat)
- [ickyicky/window-calls](https://github.com/ickyicky/window-calls)
- [sunwxg/gnome-shell-extension-unblank](https://github.com/sunwxg/gnome-shell-extension-unblank)
- [GSConnect/gnome-shell-extension-gsconnect](https://github.com/GSConnect/gnome-shell-extension-gsconnect): similar to KDE Connect, connect linux and phone.

2. `gnome-tweaks ` [#ubuntu2204]() [#apt]()

> [!TIP]
> Set wrong refresh rate can result in inferior font: `Antialising' > Subpixel/LCD`

**Bugs:**<br />

- "Suspend when laptop lid is closed" not working. Instead it disables the monitor of the laptop.

[Abhishek Prakash's "7 Ways to Customize Your Linux Desktop With GNOME Tweaks Tool"](https://itsfoss.com/gnome-tweak-tool/)

3. `gnome-shell-extensions` [#ubuntu2204]() [#apt]()

### Documentation

1. [tealdeer-rs/tealdeer](https://github.com/tealdeer-rs/tealdeer) [#fedorakde40]() [#prebuilt-binary]()

- Reconfigure `~/.config/tealdeer/config.toml`: change color and enable auto-update.
- Faster than normal [tldr](https://tldr.sh/)

### Directory structure

```txt
/opt                          # multi-user, 3rd-party, non package managers applications.
/usr/local/bin                # exe files from /opt (Flatpak system-level, ...) are symlinked to here
/home/silverbullet069/
  |--- .akashic-records       # everything about me
  |--- .dotfiles              # every config files that can be maintained externally
  |--- .fonts                 # custom fonts
  |--- .icons                 # custom icons for Icon= in .desktop files
  |--- .local/bin             # symlinked of executable files from (Fish plugins, Prebuiit binary, ...)
  |--- FOSS                   # like /opt, but for single user.
  |--- LocalRepository        # GitHub clone repositories
  |--- .personal_script       # legacy location to store automation scripts
  |--- .playground            # testing area for newly learned tech
  |--- sambashare             # file sharing via LAN
  |--- Docker                 # volume for non-dev applications run inside Docker container

Legacy:
/home/silverbullet069/
  |--- GitOpenSourceApps      # GitHub clone repositories
```

### Download Manager

1. [subhra74/xdm](https://github.com/subhra74/xdm/) [#ubuntu2204]() [#prebuilt-binary]()

- Autostart on boot.

**Bugs:**<br />

- [Can't open XDM via terminal](https://github.com/subhra74/xdm/issues/1130)
  => Solution: downgrade to v8.0.26

### Email service

1. Gmail [#saas]()

> [!TIP]
> Use **alias** on a free account:
> Privacy: create a new account, set that account as an alias for another account.
> Non-privacy: use `+` syntax. Typically for newsletter subscription so they can be filtered.

2. Proton Mail [#saas]()

- Free tier:
  - Doesn't include SMTP bridging.
  - 500MB mail storage + 2GB drive storage.

### Email client

1. Gmail Web Application [#saas]()

- Create forwarding and labelling rules.
- Slow, snappy responsive.

2. **Thunderbird** [#ubuntu2204]() [#fedorakde40]() [#flatpak]()

- For Gmail account: Can't filter emails inside Primary tag only.
  => Workaround: Filter at Web App level then it gets reflected at Thunderbird.
- Extension list:
  - Dark Reader
  - Darko.
  - Minimize on close.
- Add OpenPGP key pair. Not sending public key to key server.
- I don't move profile data to `tmpfs` partition on RAM since it's not working with Flatpak version. [Speed up firefox with RAM cache and tmpfs in Linux](https://www.pcsuggest.com/speed-up-firefox-with-ram-cache-and-tmpfs-linux/)

### File explorer

1. **GNOME Files** (formerly Nautilus) [#ubuntu2204]() [#builtin]()

**Bugs:**<br />

- Image preview not supported.
- Can't even press right-click if the directory is too crammed. Therefore can not open terminal.
- From parent directory view, can not add bookmark to a child directory.

2. **Dolphin** [#fedorakde40]() [#builtin]()

3. Thunar [#ubuntu2204]()

### File monitor

1. `inotifytools` [#fedorakde40]() [#builtin]()

2. Syncthing [#ubuntu2204]()

3. Watchman

### File sharing

1. FileZilla [#ubuntu2204]() [#apt]() [#fedorakde40]() [#dnf]()

2. `vsftpd` [#ubuntu2204]() [#apt]()

- Maintain config file [#symlink]()

**Bugs:**<br />

- Issue with Howdy integration.

3. `smbclient` [#ubuntu2204]() [#apt]()

4. `sambashare` [#ubuntu2204]() [#builtin]() [#fedorakde40]() [#dnf]()

- Create `~/.sambashare`
- Modify `ufw` to allow local connection
  => Redundant

[Fedora's Documentation "Setup](https://docs.fedoraproject.org/en-US/quick-docs/samba/)

### Financial management

1. MISA
2. Spreadsheet

- From J2TEAM.

### Flash OS Image

1. balenaEtcher [#ubuntu2204]() [#appimage]()

2. Rufus [#windows-only]()

3. [ventoy/Ventoy](https://github.com/ventoy/Ventoy) [#ubuntu2204]() [#fedorakde40]() [#prebuilt-binary]()

### Fonts and Icons

> [!TIP]
> Best fonts if they are used right: `Ubuntu, Noto, IBM Plex, Rubik, Droid`
> Fonts directory: `/usr/share/fonts` (system) or `~/.fonts` (user)
> Icons directory: `/usr/share/icons` (system) or `~/.icons` (user)
> Update font cache: `fc-cache -fv`

- Font: `Subpixel Rendering > RGB`
- List of installed fonts:
  - **Noto, Noto Sans** [#fedorakde40]()
  - Cascadia Code. [#fedorakde40]()
  - FiraCode NF. [#fedorakde40]()
  - MesloLGS NF. [#fedorakde40]()
- List of installed icons:
  - La Capitaine [#ubuntu2204]()

### Games

#### 1. Steins;Gate Steam HD (2009)

#### 2. Steins;Gate JAP (2009)

#### 3. Steins;Gate Elite (2018)

**Tweaks:**

- Running **Steins;Gate Elite** launcher outputs error: **Launch is failed**.
  => Workaround: Add command argument `EN` (maybe anything will do?)

- Running **Steins;Gate Elite** launcher, blank menu screen appeared (without spinning satelite image).
  => Cause (speculation): CRC mismatch from `bg.mpk` file in FitGirl's release. Each installation attempt extracts different amount of data inside this file, so they generates a different CRC hash.
  => Workaround: download from another publisher.

[FAQ/WORKAROUNDS](https://steamcommunity.com/app/412830/discussions/0/4340984986767980571)

### Gaming Platform

#### 1. Steam

[#ubuntu2204]() [#prebuilt-binary]() [#fedorakde40]() [#rpmfusion]()

**Tweaks:**

- UI is too small for high DPI monitor:

  - Add official environment variable: `env STEAM_FORCE_DESKTOPUI_SCALING=`. [#desktop]()
  - GTK-only (legacy): add `env GTK_SCALE=`.
  - KDE-only (untested): add `env QT_SCALE_FACTOR=`, or `QT_AUTO_SCREEN_SCALE_FACTOR= QT_SCREEN_SCALE_FACTORS=`

- Steam are banned from all Vietnam's ISPs.
  => Easy way: Write a script to turn on Cloudflare WARP along with Steam.
  => Advanced: Route `store.steampowered.com` access through WireGuard connecting to CloudFlare's VPN server.

#### 2. Lutris

Same as [Virtualization > 4. Lutris](#4-Lutris)

### Image editor

1. GIMP and `gimp-plugin-registry` [#ubuntu2204]() [#apt]() [#fedorakde40]() [#flatpak]()

2. Pinta [#ubuntu2204]() [#snap]()

3. Adobe (SaaS)

4. Canva (SaaS)

5. ImageMagick (`magick`) [#fedorakde40]() [#builtin]()

### Image viewer

1. GNOME Image Viewer [#ubuntu2204]() [#builtin]()

- Simple, decent.

2. Gwenview [#fedorakde40]() [#flatpak]()

### Input method

1. [BambooEngine/ibus-bamboo](https://github.com/BambooEngine/ibus-bamboo) [#ubuntu2204]() [#ppa]()

**Bugs:**<br />

- Only `ForwardKeyEvent I` setting worked, working inconsistency.
- Having issues when integrate with `keyd` to use hotkeys mimicing arrow key navigation when editing documents on Google Docs SAAS on Mozilla Firefox.

2. `ibus-panel-emoji` [#ubuntu2204]() [#builtin]()

**Bugs:**<br />

- The global hotkey: `Ctrl + .` was taken
  => Workaround: Run `gsettings set org.freedesktop.ibus.panel.emoji hotkey "[]`

3. `fcitx5` [#fedorakde40]() [#dnf]()

- `fcitx` without `5` is depreciated.
- Create script `~/.config/plasma-workspace/env/im.sh` to define global env vars. Application-specific env vars are placed inside their `.desktop` file.
- Autostart using XDG autostart. [#xdgautostart]()
- List of added Input Method Engine:

  - `fcitx5-unikey` - Vietnamese
  - `fcitx5-mozc` - Japanese
    - Customize some keyboard shortcuts
    - Docs: https://github.com/google/mozc/blob/master/docs/configurations.md

- Switch between different character sets:

**Bugs:**<br />

- Bug: `im-chooser` can't be run inside Wayland environment, user accidentally chose `X compose table` and can't change back to other input methods.
  => Workaround: remove `~/.config/imsettings/inputrc` (untested) or symlink to different file in `/etc/X11/xinit/xinput.d/`
- Application Launcher input method isn't stayed at English.

[Fcitx5's Docs "Setup Fcitx5"](https://fcitx-im.org/wiki/Setup_Fcitx_5)
[Fcitx5's Docs "Using Fcitx5 on Wayland"](https://fcitx-im.org/wiki/Using_Fcitx_5_on_Wayland)
[JorgeLDB's comment on Reddit](https://www.reddit.com/r/kde/comments/1ex0xk0/comment/ll89ids)

4. `im-chooser` [#fedorakde40]() [#builtin]()

- Uninstalled.

### Keyboard autoinput

1. [ReimuNotMoe/ydotool](https://github.com/ReimuNotMoe/ydotool) [#ubuntu2204]() [#apt]()

- Worked on Wayland.
- Could be a security issue if use it for automation so I don't use it again.

### Keyboard input remapper

1. **[rvaiya/keyd](https://github.com/rvaiya/keyd)** [#ubuntu2204]() [#selfbuilt]()

- Maintain `/etc/keyd/default.conf` config file. [#symlink]()

2. **[evremap](https://github.com/wez/evremap)** [#fedorakde40]() [#selfbuilt]()

- Maintain remapper config: `evremap.toml`
- Maintain service file: `evremap.service` [#synclink]()
- Symlink `~/FOSS/evremap/target/release/evremap` to `/usr/bin/evremap`

**Bugs:**<br />

- [evremap's running permission issues](https://github.com/wez/evremap?tab=readme-ov-file#running-it)
  => Workaround: Apply [jbriales's issue](https://github.com/wez/evremap/issues/21). According to [group's not applied after logout](https://askubuntu.com/a/455442), it's best to reboot to perminently apply `input` group.

3. [One-on-one key mapping without 3rd-party solutions](https://www.reddit.com/r/linux_gaming/comments/nypsi1/updated_guide_to_remapping_keys_on_linux_using/)

4. **Keyboard with QMK/VIA support**

> [!IMPORTANT]
> This is the best solution if your workplace's security policy doesn't allow installing 3rd-party softwares.

- My `Keychron K3` isn't supported.

5. `libinput-tools` [#ubuntu2204]() [#apt]()

### Language Learning

> Spaced Repetition System, also known as the "Leitner System".

1. **Anki-family** (Anki desktop, AnkiWeb, AnkiDroid (android))

**List of plugins:**

- AnkiConnect.
- Ankimon by Unlucky-life. [#uninstalled]()
- Customize Keyboard Shortcuts.
- Review Heatmap.

**List of considered plugins:**

- Image Occlusion Enhanced.
- Speed Focus Mode.
- AnkiDraw.
- Cloze Overlapper.

2. SuperMemo
   **Why I haven't tried it yet?**

- There aren't any Linux port so it must be run using virtualization.

### Linux to Phone

1. FileZilla + FTP server [#ubuntu2204]()

- Half-measure. It's too manually.

2. KDE Connect [#fedorakde40]() [#builtin]()

### Manufacturer software

1. Lenovo Vantage [#windows-only]()

> [!WARNING]
> Old hardwares install old driver cause compatibility issues with Windows 11.

- Disable Zero Touch Login
- Turn on Conservation Mode
- Turn on Rapid Charge (only in Windows ?)

2. Samsung Magician [#windows-only]

3. CrystalDiskInfo [#windows-only]

### Measurement

1. `kruler` [#fedorakde40]() [#flatpak]()

2. `screenruler` [#ubuntu2204]() [#apt]()

### Mouse programming

1. EDRA EM6502 [#windows-only]()

### Office suite

1. Microsoft Office [#windows-only]()

- Cracked via [massgravel/Microsoft-Activation-Scripts](https://github.com/massgravel/Microsoft-Activation-Scripts)

2. LibreOffice [#fedorakde40]() [#flatpak]()

3. Google Docs, Google Spreadsheet, Google Presentation [#saas]()

### Package manager, format and repository

1. `apt`, `.deb` and PPA [#ubuntu2204]() [#builtin]()
   > [!TIP] > `apt-get` is best for automated scripts. `apt` is best for interactive use.

- Turn on all Repository at `Software & Updates > Ubuntu XX.XX Software > Downloadable from the Internet`, except for source code.

2. `dnf`, `.rpm` and RPMFusion, COPR [#fedorakde40]() [#builtin]()

> [!NOTE]
> Fedora 41 will install `dnf5`.

> [!TIP]
> View list: `dnf group list -v`
> Force reset metadata using `sudo dnf --refresh [COMMAND]`
> Search: `dnf search [PACKAGE_NAME]` or `rpm -q [PACKAGE_NAME]`
> List GPG pubkey: `sudo rpm -q gpg-pubkey --qf '%{NAME}-%{VERSION}-%{RELEASE}\t%{SUMMARY}\n'`
> Delete GPG pubkey: `sudo rpm -e [GPG_KEY]`
> Repositories directory: ` cd /etc/yum.repos.d/`

- Installed Development Tools: `sudo dnf groupinstall "Development Tools"`
- Optimize by modifying `/etc/dnf/dnf.conf`:
  - Unnecessary `deltarpm=true`
- Enable RPMFusion:
  - Using `Discover`
  - Using CLI: `sudo dnf install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm`

3. Snap [#ubuntu2204]() [#builtin]()
   > [!WARNING]
   > This is Ubuntu's proprietary package manager. I would rather not using it.

- Snap packages are installed into `/` partition by default.
  => Workaround: Relocate from `/` to `/home`, from [AskUbuntu user](https://askubuntu.com/a/1070635), modified version `~/.dotfiles/relocate_script.sh`

4. PyPI and Pip

- Install Python packages.

[#bugs]():

- During the time that I had to install Pytorch with ROCm support, the package's size is 4GB. Pip package when downloaded are stored inside `/tmp`, with the default storage size being half of total RAM (in my case, 8GB), it's simply not enough to handle the download. To tackle this problem:

* Quick workaround: run `TMPDIR=/var/tmp pip install --cache-dir=$TMPDIR [PACKAGE_NAME]`
* Long-term fix: `sudo mount -o remount,size=16G /tmp/`, check by running `df -h /tmp`

**Bugs:**<br />

- After relocating, by checking `journalctl -b | grep 'AVC apparmor="DENIED"`, every snap package have their permission denied
  => Solution: Default snap package are installed at system-level. Reinstall them at user-level.

4. Flatpak [#ubuntu2204]() [#apt]() [#fedorakde40]() [#builtin]()

**Issues of current model of packaging:**<br />

- Duplicate work when packaging apps: Linux distro diverses in Package Manager, package format and repository:
  - 1 application, 1 maintainer, 1 package format for 1 package manager.
  - What about apps that have no maintainers, only devs? Devs have to learn the language of each package format and does packaging for themselves.
  - Some apps only focus on some distros (Debian-family (Ubuntu) or Red Hat-family (Fedora) have the most support).
- There are apps that requires user builds their own binary. No package.
- Cross-distro packaging is time-consuming.
- Apps that are unmaintained and outdated.

**Flatpak proposes:**

- Run on any Linux distros.
- No amount of time will be wasted to package.
- Stability and Sandbox: Flatpak apps and runtimes are isolated from host, and isolated from one to another. Their breckage won't affect outer system. They have limited access to host environment (not by default, need some tweaks).
- Rootless install: No need `sudo` to install Flatpak app or a runtime.

**Flatpak drawbacks:**

- Immature data integrity checking mechanism. [flathub/flathub#1498](https://github.com/flathub/flathub/issues/1498)

> [!TIP]
> Always prefers Flatpak applications to distro's package manager.
> **Flathub** is not the only Flatpak repository (the best thou). Fedora maintained their own Flatpak repository as well.
>
> - System-level app's `.desktop` file location: `/usr/local/share/applications/`
> - User-level app's `.desktop` file location: `~/.local/share/flatpak/exports/share/applications`
> - Add custom icon by `/home/$USER/.local/share/flatpak/app/[APP_NAME]/current/active/export/share/icons/` and symlinked them to `/home/$USER/.local/share/flatpak/exports/share/icons/`
> - Print app directory: `flatpak info --show-location [NAME]`
> - Print permissions: `flatpak info --show-permissions [NAME]`
> - Print the recent versions: `flatpak remote-info --log flathub [NAME]`
> - Prevent auto update `flatpak mask -v [-u|--user] [--system] [NAME]`
> - Allow auto update `flatpak mask -v [-u|--user] [--system] --remove [NAME]`
> - Flatpak by default does not touch the configuration/settings when uninstalling. Delete app + data `flatpak uninstall --delete-data [NAME]`
> - Remove unused runtime `flatpak uninstall --unused`. There are some runtimes called "extensions" will be pinned and not get removed. You will have to manually remove them.

> [!WARNING]
> Don't use `flathubbeta`, it's unstable.
> Flatpak consumes a lot of disk storage.

**Tweaks:**

- Enable `flathub` for both **system** and **user**:

```sh
flatpak remote-add --if-not-exists --user flathub https://flathub.org/repo/flathub.flatpakrepo && flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
```

- (Ubuntu only) Show Flatpak applications in Ubuntu Software Center by installing `gnome-software-plugin-flatpak`. Fedora's Discover can show Flatpak applications by default.
- (Ubuntu-only) Install Flatseal to manage permissions. Fedora's `System Settings > Security & Privacy > Application Permissions > Flatpak Permissions` can handle permissions by default.
- Warning log `flatpak /gdk/wayland/gdkcursor-wayland.c:210 cursor image size (64) not an integer multiple of theme size (24)`  
  => You're using unsupported cursor theme (in my case, it's `KDE Breeze`). Switch to `High Contrast`.
- Warning log `Gsk-WARNING **: 01:43:13.566: Clipping is broken, everything is clipped, but we didn't early-exit.`  
  => Filter out these output by running `flatpak run [APP_NAME] 2>&1 | grep --line-buffered -v -e "Clipping is broken, everything is clipped, but we didn't early-exit." -e '^[[:space:]]*$'`

[Why Flatpak applications have free drivers by default?](https://www.reddit.com/r/Fedora/comments/137e194/comment/jiudiht/)

5. [TheAssassin/AppImageLauncher](https://github.com/TheAssassin/AppImageLauncher) [#ubuntu2204]() [#ppa]()

- In Fedora, `.desktop` files are maintained manually.

6. [rpmfusion-infra/fedy](https://github.com/rpmfusion-infra/fedy)

### Password manager and OS Keyring

1. **BitWarden** [#saas]()

- Cloud-based solution, need to create an account. Simple, convenient, for non-tech savvy.

2. **KeePassXC** [#fedorakde40]() [#flatpak]()

- Replace both `KWallet` and `GNOME Keyring` with `KeePassXC` as system keyring using Secret Service Integration

* (NixOS only) Put `services.gnome3.gnome-keyring.enable = lib.mkForce false;`
  [pbogdan's comment](https://discourse.nixos.org/t/how-to-disable-gnome-keyring-daemon-automatic-startup/6717/3)

* (unrelated?) Activate Secret Service API of `kwallet`

```conf
# Modify ~/.config/kwalletrc
[Wallet]
...
Enabled=false
...

[org.freedesktop.secrets]
apiEnabled=true
```

[diwow's comment](https://forum.manjaro.org/t/replace-kwallet-with-keepassxc-as-keyring/159188/8)

- Stop gnome-keyring if it's installed:

```sh
pgrep -l gnome
# Funnily, kill the first process is enough
pkill -9 [PID]
```

- Then create a new group in my `Default` database called `System Keyring` that will hold the passwords used for the keyring.
- Enable Secret Service Integration: `Tools > Settings > Secret Service Integration`
- Export entries from `System Keyring` group: `Database > Database Settings > Secret Service Integration > Expose entries under this group`

[Chuck Nemeth's "KeepassXC as the System Keyring > Configure"](https://wiki.chucknemeth.com/linux/security/keyring/keepassxc-keyring#configure)

- According to [hackerb9's post on askubuntu](https://askubuntu.com/a/1437566), there are 4 attack vectors to successfully replace
- First, prevent PAM from utilizing GNOME Keyring

```sh
# List all files that contains entries that needed to be commented out
grep -r "pam_gnome_keyring.so" /etc/pam.d/*
# Go inside each file and comment out each of them
# Re-run the command to check if you have commented out correctly
```

> [!CAUTION]
> There is a pattern called `[default=<N>]` that you have to keep in mind, according to [Gregory Lee Bartholomew's comment](https://discussion.fedoraproject.org/t/replacing-gnome-keyring-with-keepassxc/104539/8)

- Second, prevent autostart on boot and manual start some systemd's servies that utilizes GNOME Keyring: `/usr/lib/systemd/user/gnome-keyring-daemon.service` and `/usr/lib/systemd/user/gnome-keyring-daemon.socket`

```sh
systemctl --user mask gnome-keyring-daemon.service
systemctl --user mask gnome-keyring-daemon.socket
```

- Third, disable system-level XDG autostart on boot by overriding at user-level

```sh
sh -c 'grep -rl "gnome-keyring-daemon" "/etc/xdg/autostart" | while read -r SYSTEM_FILE; do echo "$SYSTEM_FILE"; BASENAME=$(basename "$SYSTEM_FILE"); LOCAL_FILE="/home/$USER/.config/autostart/$BASENAME"; echo "$LOCAL_FILE"; echo -e "[Desktop Entry]\nHidden=true" > "$LOCAL_FILE"; done'
```

[Chuck Nemeth's "KeepassXC as the System Keyring > Disable gnome-keyring"](https://wiki.chucknemeth.com/linux/security/keyring/keepassxc-keyring#disable-gnome-keyring)

- Fourth, change system-level D-Bus services by overriding at user-level

```sh
sh -c 'grep -rl "gnome-keyring-daemon" "/usr/share/dbus-1/services" | while read -r SYSTEM_FILE; do echo "$SYSTEM_FILE"; BASENAME=$(basename "$SYSTEM_FILE"); LOCAL_FILE="/home/$USER/.local/share/dbus-1/services/$BASENAME"; SERVICE_NAME=$(echo "$BASENAME" | rev | cut -d"." -f2- | rev); rsync -avz --checksum "$SYSTEM_FILE" "$LOCAL_FILE"; echo -e "[D-BUS Service]\nName=${SERVICE_NAME}\nExec=/usr/bin/flatpak run org.keepassxc.KeePassXC" > "$LOCAL_FILE"; done'
```

- Check it works by having database unlocked and use `secret-tool --unlock` command.

[Stewarts's "KeepassXC as Secret Service"](https://www.stewarts.org.uk/post/keepassxcassecretservice/)  
[ntninja's comment](https://github.com/keepassxreboot/keepassxc/issues/6274#issuecomment-810983553)

3. KWallet [#fedorakde40]() [#builtin]()

- Disable PAM on KWallet by overriding its XDG autostart file. [#xdgautostart]()
- Pop-up appears when there is a password prompt (Wifi, LUKS decrypt, ...), therefore disable KWallet by adding `Enabled=true` to `~/.config/kwalletrc` [#hardlink]()

4. GNOME Keyring [#fedorakde40]() [#builtin]()

> [!TIP]
> It's a D-Bus service, so it provides varius D-Bus APIs to access its data.
> Check status: `busctl --user get-property org.freedesktop.secrets /org/freedesktop/secrets/collection/login org.freedesktop.Secret.Collection Locked`

### PDF editor

1. Firefox PDF editor [#builtin]()

2. Okular [#ubuntu2204]() [#snap]() [#fedorakde40]() [#flatpak]()

### Personal Media Server (Servarr stack)

[#fedorakde40]()

**What are Direct Play, Direct Stream and Transcoding?**<br />

- **Direct Play**: client app tells the server that itself and the device it's running on can serve the audio and/or video formats from the content as-is, without needing the server to do transcoding.
- **Direct Stream**: occurs if either audio, video, container or subs are not supported by client app. The server converts the media to a format that client can process. This conversion process can be either **Transcoding** or **Remuxing**.
- **Transcoding**: server converts the video/audio formats to formats that client can process. This process was CPU intensive before but can be accomplished using hardware accelleration via GPU now.
  - Decoding is less intensive than encoding.
  - Audio transcoding is less intensive than video transcoding.
- **Remuxing**: server converts the file container to a container that client can process. The least intensive process.

> [!CAUTION]
> Subs can unintentionally trigger Direct Stream, **Remuxing** if subs are remuxed to different formats, or **Video Transcoding** when subs are burned into original file.

**What are Uncompressed, Compressed Lossless and Compressed Lossy?**<br />

- **Uncompressed**: most info, compatible with old software, big size however. E.g. WAV, LPCM, ...
- **Compressed Lossless**: insignificant data loss with less size. E.g. FLAC, ALAC, ...
- **Compressed Lossy**: acceptable data loss, small size. E.g. MP3, AAC, ...

**4 different types of subtitles, unrelated to subtitle formats?**<br />

1. **Closed**: generic name for Forced, SDH, Closed Captitioning, Normal.

- Forced: provided when chars speak foreign/alien/sign language.
- SDH and Closed Captioning: for deaf and hard of hearing, include extra info like background noise, music played, ...
- Normal: can be turned on/off.

2. **Burned-in**: perminantly placed in video, can't be turned off.

[Jellyfin's Docs "Types of Subtitles"](https://jellyfin.org/docs/general/clients/codec-support/#types-of-subtitles)

**Table of media interconnection:**<br />
| | Samsung TV 4K | BT200 | Chromecast 4K | Jellyfin |
|-----------------|:--------------:|:-------------:|:------------------------------:|:----------------------:|
|Video codecs | via CC | unrelated | H264, VP8 | H264,H265,VP9,AV1 |
|HDR formats | 10/10+, no DV | unrelated | 10/10+,HLG,DV (P5,8, no 7) | 10/10+ |
|Audio codecs | via CC + BT200 |SBC (aptX LL ?)| DD(AC3),DD+(EAC3),FLAC,AAC,WAV | DD(AC3),DD+(EAC3),FLAC,|
| | | | (LPCM),DTS(?),MP3,Opus,Vorbis | AAC,DTS,MP3,Opus |
|Audio passthrough| ? | unsupported | DD(AC3),DD+(EAC3),DA | ? |
|Media container | via CC | unrelated | MP4, OGG, WAV, MP3, MP2T, WEBM | _MKV_,MP4,TS,OGG,WEBM |

\*MKV container have the most number of supported subtitle format.

[Understanding Bluetooth codecs](https://www.soundguys.com/understanding-bluetooth-codecs-15352/)  
[Google Cast Docs "Supported Media for Google Cast" (until 2024-10-02)](https://developers.google.com/cast/docs/media)

**HDR formats list:**

- Dolby Vision Profile 5: WEB-DL Dolby Vision + no HDR10 fallback
- Dolby Vision Profile 7: UHD BluRay Remuxes + UHD BluRay.
- Dolby Vision Profile 8: (Hybrid) WEB-DL (HULU), Hybrid UHD Remux, UHD BluRAY + HDR10 fallback.
- HDR10 and HDR10+: If HDR10+ metadata is ignored by the digital media server that unsupport this format, it will playback the video in HDR10.

**Audio formats list:** allow for best quality, without causing the audio signal to be compressed before reaching AVR, soundbar or speaker, especially true with Dolby TrueHD and DTS-HD MA.

- Dolby TrueHD ATMOS
- Digital Theater System X
- Dolby Digital Plus ATMOS
- Dolby TrueHD
- Digital Theater System - HD Master Audio
- FLAC/PCM
- Dolby Digital Plus (EAC3)
- Digital Theater System (DTS): decoded to LPCM on Apple TV
- AAC
- Dolby Digital (AC3): decoded to PCM on Apple TV

[What does my Media Player Support (until 2024-10-02)](https://docs.google.com/spreadsheets/d/15Wf_jy5WqOPShczFKQB28cCetBgAGcnA0mNOG-ePwDc)  
[Jellyfin's Docs "Codec Tables"](https://jellyfin.org/docs/general/clients/codec-support/)

> [!TIP]
> Jellyseerr requested yet Radarr/Sonarr not auto-downloading?
> Troubleshooting by going inside Radarr/Sonarr, do a manual interactive search and check the red marker to see the error.

[Servarr Wiki's "Radarr > Settings > Qualities Defined"](https://wiki.servarr.com/en/radarr/settings)  
[Adobe's "Understanding Audio Bitrate"](https://www.adobe.com/creativecloud/video/discover/audio-bitrate.html)

- Maintain Docker Compose file: `/opt/servarr/docker-compose.yml`
  - `hostname` is useless, `services:` name is used as DNS name (`container_name` will be used instead if provided) for inter-container communications.
  - `dns` to setup DNS resolver.
- Use `cron` to automatically update weekly. [#cronjob]()

> [!NOTE]
> Connecting via `localhost` won't work because there is no way to share host's `localhost` with container's `localhost` without using `network_mode: 'host'`.

1. Jellyfin Client, Server and `jellyfin-ffmpeg` [#flatpak]() [#docker]()

- In **Server > Playback > Transcoding** setting, choose `VA-API` as HWA (prefer on all GPUs, including Vega+ GPUs)

  - Tested. Remember to check for supported codecs and enabled them in Client app. Enabling unsupported ones will result in `ffmpeg` error.
  - Do not choose `AMF` if you're using Linux, it's based on closed-source solutions, Jellyfin can't look inside it so it's not fully supported on Linux, unlike AMF on Windows is based on open-source solutions.
  - Embedded subtitle is buggy as hell, [according to anthonylavado](https://www.reddit.com/r/jellyfin/comments/ui011w/jellyfin_subtitle_problematic_and_not_sure_why_it/i7aw478/).
  - List of plugins:
    - OpenSubtitles: Install 20 subtitles/day at MAX.
    - Subtitle Extract: Extract Burned-in subtitle into Normal subtitles.

  remember to config plugin tasks. Some aren't even configured.

- In `User > Playback > Video Advanced`, check if your sound system can process `DTS` or `TrueHD`, if not leave them.
- Change local account to easy-to-remembered password.
- In `Libraries > Libraries > 3 dots > Manage library`:
  - Disable the metadata provide `The Open Movie Database`. Change to use Local Metadata. Ref: https://jellyfin.org/docs/general/server/metadata/nfo/ to name your local metadata correctly although I assume the Kodi / Emby metadata provider had done it.
  - Enable "Only download subtitles that are a perfect match". If there isn't one available, I will do it manually.

[Jellyfin's Docs "HWA Tutorial On AMD GPU"](https://jellyfin.org/docs/general/administration/hardware-acceleration/amd/#configure-with-linux-virtualization)
[Jellyfin's Docs "Hardware Selection"](https://jellyfin.org/docs/general/administration/hardware-selection/): AMD is not supported very well.
[Jellyfin's Docs "Hardware Acceleration"](https://jellyfin.org/docs/general/administration/hardware-acceleration/)

2. Jellyseerr [#docker]()

- Change local account to easy-to-remembered password.
- To create season folders, go into _Settings > Services > Sonarr > Edit > Season Folders_

3. qBittorrent [#docker]()

- Option > Downloads > Saving Management:
  - Default Torrent Management Mode: `Automatic`
  - Default Save Path: `/data/torrents`
- `.Trash-1000` taking space? Option > Advanced > Torrent content removing mode: `Delete permanently`, according to [pedrobuffon's comment](https://www.reddit.com/r/sonarr/comments/1fv4lhd/when_importing_files_are_duplicated_on_a_trash1000/)
- Add 4 categories including save path.
- Everything except _Pre-allocate disk space for all files_, WebUI Authentication and Security [TRaSH's "qBittorrent Basic Setup"](https://trash-guides.info/Downloaders/qBittorrent/Basic-Setup/)
- If you change qBittorrent password but haven't changed it inside Sonarr/Radarr, chance that they would authenticate themselves to the point of being banned. Solution: add the following lines to `~/.config/qBittorrent/qBittorent.conf`

```sh
[Preferences]
IPFilter \ BannedIPs =
```

4. WireGuard [#docker]()

- Using **Cloudflare WARP** Profile, extracted from `wgcf`.

5. Flaresolverr [#docker]()

- Connected to **Prowlarr**.
- It couldn't bypass the rest of _1337x_ indexer urls, except `1337x.to`

6. Radarr [#docker]()

- Media Management: Min Free Space, Use Hardlinks instead of Copy, Propers and Repacks and Root Folders (2nd, 3rd and 4th are covered by TRaSH Guides), also enable "Unmonitor Deleted Movies"
- Quality: 0 to min, max to preferred.
- Indexer: delegate to **Prowlarr**, add `1337x` and `Nyaa.si` and add Multi Language `Original`, `English` to all of them, enabled _Allow Hardcoded Subs_.
  - Since Prowlarr doesn't include ALL appropriate categories, you must add them manually by editing the indexer in Radarr. Also make sure go to Prowlarr > Settings > Apps > Applications: Radarr/Sonarr and change sync level to Add and Remove only to avoid Prowlarr overriding the newly added tags.
- Download Client: **qBittorrent**.
- Quality Profile: create **UHB Bluray + WEB** - my template-custom profile with auto-update features from **Recyclarr**.
  - Preferred Language set to Any.

> [!TIP]
> Check the number of unmapped folder by visiting: `Media Management > Root Folders > /data/media/movies`, click onto the path to resolve each of them.
> Manual download film on Radarr UI (not via Jellyseerr) to fix unexpected errors.

[TRaSH Guide's "Radarr"](https://trash-guides.info/Radarr/).
[TRaSH Guide's "What does my media support"](https://trash-guides.info/Plex/what-does-my-media-player-support/).
[TRaSH Guide's "Hardlinks and Instant Moves (Atomic-Moves)"](https://trash-guides.info/Hardlinks/Hardlinks-and-Instant-Moves/)

**Bug:**

- Unknown Movies error.
  => Assumption: Must be hidden naming issue.

7. Sonarr [#docker]()

- Same as Radarr.
- For Media Managements's settings:
  - Follow [Servarr Wiki "Sonarr > Community Naming Suggestion](https://wiki.servarr.com/en/sonarr/settings#community-naming-suggestions) from [TRaSH's Guides](https://trash-guides.info/Sonarr/Sonarr-recommended-naming-scheme)
  - Change File Data: `UTC Release Date`
  - Disable "Unmonitor Deleted Episodes" since it makes no sense to have an unmonitored episode for the whole season.
- _Allow Hardcoded Subs_ option not existed.
- Unlike Radarr, `Preferred Language` option inside `Custom Profiles` not existed.
- Setup for a seperate Quality Profile on a seperate Sonarr instance dedicated to Anime is not necessary because I don't watch Anime on Sonarr or on my TV.
- Sonarr won't automatically load missing episodes, had to do it manually in Sonarr Web UI.
- In order to reorganize "flat" series episodes into their respective _Season Folder_, press the button "Preview Rename for this season".
- You might update new naming convention for Series Folder, to update the name for old series, follow ["Servarr Wiki's "Sonarr > FAQ: How can I rename my series folders""](https://wiki.servarr.com/sonarr/faq#how-can-i-rename-my-series-folders)
- Sometimes, `Standard` Series Type works better than `Anime` Series Type for anime. You can modify it either by Mass Editor (a workflow that involves _Select Series > choose series > Edit_. or from `Edit` button inside the series detail view.
- Download `Kodi (XMBC) / Emby` metadata standard and `The TVDB` metadata provider to enforce the use of local metadata.

8. Bazarr [#docker]()

- For now, it's not essential.

**Bugs:**

- Currently can't write to `/data/media/movies`. Fix later if Bazarr is needed.

9. Prowlarr [#docker]()

- Add **1337x.to**, **EZTV**, **Nyaa.si** indexers.
- Add FlareSolverr indexer procy.
  - Add `1337x` tag to use FlareSolverr to bypass Cloudflare DDOS Protection on `1337x.to`
- Add Sonarr and Radarr applications, change **Sync Profiles > Standard: Minimum Seeders** to 10.
- Add qBittorrent as download client.
- Change some UI settings.

[Servarr Docs's "Prowlarr Settings"](https://wiki.servarr.com/prowlarr/settings) and [Servarr Docs"Prowlarr FAQ"](https://wiki.servarr.com/prowlarr/faq)

**Rabbit holes**:

- Cannot connect to `1337x` torrent tracker:

  - Connection refused error.
    => Reason: ISP's DNS blocking policy.
    => Solution 1: on Fedora host, set `Cloudflare` as global DNS server and `Google` as fallback DNS server.
    => Solution 2: remove global + fallback DNS server. Set DNS server for Prowlarr image to Cloudflare's inside `docker-compose.yml`

  - SSL connection could not be established.
    => Troubleshoot on host: using `curl` shows _Connection reset by peer_ error, but Firefox worked?
    => Assumption: Firefox had `Cloudflare` as its DNS resolver with `DNS over HTTPS (DoH)`. It's the DoH that bypass ISP trapping DNS lookups using public DNS. But `curl` doesn't have DoH setting.
    [Jackett's Wiki "Troubleshooting: A connection attempt failed"](https://github.com/Jackett/Jackett/wiki/Troubleshooting/#a-connection-attempt-failed)
    => Solution 1: Run **Cloudflare WARP** on host and route Docker container's through host's network.
    => Solution 2: Run **Wireguard** Docker image, share its network with Prowlarr's, and route every traffic from Prowlarr to Wireguard.

[Linuxserver.io's "Routing Docker Host And Container Traffic Through Wireguard"](https://www.linuxserver.io/blog/routing-docker-host-and-container-traffic-through-wireguard)
[Linuxserver.io's "Advanced Wireguard Container Routing"](https://www.linuxserver.io/blog/advanced-wireguard-container-routing) (setup multiple WireGuard containers on a VPS to achieve split tunnelling)

10. Recyclarr [#docker]()

- Maintain config files: `/opt/servarr/appdata/recyclarr/config/recyclarr.yml`.

11. [qdm12/gluetun]: Lightweight, swiss-army-like VPN client, to multiple VPN service providers.

- Haven't tried.

**References**:

- https://wiki.servarr.com/useful-tools

### Personal Knowledge Management (PKM)

1. Obsidian [#ubuntu2204]() [#appimage]()

- Slow, stuttering

2. GitHub Gist [#ubuntu2204]() [#saas]()

- Hard to scale, good for quick note.

### Pomodoro timer

1. RSIBreak [#fedorakde40]() [#dnf]()

[#tweaks]():

- Disable short break.
- Set big break to 60 minutes.
- Break duration is 5 minutes.

- Default bahavior of a simple pop-up is appear in the middle of the screen and steal focus. Pretty irritating for me.
  => Create a Window Rule that override pop-up coordinates on screen and prevent focus stealing.

### Remote access desktop

1. SSH [#fedorakde40]() [#builtin]()

- Enable systemd service.
- Edit `/etc/ssh/sshd_config.d/00-custom.conf` to change default port, disable key-based authentication and prevent root access.
- Fedora utilizes `SELinux`, it denies port binding so you have to add a new rule: `semanage port -a -t ssh_port_t -p tcp 5522`, [accoring to Matt Clark](https://superuser.com/a/1122161/2206521)
- Create a new public key, manually add the public key to `~/.ssh/authorized_keys`. If you haven't disable key-based auth, you could install the public key remotely using `ssh-copy-id` command.

2. Parsec

[#ubuntu2204]() [#prebuilt-binary]() [#fedorakde40]() [#flatpak]()

> [!NOTE]
> Parsec don't have official Flatpak repository. After checking its GitHub repository, it's a repackage of the `.deb` package (SHA256 checked)
> [flathub/com.parsecgaming.parsec](https://github.com/flathub/com.parsecgaming.parsec)

- Linux can only be **the client**.
- Add custom icons file from `https://www.steamgriddb.com/icon/3711`.

### RSS Feed

1. Akregator [#fedorakde40]() [#uninstalled]()

### Sandbox

#### 1. [netblue30/firejail](https://github.com/netblue30/firejail)

- For running non-Flatpak applications.
- [lutris#1097, Hiradur's custom Lutris profile](https://github.com/lutris/lutris/issues/1097#issuecomment-1140392601). Note that you have to add more restricted rules for internet connection, accessibility to specific Game directory, ... Bottles has already provided this.

[Sandboxes vs. Containers](https://github.com/netblue30/firejail/wiki/Frequently-Asked-Questions#how-does-it-compare-with-docker-lxc-nspawn-bubblewrap)

2. [containers/bubblewrap](https://github.com/containers/bubblewrap)

3. Built-in Flatpak sandbox + `Flatseal`/_KDE System Settings > Flatpak Permissions_ for permission managing

4. Built-in Lutris sandbox

[Pjotr's "Run Browser in Sandbox"](https://easylinuxtipsproject.blogspot.com/p/sandbox.html)

### Screenshot/Screen Recoder

1. GNOME Screenshot [#ubuntu2204]() [#builtin]()

- Its screen recorder feature is horrible, uncompressed, can be played but can't be fast-forwarded.

2. `gnome-screenshot` [#ubuntu2204]() [#apt]()

- Integrate with some custom keyboard macros.

3. OBS Studio [#ubuntu2204]() [#apt]() [#fedorakde40]() [#flatpak]()

#### 4. Spectacle [#fedorakde40]() [#builtin]()

- Feature-rich.

5. Flameshot Flatpak [#fedorakde40]() [#flatpak]()

- Install `.flatpak` file on GitHub Release (I've trust issue with unverified Flathub repository)
- Multiple monitor bug on Linux Wayland KDE Plasma.

- I have tried to create custom Window Rules according to [flameshot-org/flameshot#2364](https://github.com/flameshot-org/flameshot/issues/2364) but it's not working.

### Shell

1. `sh` and `bash`

[Differences between **.profile**, **.bashrc**, **.bash_profile**](https://superuser.com/a/183980)  
[POSIX-compliant comparison](https://unix.stackexchange.com/a/72042/607715)

- Using here string is much efficient than echo piping because:
  - no subshell creation
  - no pipe overhead
  - direct input to command
  - echo pipe is better for large strings due to streaming
  - for small string, here string is better

```sh
# Here String (<<<)
message="Hello World"
grep "Hello" <<< "$message"

# Echo Piping (|)
message="Hello World"
echo "$message" | grep "Hello"
```

2. `fish` [#ubuntu2204]() [#apt]() [#fedorakde40]() [#dnf]()

- Change it to default shell: `chsh -s /usr/bin/fish`
- Add `~/.local/bin` to `fish_user_paths` universal variable

> [!CAUTION]
> Every newly path was added via this way is available in Fish shell operations ONLY!

- Run `fish_config` to change Fish color scheme.
- Install [mattgreen/lucid.fish](https://github.com/mattgreen/lucid.fish) fish prompt (minimalist + high-performance).
- Add `~/.config/fish/func.d` as 1st element of `$fish_function_path` universal variable.
- Customize shell startup using `~/.config/fish/config.fish`:
  - Add `node` to `$PATH` + upgrate to latest LTS node.
  - Activate Virtual environment if current directory contains Python virtual environment with the name `.venv`.
- Symlinked all custom scripts inside `~/.dotfiles/fish`:
  - Wrap `btrbk` backup one-liner command.
  - Wrap `Exec=` one-liner command from 3rd-party apps `.desktop` files. Usually these commands have added environment variables.
    > NOTE: for Flatpak application without added environment variables, create a symlink from their executable file to `~/.local/bin` is much more appropriate.
  - Customize `lucid.fish` fish prompt.
  - Remove fish greeting
  - Add some utilities
- Plugin Manager [jorgebucaran/fisher](https://github.com/jorgebucaran/fisher):
  - [PatrickF1/fzf.fish](https://github.com/PatrickF1/fzf.fish). Dependency: [junegunn/fzf](https://github.com/junegunn/fzf) [#dnf]().
  - [jorgebucaran/autopair.fish](https://github.com/jorgebucaran/autopair.fish)
  - [jethrokuan/z](https://github.com/jethrokuan/z)
  - [FabioAntunes/fish-nvm](https://github.com/FabioAntunes/fish-nvm)
  - [edc/bass](https://github.com/edc/bass)
  - [sharkdp/fd](https://github.com/sharkdp/fd)
  - [sharkdp/bat](https://github.com/sharkdp/bat)

[Carlos Alexandro Becker's "Why I migrated to the Fish Shell"](https://carlosbecker.com/posts/fish/)

- Better start and exit time when comparing to `zsh`, insignificant less than `sh/bash`
- `~/.config/fish/fish_variables`: Setting aliases or env vars might add overhead to shell start time, with Fish's Universal Variable, this is not an issue anymore. (if you need these aliases/env vars worked outside Fish shell, it's best to set them globally somewhere else).
- `~/.config/fish/fish_variables` and `~/.config/fish/completions`: Lazy loading of completions and functions.
- Default features that are plugins on Bash/ZSH:
  - Syntax Highlighting
  - Autosuggestions
  - Man-page completions
- Don't need `oh-my-fish`, just `fisher` is enough.
- Fish isn't POSIX-compliant, so I refrain from writing custom `.fish` scripts.
- Fish doesn't have _alias_, instead, just wrap your command in a function.
- Different syntax: `$argv[1]` instead of `$1` and `set FISH bar` instead of `FOO="bar"`
- Take _list_ differently from `sh/bash/zsh`

### Social media

1. Facebook [#saas]()

2. [sindresorhus/caprine](https://github.com/sindresorhus/caprine) [#ubuntu2204]() [#appimage]() [#fedorakde40]() [#flatpak]()

**Bugs:**<br />

- Can't sync old messages or change settings.
  => Workaround: [mquevill's comment](https://github.com/sindresorhus/caprine/issues/2167#issuecomment-2089402948)

3. Zalo [#saas]()

- (Ubuntu-only) Can't use web version on Firefox, Chrome or Edge.
- (Fedora-only) Can't use web version on Firefox, but worked on Edge. Chrome is untested.

4. Discord [#ubuntu2204]() [#deb]() [#fedorakde40]() [#flatpak]()

### Streaming service

1. Stremio [#ubuntu2204]() [#deb]() [#fedorakde40]() [#flatpak]()

- Enable `torrent.io` plugins.
- Set it to unlimited caching.

> [!TIP]
> Use with RealDebrid for maximizing experience.

> [!IMPORTANT]
> Both Stremio and RealDebrid don't seed! Consider using Servarr stack.

2. [yt-dlp/yt-dlp]() [#fedorakde40]() [#dnf]()

- Docs: [yt-dlp/yt-dlp-wiki](https://github.com/yt-dlp/yt-dlp-wiki)
- Put inside `~/FOSS` [#foss]()

> [!TIP]
> Stream Youtube directly to media player
> Uses very little resource (memory, CPU and battery consumption)
> NOTE: if you're using flatpak application, you have to specify custom config directory

```
yt-dlp --cookies-from-browser firefox -q -f ba/b -o - "URL" | cvlc -
```

### Terminal

1. Konsole [#fedorakde40]() [#builtin]()

**Bugs:**

- `Shift + Left`, `Shift + Right` swallow text selection.
  => Workaround, change Yakuake shortcuts via pressing `Ctrl + Alt + ,`

2. GNOME Terminal [#ubuntu2204]() [#builtin]()

### Text editor

1. Notepad++ [#windows-only]()

2. gedit [#ubuntu2204]() [#builtin]()

- Add `Right-click > New document` to GNOME Files: `touch ~/Templates/newfile.txt`

> [!TIP]
> To open as root, use either `admin://` or `gksudo` (legacy).

**Cons:**<br />

- Install plugin that enlarge font, but sometimes it's not working.
- Lose focus to notification when being opened via terminal
- Can't press Tab to navigate to next search result button, in modal Replace.

3. `nano` [#ubuntu2204]() [#fedorakde40]() [#builtin]()

- Apply custom Markdown style [#hardlink]()

- For Windows:
  - With normal shell privilege, `.nanorc` is loaded from `%UserProfile%\.nanorc`
  - With Administrator privilege, `.nanorc` is loaded from `%AllUsersProfile%\.nanorc`
  - The systemwide nanorc is loaded from: `%AllUsersProfile%\nanorc`

4. `vi`/`vim` [#ubuntu2204]() [#fedorakde40]() [#builtin]()

- Not my type, it needs a second brain to handle navigation.

### Text hooker

1. [Artikash/Textractor](https://github.com/Artikash/Textractor)

- Will not work for every VN engine (e.g. MAGUS engine for Steins;Gate franchise).
- Works with WINE under same prefix.

2. [Jamal's Texthooker](https://anacreondjt.gitlab.io/texthooker.html)

### Text processor

1. Overleaf [#saas]()

- Long compile time.

### To-do App

1. Todoist

[#android]()

2. Planify [#fedorakde40]() [#flatpak]()

**Bugs:**

- Cursor size is too big inside version **4.11.5**.

### Touchpad

1. `libinput-tools` [#ubuntu2204]() [#apt]()

- Reduce touchpad sensitivity.

[Ubuntu Handbook's "Adjust Touchpad Scrolling"](https://ubuntuhandbook.org/index.php/2023/05/adjust-touchpad-scrolling-ubuntu/)

### Video conference

1. Google Meet [#saas]()

2. Microsoft Teams [#saas]() [#pwa]()

### Video editor

1. Shotcut [#ubuntu2204]() [#fedorakde40]() [#flatpak]()

### Video player

1. VLC [#ubuntu2204]() [#apt]() [#fedorakde40]() [#dnf]()

- Maintain custom `.desktop` file. [#desktop]()

**Cons:**

- APT's version is too old (`v3.0.16`)

**Bugs:**

- (Ubuntu only) Stremio's "Watch on VLC" always turn off Audio Device. Needs to be turn on manually.
- (Fedora only) UI scaling not working.
  => Workaround: add `env QT_AUTO_SCREEN_SCALE_FACTOR=false QT_SCREEN_SCALE_FACTORS=3`

> [!NOTE]
> There is an unofficial Flatpak version.

2. `aplay` [#fedorakde40]() [#builtin]()

- ALSA client tool.
- Some codecs are not supported.

> [!TIP]
> List supported PCMs: `aplay -L`.

3. `pw-play` [#fedorakde40]() [#builtin]()

- PipeWire client tool

### Virtualization

#### 1. Docker

[#ubuntu2204]() [#ppa]() [#fedorakde40]() [#rpm]()

**Tweaks:**

- Add `$USER` to `docker` group in order to run Docker as non-root user.
- Autostart on boot [#systemd]().
- For a separate root and home partition setup, relocate container images by running `rsync -avuP --checksum /var/lib/docker/ ~/`

> [!CAUTION]
> Running rootless Docker can pose many problems. Don't try to do it.
> DO NOT ADD `/etc/docker/daemon.json`, explicitly specify the configuration in command.

- Specify `restart: unless-stopped` inside `docker-compose.yml` will restart service on boot.

[Sandboxes vs. Containers](https://github.com/netblue30/firejail/wiki/Frequently-Asked-Questions#how-does-it-compare-with-docker-lxc-nspawn-bubblewrap)

#### 2. VirtualBox and `virtualbox-guest-additions-iso`

[#ubuntu2204]() [#apt]()

#### 3. Bottles

[#fedorakde40]() [#flatpak]()

- GUI wrapper for WINE.
- UI minimalism, easy to configurate.
- 1 more layer of security: Sandbox per bottle.

**Tweaks:**

- By default, there is no filesystem permission. User must add their directories manually.
- UI scaling is terrible, haven't found a workaround.
  => Change DPI settings. Recommended: `240`dpi (`2560x1440, 125%`).
- Remove Templates and Downloads symlinks to `/home/$USER` via Wine configuration. Remove Z drive as well.

**Caveats:**

- Force close application not working. Have to use `htop` to check PID and `kill -9 [PID]`.
- Working directory and the directory that contains the `.exe` file must be the same, or else nothing will happen when running the executable.

#### 4. Lutris

[#fedorakde40]() [#flatpak]()

- GUI wrapper for WINE.
- Wraps multiple "Runners" - emulators, engines or transition layers - that are capable to install and play games downloaded from various sources.
- Added layer of security: _WINE sandbox_ (similar to Bottles's _Sandbox per bottle_).
- Standard procedure to test running a crack game:
  - Configure Wine params.
  - Spin up a clean Wine prefix.
  - Install any Winetricks required.
  - Copy cracked games and run it.
- Lutris has higher verbosity level than Bottles. Its log is better since running some EXE inside Bottles does not output anything.

**Tweaks:**

[DavidoTek/ProtonUp-Qt](https://github.com/DavidoTek/ProtonUp-Qt) [#fedorakde40]() [#flatpak]()

- Steam

  - GE-Proton9-20 (2024-11-14)

- Lutris Flatpak

  - Lutris-Wine: lutris-7.2.2-x86_64. (2024-11-14)
  - Kron4ek Wine-Builds Vanilla: wine-9.21-amd64-wow64. (2024-11-14)
  - Wine-GE: wine-ge-8-26-x86_64
  - Wine TKG: wine-tkg-valve-exp-bleeding-9.0.135225.2024.1113-327-x86_64
  - vkd3d-lutris: vkd3d-2.13 (2024-11-14)
  - vkd3d-proton: vkd3d-proton-2.13 (2024-11-14)
  - DXVK: dxvk-2.5 (2024-11-14)
  - DXVK Async: dxvk-gplasync-v2.4.1-1 (2024-11-14)

- By default, `--filesystem=home` is provided, restrict this access. Best practice is to allow
- Restrict Internet access globally via **Flatseal** or _KDE System Settings > Flatpak Permissions_.

> **NOTE:** Remember to unrestrict when download Runners (Wine releases, ...) and DLL components (DXVK, VKD3D, ...)

- Added 1 more layer of security: _Wine sandbox_.
- Change screen DPI settings to `240` for `2560x1440` monitor, `125%` scale.
- Remove Templates and Downloads symlinks to `/home/$USER` via Wine configuration. Remove Z drive as well.
- Generate ja_JP.UTF-8 locale by uncomment the line `ja_JP.UTF-8 UTF-8` in `/etc/locale.gen` and run `sudo locale-gen` (I assume Bash console is needed)
- Set correct environment variables (Set locale setting inside Lutris is not needed?):

```txt
LC_ALL      ja_JP.UTF-8
TZ          Asia/Tokyo
```

- Wine prefix must be blank if ProtonGE is being used on prefix for the first time (?)
- Install Textractor and setup Textractor to launch with VN simultaneously by following [TheMoeWay's "Set Up Textractor to launch with VN"](https://learnjapanese.moe/vn-linux/#set-up-textractor-to-launch-with-vn)
- Use `file` command to identify which architecture the VN is.

```txt
VN.exe: PE32 executable (GUI) Intel 80386, for MS Windows (32-bit)
VN.exe: PE32+ executable (GUI) x86-64, for MS Windows (64-bit)
```

- Add Windows Japanese fonts ([provided by TheMoeWay](https://drive.google.com/file/d/1OiBgAmt3vPRu08gPpxFfzrtDgarBGszK/view)) to Wine prefix's fonts folder `drive_c/windows/Fonts` (without editing Registry?)

- Disable External drive access by change `/media` and `/run/media` filesystem permissions to OFF.

**Q&A:**

- Q: _GE-Proton (Latest)_ is missing.  
  A: Disrupting ProtonUp-QT Wine TKG downloading and extracting process can make GE-Proton invisible to Lutris Flatpak.

- Q: _GE-Proton (Latest)_ prefix creation always failed.  
  A: Use different Wine release to create prefix.

- Q: _GE-Proton (Latest)_ requires internet connection to install `umu-launcher`.  
  A: Cre: [umu-launcher#69](https://github.com/Open-Wine-Components/umu-launcher/issues/69). Download the offline version using [ProtonUp-Qt](https://github.com/DavidoTek/ProtonUp-Qt) then add Lutris Flatpak permission to read/write `~/.local/share/Steam/compatibilitytools.d/GE-Proton...` (native Steam directory) so it's visible to Lutris Flatpak.

- Q: Running `umu-launcher` shows error: `The following file was not found in PROTONPATH: proton`  
  A: Use [ProtonUp-Qt](https://github.com/DavidoTek/ProtonUp-Qt) to install latest version of ProtonGE, then add `PROTONPATH` environment variable to Lutris according to [umu-launcher's "How do I use it?"](https://github.com/Open-Wine-Components/umu-launcher?tab=readme-ov-file#how-do-i-use-it). Also add Lutris Flatpak permission to allow read/write the directory contains **ProtonGE9-20**.

- Q: **GE-Proton (Latest)** and **ProtonGE** from ProtonUp-Qt are having the same issues  
   `pressure-vessel-wrap[70]: W: /dev/shm not shared between app instances (flatpak#4214). The Steam Overlay will not work.
steam-runtime-launch-client[70]: E: Unable to convert /app fd 16 into path: different file inside and outside sandbox`  
   A: Unsolved. Either Lutris Flatpak sandbox ([flatpak/flatpak#1756](https://github.com/flatpak/flatpak/issues/1756), [flatpak/flatpak#4124](https://github.com/flatpak/flatpak/pull/4214)) causes this, or Lutris Flatpak v0.5.17 is bugged.

[YoteZip/LinuxCrackingBible](https://github.com/YoteZip/LinuxCrackingBible)  
[Arch Wiki "Security"](https://wiki.archlinux.org/title/Security)  
[WineHQ's "FAQ: Is Wine malware-compatible?"](https://gitlab.winehq.org/wine/wine/-/wikis/FAQ#is-wine-malware-compatible)

#### 5. WINE

- WINE applications do not have read permission for X11/Wayland screen.

### VPN

1. `warp-cli` [#ubuntu2204]() [#apt]() [#fedorakde40]() [#dnf]()

- Add DNF repository: `sudo dnf config-manager --add-repo https://pkg.cloudflareclient.com/cloudflare-warp-ascii.repo`
- Allowing WARP to resolve DNS requests by creating `/etc/systemd/resolved.conf` and write

```conf
[Resolve]
ResolveUnicastSingleLabel=yes
```

- Set up WARP+ key.
  **Bugs:**
- DNS over WARP not working.

[Cloudflare Docs's "WARP modes"](https://developers.cloudflare.com/cloudflare-one/connections/connect-devices/warp/configure-warp/warp-modes/)

2. [ViRb3/wgcf](https://github.com/ViRb3/wgcf) [#fedorakde40]() [#prebuilt-binary]()

- Save to `~/FOSS` [#foss]()
- Set up WARP+ key.
- Generate WireGuard profile [#hardlink]()

3. WireGuard [#fedorakde40]() [#builtin]()

- CloudFlare WARP's Split Tunnelling requires free-tier with payment method input.
  => Split Tunnelling via WireGuard: create a WireGuard network interface, tunnel it to VPN provider (CloudFlare WARP) and route every traffic that has destination IPs resolved from banned websites through that interface.

[WireGuard's "Conceptual Overview"](https://www.wireguard.com/#conceptual-overview)

- Abstract: securely encapsulates IP packets over UDP. How? Add a WireGuard interface, configure it with private key and peers' public keys, then send packets across it. Mimic the model of SSH and Mosh.
- Add a network interface called `wg0`, it acts as a tunnel.
- Use `ifconfig` to configure `wg0`
- Use `ip route` to add/remove routes for `wg0`
- WireGuard aspects of `wg0` are configured using `wg`
- Associates tunnel IP + public keys + remote endpoints
  - Interface _sends_ a packet => peer.
  - Interface _receives_ a packet from peer.
- Cryptokey Routing:
  - Associates public keys with a list of tunnel IP addresses that are allowed inside the tunnel.
  - 1 network interface, 1 private key, N peers.
  - 1 peer, 1 public key, 1 allowed ip list.
  - Depending on the setup (client/server) peer setup can be different.
- Server configuration:
  - Each peer (a.k.a client) can send packets to the network interface with their source IP matching the corresponding list of allowed IPs.
  - If server's network interface wants to send a packet to a peer, it looks at the destination IP and compare it each peer's list of allowed IPs to know which peer to send it to.
  - There can be ONE server's source IP and it's specified in `Endpoint=` entry at client's `[Peer]` setting.
- Client configuration:
  - Single peer (a.k.a server) can send packets to the client's network interface using any source IP (to ensure client will ALWAYS GET packet from server). Some VPN providers like Cloudflare wouldn't accept allowed ip list other than 0.0.0.0/0
  - If client's network interface wants to send a packet to its single peer (a.k.a server), it will encrypt packet for the single peer with any destination IP address.
  - There can be ONE client's source IP.
  - Why only client have `Endpoint=` entry and server doesn't have it? Client needs to knows where to send encrypted data before it has received encrypted data. Client is the initiator. But server discovers the endpoint of its peers by examining the packets sent from client. The server can change its endpoint and send to its peers and they will know and update the configuration. The same could happen with client.

=> When sending packets, the list of allowed IPs behave as a _routing table_
=> When receiving packets, the list of allowed IPs behave as a _access control list_

4. Tailscale [#fedorakde42]()

- Follow [Install Tailscale on Fedora version 41 and later](https://tailscale.com/kb/1511/install-fedora-2) instructions to install Tailscale on my machine.

## Miscellaneous

1. Ubuntu 22.04

- APT addition packages: `build-essential`, `apt-transport-https`, `ubuntu-restricted-extras`, `dconf-editor`, `gir1.2-gtop2.0`, `dbux-x11`, `7z`, `zip`, `jq`, `tree`, `tmux`.
  - For `7z`, be sure to not write a whitespace between option and value, as well as `~`. E.g. `-o$HOME/Donwloads/foobar` not `-o ~/Donwloads/foobar`
- Modify Favorites app via Terminal:

```sh
gsettings [get|set] org.gnome.shell favorite-apps
dconf [read|write] /org/gnome/shell/favorite-apps "['firefox.desktop', 'org.gnome.Nautilus.desktop', 'my-app.desktop'], ..."
```

- GNOME tracker DB error: `tracker3 reset -s`

2. Fedora KDE 40

- Installed `Development Tools`
- Set sudo password timeout to 60 minutes inside `/etc/sudoers.d/`
- Inside Window Rules UI, you can extract Window property by clicking `Detect Window Properties`.

3. All OS

- Print OS information in Linux:

```sh
cat /etc/os-release
lsb_release -a
hostnamectl
uname -r # kernal version
cat /etc/fedora-release # Fedora-family distro only
```

- All applications' exe files that are installed inside `/opt` are symlinked to `/usr/local/bin`. [#opt]()
- All applications' exe files that are installed inside `~/FOSS` are symlinked to `~/.local/bin`. [#foss]()
- Hardlinks are preferred when linking on the same partitions. Symlinked or Synclinked is used when linking cross-partitions.
- Permission `755` is good enough for script files inside single-user system.

- Determine an application use `Qt` or `GTK`: `ldd [APPLICATION_EXECUTABLE] | grep -i 'qt\|gtk'`.
- Verify config files: `testparm`
- Look for CLI name of a GUI app, examine `Exec=` line in its `.desktop` file.
- Print full systemctl log: `systemctl status --no-pager --full [SERVICE_NAME]`.
- Old config backup at `~/.config.bak`
- Check system-level DNS server: `resolvectl`.

- `locate` is a very powerful tool. It's very helpful in searching for undocumented files.
- `ps aux | grep [PID]` to know which applications associate with a PID.
- `pgrep` to know PID of applications via their name.
- `journalctl` to debug `systemd` service, events and actions triggered by timers.
  - `-u`: filter unit
  - `-b`: logs from current boot only
  - `-S today`: logs from today only
  - `-x`: help along log entry
  - `-f`: monitor the log
- `nice`: control application resource priority or [Ananicy](https://www.maketecheasier.com/control-apps-priorities-with-ananicy-linux/)
- `pv [FILE] | sha256mac` to print hashing progress, or install [Xfennec/progress](https://github.com/Xfennec/progress).
- `chroot` "jail" is a way to isolate a process and its children from the rest of the system, idea is to create a directory where user copy or symlink in all of system files needed for a process to run, then use chroot to change the root directory to be at the base of this new tree.

[A very good blog about system administation](https://blogd.net)
