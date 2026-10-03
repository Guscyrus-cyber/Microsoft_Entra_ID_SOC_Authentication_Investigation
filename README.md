**Microsoft Entra ID SOC Lab — Setup and Preparation**

Steps Completed Before the Formal SOC Investigation

1.  I signed in to the Microsoft Azure Portal using the Azure administrator account.

2.  I opened Microsoft Entra ID from the Azure portal.

3.  I confirmed access to the Microsoft Entra admin center and the Entra ID tenant.

4.  Navigating to Identity → Users → All users.

5.  Creating a test user for the SOC lab:

    - Display name: SOC Test User

    - Username: testuser01@liberty566yahoo.onmicrosoft.com

6.  I used the temporary password generated for SOC Test User.

7.  I opened a separate browser/browser session so the administrator account and test-user account could be used independently.

8.  I attempted to sign in as SOC Test User using the temporary credentials.

9.  I generated authentication activity by performing different sign-in attempts, including unsuccessful sign-ins.

10. I completed the required password-related authentication steps when prompted by Microsoft.

11. Now I returned to the Microsoft Entra administrator session.

12. Navigating to the Sign-in logs for the test user.

13. I confirmed that Microsoft Entra ID recorded the authentication activity for SOC Test User.

14. Verifiing that the sign-in logs contained multiple authentication results:

    - Success

    - Failure

    - Interrupted

15. I confirmed that the logs contained different IP addresses associated with the authentication attempts.

16. Then I confirmed that the sign-in records included authentication requirements such as:

    - Single-factor authentication

    - Multifactor authentication

17. I confirmed that Conditional Access showed Not Applied for the observed events.

18. I identified several Microsoft Entra sign-in error codes in the generated authentication activity, including:

    - 0 — successful authentication

    - 50089

    - 50126

    - 50072

    - 50055

19. I located a specific failed authentication event for later SOC investigation:

    - User: SOC Test User

    - Status: Failure

    - Sign-in error code: 50126

    - IP address: 108.28.79.19

20. Stopping before opening the failed event so that the actual SOC investigation could begin as a separate, clearly documented lab.

> **Microsoft Entra ID SOC Authentication Investigation**\
> \
> Introduction

This lab introduces a realistic SOC Tier 1/Tier 2 identity and authentication investigation using Microsoft Entra ID.

Microsoft Entra ID is Microsoft's cloud-based identity and access management platform. In an enterprise environment, it records authentication activity involving users, applications, devices, IP addresses, authentication methods, and security controls. For a SOC analyst, these identity logs are important because compromised credentials and suspicious sign-ins are common starting points for security incidents.

In the preparation phase, I created SOC Test User and intentionally generated different authentication events. Microsoft Entra ID has now recorded successful, failed, and interrupted sign-in attempts, giving me a real authentication data to investigate rather than relying on a prepared dataset.

What I Will Do in This Lab

I will work through the authentication activity from the perspective of a SOC analyst. The investigation will include:

1.  Investigate a failed sign-in event and examine the event details rather than relying only on the sign-in log table.

2.  Analyze the user identity, timestamp, source IP address, application, location, authentication requirement, and sign-in status.

3.  Interpret Microsoft Entra sign-in error codes, beginning with the 50126 failure I generated.

4.  Examine authentication details to determine why authentication succeeded, failed, or was interrupted.

5.  Compare successful and failed authentication attempts to identify patterns that could indicate normal user error or suspicious activity.

6.  Evaluate IP addresses and location information to determine whether authentication activity deserves further investigation.

7.  Examine MFA information and understand how multifactor authentication affects an identity investigation.

8.  Review Conditional Access information and understand what Not Applied means in the context of the sign-in.

9.  Perform SOC triage by deciding whether observed activity should be classified as benign, suspicious, or potentially malicious.

10. Document the investigation using the type of evidence and reasoning expected from a SOC analyst.

11. Map relevant activity to MITRE ATT&CK, where appropriate, particularly techniques associated with credential attacks and account access.

12. Develop an escalation decision—whether the event should be closed, investigated further, or escalated to a Tier 2 analyst.

### Main Objective

The goal is to develop the analyst thought process:

Alert / Sign-in Event → Evidence Collection → Authentication Analysis → Correlation → Risk Assessment → Verdict → Documentation → Escalation

