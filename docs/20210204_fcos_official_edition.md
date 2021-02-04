# Fedora CoreOS Meetup - Fedora CoreOS as an Official Edition

Fedora Change : https://fedoraproject.org/wiki/Changes/FedoraCoreOS

Key concepts :
* There is no Fedora CoreOS 33 or 34, I don't think we want to align with the 6 months release cadence of other Fedora Editions.
* How can we integrate within Fedora's process, keeping our fortnigthly release cycle.


General Feedback was how do we integrate with the Fedora process

Fedora's [Edition promotion policy](https://docs.fedoraproject.org/en-US/council/policy/edition-promotion-policy/)
* Development and how we integrate with Fedora's change proposal process ? (Review, Propose Changes)
* Go/No Go process ? Release Blockers
* Release Criteria ?
* When do we switch streams to the latest Fedora base (F33 -> F34, etc...)?
* How do we coordinate with other teams
    * Docs
    * Marketing
    * Translation
    * Magazine
    * Web
* How much effort do we want to put into making FCOS an edition ? What are the benefits ?
* Have you asked anyone who has gone through this process if it was useful to them?

## Notes

- [miabbott/cverna] Short introduction
- [mattdm] how does "we don't have releases" work with the release blocker process?
- [bgilbert] we ship the stable stream later than major Fedora releases for this reason; may not be desirable in all cases.  if there is a blocker that only affects FCOS, we may not want to hold the other releases.
- [mattdm] publicity is a factor of concern here
- [bcotton] user perspective on release day is problematic; "why am i getting older Fedora bits?"
- [walters] we think we can address all these concerns over time. i.e. ubuntu has similar issues with software upater/apt - https://www.reddit.com/r/Ubuntu/comments/aofv57/software_updater_lags_behind_apt/
- [sumantro] Blocker Bugs for Fedora tracked in BZ; FCOS tracks issues in GH
- [bgilbert] FCOS is an appliance, uses automatic updates.  Breaking updates incentivizes users to turn off auto-updates. Streams exist towards this goal.
- [mattdm] automatic updates are a selling point; we should use it to our advantage
- [travier/bcotton/mattdm] <discussion of how to rectify how classic Fedora + FCOS handle change proposals, etc>
- [jligon] e.g. If I want to remove Docker from FCOS, do I submit a change to FCOS only, Fedora proper, somewhere on GH?
- [mattdm] feels like a self-contained change; would be discussed by stakeholders and publicized appropriately (picked up by LWN, Phoronix, etc)
- [mattdm] FCOS changes being part of existing Fedora change process is desirable
- [walters] bootupd should have been a change request; but sometimes we need to ship something downstream faster than Fedora allows
- [bgilbert] prefer not stacking big changes around major release; easier for change management
- [bcotton] window between self-contained change proposal and GA is only 3months
- [cglombek] there are still usecases where Fedora Server is better suited (firewall, RPM modules, etc)
- [mattdm] FCOS will likely sit alongside Fedora Server for a while
- [walters] I don't think it's a good idea overall to chain FCOS Edition status into Server's edition status
- [mattdm] should develop an async process for ???
- [cverna] promoting between streams is gated on testing; what does the formalized process look like
- [cglombek] https://github.com/coreos/enhancements that are going to affect the rest of Fedora, we should "upstream" those enhancements to proper Fedora Change Requests.  conversely, Fedora Chagne Requests that affect FCOS should get better review by FCOS
- [walters] we could use an arbitrary component in BZ to capture problems for FCOS
- [dmabe] there is an FCOS component, but it directs folks to use GH issue tracker
- [sumantro] get some basic criteria around stream promotion; can't catch everything in CI; https://fedoraproject.org/wiki/Fedora_Release_Criteria
- [walters] A good example of not-CI currently for us is multi-arch
- [bgilbert] our decision process so far has been case-by-case and consensus-driven
- [jlebon] we should be doing more talking/communication on Fedora devel around change requests that affect FCOS
- [jligon] is there a tradeoff where becoming an official top-level edition where some decision making is surrendered?
- [bcotton] there is some latitude for editions for change proposals; there is marketing/UX benefits to be closer to the rest of Fedora. tl;dr - case by case basis
- [travier] our release criteria exists in CI; we do evaluate each update that we ship is safe to use. when issues are found, we have more options to prevent those issues from being released (i.e downgrades, pinned packages, etc)
- [bgilbert] we snapshot bodhi stable that gets promoted into the testing stream; pkgs are not pinned for an extended amount of time. we do more post-processing than most of Fedora.
- [mattdm] it would be beneficial to check in with mindshare team regularly
- **[sumantro] would like to volunteer to be mindshare rep for FCOS**

