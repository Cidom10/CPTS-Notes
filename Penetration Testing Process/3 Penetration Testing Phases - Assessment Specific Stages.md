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

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE1OTM2MTgyNDAsLTE4MDUxNjkzNiwtMT
Q5MTc0NzUzNl19
-->