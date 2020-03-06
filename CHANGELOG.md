# 1.3.0 2020-03-05
POSSIBLE BREAKING CHANGE: IPS "pass" and "monitor" were crossed. Please check!
* Better handling of IPS rules *HOWEVER* minor change to handling of IPS
Profiles means that "pass" and "monitor" were handled the wrong way around. If
you have used these actions, you will need to verify that you have the right
settings. 

Pass = "action: pass log: disable"
Monitor = "action: pass log: enable"
* Add quarantine to the IPS rules
* Add testing for IPS Signature Names to add to the IPS Sensor/Profile
* Add IPS IP Exemptions for Signatures.
* Add Documentation Directory (currently just a "simple firewall and policy")
* Added extra rulegroup passable options.

# 1.2.3 2019-08-07
* Add log-packet to IPS Sensor roles

# 1.2.2 2019-07-25
* Add src-filter to VIP objects in E02.
* Improved titles and comments in A00 and F07.
* Include a login test as well as a response test in A00.

# 1.2.1 2019-07-18
* Move to using `log_from_start` instead of `logstart` in rules
* Setup a new `log_from_start` value as a default value across the rulebase
* Add rulegroup selecter for `log_from_start`, `log`, `status` in addition to 
`nat`, `ips_enable`, `ssl_ssh_profile` and `ips_sensor`.

# 1.2.0 2019-07-12
* Allow SSL-Exception lists in SSL Inspection Profiles

# 1.1.7 2019-07-09
* Fixed typo in FQDN object creation

# 1.1.6 2019-07-09
* Fixed typos

# 1.1.5 2019-07-09
* Allow externally managed IPS and SSL Inspection profiles

# 1.1.4 2019-07-05
* Identified issue where no_proxy contains a hostname

# 1.1.3 2019-07-02
* Add Source NAT that isn't a Hide NAT (Closes #6)

# 1.1.2 2019-07-01
* Resolved issue where missing parameters cause the run to fail

# 1.1.1 2019-07-01
* Added rulegroup level nat definition, improved variable type detection 
for lists in source, destination and services.

# 1.1.0 2019-07-01
* Added IPS inspection
* Fix typo in assert statements around logstart
* Made async'able actions optionally async'd rather than enforced.
* Added versions to meta/main.yml and added changelog.

# 1.0.0 2019-06-09
* "Simple" policies work! Using git workflows and group_vars yml files :)
* Changelog before this point - not recorded.
