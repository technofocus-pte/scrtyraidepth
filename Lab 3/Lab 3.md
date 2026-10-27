# Lab 3: Apply principles of Zero Trust to Microsoft Copilots

### **Lab Overview**

In this hands-on lab, you secure a Microsoft 365 Copilot deployment by
applying the three Zero Trust principles: verify explicitly, use least
privileged access, and assume breach. You work through the seven
protection layers in the order Microsoft recommends, deploying one
representative control per layer, and then validate that Copilot only
surfaces data a user is already allowed to see.

**Scenario:** Contoso Ltd. is rolling out Microsoft 365 Copilot to a
pilot group. Before licenses are assigned, the security team must
confirm that sensitive files are labeled, sign-ins are verified, apps
and devices are managed, threats are detected, Teams collaboration is
controlled, and oversharing is removed.

### **Learning Objectives**

By the end of this lab, you will be able to:

1.  **Explain** the three Zero Trust principles and how they apply to
    Microsoft 365 Copilot.

2.  **Describe** the seven protection layers and the order in which
    Microsoft recommends applying them before assigning Copilot
    licenses.

3.  **Deploy** one representative control for each layer: data
    protection, identity and access, App Protection, device management,
    threat protection, secure Teams collaboration, and minimum user
    permissions.

4.  **Remove** existing oversharing so each user has just enough access
    to the data they need.

5.  **Validate** that Copilot honors permissions, labels, and
    Conditional Access by testing an authorized and an unauthorized
    user.

6.  **Confirm** that Copilot interactions are recorded in Microsoft
    Purview Audit.

### **Lab Prerequisites**

- A **Microsoft 365 lab or trial tenant** with admin access. Never use a
  production tenant.

- **Microsoft 365 E3 (or Business Premium)** for Microsoft Entra ID P1,
  Microsoft Intune, Microsoft Defender for Office 365 Plan 1, and
  sensitivity labels. E5 is not required.

- **Microsoft 365 Copilot** licenses available for at least two pilot
  users (this also provides the SharePoint Advanced Management features
  used in Exercises 2 and 8)

- An admin account that holds, or can activate, these roles: **Global
  Administrator** (or the least-privileged set of Compliance,
  Conditional Access, Security, Intune, SharePoint, and Teams
  Administrator)

- Basic familiarity with Microsoft 365 services, identity, devices, and
  file permissions

- The lab accounts and data below, **created before the session** (not
  part of the lab time)

| **Account or object** | **Purpose** |
|----|----|
| [admin@\<tenant\>.onmicrosoft.com](mailto:admin@<tenant>.onmicrosoft.com) | Lab administrator |
| CopilotPilot1 (user) | Member of **Copilot pilot users** and **Finance Members**. Will be licensed for Microsoft 365 Copilot. |
| CopilotPilot2 (user) | Member of **Copilot pilot users** only; not a Finance member. Will be licensed for Microsoft 365 Copilot. |
| Copilot pilot users (security group) | Scopes Conditional Access, Intune, and label policies |
| Finance Members (security group) | Will replace broad access on the Finance site. Contains CopilotPilot1. |
| Finance (SharePoint site and Teams team) | Team-connected site that is currently shared with **Everyone except external users**. Holds Salary-review-2026.xlsx with test salary data. |
| Break-glass account | Emergency access account excluded from all Conditional Access policies |
| Windows 11 test device | Enrolled in Intune and signed in with a pilot account (used in Exercises 5 and 9) |

**Note:** Portal menus change often. If a label in a step does not match
your screen, use the portal search box to find the same setting by name.

## Exercise 1 — Prepare for Microsoft Copilots using Zero Trust

Scenario: You need to understand what you are protecting and why, before
you change any settings. Estimated time: 5 minutes.

### 

### **Task 1 — Review the Zero Trust principles**

Zero Trust treats every request as if it came from an uncontrolled
network: never trust, always verify. Keep this table in mind for the
rest of the lab.

| **Principle** | **Definition** | **How it applies to Copilot** |
|----|----|----|
| **Verify explicitly** | Always authenticate and authorize based on all available data points. | Validate user credentials, device requirements, and app permissions and behaviors. |
| **Use least privileged access** | Limit access with just-in-time and just-enough-access, risk-based policies, and data protection. | Remove oversharing; protect data with sensitivity labels and DLP policies. |
| **Assume breach** | Minimize blast radius, segment access, and use analytics to detect threats. | Use mailbox protection and Microsoft Defender to prevent, detect, and respond to attacks. |

### **Task 2 — Understand how Copilot uses your data**

