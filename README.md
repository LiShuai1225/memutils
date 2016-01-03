# memutils

This repository contains tools to assist investigation of memory usage in linux.

## memtop

__NOTE:__ This tool elevates by sudo'ing by default, so it can access the
`/proc/$pid/smap` file for processes owned by all users.

__NOTE:__ This tool assumes relatively recent bash and standard linux utility
functions like `awk` and `paste`. If it doesn't work, check that all the tools
used are up to date.

This tool outputs the memory usage, calculated using
[Proportional Set Size (PSS)][pss] for processes running on the system.

PSS ends up being a better measure of _actual_ memory usage than say resident
pages as it takes into account shared memory like dynamically linked libraries,
etc. but avoids double-counting by dividing the shared pages usage by the number
of processes using those shared pages. This works because if you were to sum
each process's PSS you'd end up with the memory used by all shared and unshared
pages, i.e. total memory usage. Additionally, it's fair distributes shared
memory usage amongst processes that use it - each is treated as if it uses an
even proportion.

### Usage

Simple run `memtop` to output a list of `PID, usage (KiB), command line`, e.g.:

```
[~]$ memtop
  1638      112 /opt/google/chrome/chrome-sandbox/opt/google/chrome/nacl_helper
...
 27343   112660 /opt/google/chrome/chrome --type=renderer --enable-display-list-2d-canvas --enab
 17092   127260 /usr/share/spotify/spotify
  1668   151431 /proc/self/exe--type=gpu-process--channel=1624.0.1077674023--disable-breakpad--s
 17131   181799 /usr/share/spotify/spotify --type=renderer --no-sandbox --lang=en-US --lang=en-U
 24668   225407 /opt/google/chrome/chrome --type=renderer --enable-display-list-2d-canvas --enab
  1624   250121 /opt/google/chrome/chrome
  1086   497735 /usr/bin/python2/usr/bin/offlineimap
  8917   768734 mutt
```

The output is always sorted by memory usage ascending. You can output
human-readable sizes using the `-h` flag:

```
[~]$ memtop -h
  1638   112KiB /opt/google/chrome/chrome-sandbox/opt/google/chrome/nacl_helper
...
 27343   111MiB /opt/google/chrome/chrome --type=renderer --enable-display-list-2d-canvas --enab
 17092   125MiB /usr/share/spotify/spotify
  1668   148MiB /proc/self/exe--type=gpu-process--channel=1624.0.1077674023--disable-breakpad--s
 17131   178MiB /usr/share/spotify/spotify --type=renderer --no-sandbox --lang=en-US --lang=en-U
 24668   221MiB /opt/google/chrome/chrome --type=renderer --enable-display-list-2d-canvas --enab
  1624   245MiB /opt/google/chrome/chrome
  8917   751MiB mutt
```

Note that output of the command lines is truncated, this is for readability,
however you can turn it off with the `-l` flag, e.g.:

