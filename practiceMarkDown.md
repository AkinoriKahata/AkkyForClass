## Security Principles
What are the security properties of the system? How does it adhere to the principles for secure system design? What is the reference monitor in the system, and how does it provide complete mediation, tamperproof-ness, and how does it argue trustworthiness?

### Hypothesis and questions
- dom0 is key components of the xen. domU have to communicate with dom0 to use syscall(hypercall in VM). Thanks to this architecture, xen may check the critical activities of domUs by dom0.
- But, one recent optimization is the dom0-less static partitioning to avoid the complexity of running an extra management virtual machine after bootup. Also, even though dom0 is used, there is the option for xen PCI passthrough, which gives the VM guest more rights - instead of sending requests to the event channel via dom0, the privileged domU can bypass the access the hardware directly (much faster but less secure). How to keep secure in these cases?

### Xen's security feature
<The point of Over whole architecture>
1. Key component dom0
- only dom0 can communicate with hardware.
- domains are separated each other.

2. XSM (Xen Security Module) and FLASK (Flux Advanced Security Kernel)
- Provides Mandatory Access contro.
- control VMM operation
- Security ENriched LInux (SELinux) in dom0; use network to communicate

3. TCB
- For tamper proof, Xen has a much larger TCB, and more flexible

4. Verification
- Xode - 200k + LOC
- Policy - SELinux style

### Code analysis of Each feature
(1) dom0 and domU

(2) Mandatory Access control provided by XSM
a. Type
b. Role
c. User

(3) TCB


(4) Tamper proof


<Key source cord>
xen/xen/xsm/flask/ss/policydb.h
xen/xen/xsm/flask/ss/policydb.c
xen/tools/flask/policy/modules/xen.te
 