By the end of the lab, a SOC analyze will be able to look at a Microsoft Entra ID sign-in event and explain what happened, which account was involved, where the attempt originated, why it failed or succeeded, whether it appears suspicious, what additional evidence should be checked, and what action a SOC analyst should take.

This will also establish a foundation for later Microsoft Sentinel and KQL identity investigations, where the same type of authentication activity can be investigated and correlated at SIEM level.

Step 1 — Opening the Failed Sign-In Event

Status: Failure\
Sign-in error code: 50126 which is Microsoft Entra ID error code\
IP address: 108.28.79.19\
Location: Lorton, Virginia, US\
Conditional Access: Not Applied\
Authentication: Single-factor authentication (Image 1)\
\
Step 2 — Initial SOC Triage

From the event, I can establish:

- Date/time: August 15, 2026 — 11:53:48 PM

- User: SOC Test User

- Username: testuser01@liberty566yahoo.onmicrosoft.com

- Status: Failure

- Error code: 50126

- Failure reason: Error validating credentials due to invalid username or password

- Authentication requirement: Single-factor authentication

- Application: AMC PROD

- Client app: Browser

- Agent type: Not Agentic

- User agent: Chrome 151 on macOS

- Flagged for review: No

At this point, I know what failed and which account was targeted, but I still don't have enough evidence to decide whether this was simply a mistyped password or suspicious activity. (Images 2, 3, and 4)

Step 3 — Location Analysis

The Location tab provides information about the network origin of the failed authentication attempt.

The following evidence is recorded:

- Location: Lorton, Virginia, US

- IP address: 108.28.79.19

- Autonomous System Number (ASN): 701

- Through Global Secure Access: No

- Named location: No network details

### SOC Analysis

The source IP address is an important Indicator of Compromise (IOC) during authentication investigations. A SOC analyst can correlate this IP address with other sign-in events and determine whether multiple authentication failures originated from the same source.

The geographic location alone does not prove that an authentication attempt is legitimate or malicious. IP geolocation is approximate and can also be affected by VPNs, proxies, mobile networks, and ISP routing.

In this event, the combination of error code 50126 and source IP 108.28.79.19 establishes that an invalid credential attempt against SOC Test User originated from this network address.\
(Image 5)

Step 4 — Device Information Analysis

The Device info tab provides additional context about the system used for the failed authentication attempt.

The event shows:

- Browser: Chrome 151.0.0

- Operating System: macOS

- Compliant: No

- Managed: No

- Device ID: Not recorded

- Join Type: Not recorded

### SOC Analysis

The sign-in originated from a macOS system using Google Chrome.

Two fields are particularly important:

Managed: No — Microsoft Entra ID does not identify this device as an organization-managed device.

Compliant: No — the device is not recorded as meeting an organization's device-compliance requirements.

These values do not by themselves indicate malicious activity. For example, a legitimate user may authenticate from a personal device. However, in an enterprise SOC investigation, an authentication failure from an unmanaged/noncompliant device can increase the importance of checking the event against the user's normal authentication behavior.

At this stage, the investigation has established:

User → Failed password → Source IP → Geographic location → Browser → Operating system → Device management status (Image 6)

Step 5 — Authentication Details Analysis

The event shows:

- Authentication method: Password

- Authentication method detail: Password in the cloud

- Succeeded: false

- Result detail: Invalid username or password

What false Means

Succeeded: false means the password authentication attempt failed.

It does not mean that “Password in the cloud” is false. Password in the cloud describes where/how the password authentication was processed, while the separate Succeeded field reports the result.

Microsoft Entra ID therefore recorded:

A password-based cloud authentication attempt occurred, but the credentials presented during that attempt were not successfully validated.

Because the exact credentials entered at 11:53:48 PM are not available in my sign-in log, the investigation should not claim that a wrong password was deliberately entered Which I signed in correctly.

From a SOC perspective, the correct conclusion at this stage is:

The authentication failed because Microsoft Entra ID could not validate the supplied username/password credentials. The available evidence does not yet establish whether this resulted from normal user error or suspicious credential activity.

This uncertainty is realistic in SOC investigations. The next step is to examine surrounding authentication events and determine whether a pattern of failures exists. (Image 7)

Step 6 — Authentication Result Analysis

The Authentication Details confirm the following:

- Authentication method: Password

- Authentication method detail: Password in the cloud

- Succeeded: false

- Result: Invalid username or password

