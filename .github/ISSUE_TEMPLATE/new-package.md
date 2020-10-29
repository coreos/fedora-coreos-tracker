---
name: Request a new package
about: Ask for a new package to be added to Fedora CoreOS
title: 'New Package Request: <package name>'
labels: ''
assignees: ''

---

Please try to answer the following questions about the package you are requesting:

1. What, if any, are the additional dependencies on the package? (i.e. does it pull in Python, Perl, etc)

2. What is the size of the package and its dependencies?

3. What problem are you trying to solve with this package? Or what functionality does the package provide?

4. Can the software provided by the package be run from a container?  Explain why or why not.

5. Can the tool(s) provided by the package be helpful in debugging container runtime issues?

6. Can the tool(s) provided by the package be helpful in debugging networking issues?

7. Is it possible to layer the package onto the base OS as a day 2 operation?  Explain why or why not.

8. In the case of packages providing services and binaries, can the packaging be adjusted to just deliver binaries?

9. Can the tool(s) provided by the package be used to do things we’d rather users not be able to do in FCOS? (e.g. can it be abused as a Turing complete interpreter?)

10. Does the software provided by the package have a history of CVEs?
