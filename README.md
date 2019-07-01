FortiGate Policy
================

This role takes a rulebase layout, lints the resulting objects, then validates that the objects are used correctly in the 
policy. Finally, it installs the policy on a FortiNet FortiGate Firewall, and removes all non-implemented objects.

Requirements
------------

    fortiosapi>=0.10.4
    netaddr>=0.7.19

Role Variables
--------------
### Built-in values
A built FortiNet FortiGate or FortiOS appliance has a number of pre-defined address objects and services. These are defined in vars/main.yml in as concise a
form as possible. Typically, it's easier to use the built-in objects.

### Addresses, Services and Policies

Here's a sample of host/address objects that can be created:

    ---
    # Note that the prefix here ('mygroup') can be anything unique to the groups
    # assigned to this firewall.
    mypolicy_address_objects:
      test_net_1:
      # comment: "Optional comment can be added with just `comment:` in the object"
        ip: "192.0.2.0/24"
      host_a_in_test_net_2:
        comment: "Here we have a comment added"
        ip: "198.51.100.10"
      example_com:
        fqdn: example.com
      spanish_region:
        geo: ES
      webservers:
      # sdn: azure
      # comment: "If you've defined multiple SDN connectors, e.g. aws, azure, and openstack, you'd specify this with the sdn tag. It defaults to 'azure'"
        dynamic: "tag.webserver=true"

Here's how you would specify a host group. Note that due to how the playbook is delivered, you can only have 5 levels of recursion between groups, as follows:

    ---
    mypolicy_address_groups:
      example_sites:
      - example_com
      - example_net
      
      test_nets:
      - test_net_1
      - host_a_in_test_net_2
      
      recursion_level_2:
      - example_sites
      - test_nets
      
      recursion_level_3:
      - recursion_level_2
      
      recursion_level_4:
      - recursion_level_3
      
      recursion_level_5:
      - recursion_level_4

Services can, in normal FortiGate structure, be bundled into three separate groups; TCP/UDP, ICMP and IP. Here are five examples:

    ---
    mypolicy_service_objects:
      irc:
        tcp: "6660-6669" "7000" "194" "994"
      mosh:
        udp: "60000-61000"
      dns:
        tcp: "53"
        udp: "53"
      egp:
        ip: "8"
      icmp_redirect:
        icmp_type: "5"
        icmp_code: "0"

Like with host groups, there's a services group structure. Again, this is limited to 5 levels of recursion.

    ---
    mypolicy_service_groups:
      ssh_and_mosh:
      - SSH
      - mosh

Policies can cross multiple files, and are gathered into 100's of rules, following this pattern:

    ---
    # This next line sets the policyid prefix to "10"
    rulegroup10:
      # This next line sets the policyid suffix to 00 - the policyid is 1000
      rule00:
        # This next line is optional. Without it, it will set the value to "Policy_XXYY" where XX is the policyid prefix, and YY is the suffix
        name: Drop broadcast traffic
        # These sources and destinations need to be specified, either in *_address_objects, *_address_groups or in built_in_hosts.
        #   They can either be individual strings (like the following examples), or lists (like in the next rulegroup)
        sources: broadcast_addresses
        destinations: all
        # These services need to be specified in *_service_objects, *_service_groups, built_in_services or built_in_service_groups.
        #   They can either be individual strings (like the following examples), or lists (like in the next rulegroup)        
        services: ALL
        # actions are "allow" (default) and "deny"
        action: deny
        # Log is "all" (default) or "utm" or "none". To also enable logging from the start, set logstart: enable
        log: "none"
        # If you want the rule to be created, but marked as "disabled" set the following to true (options are true/yes/false/no)
        disabled: false
    
    ---
    # This next line sets the policyid prefix to "20"
    rulegroup20:
      # This policyid will be 2001
      rule01:
        sources:
        - test_net_1
        - test_net_2
        destinations: test_net_3
        # Force hide-nat behind the firewall interface
        nat: true
        services:
        - HTTP
        - HTTPS
      rule02:
        # Internet Services rely on a FortiCare subscription, and don't have "services".
        src_internet_service:
        - Google_Google_Bot
        dst_internet_service:
        - Amazon_Web
    
    ---
    # The next few sections are purely about defining IPS policies. If you are not doing IPS, these are not relevant to you :)
    rulegroup30:
      # This rule will have the policy id of 3001 and will perform IPS/SSL inspection against items that match this firewall
      #   policy.
      rule01:
        sources: somesource
        destinations: somedestination
        services: HTTPS
        ssl_ssh_profile: inspection_profile_group_a
        ips_sensor: ips_profile_group_a

IPS and SSL Inspection profiles are defined as follows:

    ---
    ips_profiles:
      monitor_only:
        entries:
        - severity: [info, low, medium, high, critical]
          status: enable
          action: pass
    ssl_ssh_inspect_profiles:
      corporate_inspection:
        # Note, this currently must be loaded on the device itself
        ca: corporate_ca_certificate
        
    ---
    # Create a file which defines the "default" IPS and SSL profiles for all rules
    # Note this would enable IPS inspection on all subsequent rules!
    #
    # Default SSL/SSH Inspection Profile for all rules
    ssl_ssh_profile: default_inspection_profile
    # Default IPS Sensor Profile for all rules
    ips_sensor: default_ips_profile
    
    ---
    rulegroup40:
      # Default Disable IPS rules across this rulegroup
      ips_enable: false
      # By default, use this SSL/SSH Inspection Profile for this rulegroup
      ssl_ssh_profile: inspection_profile_group_b
      # By default, use this IPS Sensor Profile for this rulegroup
      ips_sensor: ips_profile_group_b
      rule01:
        # This rule uses the default SSL and IPS profiles for this rule group
        #   and overrides the default IPS disable setting for this group..
        sources: newsource1
        destinations: newdestination1
        services: SSH
        ips_enable: true
      rule02: 
        # This rule uses the default IPS Sensor profile, but a separate SSL inspection profile
        sources: newsource2
        destinations: newdestination2
        services: SSH
        ips_enable: true
        ssl_ssh_profile: inspection_profile_group_c
      rule03:
        # This rule has NO IPS because it's disabled at the group level
        sources: noipssource
        destination: noipsdestination
        services: HTTPS

### Flags

All these variables are set from either the group vars, or by passing "extra" values (see the example playbook below)

* `run_actions_async` - default 'false': At the cost of massive CPU spike,
perform creation actions for objects (not groups) as a parallell activity.

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
  * `create_security_profiles`: Implement the Security Profiles as defined
in the git repo.

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
      hosts: all
      vars:
        dry_run: True
      roles:
        - e-and-cs.fortigate_policy

    - name: Implement changes, without removing any configuration from the firewall
      hosts: all
      vars:
        validate_all: False
      roles:
        - e-and-cs.fortigate_policy

    - name: Just re-order and remove items from the firewall. Don't try and create anything new.
      hosts: all
      vars:
        validate_all: False
        create_all: False
      roles:
        - e-and-cs.fortigate_policy

Extra-Vars method:

    ---
    - hosts: all
      roles:
        - e-and-cs.fortigate_policy

`ansible-playbook site.yml -e "{'validate_all': False, 'prune_all': True}"`

License
-------

MIT

Author Information
------------------

[Jon Spriggs](mailto:jon.spriggs@uk.fujitsu.com) is a Technical Consultant for Fujitsu, in the Enterprise & Cyber Security Business Unit.
