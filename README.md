
Welcome to the Fedora CoreOS issue tracker. This tracker will be used
to discuss new features for Fedora CoreOS and also important bugs that
are affecting the project. Tickets with the `meeting` label will be
taken as agenda items during the meetings. This repo is to be used primarily
for development purposes. If you are a user and have questions please use
the [forum or the mailing list](#communication-channels-for-fedora-coreos).

# Fedora CoreOS Working Group

Fedora CoreOS is an automatically updating, minimal, monolithic,
container-focused operating system, designed for clusters but also
operable standalone, optimized for Kubernetes but also great without
it. It aims to combine the best of both CoreOS Container Linux and
Fedora Atomic Host, integrating technology like Ignition from Container
Linux with rpm-ostree and SELinux hardening from Project Atomic. Its
goal is to provide the best container host to run containerized workloads
securely and at scale.

The Fedora CoreOS Working Group works to bring together the various
technologies and produce Fedora CoreOS.

# Get Fedora CoreOS

Future Link to download page

# Communication channels for Fedora CoreOS

- main mailing list: [coreos@lists.fedoraproject.org](https://lists.fedoraproject.org/archives/list/coreos@lists.fedoraproject.org/)
- status mailing list: [coreos-status@lists.fedoraproject.org](https://lists.fedoraproject.org/archives/list/coreos-status@lists.fedoraproject.org/) (announcements/important messages)
- `#fedora-coreos` on IRC (Freenode)
- forum at [https://discussion.fedoraproject.org/c/server/coreos](https://discussion.fedoraproject.org/c/server/coreos)
- feature planning and important issue tracking at [github.com/coreos/fedora-coreos-tracker](https://github.com/coreos/fedora-coreos-tracker)
- website at [https://coreos.fedoraproject.org/](https://coreos.fedoraproject.org/)
- Twitter: [@coreos](https://twitter.com/coreos) (specific to CoreOS and containers) and [@fedora](https://twitter.com/fedora) (all Fedora and other relevant news)

# Roadmap/Plans

The first release of Fedora CoreOS will be a 
[preview release](https://github.com/coreos/fedora-coreos-tracker/issues/145)
followed by a subsequent stable release. You can see a link to
our project boards in our [ROADMAP.md](./ROADMAP.md).

# Meetings

The Fedora CoreOS Working Group has a weekly meeting. The meeting usually
happens in `#fedora-meeting-1` on irc.freenode.net and the schedule for the
meeting can be found here: https://apps.fedoraproject.org/calendar/CoreOS
Currently, meetings are at `16:30 UTC` on Wednesdays.

## Steps to run the meeting

- Navigate to `#fedora-meeting-1` on freenode
- Type `#startmeeting fedora_coreos_meeting`
- `#topic roll call`

Wait for 2-4 minutes for people to check in for the roll call.

- `#chair` all the people present for the meeting
- `#topic Action items from last meeting`

Find the last meeting log from
[meetbot](https://meetbot-raw.fedoraproject.org/teams/fedora_coreos_meeting)
and post the action items in the meeting for people to
update the status of.

- After they are done move to each `meeting` ticket from
  [this tracker](https://github.com/coreos/fedora-coreos-tracker/labels/meeting)

Do the following for each ticket

- `#topic` Ticket subject
- `#link` link\_to\_the\_ticket

During the meeting, you can give people action items for them to complete:

- `#action <nickname>` description of what needs to be done

When all topics are over, go for open floor:

- `#topic Open Floor`

After open floor, end the meeting.

- `#endmeeting`

When convenient, send an email to `coreos@lists.fedoraproject.org` with the
details of the meeting from [meetbot page](https://meetbot.fedoraproject.org/sresults/?group_id=fedora_coreos_meeting&type=team).
Minutes in textual format are directly available using `.txt` as URL extension.
It's easiest to get the Minutes/Minutes (text)/Log URLs by copying the
footer that Meetbot prints after `#endmeeting`.

The usual format follows:

```
Subject:  Fedora CoreOS Meeting Minutes year-mm-dd

Body:

Minutes: <URL to meetbot .html>
Minutes (text): <URL to meetbot .txt>
Log:  <URL to meetbot .log.html>

<Copy/paste content of meetbot .txt>
```

You can see examples in the [archives](https://lists.fedoraproject.org/archives/list/coreos@lists.fedoraproject.org/)

# Voting

On some topics we will need to vote. The following rules apply to the voting
process.

## For Regularly Scheduled Meetings

A quorum for the meeting is 5 people, or 51% of the members of the WG listed
below, which ever is lower. Voting items must pass with a majority of the
members voting at the meeting.

## For General Ad-Hoc Votes

- All ad-hoc votes will be held via [tracker issues](https://github.com/coreos/fedora-coreos-tracker/).
- Ad-hoc votes must be announced on the current primary mailing list for Fedora Atomic (atomic-devel).
- Ad-hoc votes must be open for at least three working days (see below) after the announcement.

At least 5 people must vote, or 51% of the WG membership, whichever is
less. Votes are "+1" (in favor), "-1" (against), or +0 (abstain). Votes
pass by a simple majority of those voting.

## For Urgent Ad-Hoc Votes

- All ad-hoc votes will be held via tracker issues in the fedora-coreos-tracker repo.
- Ad-hoc votes must be announced on the current primary mailing list for Fedora CoreOS.
- Ad-Hoc votes must be open for at least three hours after the announcement.

At least 5 people must vote, or 51% of the WG membership, whichever is less. Votes are "+1" (in favor), "-1" (against), or +0 (abstain). Votes pass by a 2/3 majority of those voting (round up).

Working days: non-holiday weekdays. Relevant holidays are the national holidays of the USA, Western Europe, and India.

# Working Group Members and Points of Contact

TBD
