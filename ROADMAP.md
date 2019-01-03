This is a rough timeline/roadmap for the work we know we need to do
for Fedora CoreOS and Fedora 30. It is not complete but it is a start
at trying to wrangle all of the things we've discussed onto a calendar
so we can prioritize some things and let others wait until later (i.e.
can be done after first ship date).

## Fedora 30 schedule as [documented on the WIKI](https://fedoraproject.org/wiki/Releases/30/Schedule)
- 2019-01-29  Change Checkpoint: Proposal submission deadline (Self Contained Changes)
- 2019-02-19  Branch Fedora 30 from Rawhide (Rawhide becomes future F31)              
- 2019-03-05  Beta Freeze / Bodhi Activation                                      
- 2019-03-26  Beta Release (Preferred Target)                                         
- 2019-04-02  Beta Release (Target #1)                                                
- 2019-04-16  Final Freeze                                                        
- 2019-04-30  Fedora 30 Final Release (GA) (Preferred Target)                         
- 2019-05-07  Fedora 30 Final Release (GA) (Target #1)                                

### December
- 2018-12-16
    - ~~**H** - **finalize strategy** *Firewall Management [#26](https://github.com/coreos/fedora-coreos-tracker/issues/26)*~~
    - ~~**H** - **investigate** *no cloud agents [#95](https://github.com/coreos/fedora-coreos-tracker/issues/95)*~~
        - ~~azure [#65](https://github.com/coreos/fedora-coreos-tracker/issues/65). open new tickets for work items~~
    - ~~**M** - **collaborate** *Talk to Fedora kernel team about FCOS stream design [#80](https://github.com/coreos/fedora-coreos-tracker/issues/80)*~~
- 2018-12-23
    - Holidays - Go Rest!!
- 2018-12-30
    - **H** - **investigate** *no cloud agents [#95](https://github.com/coreos/fedora-coreos-tracker/issues/95)*
        - gce [#67](https://github.com/coreos/fedora-coreos-tracker/issues/67), open new tickets for work items
    - **H** - **finalize strategy** *ostree mirroring for better UX [#54](https://github.com/coreos/fedora-coreos-tracker/issues/54)*
    - **M** - **complete** *bare metal installer: POC [#91](https://github.com/coreos/fedora-coreos-tracker/issues/91)*
        - Proof of concept complete
    - **H** - **finalize strategy**,**collaborate** *Network Management [#24](https://github.com/coreos/fedora-coreos-tracker/issues/24)*
        - gaps identified feature work requested


### January
- 2019-01-07
    - **H** - **investigate** *no cloud agents [#95](https://github.com/coreos/fedora-coreos-tracker/issues/95)*
        - aws [#66](https://github.com/coreos/fedora-coreos-tracker/issues/66), open new tickets for work items
    - **H** - **collaborate** *fedora releng integration [#44](https://github.com/coreos/fedora-coreos-tracker/issues/44)*
    - **H** - **investigate** *no cloud agents [#95](https://github.com/coreos/fedora-coreos-tracker/issues/95)*
        - openstack [#68](https://github.com/coreos/fedora-coreos-tracker/issues/68), packet [#69](https://github.com/coreos/fedora-coreos-tracker/issues/69), open new tickets for work items
- 2019-01-14
    - **H** - **finalize strategy** *Kubernetes/OKD strategy [#93](https://github.com/coreos/fedora-coreos-tracker/issues/93)*
    - **H** - **investigate** *no cloud agents [#95](https://github.com/coreos/fedora-coreos-tracker/issues/95)*
        - virtualbox [#73](https://github.com/coreos/fedora-coreos-tracker/issues/), qemu [#74](https://github.com/coreos/fedora-coreos-tracker/issues/74), open new tickets for work items
    - **M** - **finalize strategy** *Collect metrics from Fedora CoreOS machines design [#86](https://github.com/coreos/fedora-coreos-tracker/issues/86)*
- 2019-01-21
    - Week of Devconf.cz
    - **H** - **investigate** *no cloud agents [#95](https://github.com/coreos/fedora-coreos-tracker/issues/95)*
        - vmware [#70](https://github.com/coreos/fedora-coreos-tracker/issues/70), digitalocean [#71](https://github.com/coreos/fedora-coreos-tracker/issues/71), open new tickets for work items
    - **M** - **finalize strategy** *burndown python dependencies [#92](https://github.com/coreos/fedora-coreos-tracker/issues/92)*
    - **L** - **complete** *merge of fedora-toolbox and coreos-toolbox efforts [#90](https://github.com/coreos/fedora-coreos-tracker/issues/90)*
- 2019-01-28
    - **M** - **complete** *Host Installer for Fedora CoreOS (bare metal) [#50](https://github.com/coreos/fedora-coreos-tracker/issues/50)*
        - Action items, gaps identified from POC ([#91](https://github.com/coreos/fedora-coreos-tracker/issues/91)) have been fixed


### February 
- 2019-02-04
    - **H** - **finalize strategy** *Container Linux migration tools and documentation [#48](https://github.com/coreos/fedora-coreos-tracker/issues/48)*
    - **M** - **finalize strategy** *Determine how to handle automatic rollback [#47](https://github.com/coreos/fedora-coreos-tracker/issues/47)*
- 2019-02-11 
    - **M** - **finalize strategy** *Equivalent to system containers from Fedora Atomic in Fedora CoreOS design [#37](https://github.com/coreos/fedora-coreos-tracker/issues/37)*
- 2019-02-18 
    - 2019-02-19  Branch Fedora 30 from Rawhide (Rawhide becomes future F31)              
- 2019-02-25 
    - **H** - **finalize strategy** *Throttled update rollouts [#83](https://github.com/coreos/fedora-coreos-tracker/issues/83)*
    - **H** - **complete** *action items from fedora releng integration discussion ([#44](https://github.com/coreos/fedora-coreos-tracker/issues/44))*
    - **H** - **complete aws, azure** *no cloud agents [#95](https://github.com/coreos/fedora-coreos-tracker/issues/95)*
        - https://github.com/coreos/coreos-metadata/issues/120
        - https://github.com/coreos/fedora-coreos-tracker/issues/4


### March       
- 2019-03-04
    - **M** **strategize** *reboot coordination: locksmith successor design [#3](https://github.com/coreos/fedora-coreos-tracker/issues/3)*
    - **H** - **complete gce** *no cloud agents [#95](https://github.com/coreos/fedora-coreos-tracker/issues/95)*
- 2019-03-11 
    - **H** - **complete openstack, packet** *no cloud agents [#95](https://github.com/coreos/fedora-coreos-tracker/issues/95)*
- 2019-03-18 
    - **H** - **complete virtualbox, qemu** *no cloud agents [#95](https://github.com/coreos/fedora-coreos-tracker/issues/95)*
- 2019-03-25 
    - **H** - **complete vmware, digitalocean** *no cloud agents [#95](https://github.com/coreos/fedora-coreos-tracker/issues/95)*

### April
- 2019-04-01
- 2019-04-08
- 2019-04-15
    - 2019-04-16  Final Freeze                                                        
- 2019-04-22
- 2019-04-29
    - 2019-04-30  Fedora 30 Final Release (GA) (Preferred Target)

### May
- 2019-05-06
    - 2019-05-07  Fedora 30 Final Release (GA) (Target #1)
