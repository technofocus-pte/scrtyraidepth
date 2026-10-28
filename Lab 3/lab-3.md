# Lab 3: Apply principles of Zero Trust to Microsoft Copilots

### **Lab Overview**

In this hands-on lab, you secure a Microsoft 365 Copilot deployment by applying the three Zero Trust principles: verify explicitly, use least privileged access, and assume breach. You work through the seven protection layers in the order Microsoft recommends, deploying one representative control per layer, and then validate that Copilot only surfaces data a user is already allowed to see.

**Scenario:** Contoso Ltd. is rolling out Microsoft 365 Copilot to a pilot group. Before licenses are assigned, the security team must confirm that sensitive files are labeled, sign-ins are verified, apps and devices are managed, threats are detected, Teams collaboration is controlled, and oversharing is removed.

### **Learning Objectives**

By the end of this lab, you will be able to:

1.  **Explain** the three Zero Trust principles and how they apply to Microsoft 365 Copilot.

2.  **Describe** the seven protection layers and the order in which Microsoft recommends applying them before assigning Copilot licenses.

3.  **Deploy** one representative control for each layer: data protection, identity and access, App Protection, device management, threat protection, secure Teams collaboration, and minimum user permissions.

4.  **Remove** existing oversharing so each user has just enough access to the data they need.

5.  **Validate** that Copilot honors permissions, labels, and Conditional Access by testing an authorized and an unauthorized user.

6.  **Confirm** that Copilot interactions are recorded in Microsoft Purview Audit.

### **Lab Prerequisites**

- A **Microsoft 365 lab or trial tenant** with admin access. Never use a production tenant.

- **Microsoft 365 E3 (or Business Premium)** for Microsoft Entra ID P1, Microsoft Intune, Microsoft Defender for Office 365 Plan 1, and sensitivity labels. E5 is not required.

- **Microsoft 365 Copilot** licenses available for at least two pilot users (this also provides the SharePoint Advanced Management features used in Exercises 2 and 8)

- An admin account that holds, or can activate, these roles: **Global Administrator** (or the least-privileged set of Compliance, Conditional Access, Security, Intune, SharePoint, and Teams Administrator)

- Basic familiarity with Microsoft 365 services, identity, devices, and file permissions

- The lab accounts and data below, **created before the session** (not part of the lab time)

| **Account or object**                                                     | **Purpose**                                                                                                                                |
|---------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------|
| [admin@\<tenant\>.onmicrosoft.com](mailto:admin@<tenant>.onmicrosoft.com) | Lab administrator                                                                                                                          |
| CopilotPilot1 (user)                                                      | Member of **Copilot pilot users** and **Finance Members**. Will be licensed for Microsoft 365 Copilot.                                     |
| CopilotPilot2 (user)                                                      | Member of **Copilot pilot users** only; not a Finance member. Will be licensed for Microsoft 365 Copilot.                                  |
| Copilot pilot users (security group)                                      | Scopes Conditional Access, Intune, and label policies                                                                                      |
| Finance Members (security group)                                          | Will replace broad access on the Finance site. Contains CopilotPilot1.                                                                     |
| Finance (SharePoint site and Teams team)                                  | Team-connected site that is currently shared with **Everyone except external users**. Holds Salary-review-2026.xlsx with test salary data. |
| Break-glass account                                                       | Emergency access account excluded from all Conditional Access policies                                                                     |
| Windows 11 test device                                                    | Enrolled in Intune and signed in with a pilot account (used in Exercises 5 and 9)                                                          |

**Note:** Portal menus change often. If a label in a step does not match your screen, use the portal search box to find the same setting by name.

## Exercise 1 — Prepare for Microsoft Copilots using Zero Trust

Scenario: You need to understand what you are protecting and why, before you change any settings. Estimated time: 5 minutes.

### 

### **Task 1 — Review the Zero Trust principles**

Zero Trust treats every request as if it came from an uncontrolled network: never trust, always verify. Keep this table in mind for the rest of the lab.

