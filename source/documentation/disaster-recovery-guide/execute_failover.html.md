# Execute a Failover

**Prerequisites**:

* The Manager and hosts in the secondary site are running.
* Replicated storage domains are in read/write mode.
* No replicated storage domains are attached to the secondary site.
* A machine running Red Hat Ansible Engine that can connect via SSH to the Manager in the primary and secondary site, with the required packages and files:
  * The `oVirt.disaster-recovery` package.
  * The mapping file and required failover playbook.

**Important:** Sanlock must release all storage locks from the replicated storage domains before the failover process starts. These locks should be released automatically approximately 80 seconds after the disaster occurs.

This example uses the `dr-rhv-failover.yml` playbook created earlier.

**Executing a failover**:

Run the failover playbook with the following command:
```
# ansible-playbook dr-rhv-failover.yml --tags "fail_over"
```

When the primary site becomes active, ensure that you clean the environment before failing back. See <<clean>> for more information.
