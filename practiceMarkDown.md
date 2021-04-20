## Security Principles
What are the security properties of the system? How does it adhere to the principles for secure system design? What is the reference monitor in the system, and how does it provide complete mediation, tamperproof-ness, and how does it argue trustworthiness?

### Hypothesis and questions
- dom0 is key components of the xen. domU have to communicate with dom0 to use syscall(hypercall in VM). Thanks to this architecture, xen may check the critical activities of domUs by dom0.
- But, one recent optimization is the dom0-less static partitioning to avoid the complexity of running an extra management virtual machine after bootup. Also, even though dom0 is used, there is the option for xen PCI passthrough, which gives the VM guest more rights - instead of sending requests to the event channel via dom0, the privileged domU can bypass the access the hardware directly (much faster but less secure). How to keep secure in these cases?

### Xen's security feature
**The point of Over whole architecture**
1. Key component dom0
 - Only dom0 can communicate with hardware. domains are separated each other.

2. XSM (Xen Security Module) and FLASK (Flux Advanced Security Kernel)
 - Provides **Mandatory Access control**. Control VMM operation. Security ENriched LInux (SELinux) in dom0; use network to communicate

3. TCB
 - For tamper proof, Xen has a much larger TCB, and more flexible

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
- there are three type of policies
 a. Type  - Define which hypercall can be executed.
 b. Role  - Each role has set of types which belong to the role.
 c. User  - Each user has set of roles which belong to the user.

- Policies are written in TE(type enforcement) file, like **dom0.te**. General format is below.
```
*allows<ource type> <target type>:<security class> <hypercall>; 
```
- The example below means that enables the dom0_t type to execute hypercalls in the xen class targeting a xen_t type. By using these files, type can be implemented each module, and each type are allocated to role, and users.  

#### example code from "dom0.te"
```
allow dom0_t xen_t:xen {
	settime tbufcontrol readconsole clearconsole perfcontrol mtrr_add
	mtrr_del mtrr_read microcode physinfo quirk writeconsole readapic
	writeapic privprofile nonprivprofile kexec firmware sleep frequency
	getidle debug getcpuinfo heap pm_op mca_op lockprof cpupool_op
	getscheduler setscheduler hypfs_op
};
```
These 



(3) TCB


(4) Tamper proof


<Key source cord>
xen/xen/xsm/flask/ss/policydb.h
xen/xen/xsm/flask/ss/policydb.c
xen/tools/flask/policy/modules/xen.te
 
