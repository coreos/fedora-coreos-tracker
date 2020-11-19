---
name: Request a new platform
about: Ask for Fedora CoreOS to support a new cloud environment
title: 'Platform Request: <platform name>'
labels: 'area/platforms, kind/enhancement'
assignees: ''

---

In order to implement support for a new cloud platform in Fedora CoreOS, we need to know several things about the platform.  Please try to answer as many questions as you can.

- [ ] Why is the platform important?  Who uses it?

- [ ] What is the official name of the platform?  Is there a short name that's commonly used in client API implementations?

- [ ] How can the OS retrieve instance userdata?  What happens if no userdata is provided?

- [ ] Does the platform provide a way to configure SSH keys for the instance?  How can the OS retrieve them?  What happens if none are provided?

- [ ] How can the OS retrieve network configuration?  Is DHCP sufficient, or is there some other network-accessible metadata service?

- [ ] In particular, how can the OS retrieve the system hostname?

- [ ] Does the platform require the OS to have a specific console configuration?

- [ ] Is there a mechanism for the OS to report to the platform that it has successfully booted?  Is the mechanism required?

- [ ] Does the platform have an agent that runs inside the instance?  Is it required?  What does it do?  What language is it implemented in, and where is the source code repository?

- [ ] How are VM images uploaded to the platform and published to other users?  Is there an API?  What disk image format is expected?

- [ ] Are there any other platform quirks we should know about?