A Copilot prompt passes through four components:

- **User devices** running Microsoft 365 apps, where users enter prompts

- The **Copilot service**, which orchestrates the response

- The **large language model (LLM)**, which generates the response

- **Microsoft Graph**, which retrieves your tenant’s data

**Note:** Copilot results contain only data the signed-in user is
already allowed to access. Copilot does not create new permissions; it
exposes existing ones faster. This is why oversharing must be fixed
before licenses are assigned.

### **Task 3 — Map the seven protection layers**

Microsoft recommends applying the layers in this order. Each remaining
exercise covers one layer.

| **Step** | **Protection layer** | **Zero Trust principle(s)** | **Exercise** |
|----|----|----|----|
| 1 | Data protection and compliance tools | Verify explicitly, Use least privileged access | 2 |
| 2 | Identity and access policies | Verify explicitly, Use least privileged access | 3 |
| 3 | App Protection policies | Use least privileged access, Assume breach | 4 |
| 4 | Device management and protection | Verify explicitly | 5 |
| 5 | Threat protection services | Assume breach | 6 |
| 6 | Secure collaboration for Microsoft Teams | Verify explicitly, Use least privileged access | 7 |
| 7 | Minimum user permissions to data | Use least privileged access | 8 |

## Exercise 2 — Deploy data protection

Scenario: The Finance site holds sensitive salary data. You hide it from
Copilot while you label and protect it. Zero Trust principles: Verify
explicitly, Use least privileged access. Estimated time: 10 minutes.

### **Task 1 — Hide the Finance site from Copilot**

On your lab computer, open **SharePoint Online Management Shell** and
run the following commands, replacing \<tenant\> with your tenant name:

1.  Paste the following command:\
    ***Connect-SPOService -Url <https://finance-admin.sharepoint.com>***

<img src="converted/Lab 3/media/media/image3f.png"
style="width:6.5in;height:3.38542in" />