| **Principle**                   | **Definition**                                                                                   | **How it applies to Copilot**                                                             |
|---------------------------------|--------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------|
| **Verify explicitly**           | Always authenticate and authorize based on all available data points.                            | Validate user credentials, device requirements, and app permissions and behaviors.        |
| **Use least privileged access** | Limit access with just-in-time and just-enough-access, risk-based policies, and data protection. | Remove oversharing; protect data with sensitivity labels and DLP policies.                |
| **Assume breach**               | Minimize blast radius, segment access, and use analytics to detect threats.                      | Use mailbox protection and Microsoft Defender to prevent, detect, and respond to attacks. |

### **Task 2 — Understand how Copilot uses your data**

A Copilot prompt passes through four components:

- **User devices** running Microsoft 365 apps, where users enter prompts

- The **Copilot service**, which orchestrates the response

- The **large language model (LLM)**, which generates the response

- **Microsoft Graph**, which retrieves your tenant’s data

**Note:** Copilot results contain only data the signed-in user is already allowed to access. Copilot does not create new permissions; it exposes existing ones faster. This is why oversharing must be fixed before licenses are assigned.

### **Task 3 — Map the seven protection layers**

Microsoft recommends applying the layers in this order. Each remaining exercise covers one layer.

| **Step** | **Protection layer**                     | **Zero Trust principle(s)**                    | **Exercise** |
|----------|------------------------------------------|------------------------------------------------|--------------|
| 1        | Data protection and compliance tools     | Verify explicitly, Use least privileged access | 2            |
| 2        | Identity and access policies             | Verify explicitly, Use least privileged access | 3            |
| 3        | App Protection policies                  | Use least privileged access, Assume breach     | 4            |
| 4        | Device management and protection         | Verify explicitly                              | 5            |
| 5        | Threat protection services               | Assume breach                                  | 6            |
| 6        | Secure collaboration for Microsoft Teams | Verify explicitly, Use least privileged access | 7            |
| 7        | Minimum user permissions to data         | Use least privileged access                    | 8            |

## Exercise 2 — Deploy data protection

Scenario: The Finance site holds sensitive salary data. You hide it from Copilot while you label and protect it. Zero Trust principles: Verify explicitly, Use least privileged access. Estimated time: 10 minutes.

### **Task 1 — Hide the Finance site from Copilot**

On your lab computer, open **SharePoint Online Management Shell** and run the following commands, replacing \<tenant\> with your tenant name:

1.  Paste the following command:  
    ***Connect-SPOService -Url <https://finance-admin.sharepoint.com>***

![Lab 3 screenshot 1](media/image-01.png)

