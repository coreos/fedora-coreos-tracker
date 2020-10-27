# Request to Include a New Package in Fedora CoreOS

If you would like to propose the inclusion of a new package into the base
content set of Fedora CoreOS, please open a [new issue](https://github.com/coreos/fedora-coreos-tracker/issues/new) 
with the following questions answered.  The more detail provided for each
question, the better informed everyone will be.

Please title the new issue: `Package Request: <name of package>`

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