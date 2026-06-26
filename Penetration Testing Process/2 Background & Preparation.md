
Blackbox	
- Minimal. Only the essential information, such as IP addresses and domains, is provided.

Greybox	
- Extended. In this case, we are provided with additional information, such as specific URLs, hostnames, subnets, and similar.

Whitebox	
- Maximum. Here everything is disclosed to us. This gives us an internal view of the entire structure, which allows us to prepare an attack using internal information. We may be given detailed configurations, admin credentials, web application source code, etc.

Red-Teaming	
- May include physical testing and social engineering, among other things. Can be combined with any of the above types.

Purple-Teaming	
- It can be combined with any of the above types. However, it focuses on working closely with the defenders.

=======================================================

**Computer Fraud and Abuse Act (CFAA)** - No computer access w/o authorization when hacking, etc

**Digital Millennium Copyright Act (DMCA)** - No hacking to protect copyrighted works

**Electronic Communications Privacy Act (ECPA)** - No storing communications w/o consent

**HIPAA** - No using private health info

**Children's Online Privacy Protection Act (COPPA)** - Regulates storing info of children under 13

---

**Precautions for PenTests**
1. Obtain written consent from the owner or authorized representative of the computer or network being tested
2. Conduct the testing within the scope of the consent obtained only and respect any limitations specified
3. Take measures to prevent causing damage to the systems or networks being tested
4. Do not access, use or disclose personal data or any other information obtained during the testing without permission
5. Do not intercept electronic communications without the consent of one of the parties to the communication
6. Do not conduct testing on systems or networks that are covered by the Health Insurance Portability and Accountability Act (HIPAA) without proper authorization

---

**Deterministic Process** - each stats is dependent on and determined by past states
**Stochastic Process** - One state follows from others with only a certain probability

![Penetration Testing Process](https://cdn.services-k8s.prod.aws.htb.systems/content/modules/90/0-PT-Process.png)

### Pre-Engagement
Educating the client and adjusting the contract
Use in-person meetings or conference calls to set up:
- Non-Disclosure Agreement (NDA)
- Goals
- Scope
- Time Estimate
- Rules of Engagement

### Information Gathering
Search for information about the target company and the software and hardware in use to find potential security gaps that we may be able to leverage for a foothold.

### Vulnerability Assessment
Analyze results from Info Gathering stage
Analyze systems, apps, version for possible attack vectors
Used to determine threat level and susceptibility of infrastructure to cyber attack

### Exploitation
Use results to test attacks against the potential vectors
Execute them against target systems to gain access

### Post-Exploitation
After gaining access, try to escalate privileges
Hunt for sensitive data worth protecting (pillaging)

### Lateral Movement
Movement within internal network to access additional hosts at the same or. higher privilege level

### Proof-of-Concept
Document, step-by-step, the steps we took to achieve network compromise or some level of access
Paint a picture of how we were able to chain together multiple weaknesses to reach our goal so they can see a clear picture of how each vulnerability fits in and help prioritize their remediation efforts

### Post-Engagement
Detailed documentation is prepared for both administrators and client company management to understand the severity of the vulnerabilities found
Clean up all traces of our actions on all hosts and servers

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTcyMzg2MTMzOV19
-->