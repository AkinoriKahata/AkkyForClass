## Security Principles
What are the security properties of the system? How does it adhere to the principles for secure system design? What is the reference monitor in the system, and how does it provide complete mediation, tamperproof-ness, and how does it argue trustworthiness?

### Hypothesis and questions
- dom0 is key components of the xen. domU have to communicate with dom0 to use syscall(hypercall in VM). Thanks to this architecture, xen may check the critical activities of domUs by dom0.
- But, one recent optimization is the dom0-less static partitioning to avoid the complexity of running an extra management virtual machine after bootup. Also, even though dom0 is used, there is the option for xen PCI passthrough, which gives the VM guest more rights - instead of sending requests to the event channel via dom0, the privileged domU can bypass the access the hardware directly (much faster but less secure). How to keep secure in these cases?

### Xen's security feature
**The point of Over whole architecture**
1. Dom0 (aka Control Domain) as reference monitor
- Dom0 (domain0) is initial domain which is started at boot timing of hyperxen hyper visor, and it is key component for security. Dom0 has privilege to manages domU (domain user). Basically, only dom0 can access the hardware directly. DomU have to communicate with dom0 to use syscall(hypercall in VM). Thanks to this architecture, xen may check the critical activities of domUs by dom0.
- But sometimes, domU want to access drivers directly. In these cases, there are mechanism of DriverDomain, and also hardware can be passed thourh to the domU.

2. XSM (Xen Security Module) and FLASK (Flux Advanced Security Kernel)
- Provides **Mandatory Access control** like Security ENriched LInux (SELinux).

3. TCB (Trusted Computer Base)
- Xen's TCB is relatively large. Xen's TCB includes Contorol domain (Dom0) and hypervisor layer. Dom0 has toolstacks, dom0 kernel whicha includes drivers. Hypervisor layers has memory management unit (mmu), scheduler and xen security module (xsm). There is difficult concept that Large TCB is good or not but ....

4. Verification
- Xode - 200k + LOC
- Policy - SELinux style

### Code analysis of Each feature
(1) The relationship dom0 and domU
a. eventchannel
- Since domU can not use hypercall, there is a mechanism which can enables domU to communicate with dom0. Eeach domain can communicate via event_chnnel. This channel managed by event_fifo.c.

#### example code from event_channel.c
```
static struct evtchn *alloc_evtchn_bucket(struct domain *d, unsigned int port)
{ .....
    chn = xzalloc_array(struct evtchn, EVTCHNS_PER_BUCKET);
    ...
    for ( i = 0; i < EVTCHNS_PER_BUCKET; i++ )
    {
        chn[i].port = port + i;
        rwlock_init(&chn[i].lock);
    }
    return chn;
}
```

(2) Mandatory Access control provided by XSM
- there are three type of policies.
 a. Type  - Define which hypercall can be executed.
 b. Role  - Each role has set of types which belong to the role.
 c. User  - Each user has set of roles which belong to the user.

- Policies are written in TE(type enforcement) file, like **dom0.te**. General format is below.
```
*allows<ource type> <target type>:<security class> <hypercall>; 
<example code from "dom0.te">
allow dom0_t xen_t:xen {
	settime tbufcontrol readconsole clearconsole perfcontrol mtrr_add
	mtrr_del mtrr_read microcode physinfo quirk writeconsole readapic
	writeapic privprofile nonprivprofile kexec firmware sleep frequency
	getidle debug getcpuinfo heap pm_op mca_op lockprof cpupool_op
	getscheduler setscheduler hypfs_op
};
//dom0_t type can execute hypercalls in the xen class targeting a xen_t type.
```
- By using these files, type can be implemented each module, and each type are allocated to role, and users. These set of security attributes associated are called **security context**. These contexts are labeled by security id **sid**. Example of sids are "xen" contains "system_u:system_r:xen_t,s0, "dom0" contains "system_u:system_r:dom0_t,s0", and so on. These sids are put on **sidtab**, and those policies are contained by **policydb**.
- 

(3) TCB

(4) Tamper proof

### Advanced Feature (domain0 less architecture)
- To improve 

(1) device model stub domain


A stub domain is a specialized system domain running on a Xen host used in order to disaggregate the control domain ("domain 0"). Many stub domains are based on the Mini-OS tiny OS (see extras/mini-os in the Xen source code). Some work has been done on Linux stub domains.

In the specific case of a device model stub domain this system domain is used to run the Qemu device model associated with an HVM domain. Often the Qemu device model would be a normal process running in the domain 0 environment.

Device model stub domains were first implemented by Samuel Thibault in Xen 3.3 in 2008.