2.  Paste the following command:  
      
    ***Set-SPOSite \`-Identity "<https://gpssandboxlabs100174.sharepoint.com/sites/finance>" -RestrictContentOrgWideSearch \$true***

![Lab 3 screenshot 2](media/image-02.png)

![Lab 3 screenshot 3](media/image-03.png)

**Note:** This setting (Restricted Content Discovery) hides the site from organization-wide search and Copilot, but it does not change permissions. It is a temporary safeguard while you clean up access. You turn it off in Exercise 8, once permissions are fixed.

### **Task 2 — Create, publish, and apply a sensitivity label**

1.  Sign in to the **Microsoft Purview portal** (purview.microsoft.com) as the lab administrator.  
    ![Lab 3 screenshot 4](media/image-04.png)

2.  Go to **Solutions \> Information Protection \> Sensitivity labels.**

> ![Lab 3 screenshot 5](media/image-05.png)

3.  Select **+ Create a label**.

![Lab 3 screenshot 6](media/image-06.png)

4.  Enter the following details:

    1.  Name: **Highly Confidential**

    2.  Display name: **Highly Confidential**

    3.  Description for users: **Restricted business data. Do not share outside the approved group.**

![Lab 3 screenshot 7](media/image-07.png)

5.  For **Scope**, select **Files & other data assets** and **Emails**.

![Lab 3 screenshot 8](media/image-08.png)

6.  Select **Control access**, then assign permissions to the **Copilot pilot users** group.

![Lab 3 screenshot 9](media/image-09.png)

![Lab 3 screenshot 10](media/image-10.png)

7.  Select **Create label**, then **Publish label**. In the label policy, publish to Finance **users**, name the policy **Copilot pilot label policy**, and select **Submit**.

![Lab 3 screenshot 11](media/image-11.png)

8.  Open **Salary-review-2026.xlsx** in the Finance site in Excel for the web.

![Lab 3 screenshot 12](media/image-12.png)

9.  Select **Sensitivity**, and apply **Highly Confidential**.  
    ![Lab 3 screenshot 13](media/image-13.png)

**Note:** A newly published label can take time to appear in Office apps. If it is not available yet, continue with the lab and apply it before Exercise 9. Also confirm that sensitivity labels are enabled for Office files in SharePoint and OneDrive, or Copilot cannot read labeled, encrypted files correctly.

### **Task 3 — Create a DLP policy in simulation mode**

1.  In the Microsoft Purview portal, go to **Solutions \> Data loss prevention.**  
    ![Lab 3 screenshot 14](media/image-14.png)

2.  Select **Policies**, then select **+ Create policy**.

![Lab 3 screenshot 15](media/image-15.png)

3.  Choose the **Custom** template and name the policy **DLP – Financial data**.  
    ![Lab 3 screenshot 16](media/image-16.png)

![Lab 3 screenshot 17](media/image-17.png)

4.  Select the locations **Exchange email**, **SharePoint sites**, **OneDrive accounts**, and **Teams chat and channel messages**.

![Lab 3 screenshot 18](media/image-18.png)

5.  Create a rule named- New_rule and Set the action to **Restrict access or encrypt the content.**  
    ![Lab 3 screenshot 19](media/image-19.png)

![Lab 3 screenshot 20](media/image-20.png)

6.  Select Submit.  
    ![Lab 3 screenshot 21](media/image-21.png)

7.  Select **Run the policy in simulation mode**, then submit the policy.  
    ![Lab 3 screenshot 22](media/image-22.png)

**Note: Check your work against this list:**

- Is the Highly Confidential label published to the Copilot pilot users group, and applied to Salary-review-2026.xlsx?

- Does DLP – Financial data show a status of simulation mode?

- Has the Finance site dropped out of organization-wide search results for a pilot user? (This can take a while to apply.)

## Exercise 3 — Deploy identity and access

Scenario: Attackers should not be able to use stolen credentials to find data quickly through Copilot. Zero Trust principles: Verify explicitly, Use least privileged access. Estimated time: 8 minutes.

### **Task 1 — Create a multifactor authentication policy**

1.  Sign in to the **Microsoft Entra admin center** (entra.microsoft.com) and go to **Entra ID \> Conditional Access \> Policies**.![Lab 3 screenshot 23](media/image-23.png)

2.  Select **+ Create new policy from templates**, then choose the **Secure foundation** category.

![Lab 3 screenshot 24](media/image-24.png)

3.  Select **Require multifactor authentication for all users**, then continue to review the policy.

![Lab 3 screenshot 25](media/image-25.png)

4.  Under **Users**, confirm **All users** is included and exclude the **break-glass account**.

![Lab 3 screenshot 26](media/image-26.png)

5.  Set **Enable policy** to **Report-only**, then create the policy.

![Lab 3 screenshot 27](media/image-27.png)

**Warning:** Always exclude at least one emergency access account from Conditional Access policies to avoid locking yourself out.

### **Task 2 — Review the impact and enforce the policy**

1.  Go to **Entra ID \> Monitoring & health \> Sign-in logs** and open a recent sign-in by a pilot user.

![Lab 3 screenshot 28](media/image-28.png)

2.  Select the **Report-only** tab and confirm the new policy appears in the results.

![Lab 3 screenshot 29](media/image-29.png)

![Lab 3 screenshot 30](media/image-30.png)![Lab 3 screenshot 30](media/image-30.png)

3.  Return to the policy and change **Enable policy** to **On**.

![Lab 3 screenshot 31](media/image-31.png)

4.  Select the **Report-only** tab and confirm the new policy appears in the results.  
      
    ![Lab 3 screenshot 32](media/image-32.png)

**Note: Check your work against this list:**

- Is a pilot user prompted for MFA when signing in to Microsoft 365 Copilot?

- Do the sign-in logs show the policy applied as Success?

## Exercise 4 — Deploy App Protection policies

Scenario: Copilot-generated content must not be copied into unmanaged apps, even on personal phones. Zero Trust principles: Use least privileged access, Assume breach. Estimated time: 7 minutes.

### **Task 1 — Create an App Protection policy for iOS/iPadOS**

1.  Sign in to the **Microsoft Intune admin center** ([intune.microsoft.com](https://intune.microsoft.com/auth/login/)).

![Lab 3 screenshot 33](media/image-33.png)

2.  Go to **Apps \> Protection \> + Create policy \>Select iOS/iPadOS**.  
    ![Lab 3 screenshot 34](media/image-34.png)

3.  On **Basics**, enter the name **APP – Copilot pilot – iOS**.

![Lab 3 screenshot 35](media/image-35.png)

4.  On **Apps**, select **Core Microsoft Apps** and confirm Microsoft 365 Copilot, Microsoft Teams, Microsoft Outlook, Microsoft Word, and Microsoft Edge are included.

![Lab 3 screenshot 36](media/image-36.png)

5.  On **Data protection**, set:

    1.  Send org data to other apps: **Policy managed apps**

    2.  Restrict cut, copy, and paste between other apps: **Policy managed apps with paste in**

    3.  Save copies of org data: **Block**

![Lab 3 screenshot 37](media/image-37.png)

6.  On **Access requirements**, require a **PIN for access.**

![Lab 3 screenshot 38](media/image-38.png)

7.  On **Conditional launch**, set **Max PIN attempts** to **5** with the action **Reset PIN**, and **Jailbroken/rooted devices** to **Block access**.

![Lab 3 screenshot 39](media/image-39.png)

8.  On **Assignments**, select **Copilot pilot users.**  
    ![Lab 3 screenshot 40](media/image-40.png)

9.  Then select **Create**.

![Lab 3 screenshot 41](media/image-41.png)

**Note: Check your work against this list:**

- Does the policy appear under Apps \> Protection with the Copilot pilot users group assigned?

- If you have a test phone: is the pilot user asked for an app PIN, and is a Copilot response blocked from pasting into a personal notes app?

## Exercise 5 — Deploy device management protection

Scenario: Only healthy, compliant devices should be able to reach Copilot. Zero Trust principle: Verify explicitly. Estimated time: 8 minutes.

### **Task 1 — Create a Windows compliance policy**

1.  In the Microsoft Intune admin center, go to **Devices \> Compliance \> + Create policy \> Windows 10 and later**.

![Lab 3 screenshot 42](media/image-42.png)

2.  Name the policy **Compliance – Windows – Copilot pilot**.

![Lab 3 screenshot 43](media/image-43.png)

![Lab 3 screenshot 44](media/image-44.png)

3.  On **Compliance settings**, set the following values:

    1.  Require BitLocker: **Require**

    2.  Require Secure Boot to be enabled on the device: **Require**

    3.  Firewall: **Require**

    4.  Microsoft Defender Antimalware: **Require**

![Lab 3 screenshot 45](media/image-45.png)

4.  Under **Actions for noncompliance**, keep **Mark device noncompliant** at **0 days**.

![Lab 3 screenshot 46](media/image-46.png)

5.  On **Assignments**, select **Copilot pilot users**, then select **Create**.

![Lab 3 screenshot 47](media/image-47.png)![Lab 3 screenshot 48](media/image-48.png)

### **Task 2 — Require a compliant device in Conditional Access**

1.  In the Microsoft Entra admin center, select Conditional Access.  
    ![Lab 3 screenshot 49](media/image-49.png)

2.  Paste the policy name as **Require compliant device for Microsoft 365**. Under **Users**, include **Copilot pilot users** and exclude the **break-glass account**.

![Lab 3 screenshot 50](media/image-50.png)

3.  Under **Grant**, select **Require device to be marked as compliant**, and set **Enable policy** to **Report-only**.

![Lab 3 screenshot 51](media/image-51.png)

4.  In the Intune admin center, go to **Devices \> All devices** and see if the Windows 11 test device shows **Compliant**. If not, follow step 5.  
    ![Lab 3 screenshot 52](media/image-52.png)

5.  Return to the policy and change **Enable policy** to **On**.

![Lab 3 screenshot 53](media/image-53.png)

**Note: Check your work against this list:**

- Does the test device show Compliant in Devices \> All devices? (A first check-in can take a few minutes.)

- Is a browser session from a non-enrolled device blocked from Microsoft 365 Copilot?

## Exercise 6: Validate Copilot behavior

**Scenario:** You remove the temporary safeguard, confirm Copilot honors every layer you configured, then license the protected pilot group. **Estimated time:** 8 minutes.

### Task 1: Fix access and assign Copilot licenses

1.  Navigate to SharePoint site- <https://gpssandboxlabs100174.sharepoint.com/_layouts/15/sharepoint.aspx/discover>

![Lab 3 screenshot 54](media/image-54.png)

2.  Go to Sites \> Active sites, open the Finance site.  
    ![Lab 3 screenshot 55](media/image-55.png)

3.  Open Settings icon\>Site Permissions.  
    ![Lab 3 screenshot 56](media/image-56.png)

4.  Remove Everyone except external users and confirm Finance Members has access.  
    ![Lab 3 screenshot 57](media/image-57.png)

5.  Turn off the temporary safeguard from Exercise 2:

powershell

Set-SPOSite -Identity [https://\<tenant\>.sharepoint.com/sites/Finance](https://<tenant>.sharepoint.com/sites/Finance) -RestrictContentOrgWideSearch \$false

![Lab 3 screenshot 58](media/image-58.png)

6.  In the Microsoft 365 admin center (admin.microsoft.com), go to Billing \> Licenses \> Microsoft 365 E7.  
    ![Lab 3 screenshot 59](media/image-59.png)

7.  Select Assign licenses, choose CopilotPilot1 and Copilot pilot users.

![Lab 3 screenshot 60](media/image-60.png)

### Task 2: Test permission trimming

1.  Open the Microsoft 365 Copilot app.

2.  Paste the following promt: Summarize the salary review for 2026.

![Lab 3 screenshot 61](media/image-61.png)

3.  Confirm Copilot does not return content from Salary-review-2026.xlsx.

![Lab 3 screenshot 62](media/image-62.png)

4.  Sign in as CopilotPilot1 (a Finance member) and repeat the prompt.

5.  Confirm the response cites Salary-review-2026.xlsx and shows the Highly Confidential label.  
    ![Lab 3 screenshot 63](media/image-63.png)

Note: Permission and search changes can take a few minutes to propagate. If CopilotPilot1 gets no results, wait and try again.

## Lab Summary

- You prepared a Microsoft 365 tenant for Copilot by applying all seven Zero Trust protection layers, with one representative control for each, and then confirmed that Copilot only surfaces data each user is allowed to see. The key takeaway: Copilot respects the permissions you already have, so fix oversharing and protect data before you license users.

- (Use the layer table from Exercise 1 as the quick-check. No need to repeat it here.)

- Clean up (optional): Switch the lab Conditional Access policies to Off if the tenant is shared, and remove Copilot licenses from the test accounts.
