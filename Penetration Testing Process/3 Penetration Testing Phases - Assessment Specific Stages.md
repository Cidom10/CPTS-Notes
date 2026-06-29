Entirety of pre-engagement consists of:
1. Scoping questionnaire
2. Pre-engagement meeting
3. Kick-off meeting

| Type | Description |
|--|--|
| Unilateral NDA | This type of NDA obligates only one party to maintain confidentiality and allows the other party to share the information received with third parties. 
| Bilateral NDA | In this type, both parties are obligated to keep the resulting and acquired information confidential. This is the most common type of NDA that protects the work of penetration testers. |
| Multilateral NDA | Multilateral NDA is a commitment to confidentiality by more than two parties. If we conduct a penetration test for a cooperative network, all parties responsible and involved must sign this document. |

### Documents Needed for a Pentest
(Could break Computer Misuse Act if not signed)

| Document | Timing for Preparation |
| -- | -- |
| Non-Disclosure Agreement (NDA) | After initial contact|
| Scoping Questionnaire | Before pre-engagement |
| Scoping Document | During pre-engagement |
| Pentest Proposal (Contract/SoW) | During pre-engagement |
| Rules of Engagement (RoE) | Before kick-off meeting |
| Contractors Agreement (physical assessments) | Before kick-off meeting |
| Reports | During and After conducted pentest |

Typical Questions in a Scoping Questionnaire:

 - How many expected live hosts?
 - How many IPs/CIDR ranges in scope?
 - How many Domains/Subdomains are in scope?
 - How many wireless SSIDs in scope?
 - How many web/mobile applications? If testing is authenticated, how many roles (standard user, admin, etc.)?
 - For a phishing assessment, how many users will be targeted? Will the client provide a list, or we will be required to gather this list via OSINT?
 - If the client is requesting a Physical Assessment, how many locations? If multiple sites are in-scope, are they geographically dispersed?
 - What is the objective of the Red Team Assessment? Are any activities (such as phishing or physical security attacks) out of scope?
 - Is a separate Active Directory Security Assessment desired?
 - Will network testing be conducted from an anonymous user on the network or a standard domain user?
 - Do we need to bypass Network Access Control (NAC)?
 - Is the Penetration Test black box (no information provided), grey box (only IP address/CIDR ranges/URLs provided), white box (detailed information provided)
-  Would they like us to test from a non-evasive, hybrid-evasive (start quiet and gradually become "louder" to assess at what level the client's security personnel detect our activities), or fully evasive.

Checklist for a Contract can be found [here](https://academy.hackthebox.com/app/module/90/section/937) (table is really long)

Checklist for a Rules of Engagement can be found at the same link (table is really long)

### Kick-Off Meeting
After signing all contracts and docs, in-person
Includes client POCs, tech support staff, pentest team, and sometimes a Project Manager
Inform customer about potential risks

### Information Gathering
This is the phase in which we gather all available information about the company, its employees and infrastructure, and how they are organized.
-   Open-Source Intelligence
-   Infrastructure Enumeration
-   Service Enumeration
-   Host Enumeration

### Infrastructure Enumeration
During the infrastructure enumeration, we try to overview the company's position on the internet and intranet. 
Use OSINT and the first active scans. 
Use DNS to create a map of the client's servers and hosts 
Develop an understanding of how their infrastructure is structured. This includes name servers, mail servers, web servers, cloud instances, and more. 
Make a list of hosts and IPs and compare them to our scope to see if they are included and listed.
Also try to determine their security measures

### Service Enumeration
Find services that interact with the host or server over network
Find version, provided info, and reason it can be used

### Host Enumeration
Identify OS, services, service versions, etc of every host in the scoping document
Internally, look for sensitive files, services, scripts, apps, info, etc

### Pillaging
After Post-Exploitation, collect sensitive info on exploited hosts
Can show the impact of a potential attack
Used for further steps to escalate privileges or move laterally


### Vulnerability Assessment

An analysis is a detailed examination of an event or process, describing its origin and impact, that with the help of certain precautions and actions, can be triggered to support or prevent future occurrences.

| Analysis Type | Description |
| -- | -- |
| Descriptive | Descriptive analysis is essential in any data analysis. On the one hand, it describes a data set based on individual characteristics. It helps to detect possible errors in data collection or outliers in the data set. |
| Diagnostic | Diagnostic analysis clarifies conditions' causes, effects, and interactions. Doing so provides insights that are obtained through correlations and interpretation. We must take a backward-looking view, similar to descriptive analysis, with the subtle difference that we try to find reasons for events and developments. |
| Predictive | By evaluating historical and current data, predictive analysis creates a predictive model for future probabilities. Based on the results of descriptive and diagnostic analyses, this method of data analysis makes it possible to identify trends, detect deviations from expected values at an early stage, and predict future occurrences as accurately as possible. |
| Prescriptive | Prescriptive analytics aims to narrow down what actions to take to eliminate or prevent a future problem or trigger a specific activity or process. |

### Exploitation

Once vulnerabilities are found, we can prioritize attacks
- Probability of success
- Complexity
- Probability of damage

CVSS Scoring, NVD Calculator

### Data Exfiltration

Important Industry Standards:
| Info Type | Regulation |
| -- | -- |
| Credit Card | PCI-DSS |
| e-Patient Health Info | HIPAA |
| Private Banking Info | Gramm-Leach-Bliley (GLBA) |
| Government Info | FISMA |

Frameworks tat
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIwMDUyNDQ4OTgsLTU3NTcyMjc0OSwtMT
ExODUzODA3NCwtMTIyODU5MjQ4NCw4NDY3MjEwMDgsLTE1OTM2
MTgyNDAsLTE4MDUxNjkzNiwtMTQ5MTc0NzUzNl19
-->