2.  Paste the following command:\
    \
    ***Set-SPOSite \`-Identity
    "<https://gpssandboxlabs100174.sharepoint.com/sites/finance>"
    -RestrictContentOrgWideSearch \$true***

<img src="converted/Lab 3/media/media/image40.png"
style="width:6.5in;height:1.33333in" />

<img src="converted/Lab 3/media/media/image41.png"
style="width:6.5in;height:1.33333in" />

**Note:** This setting (Restricted Content Discovery) hides the site
from organization-wide search and Copilot, but it does not change
permissions. It is a temporary safeguard while you clean up access. You
turn it off in Exercise 8, once permissions are fixed.

### **Task 2 — Create, publish, and apply a sensitivity label**

1.  Sign in to the **Microsoft Purview portal** (purview.microsoft.com)
    as the lab administrator.\
    <img src="converted/Lab 3/media/media/image.png"
    style="width:6.25in;height:2.40625in" />

2.  Go to **Solutions \> Information Protection \> Sensitivity labels.**

> <img src="converted/Lab 3/media/media/image2.png"
> style="width:6.5in;height:2.35417in" />

3.  Select **+ Create a label**.

<img src="converted/Lab 3/media/media/image3.png"
style="width:6.5in;height:2.09375in" />

4.  Enter the following details:

    1.  Name: **Highly Confidential**

    2.  Display name: **Highly Confidential**

    3.  Description for users: **Restricted business data. Do not share
        outside the approved group.**

<img src="converted/Lab 3/media/media/image4.png"
style="width:6.5in;height:3.30208in" />

5.  For **Scope**, select **Files & other data assets** and **Emails**.

<img src="converted/Lab 3/media/media/image5.png"
style="width:6.5in;height:3.30208in" />

6.  Select **Control access**, then assign permissions to the **Copilot
    pilot users** group.

<img src="converted/Lab 3/media/media/image6.png"
style="width:6.5in;height:3.29167in" />

<img src="converted/Lab 3/media/media/image8.png"
style="width:6.5in;height:3.27083in" />

7.  Select **Create label**, then **Publish label**. In the label
    policy, publish to Finance **users**, name the policy **Copilot
    pilot label policy**, and select **Submit**.

<img src="converted/Lab 3/media/media/image9.png"
style="width:6.5in;height:3.30208in" />

8.  Open **Salary-review-2026.xlsx** in the Finance site in Excel for
    the web.

<img src="converted/Lab 3/media/media/imagea.png"
style="width:6.25in;height:2.59375in" />

9.  Select **Sensitivity**, and apply **Highly Confidential**.\
    <img src="converted/Lab 3/media/media/imageb.png"
    style="width:6.25in;height:4.21875in" />

**Note:** A newly published label can take time to appear in Office
apps. If it is not available yet, continue with the lab and apply it
before Exercise 9. Also confirm that sensitivity labels are enabled for
Office files in SharePoint and OneDrive, or Copilot cannot read labeled,
encrypted files correctly.

### **Task 3 — Create a DLP policy in simulation mode**

1.  In the Microsoft Purview portal, go to **Solutions \> Data loss
    prevention.**\
    <img src="converted/Lab 3/media/media/imagec.png"
    style="width:3.52083in;height:5.43812in" />

2.  Select **Policies**, then select **+ Create policy**.

<img src="converted/Lab 3/media/media/imaged.png"
style="width:6.5in;height:2.73958in" />

3.  Choose the **Custom** template and name the policy **DLP – Financial
    data**.\
    <img src="converted/Lab 3/media/media/imagee.png"
    style="width:6.25in;height:2.9375in" />

<img src="converted/Lab 3/media/media/imagef.png"
style="width:6.5in;height:3.03125in" />

4.  Select the locations **Exchange email**, **SharePoint sites**,
    **OneDrive accounts**, and **Teams chat and channel messages**.

<img src="converted/Lab 3/media/media/image10.png"
style="width:6.5in;height:3.30208in" />

5.  Create a rule named- New_rule and Set the action to **Restrict
    access or encrypt the content.**\
    <img src="converted/Lab 3/media/media/image11.png"
    style="width:6.25in;height:3.1875in" />

<img src="converted/Lab 3/media/media/image12.png"
style="width:6.25in;height:2.92708in" />

6.  Select Submit.\
    <img src="converted/Lab 3/media/media/image13.png"
    style="width:6.25in;height:3.17708in" />

7.  Select **Run the policy in simulation mode**, then submit the
    policy.\
    <img src="converted/Lab 3/media/media/image14.png"
    style="width:6.25in;height:2.78125in" />

**Note: Check your work against this list:**

- Is the Highly Confidential label published to the Copilot pilot users
  group, and applied to Salary-review-2026.xlsx?

- Does DLP – Financial data show a status of simulation mode?

- Has the Finance site dropped out of organization-wide search results
  for a pilot user? (This can take a while to apply.)

## Exercise 3 — Deploy identity and access

Scenario: Attackers should not be able to use stolen credentials to find
data quickly through Copilot. Zero Trust principles: Verify explicitly,
Use least privileged access. Estimated time: 8 minutes.

### **Task 1 — Create a multifactor authentication policy**

1.  Sign in to the **Microsoft Entra admin center**
    (entra.microsoft.com) and go to **Entra ID \> Conditional Access \>
    Policies**.<img src="converted/Lab 3/media/media/image15.png"
    style="width:6.25in;height:3.22917in" />

2.  Select **+ Create new policy from templates**, then choose the
    **Secure foundation** category.

<img src="converted/Lab 3/media/media/image16.png"
style="width:6.5in;height:2.33333in" />

3.  Select **Require multifactor authentication for all users**, then
    continue to review the policy.

<img src="converted/Lab 3/media/media/image17.png"
style="width:6.5in;height:2.08333in" />

4.  Under **Users**, confirm **All users** is included and exclude the
    **break-glass account**.

<img src="converted/Lab 3/media/media/image18.png"
style="width:6.25in;height:3.94792in" />

5.  Set **Enable policy** to **Report-only**, then create the policy.

<img src="converted/Lab 3/media/media/image19.png"
style="width:6.5in;height:4.09375in" />

**Warning:** Always exclude at least one emergency access account from
Conditional Access policies to avoid locking yourself out.

### **Task 2 — Review the impact and enforce the policy**

1.  Go to **Entra ID \> Monitoring & health \> Sign-in logs** and open a
    recent sign-in by a pilot user.

<img src="converted/Lab 3/media/media/image1a.png"
style="width:6.5in;height:2.86458in" />

2.  Select the **Report-only** tab and confirm the new policy appears in
    the results.

<img src="converted/Lab 3/media/media/image1b.png"
style="width:6.25in;height:3.95833in" />

<img src="converted/Lab 3/media/media/image1c.png" /><img src="converted/Lab 3/media/media/image1d.png" />

3.  Return to the policy and change **Enable policy** to **On**.

<img src="converted/Lab 3/media/media/image1e.png"
style="width:6.5in;height:3.32292in" />

4.  Select the **Report-only** tab and confirm the new policy appears in
    the results.\
    \
    <img src="converted/Lab 3/media/media/image1f.png"
    style="width:6.25in;height:3.98958in" />

**Note: Check your work against this list:**

- Is a pilot user prompted for MFA when signing in to Microsoft 365
  Copilot?

- Do the sign-in logs show the policy applied as Success?

## Exercise 4 — Deploy App Protection policies

Scenario: Copilot-generated content must not be copied into unmanaged
apps, even on personal phones. Zero Trust principles: Use least
privileged access, Assume breach. Estimated time: 7 minutes.

### **Task 1 — Create an App Protection policy for iOS/iPadOS**

1.  Sign in to the **Microsoft Intune admin center**
    ([intune.microsoft.com](https://intune.microsoft.com/auth/login/)).

<img src="converted/Lab 3/media/media/image20.png"
style="width:6.5in;height:2.28125in" />

2.  Go to **Apps \> Protection \> + Create policy \>Select
    iOS/iPadOS**.\
    <img src="converted/Lab 3/media/media/image21.png"
    style="width:6.25in;height:3.20833in" />

3.  On **Basics**, enter the name **APP – Copilot pilot – iOS**.

<img src="converted/Lab 3/media/media/image22.png"
style="width:6.5in;height:2.79167in" />

4.  On **Apps**, select **Core Microsoft Apps** and confirm Microsoft
    365 Copilot, Microsoft Teams, Microsoft Outlook, Microsoft Word, and
    Microsoft Edge are included.

<img src="converted/Lab 3/media/media/image23.png"
style="width:6.5in;height:3.94792in" />

5.  On **Data protection**, set:

    1.  Send org data to other apps: **Policy managed apps**

    2.  Restrict cut, copy, and paste between other apps: **Policy
        managed apps with paste in**

    3.  Save copies of org data: **Block**

<img src="converted/Lab 3/media/media/image24.png"
style="width:6.5in;height:3.9375in" />

6.  On **Access requirements**, require a **PIN for access.**

<img src="converted/Lab 3/media/media/image25.png"
style="width:6.5in;height:3.92708in" />

7.  On **Conditional launch**, set **Max PIN attempts** to **5** with
    the action **Reset PIN**, and **Jailbroken/rooted devices** to
    **Block access**.

<img src="converted/Lab 3/media/media/image26.png"
style="width:6.5in;height:3.9375in" />

8.  On **Assignments**, select **Copilot pilot users.**\
    <img src="converted/Lab 3/media/media/image27.png"
    style="width:6.25in;height:3.77604in" />

9.  Then select **Create**.

<img src="converted/Lab 3/media/media/image28.png"
style="width:6.5in;height:3.94792in" />

**Note: Check your work against this list:**

- Does the policy appear under Apps \> Protection with the Copilot pilot
  users group assigned?

- If you have a test phone: is the pilot user asked for an app PIN, and
  is a Copilot response blocked from pasting into a personal notes app?

## Exercise 5 — Deploy device management protection

Scenario: Only healthy, compliant devices should be able to reach
Copilot. Zero Trust principle: Verify explicitly. Estimated time: 8
minutes.

### **Task 1 — Create a Windows compliance policy**

1.  In the Microsoft Intune admin center, go to **Devices \> Compliance
    \> + Create policy \> Windows 10 and later**.

<img src="converted/Lab 3/media/media/image29.png"
style="width:6.5in;height:3.33333in" />

2.  Name the policy **Compliance – Windows – Copilot pilot**.

<img src="converted/Lab 3/media/media/image2a.png"
style="width:6.5in;height:5.85417in" />

<img src="converted/Lab 3/media/media/image2b.png"
style="width:6.5in;height:3.92708in" />

3.  On **Compliance settings**, set the following values:

    1.  Require BitLocker: **Require**

    2.  Require Secure Boot to be enabled on the device: **Require**

    3.  Firewall: **Require**

    4.  Microsoft Defender Antimalware: **Require**

<img src="converted/Lab 3/media/media/image2c.png"
style="width:6.5in;height:3.9375in" />

4.  Under **Actions for noncompliance**, keep **Mark device
    noncompliant** at **0 days**.

<img src="converted/Lab 3/media/media/image2d.png"
style="width:6.5in;height:2.21875in" />

5.  On **Assignments**, select **Copilot pilot users**, then select
    **Create**.

<img src="converted/Lab 3/media/media/image2e.png"
style="width:6.5in;height:3.07292in" /><img src="converted/Lab 3/media/media/image2f.png"
style="width:6.5in;height:3.94792in" />

### **Task 2 — Require a compliant device in Conditional Access**

1.  In the Microsoft Entra admin center, select Conditional Access.\
    <img src="converted/Lab 3/media/media/image30.png"
    style="width:6.25in;height:1.875in" />

2.  Paste the policy name as **Require compliant device for Microsoft
    365**. Under **Users**, include **Copilot pilot users** and exclude
    the **break-glass account**.

<img src="converted/Lab 3/media/media/image31.png"
style="width:6.5in;height:3.52083in" />

3.  Under **Grant**, select **Require device to be marked as
    compliant**, and set **Enable policy** to **Report-only**.

<img src="converted/Lab 3/media/media/image32.png"
style="width:6.5in;height:4.09375in" />

4.  In the Intune admin center, go to **Devices \> All devices** and see
    if the Windows 11 test device shows **Compliant**. If not, follow
    step 5.\
    <img src="converted/Lab 3/media/media/image33.png"
    style="width:6.25in;height:3.13542in" />

5.  Return to the policy and change **Enable policy** to **On**.

<img src="converted/Lab 3/media/media/image34.png"
style="width:6.5in;height:4.125in" />

**Note: Check your work against this list:**

- Does the test device show Compliant in Devices \> All devices? (A
  first check-in can take a few minutes.)

- Is a browser session from a non-enrolled device blocked from Microsoft
  365 Copilot?

## Exercise 6: Validate Copilot behavior

**Scenario:** You remove the temporary safeguard, confirm Copilot honors
every layer you configured, then license the protected pilot group.
**Estimated time:** 8 minutes.

### Task 1: Fix access and assign Copilot licenses

1.  Navigate to SharePoint site-
    <https://gpssandboxlabs100174.sharepoint.com/_layouts/15/sharepoint.aspx/discover>

<img src="converted/Lab 3/media/media/image35.png"
style="width:6.5in;height:3.51042in" />

2.  Go to Sites \> Active sites, open the Finance site.\
    <img src="converted/Lab 3/media/media/image36.png"
    style="width:4.40625in;height:3.48122in" />

3.  Open Settings icon\>Site Permissions.\
    <img src="converted/Lab 3/media/media/image37.png"
    style="width:6.25in;height:2.4375in" />

4.  Remove Everyone except external users and confirm Finance Members
    has access.\
    <img src="converted/Lab 3/media/media/image38.png"
    style="width:6.25in;height:1.94792in" />

5.  Turn off the temporary safeguard from Exercise 2:

powershell

Set-SPOSite -Identity
[https://\<tenant\>.sharepoint.com/sites/Finance](https://<tenant>.sharepoint.com/sites/Finance)
-RestrictContentOrgWideSearch \$false

<img src="converted/Lab 3/media/media/image39.png"
style="width:6.5in;height:1.64583in" />

6.  In the Microsoft 365 admin center (admin.microsoft.com), go to
    Billing \> Licenses \> Microsoft 365 E7.\
    <img src="converted/Lab 3/media/media/image3a.png"
    style="width:6.25in;height:2.97917in" />

7.  Select Assign licenses, choose CopilotPilot1 and Copilot pilot
    users.

<img src="converted/Lab 3/media/media/image3b.png"
style="width:6.5in;height:3.61458in" />

### Task 2: Test permission trimming

1.  Open the Microsoft 365 Copilot app.

2.  Paste the following promt: Summarize the salary review for 2026.

<img src="converted/Lab 3/media/media/image3c.png"
style="width:6.5in;height:2.59375in" />

3.  Confirm Copilot does not return content from
    Salary-review-2026.xlsx.

<img src="converted/Lab 3/media/media/image3d.png"
style="width:6.5in;height:1.16667in" />

4.  Sign in as CopilotPilot1 (a Finance member) and repeat the prompt.

5.  Confirm the response cites Salary-review-2026.xlsx and shows the
    Highly Confidential label.\
    <img src="converted/Lab 3/media/media/image3e.png"
    style="width:6.25in;height:3.97917in" />

Note: Permission and search changes can take a few minutes to propagate.
If CopilotPilot1 gets no results, wait and try again.

## Lab Summary

- You prepared a Microsoft 365 tenant for Copilot by applying all seven
  Zero Trust protection layers, with one representative control for
  each, and then confirmed that Copilot only surfaces data each user is
  allowed to see. The key takeaway: Copilot respects the permissions you
  already have, so fix oversharing and protect data before you license
  users.

- (Use the layer table from Exercise 1 as the quick-check. No need to
  repeat it here.)

- Clean up (optional): Switch the lab Conditional Access policies to Off
  if the tenant is shared, and remove Copilot licenses from the test
  accounts.
