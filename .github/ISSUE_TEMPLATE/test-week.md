---
name: Schedule a Fedora Test Week
about: Schedule a Fedora Test Week for a new Fedora major release
title: ''
labels: 'community', 'meeting'
assignees: ''
---

## Initial Tasks (to be done at least one week before test week)

- [ ] Open this ticket in the `fedora-coreos-tracker` repo
- [ ] Open a ticket with [Fedora QA](https://pagure.io/fedora-qa/issues).
  - [F36 example](https://pagure.io/fedora-qa/issue/695)

To be done after the Fedora QA folks have taken action on the QA ticket:

- [ ] Confirm the Test Day page is created on the Fedora Wiki.
  - For example: <https://fedoraproject.org/wiki/Test_Day:Fedora_36_CoreOS_2022-04-04>
- [ ] Confirm the Test Day results app is created.
  - For example: <https://testdays.fedoraproject.org/events/131>
- [ ] Choose a day during the Test Week to host a video session for live debug help
- [ ] Setup a Google Meet or other video conference session
- [ ] Create a HackMD doc for capturing notes during live video session
- [ ] Find volunteers to enumerate new documentation + test cases required for Test Week
  - Best done via dedicated video session
- [ ] File an issue on `fedora-coreos-tracker` with TODO items.
  - For example: <https://github.com/coreos/fedora-coreos-tracker/issues/1147>
- [ ] File a ticket requesting a Fedora badge is created
  - For example: <https://pagure.io/fedora-badges/issue/871>

## Announcing Test Week

Should be completed after the Initial Tasks are done

- [ ] Draft an email to <coreos@lists.fedoraproject.org> announcing the Test Week
  - [ ] Include a link to the Fedora Wiki
  - [ ] Include a link to the Test Day results app
  - [ ] Include a link to `fedora-coreos-tracker` for Test Week
  - [ ] Include a link to the video conference
  - [ ] Include a link to the HackMD doc
- [ ] Cross-post announcement email to discussion.fedoraproject.org with `#coreos` tag

- Example format: <https://hackmd.io/lCrVoW_RSMCRf2neiGhiPg>

## During Test Week

- Monitor `fedora-coreos-tracker` for new issues reported as part of Test Week
- Monitor #fedora-coreos on IRC for new issues reported as part of Test Week
- Ensure there is one or more representatives of Fedora CoreOS team present for live video session

## After Test Week

- [ ] Update `fedora-coreos-tracker` ticket with any issues found
- [ ] Update `fedora-coreos-tracker` ticket with any documentation updates made
- [ ] Review Test Day results app and follow-up on any errors reported, if possible
- [ ] Follow up with the Fedora Badges ticket with Fedora Account System (FAS) usernames that participated in Test Week
