# One Firewall with a Simple Policy

This example has a single firewall (called "my_firewall" - unimaginatively),
has an Active Directory and RDS environment inside it. Here's a simple network
diagram.

```
                          | Unprotected ingress network (Port1)
                          |
                +-------------------+
Mgmt Env 1 --\  |                   |  /-- Web Network 1
       Port2 |--|    my_firewall    |--| Port4
Mgmt Env 2 --/  |                   |  \-- Web Network 2
                +-------------------+
                          |
                          | Admin Network (Port3)
```

To separate out the "context", all items used in the policy live in 
`objects.yml` (but could equally be split into individual files per item)
while the policy lives in `policy.yml` (but could be split into individual
files per rulegroup.