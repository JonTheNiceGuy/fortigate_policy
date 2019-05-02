Firewall Policy
=========

This role takes a rulebase layout, lints the resulting objects, then validates that the objects are used correctly in the 
policy. Finally, it installs the policy on a FortiNet FortiGate Firewall, and removes all non-implemented objects.

Requirements
------------

fortiosapi>=0.10.4
netaddr>=0.7.19

Role Variables
--------------

All these variables are set from either the group vars, or by passing "extra"
* `validate_all` - Default `True`: Ensures all the following flags match
certain linting and validation checks
  * `validate_addresses`: Checks just Address objects and groups
  * `validate_services`: Checks just Service objects and groups
  * `validate_policies`: Checks that Policy items are valid and that
rules have all requisite items in them.
  * `validate_names`: Confirms that there are no naming conflicts

* `dry_run` - Default `False`: Do not perform any of the creation, reorder or prune actions listed below.

* `create_all` - Default `True`: Enables the following flags
  * `prevent_conflicts`: Ensures that service and address objects don't
have name conflicts with the group objects of the same type.
  * `create_addresses`: Creates all address object and groups that are
used in the policy.
  * `create_services`: Creates all service object and groups that are
used in the policy.
  * `create_policy`: Implements the policy as defined in the git repo.

* `reorder_policy` - Default `True`: Re-order the Firewall Policy to match the rule numbers

* `prune_all` - Default `False`: Enables the following flags
  * `prune_policy`: Remove policy rules which are not explictly defined in the playbook targetted at this host.
  * `prune_addresses`: Remove unused address objects which are not explicitly defined in the policy which will be deployed to this host.
  * `prune_services`: Remove unused services which are not explicitly defined in the policy which will be deployed to this host.

Dependencies
------------

None

Example Playbook
----------------

Multiple Plays Method

    ---
    - name: Test policy and dump output
      hosts: all:!localhost
      vars:
        dry_run: True
      roles:
        - Firewall.Policy

    - name: Implement changes, without removing any configuration from the firewall
      hosts: all:!localhost
      vars:
        validate_all: False
      roles:
        - Firewall.Policy

    - name: Just re-order and remove items from the firewall. Don't try and create anything new.
      hosts: all:!localhost
      vars:
        validate_all: False
        create_all: False
      roles:
        - Firewall.Policy

Extra-Vars method:

    ---
    - hosts: all:!localhost
      roles:
        - Firewall.Policy

`ansible-playbook site.yml -e "{'validate_all': False, 'prune_all': True}"`

License
-------

TBC

Author Information
------------------

[Jon Spriggs](mailto:jon.spriggs@uk.fujitsu.com) is a Technical Consultant for Fujitsu, in the Enterprise & Cyber Security Business Unit.
