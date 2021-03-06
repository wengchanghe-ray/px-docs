---
layout: page
title: "Create and Manage Snapshots 1.3"
keywords: portworx, pxctl, snapshot, reference
sidebar: home_sidebar
redirect_from: "/snapshot.html"
meta-description: "Learn how container volume snapshots can be created explicitly by pxctl snapshot create commands or through a schedule that is set on the volume. Try today!"
---

* TOC
{:toc}

Snapshots are efficient point-in-time read-only copies of volumes.
Snapshots created can be used to read data from the snapshot, restore data from the snapshot and also create clones from the snapshot.
They are implemented using a copy-on-write technique, so that they only use space in places where they differ from their parent volume.
Snapshots can be created explicitly by `pxctl snapshot create` commands or through a schedule that is set on the volume.

## `pxctl` Snapshot Commands

Snapshots are managed with the `pxctl volume snapshot` command.

{% include pxctl/volume/volume-snap-help-1.3.md %}

### Creation Snapshots

To create a user snaphsot for a volume , Use `pxctl snapshot create` command.
```
# pxctl volume snapshot create --name mysnap --label color=blue,fabric=wool myvol
Volume snap successful: 234835613696329810
```

The string of digits in the output is the volume ID of the new snapshot.  You can use this ID(`234835613696329810`) or the name(`mysnap`), to refer
to the snapshot in subsequent `pxctl` commands.  The label values allow you to tag the snapshot with descriptive information of your choosing.
You can use them to filter the output of the `pxctl volume list` command.

There is an implementation limit of 64 snapshots per volume.

If a volume replica is on a pool which has a label "no_snapshot" with any value, snapshot will not be created for that replica
(version 1.7 and later)

### Listing Snapshots

{% include pxctl/volume/volume-list-help-1.3.md %}

{% include pxctl/volume/volume-snap-list-1.3.md %}

### Deleting Snapshots

{% include pxctl/volume/volume-snap-delete-help-1.3.md %}

{% include pxctl/volume/volume-snap-delete-1.3.md %}


### Schedule Policy

To create a snapshot policy, use `pxctl sched-policy create` command.
```
NAME:
   pxctl sched-policy create - Create a schedule policy

USAGE:
   pxctl sched-policy create [command options] policy-name

OPTIONS:
   --periodic mins,k, -p mins,k                  periodic snapshot interval in mins,k (keeps 5 by default), 0 disables all schedule snapshots
   --daily hh:mm,k, -d hh:mm,k                   daily snapshot at specified hh:mm,k (keeps 7 by default)
   --weekly weekday@hh:mm,k, -w weekday@hh:mm,k  weekly snapshot at specified weekday@hh:mm,k (keeps 5 by default)
   --monthly day@hh:mm,k, -m day@hh:mm,k         monthly snapshot at specified day@hh:mm,k (keeps 12 by default)
```

The below example creates a policy `p1` with periodic and weekly schedules.
```
# pxctl sched-policy create --periodic 60,5 --weekly sunday@12:00,4 p1
```

Schedule policies can be addded to the volume either during  volume create or after volume create.
```
# pxctl volume create --policy p1 vol1
(or)
# pxctl volume snap-interval-update --policy p1 vol1
```

### Listing Schedule Policies

To list the schedule policies, Use `pxctl sched-policy  list` command
```
NAME:
   pxctl sched-policy list - List all schedule policies

USAGE:
      pxctl sched-policy list [arguments...]
```

### Update Schedule Policy

To update the schedule policy, Use `pxctl sched-policy  update` command
```
NAME:
  pxctl sched-policy update - Update a schedule policy

USAGE:
   pxctl sched-policy update [command options] policy-name

OPTIONS:
   --periodic mins,k, -p mins,k                  periodic snapshot interval in mins,k (keeps 5 by default), 0 disables all schedule snapshots
   --daily hh:mm,k, -d hh:mm,k                   daily snapshot at specified hh:mm,k (keeps 7 by default)
   --weekly weekday@hh:mm,k, -w weekday@hh:mm,k  weekly snapshot at specified weekday@hh:mm,k (keeps 5 by default)
   --monthly day@hh:mm,k, -m day@hh:mm,k         monthly snapshot at specified day@hh:mm,k (keeps 12 by default)
```