```
[~]$ memtop -hl
  1638   112KiB /opt/google/chrome/chrome-sandbox/opt/google/chrome/nacl_helper
...
 17131   178MiB /usr/share/spotify/spotify --type=renderer --no-sandbox --lang=en-US --lang=en-US --log-severity=disable --product-version=Spotify/1.0.19.106 --disable-spell-checking --enable-delegated-renderer --num-raster-threads=4 --gpu-rasterization-msaa-sample-count=8 --content-image-texture-target=3553 --video-image-texture-target=3553 --disable-accelerated-video-decode --channel=17092.1.230707511 --v8-natives-passed-by-fd --v8-snapshot-passed-by-fd
 24668   221MiB /opt/google/chrome/chrome --type=renderer --enable-display-list-2d-canvas --enable-experimental-canvas-features --lang=en-GB --force-fieldtrials=*AffiliationBasedMatching/Enabled/AppBannerTriggering/Aggressive/CaptivePortalInterstitial/Enabled/*ChildAccountDetection/Disabled/*ChromeSuggestions/Default/*ClientSideDetectionModel/Model0/*CrossDevicePromo/28DaySingleProfile/*DomRel-Enable/enable/*EnhancedBookmarks/Default/*ExtensionContentVerification/Enforce/*ExtensionDeveloperModeWarning/Default/InstanceID/Enabled/*OmniboxBundledExperimentV1/Unused_2/PasswordBranding/Disabled/*PasswordGeneration/Disabled/ReportCertificateErrors/ShowAndPossiblySend/*ResourcePriorities/Disabled/SHA1IdentityUIWarning/Enabled/SHA1ToolbarUIJanuary2016/Warning/SHA1ToolbarUIJanuary2017/Error/*SafeBrowsingIncidentReportingService/Default/SafeBrowsingReportPhishingErrorLink/Enabled/SafeBrowsingSocialEngineeringStrings/Enabled/SafeBrowsingUnverifiedDownloads/DisableByParameterExe/*SafeBrowsingUpdateFrequency/Default/SessionRestoreBackgroundLoading/Restore/SlimmingPaint/EnableSlimmingPaint/*UMA-Population-Restrict/normal/*UMA-Uniformity-Trial-100-Percent/group_01/*UMA-Uniformity-Trial-20-Percent/default/*UMA-Uniformity-Trial-50-Percent/default/*VarationsServiceControl/Interval_30min/WebRTC-PeerConnectionDTLS1.2/Enabled/ --extension-process --enable-webrtc-hw-h264-encoding --enable-offline-auto-reload --enable-offline-auto-reload-visible-only --num-raster-threads=4 --content-image-texture-target=3553,3553,3553,3553,3553,3553,3553,3553,3553,3553,3553,3553,3
  1624   245MiB /opt/google/chrome/chrome
  8917   751MiB mutt
```

If you run chrome you probably don't want to use this flag :)

Alternatively, you can simply use the 'command name' of each process (limited to
16 characters but often clearer, taken from `/proc/$pid/comm`) using the `-c`
flag:

```
[~]$ memtop -hc
  1638   112KiB chrome-sandbox
...
 27343   112MiB chrome
 17092   125MiB spotify
  1668   148MiB chrome
 17131   178MiB spotify
 24668   221MiB chrome
  1624   245MiB chrome
  8917   751MiB mutt
```

Finally, there's what I feel what is probably the most useful format - you can
_group by_ command name and sum accordingly giving an overall total for each
process with the same command name:

```
[~]$ memtop -gh
  170KiB agetty
  226KiB chrome-sandbox
  275KiB cat
  458KiB rtkit-daemon
  485KiB i3status
  565KiB dhcpcd
  586KiB systemd-timesyn
  637KiB udevil
  646KiB crond
  695KiB ssh-agent
  884KiB measure-net-spe
  902KiB bash
  903KiB devmon
  907KiB systemd-logind
  962KiB gconfd-2
  970KiB memtop
  1.1MiB at-spi-bus-laun
  1.2MiB nacl_helper
  1.3MiB at-spi2-registr
  1.4MiB gvfsd-fuse
  1.5MiB (sd-pam)
  1.6MiB gvfsd
  1.6MiB sshd
  1.8MiB sudo
  2.0MiB systemd-udevd
  2.2MiB accounts-daemon
  2.2MiB dbus-daemon
  2.6MiB upowerd
  3.4MiB systemd
  3.8MiB lightdm
  4.2MiB aspell
  6.5MiB i3bar
  7.2MiB i3
   14MiB polkitd
   14MiB pulseaudio
   19MiB zsh
   27MiB urxvt
   30MiB pavucontrol
   36MiB docker
   43MiB systemd-journal
   54MiB emacs
   55MiB Xorg
  108MiB vlc
  418MiB spotify
  486MiB offlineimap
  751MiB mutt
  1.5GiB chrome
```

[pss]:https://lwn.net/Articles/230975/
