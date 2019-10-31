---
title: Mac Pools Functionality Revisited
category: spike
authors: Eitan Raviv
date: Oct 2019
---

Mac Pools Functionality Revisited
================================================================

Evaluate the functionality existing in oVirt 4.4 based on experience gained in the last few releases.

Review of historic evolution of functionality
----------------------------------------------------------------

* Phase 1 - single MAC pool with single fixed small address range

  The first functionality available in oVirt was a single Mac Pool for the whole engine. 
  It had a small constant range that was bought by Kumranet from IANA and was visible in engine-config only.
  The same range was deployed in all oVirt installations, which contradicts the goal of being universally unique, as is
  the claimed goal of MAC addresses.
  
  The first problem to surface was that since the range was too small users needed to enlarge it so required it be accessible
  from the web-admin. 

* Phase 2 - MAC pool per DC

  The next enhancement was mac pools per Data Center, but there is no account why it was decided that the user should have
  the ability to allow duplicate macs and overlapping ranges. 

* Phase 3 - MAC pool per Cluster

  At a later stage in conjunction with Cloud-Forms which only had a concept of a Cluster and not of a Data Center it was
  contemplated to break down engine to independent Cluster level pieces, and to that end the mac pools were moved to the 
  cluster level.

  Another use case that benefits from this functionality is that of assigning a specific range to a dhcp server so IPs are
  generated according to the range. The cons for this use case are is not always feasible with physical NICs for two reasons:
  - physical NICs have random MAC addresses
  - physical network card replacement

* Phase 4 - MAC pool per VM pool - requested but not implemented

  There was once also a requirement from a single customer to have a MAC pool\range per VM pool in order to identify it 
  on the network but it was not implemented in engine due to complexity.

* Phase 5 - multiple ranges per pool, with overlaps

  The justification for supporting multiple ranges per pool is lost in the mists of time. Currently no real justification 
  for this design is apparent.

* Phase 6 - allow duplicate MAC addresses

  According to GSS users might need this feature for windows clustering.
  Another valid use case might be to have a VM in one DC be an identical copy of another VM in another DC for fail-over 
  purposes. When one  VM fails, and RHV is configured with HA, the second VM will start with same network configurations.
  Thirdly, for a critical VM, it is possible to create a second VM with same MAC address for a critical failure on the first
  VM. 

* Phase X - learning from past experience in CNV
  A single mac pool (with a single large random range) is now implemented in CNV. 
  This is because this is the most convenient and simple solution.


Review of some (closed) bugs encountered in past revisions
----------------------------------------------------------

  ###### Not blocking users from disallowing duplicates on mac pool that contains duplicates causes initialization failure of mac pools

   * BZ# 1561080, 1561081, 1561865, 1554180
   * fix: Prevent unsetting 'Allow Duplicates' while duplicate MACs exist in pool
 
  ###### Overlapping mac pool ranges cause "mac already in use" errors
  
   * BZ# 1593800,1492577, 1537414
   * fix: inter-mac-pool overlapping ranges: warn if found (4.3), forbid for new mac pools (4.4)
   * fix: intra-mac-pool overlapping ranges: warn if found (4.3), forbid for new mac pools (4.4)
   
  ###### Bug 1435485 - Pool - VMs are still created with duplicate MAC addresses after 4.0.7 upgrade 
  ###### Bug 1395462 - Vm Pool - VMs are created with duplicate MAC addresses 
  ###### Bug 1374619 - Engine generates duplicated mac addresses
  ###### Bug 1400043 - Vm Pool - VMs are created with duplicate MAC addresses
  
  ###### Bug 1446913 - Engine tries to re-assign MAC address from the wrong pool after VM was moved to another cluster
  
 
List of currently open bugs
---------------------------
 
  ###### overlapping mac pool ranges cause "mac already in use" errors

  * BZ# 1767319
  * fix (TBD): disallow updating overlapping mac pool ranges

  ###### If an in-use MAC is held by a VM on a different cluster, the engine does not attempt to get the next free MAC

  * BZ# 1760170
  * fix (TBD): add validation that MAC address is not allocated on all of the system should be carried out when getting 
    a MAC from the pool and not only when assigning a MAC to the vNic.

  ###### Can't import guest from export domain because of MAC duplication even if there is no guest listed on rhv

  * BZ# 1762141
  * fix (TBD): possibly the imported guest has MAC addresses on its vNic outside the range of the pool. add specific error
    handling and detailed user messages.


Propositions for of Mac Pool functionality
----------------------------------------------------------------

  ###### Remove cluster level MAC pools
    
  Since we cannot justify cluster-level MAC pools, it is possible to remove this option from the documentation and deprecate
  it in the API.

  ######