### Delete Schedule Policy

To delete the schedule policy, Use `pxctl sched-policy delete` command.
```
NAME:
   pxctl sched-policy delete - Delete a schedule policy

USAGE:
   pxctl sched-policy delete policy-name
```

### Snapshot Schedules

Creation of snapshot schedules during volume create uses four scheduling options [--periodic, --daily, --weekly and --monthly], which you can combine as desired.
Scheduled snapshots have names of the form `<Parent-Name>_<freq>_<creation_time>`, where `<freq>` denotes the schedule frequency, i.e., periodic, daily, weekly, monthly.
For example,
```
myvol_periodic_2018_Feb_26_21_12
myvol_daily_2018_Feb_26_12_00
```

The example below sets a schedule of periodic snapshot for every 60 min and daily snapshot at 8:00am and weekly snapshot on friday at 23:30pm and monthly snapshot on the 1st of the month at 6:00am.
```
pxctl volume create --periodic 60 --daily @08:00 --weekly Friday@23:30 --monthly 1@06:00 myvol
```

The example below keeps a count of 10 periodic snapshot that triggers every 120 min and 3 daily snapshots that tirggers at 8:00am
```
pxctl volume create --periodic 120,10 --daily @08:00,3 myvol
```
Once the count is reached, the oldest existing one will be deleted if necessary.


### Changing Snapshot Schedule

To change the snapshot schedule for a given volume, use `pxctl volume snap-interval-update` command
```
NAME:
   pxctl volume snap-interval-update - Update volume configuration

USAGE:
   pxctl volume snap-interval-update [command options] volume-name-or-ID

OPTIONS:
   --periodic mins,k, -p mins,k                  periodic snapshot interval in mins,k (keeps 5 by default), 0 disables all schedule snapshots
   --daily hh:mm,k, -d hh:mm,k                   daily snapshot at specified hh:mm,k (keeps 7 by default)
   --weekly weekday@hh:mm,k, -w weekday@hh:mm,k  weekly snapshot at specified weekday@hh:mm,k (keeps 5 by default)
   --monthly day@hh:mm,k, -m day@hh:mm,k         monthly snapshot at specified day@hh:mm,k (keeps 12 by default)
   --policy value, --sp value                    policy names separated by comma
```

In the below example, the old snapshot schedule is replaced with 5 daily snapshot triggering at 15:00pm
```
# pxctl volume snap-interval-update --daily @15:00,5 myvol
```

### Disabling Scheduled Snapshots

To disable scheduled snapshot for a given volume, use `--periodic 0` on snap-interval-update.
```
pxctl volume snap-interval-update --periodic 0 myvol
```

### View Snapshot Schedule on Volume

If a schedule is set on a volume and to view that schedule use `pxctl volume inspect` command.
```
# pxctl volume inspect myvol
Volume	:  1125771388930868153
	Name            	 :  myvol
	Size            	 :  1.0 GiB
	Format          	 :  ext4
	HA              	 :  1
	IO Priority     	 :  LOW
	Creation time   	 :  Feb 26 18:06:31 UTC 2018
	Snapshot        	 :  daily @15:00,keep last 5
	Shared          	 :  no
	Status          	 :  up
	State           	 :  Attached: minion1
	Device Path     	 :  /dev/pxd/pxd1125771388930868153
	Reads           	 :  54
	Reads MS        	 :  152
	Bytes Read      	 :  1105920
	Writes          	 :  53
	Writes MS       	 :  841
	Bytes Written   	 :  16891904
	IOs in progress 	 :  0
	Bytes used      	 :  48 MiB
	Replica sets on nodes:
		Set  0
			Node 		 :  70.0.34.84 (Pool 0)
	Replication Status	 :  Up

```