- Event time: August 15, 2026, 11:53:48 PM

### SOC Finding

Microsoft Entra ID received a password-based authentication attempt for SOC Test User, but the supplied credentials could not be validated.

At this stage, this event serves as a baseline failed-authentication event.

Step 7 — Conditional Access Analysis

The Conditional Access tab shows:

- Policy Name: Not applicable

- Grant Controls: None applied

- Session Controls: None applied

- Result: No Conditional Access policy evaluated for this event

### SOC Analysis

Not applicable means that a Conditional Access policy did not apply to this particular sign-in.

Conditional Access can enforce security requirements such as MFA, compliant devices, approved locations, or access restrictions. In this event, no such Conditional Access policy affected the authentication attempt.

This is useful evidence because the failure was caused by the credential validation itself (50126), rather than a Conditional Access policy blocking access. (Image 8)\

Step 8 — Correlation of Repeated Failed Sign-Ins

The event currently open is another 50126 failure:

- Date/time: August 15, 2026 — 11:52:18 PM

- Status: Failure

- Error code: 50126

- Failure reason: Invalid username or password

- User: SOC Test User

- Authentication requirement: Single-factor authentication

This is important because the previous event occurred at 11:53:48 PM with the same error code.

The log table also shows another 50126 event at 11:52:00 PM.

Therefore, at least three failed credential-validation events occurred within approximately two minutes:

11:52:00 → 11:52:18 → 11:53:48

### SOC Assessment

A single 50126 failure can easily result from a mistyped password. Multiple failures within a short period deserve correlation.

However, these events should not yet be classified as brute force. Additional evidence is required, especially the source IP, device, application, and surrounding successful authentication events. (Image 9)

Step 9 — MFA Interruption Analysis

This event at 11:58:44 PM is different from the earlier 50126 failures.

The event shows:

- User: SOC Test User

- Time: August 15, 2026 — 11:58:44 PM

- Status: Interrupted

- Authentication requirement: Multifactor authentication

- Additional details: The user was presented with options to provide contact information for MFA.

### SOC Analysis

This indicates that the authentication process reached an MFA-related stage, but the sign-in process was not completed.

This is an important distinction:

Failure (50126) → credentials could not be validated.

Interrupted (MFA) → the authentication workflow progressed further but was interrupted by an additional authentication/registration requirement.

The sequence is therefore becoming more useful for correlation:

Repeated credential failures → later authentication progression → MFA requirement/interruption

A SOC analyst would examine whether these events belong to the same source and whether a successful authentication occurred afterward. (Image 10)

Step 10 A — Correlating the MFA Events

The 11:57:52 PM event shows:

- User: SOC Test User

- Status: Interrupted

- Authentication requirement: Multifactor authentication

- Additional details: Contact information options were presented so MFA could be completed.

- Correlation ID: ba85226c-f60e-4b04-81c2-b907365b6acd

An important finding appears here: the 11:57:52 PM event and the previously examined 11:58:44 PM event have the same Correlation ID.

### SOC Analysis

A Correlation ID helps associate related authentication activity belonging to the same authentication flow or transaction.

Therefore, these two events should not automatically be interpreted as two independent suspicious login attempts. They are related events within the same MFA authentication sequence.

The authentication timeline currently contains:

11:52:00 → Failed credentials (50126)\
11:52:18 → Failed credentials (50126)\
11:53:48 → Failed credentials (50126)\
11:57:52 → MFA authentication interrupted\
11:58:44 → MFA authentication interrupted (Image 11)

Step 10 B: Investigate the Event Between Failure and MFA

There is another event visible at:

8/15/2026, 11:54:56 PM

The 11:54:56 PM confirms another MFA-interrupted event, so further repetition would add little value. (Image 12)\

Step 10 C— Simulated Suspicious Authentication by Kali Linux

The next objective is to generate controlled failed authentication attempts from Kali Linux against the lab account using deliberately incorrect passwords. These events will then be examined in Microsoft Entra ID and compared with the baseline events.

Step 11 — Kali Linux Network Identification

The Kali Linux network configuration confirms:

- Active interface: wlan0

- Kali private IPv4 address: 192.168.1.176

- Subnet: /24

- Loopback: 127.0.0.1

- Ethernet (eth0): Down

- Wireless (wlan0): Up

The address 192.168.1.176 is the private LAN address of the Kali system. Microsoft Entra ID normally records the public Internet-facing IP address, not this private address. (Images 13 and 14)

