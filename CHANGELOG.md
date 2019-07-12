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
