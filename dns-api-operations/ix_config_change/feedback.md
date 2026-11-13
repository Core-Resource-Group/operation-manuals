# Feedback for InfoBlox config change manual

**Questions**
* Title is "config change", but op refers to cutover to new app version. Also states that no application pods are touched...? Doesn't seem to make sense. Is new application deployment handled by a different workflow?
* Why is the application configuration for the new server not applied _before_ it is brought online and made accessible? This order of operations also doesn't seem to make sense, but I might not be aware of something here.

**Section 4 - Variables**
* "Risk Level" and "User Impact" are static values, only used in announcement. These should be removed from the variables list and the expected values moved into the announcement message template.

**Section 11 - Operation**
* General : If reasonable, dual-target traffic should be enabled and verified in Standby Cluster before Active Cluster (swap steps 2+3).
* General : After application of new routing configuration, expected/functional routing should always be confirmed (if possible).
* Step 1 : `Availability Zone: local` -> should this not be `Availability Zone: {AVAILABILITY_ZONE}`, with the target AZ defined?

**Low priority**
* Censoring of the product name may be flagged as strange / not approved. If obfuscation is strictly needed for some reason, `I*****x` should be replaced with an appropriate regular-word. Alternative: restrict access to the template/manual pages to (UCP + Compute + Managers + TASC reviewers) and just write InfoBlox.
* As-is, it's pretty obvious what system is being used. There are very few DNS solutions appropriate for our scale, and it takes a ~30-second google search of the first+last characters to identify. Obfuscation here provides essentially zero security.
