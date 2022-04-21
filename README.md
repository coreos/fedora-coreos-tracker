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

[Download Fedora CoreOS.](https://getfedora.org/coreos/download/)

# Communication channels for Fedora CoreOS

- main mailing list: [coreos@lists.fedoraproject.org](https://lists.fedoraproject.org/archives/list/coreos@lists.fedoraproject.org/)
- status mailing list: [coreos-status@lists.fedoraproject.org](https://lists.fedoraproject.org/archives/list/coreos-status@lists.fedoraproject.org/) (announcements/important messages)
- IRC/Matrix: [`#fedora-coreos` on Libera.Chat](https://web.libera.chat/#fedora-coreos) (ircs://irc.libera.chat:6697/#fedora-coreos) or [`#coreos:fedoraproject.org` on Matrix](https://matrix.to/#/#coreos:fedoraproject.org)
- forum at [https://discussion.fedoraproject.org/tag/coreos](https://discussion.fedoraproject.org/tag/coreos)
- feature planning and important issue tracking at [github.com/coreos/fedora-coreos-tracker](https://github.com/coreos/fedora-coreos-tracker)
- website at [https://getfedora.org/coreos/](https://getfedora.org/coreos/)
- documentation at [https://docs.fedoraproject.org/en-US/fedora-coreos/](https://docs.fedoraproject.org/en-US/fedora-coreos/)
- Twitter: [@fedoracoreos](https://twitter.com/fedoracoreos)

# Roadmap/Plans

Fedora CoreOS is available for general use and no longer in preview.  We're
continuing to add more platforms and functionality, fix bugs, and write
documentation.  Please try out Fedora CoreOS and give us feedback!

# Adding Packages to Fedora CoreOS

We often find people asking for a particular package to be added to the base set of
packages included in Fedora CoreOS. One of the goals of Fedora CoreOS is to
remain as lean as possible, without impacting overall usability for our users.
Thus, new package requests are carefully scrutinized to weigh the benefits and
drawbacks of adding an additional package.

If you would like to propose the inclusion of a new package in the base set of packages,
please file a [new package request](https://github.com/coreos/fedora-coreos-tracker/issues/new?labels=kind/enhancement&template=new-package.md&title=New+Package+Request%3A+%3Cpackage+name%3E).

# Releases

See [RELEASES.md](RELEASES.md).

# Meetings

The Fedora CoreOS Working Group has a weekly meeting. The meeting usually
happens in `#fedora-meeting-1` on irc.libera.chat and the schedule for the
meeting can be found here: https://calendar.fedoraproject.org/CoreOS/
Currently, meetings are at `16:30 UTC` on Wednesdays.

## Steps to run the meeting

- `cd` to a local checkout of this repo and `git pull`
- Ping [meeting people](https://github.com/coreos/fedora-coreos-tracker/blob/main/meeting-people.txt) in `#fedora-coreos` on libera.chat
    - `bash meeting-people.txt`
    - copy lines of output and paste into `#fedora-coreos` channel
- Navigate to `#fedora-meeting-1` on libera.chat
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

Then, when convenient:

- Remove `meeting` labels from [tickets that were discussed](https://github.com/coreos/fedora-coreos-tracker/labels/meeting)

- Send an email to [coreos@lists.fedoraproject.org](mailto:coreos@lists.fedoraproject.org) with the
details of the meeting from [meetbot page](https://meetbot.fedoraproject.org/sresults/?group_id=fedora_coreos_meeting&type=team).
Minutes in textual format are directly available using `.txt` as URL extension.
It's easiest to get the Minutes/Minutes (text)/Log URLs by copying the
footer that Meetbot prints after `#endmeeting`. You can see examples in the
[archives](https://lists.fedoraproject.org/archives/list/coreos@lists.fedoraproject.org/);
the usual format follows:

```
Subject:  Fedora CoreOS Meeting Minutes year-mm-dd

Body:

Minutes: <URL to meetbot .html>
Minutes (text): <URL to meetbot .txt>
Log:  <URL to meetbot .log.html>

<Copy/paste content of meetbot .txt>
```

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

Please see [meeting-people.txt](https://github.com/coreos/fedora-coreos-tracker/blob/main/meeting-people.txt).
