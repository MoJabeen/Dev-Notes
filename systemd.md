---
author:
  - Mo D Jabeen
date: 2023-10-01
title: Systemd
---

Systemd is an improved version of service, controlled via systemctl. Allows you to initalise components after the linux kernel is booted. Also managed services and daemons. Systemd used units controlled via a unit file.

# General Service Control

```bash
systemctl ...

start app.service //.service not needed
stop
restart
reload
reloadorrestart

enable //start service at boot
disable

status
isactive
isenabled
isfailed

list-units //list all units

cat //show unit file
mask //stop unit from starting
edit //append to unit file

edit --full //edit unit file
```

Targets are used to group units together, used to ref a system state instead of a unit.

## Journald

Journald combines all outputs from services.

```bash
journalctl -u unit // outputs from chosen unit

-n 20 // see last 20 logs
-f // follow a log
```

# Unit files

## \[Unit\]

```bash
Description = //best to keep it short
Documentation = //man page or url
Requires = //Any units required
Wants = //Less strict than requires
BindsTo = //if unit stops this will to
Before = // Dont start these units until the associated unit has started
After = // start associated after these units
Conflicts = // dont run these units at the same time
Condition = // Only run if true, if failure skip
Assert = //if failure cause error
```

## \[Install\]

Controls enabled units

```bash
Wantedby = //similar to Wants
Requiredby = //similar to requires

Alias = 
Also = //Create a set of units to enable with this one
```

## \[Service\]

```bash
Type = // simple,notify etc
ExecStart = //commands for when started
ExecStartPost = //commands post start 
ExecReload = //commands on reload
ExecStop = // commands on stop
ExecStopPost = //commands post stop
RestartSec = // commands on restart
Restart = // commands on restart
TimeoutSec = // commands on timeout
```

## \[Path\]

```bash
PathChange = //path to monitor for changes
Unit = //units to activate on changes
MakeDirectory = //if path should be created
```

## \[Timer\]

```bash
OnActiveSec = //start after .timer service
OnBootSec = //start after boot
OnUnitActive = //time after unit activation
OnCalendar =  // use absolute time instead
AccuracySec = // accuracy level
```
### subsubsection