Step 12 — Kali Linux Public IP Confirmed

The Kali Linux system's public IP address is:

108.28.79.19

This is significant because the earlier Microsoft Entra ID 50126 events also showed 108.28.79.19.

This means the Mac and Kali Linux are currently reaching the Internet through the same public IP address, most likely because they are using the same local network/router. Therefore, the public IP alone cannot distinguish Kali activity from Mac activity.

Device/browser information and timestamps will be important for attribution.

Step 13 A— Generate the Controlled Failed Sign-In

1.  Go to the Microsoft sign-in page.

2.  Enter the lab account: testuser01@liberty566yahoo.onmicrosoft.com

3.  I deliberately enter an incorrect password.

4.  I Submit it.

This creates a known test event:

Kali Linux → Firefox → 108.28.79.19 → incorrect password → expected Entra 50126

That event can then be located and investigated in the Entra logs. (Image 15)\

Step 13 B— Controlled Failed Authentication Generated

The Microsoft login page displays:

“Your account or password is incorrect.”

This establishes a known test event:

Kali Linux → Firefox → SOC Test User → Incorrect password → Authentication failure

The event occurred at approximately 7:44 PM local time.

SOC Evidence

Microsoft Entra ID should generate a new sign-in event containing approximately:

- Status: Failure

- Error code: 50126

- Source public IP: 108.28.79.19

- Operating system: Linux

- Browser: Firefox

The browser and operating-system fields will be particularly useful because the public IP is shared with the macOS system.

Step 14 — Find the Kali Event in Entra ID

Microsoft Azure → SOC Test User → Sign-in logs

Step 15 — Kali Failed Sign-In Located

The newest event shows:

- Date: 8/16/2026, 7:37:34 PM

- User: SOC Test User

- Application: AMC PROD

- Status: Failure

- Error code: 50126

- IP address: 108.28.79.19

This is very likely the controlled failed authentication generated from Kali. The displayed time differs from the screenshot time, so the next step is to verify attribution using the device/browser evidence, rather than relying on time alone. (Images 16 and 17)

Step 16 — Kali Linux Event Confirmed

The event has now been successfully attributed to the Kali Linux authentication test.

### Evidence

Basic Info

- Date: August 16, 2026 — 7:37:34 PM

- Status: Failure

- Error code: 50126

- Failure reason: Invalid username or password

- User: SOC Test User

- Authentication: Single-factor authentication

Device Info

- Operating System: Linux

- Browser: Firefox 128.0

- Managed: No

- Compliant: No

- Device ID: Not recorded

Earlier network evidence established the public IP as:

108.28.79.19

### SOC Finding

The combination of Linux + Firefox + known test timestamp + failed credentials distinguishes this event from the earlier macOS/Chrome authentication activity, even though both systems shared the same public IP address.

The investigation therefore successfully demonstrates:

Kali Linux → Firefox → SOC Test User → Incorrect password → Entra ID → 50126 authentication failure

This is an important SOC lesson: an IP address alone may not uniquely identify a device. Authentication investigations should correlate multiple fields, including timestamp, operating system, browser, user, authentication result, and source IP. (Image 18)

Step 17 — Kali Authentication Evidence Confirmed

The Authentication Details provide the final confirmation for the simulated failed sign-in:

- Date: August 16, 2026 — 7:37:34 PM

- Authentication method: Password

- Authentication method detail: Password in the cloud

- Succeeded: false

- Result detail: Invalid username or password

### SOC Analysis

The evidence now establishes the complete authentication chain:

Kali Linux → Firefox 128 → SOC Test User → Cloud password authentication → Invalid password → Authentication failed → Entra ID error 50126

The attempt did not successfully authenticate, so no account access was obtained.

For this controlled lab, the event is a True Positive — Benign/Test Activity.

In a real SOC environment, the same unexplained combination of:

Linux device + unmanaged device + failed credentials + user account

would warrant correlation with other sign-ins before determining whether it represented user error, brute-force activity, or another credential attack.

### MITRE ATT&CK

Because the lab deliberately simulated an incorrect-password attempt against an account, the relevant ATT&CK category is:

T1110 — Brute Force

However, a single failed password attempt alone would not normally be sufficient evidence to declare a real-world brute-force attack. Repeated attempts or additional correlated evidence would be expected. (Image 19)
