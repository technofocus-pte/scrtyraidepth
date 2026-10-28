**Lab 1: Protect organizational data with sensitivity labels**

Lab Duration: 50 minutes

**Lab overview**

In this lab you build a working sensitivity-label deployment for Contoso. You enable sensitivity labels for SharePoint and OneDrive, create a label, a label group and an encrypted label for the Finance team, organize the labels, and publish them to users with a label policy.

**Scenario.** You are an Enterprise Administrator at Contoso. Leadership wants confidential finance data protected wherever it travels, without blocking everyday collaboration. You set up the labels and publish them in a pilot policy before an organization-wide rollout.

**Objectives.** By the end of this lab you will be able to:

- Define a simple label taxonomy for Contoso.

- Enable sensitivity labels in SharePoint Online and OneDrive so they can process encrypted files.

- Create a sensitivity label, a label group and a label with access control (encryption).

- Organize labels by moving a label into a label group and editing an existing label.

- Publish labels to users by creating a label publishing policy.

# Prerequisites

This lab runs in a provided lab environment, so you do not need your own Microsoft 365 tenant. Confirm the following before you start.

| **Requirement**   | **Detail**                                                                                                                                                                                                                                            |
|-------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Lab environment   | A running lab with a Windows virtual machine and the **Environment** panel on the right. The **Virtual Machine** and **Web Connect Gateway** must both show **Running** on the **Environment Status** page.                                           |
| Credentials       | The **Username** and **Temporary Access Pass** shown on the **Environment** tab of the lab panel. The Temporary Access Pass is used instead of a password.                                                                                            |
| Admin permissions | The lab account is a Global Administrator, so it can open the Microsoft Purview portal, create labels and publish them. In your own tenant, you need Global Administrator or a role such as Information Protection Admin or Compliance Administrator. |
| SharePoint admin  | SharePoint Administrator or Global Administrator, needed for the optional PowerShell task in Exercise 2.                                                                                                                                              |
| Licensing         | A subscription that includes sensitivity labels, such as Microsoft 365 E3 or E5. Encryption (access control) needs Azure Rights Management to be active for the tenant.                                                                               |
| Browser           | Microsoft Edge on the lab virtual machine.                                                                                                                                                                                                            |
| PowerShell        | Windows PowerShell with the SharePoint Online Management Shell module (Microsoft.Online.SharePoint.PowerShell), already available on the lab VM.                                                                                                      |

> **Note:** Replace tenant-specific values shown in screenshots, such as the user name and the SharePoint admin URL, with the values from your own lab panel. Screenshots in this lab hide those values.

# Exercise 1: Plan the label taxonomy

Before you create anything, decide which labels Contoso needs. Microsoft recommends starting small: pick one or two high-impact scenarios, build labels for them, validate, then expand.

## Task 1: Review the pilot taxonomy

1.  Open a blank notepad to record your decisions.

2.  Write down the pilot scenarios: every document and email carries a label, and finance data is encrypted so only the Finance team can open it.

3.  Record this taxonomy. You build it in Exercise 3.

| **Item**               | **Type**                            | **Who can open it**                                  | **Protection**                                                                   |
|------------------------|-------------------------------------|------------------------------------------------------|----------------------------------------------------------------------------------|
| General                | Label                               | Anyone in Contoso                                    | None                                                                             |
| Confidential           | Label group                         | Not applicable. A label group only organizes labels. | None. A label group has no protection settings and cannot be applied to content. |
| Confidential / Finance | Label inside the Confidential group | Users and groups you assign (Co-Author)              | Encryption through access control                                                |

Labels lower in the list carry higher sensitivity. Purview uses this order to warn users who downgrade a label.

> **Checkpoint:** You have a three-row taxonomy written down. Keep these notes open for Exercises 3 and 4.

# Exercise 2: Sign in and enable sensitivity labels for SharePoint and OneDrive

This exercise signs you in to the Microsoft Purview portal and lets SharePoint and OneDrive read labeled and encrypted Office files. That keeps coauthoring, search and eDiscovery working on protected content. Complete Task 2 **or** Task 3. The change takes about 15 minutes to take effect.

## Task 1: Sign in to the Microsoft Purview portal

1.  On the **Sign in** page, paste the **Username** and select **Next**.

![image1](media/image1.png)

2.  On **Enter Temporary Access Pass**, paste the **Temporary Access Pass** and select **Sign in**.

![image2](media/image2.png)

3.  If the **Welcome to the new Microsoft Purview portal!** dialog appears, select **Get started**.

![welcome](media/image3.png)

4.  In the left navigation, select **Solutions**, and then select **Information Protection**.

![image4](media/image4.png)

5.  In the **Information Protection** navigation, select **Sensitivity labels**.

![image5](media/image5.png)

## Task 2: Enable from the Microsoft Purview portal

1.  On the **Sensitivity labels** page, find the banner that reads “Your organization has not turned on the ability to process content in Office online files that have encrypted sensitivity labels applied…”. Select **Turn on now**.

![turnon](media/image6.png)

2.  Confirm the prompt. The banner disappears when the setting is saved.

> **Note:** If your tenant uses Microsoft 365 Multi-Geo, use Task 3 (PowerShell) and repeat it for every geo-location.

## Task 3: Enable with SharePoint Online PowerShell (alternative)

Use this option if you have Multi-Geo or prefer scripting. You need SharePoint Online Management Shell version 16.0.19418.12000 or later.

1.  On the lab virtual machine, select **Start**, type PowerShell, right-click **Windows PowerShell**, and select **Run as administrator**.

![ps_search](media/image7.png)

2.  An **Administrator: Windows PowerShell** window opens.

![ps_open](media/image8.png)

3.  Update the module if needed. Then connect with your admin account, replacing the tenant name with your own. The screenshot shows contoso as a placeholder.

```powershell
Update-Module -Name Microsoft.Online.SharePoint.PowerShell

Connect-SPOService -Url https://<your-tenant>-admin.sharepoint.com -UseSystemBrowser $true
```

![ps_connect](media/image9.png)

4.  When the **Pick an account** window opens, select the lab account to finish signing in, and then return to PowerShell.

![image10](media/image10.png)

5.  Run this command to turn on the integration.

```powershell
Set-SPOTenant -EnableAIPIntegration $true
```

![ps_set](media/image11.png)

6.  At the **Confirm** prompt, type **Y** and press **Enter**.

![ps_yes](media/image12.png)

7.  Wait about 15 minutes before you continue.

> **Checkpoint:** You are signed in to Microsoft Purview and the SharePoint and OneDrive setting is on.

# Exercise 3: Create and organize sensitivity labels

You create three items from your Exercise 1 taxonomy: the **General** label, the **Confidential** label group, and the **Finance** label. Then you move Finance into the Confidential group. The wizard page names below match the current Microsoft Purview portal.

## Task 1: Create the General label

1.  On the **Sensitivity labels** page, select **Create**.

![e3_labels_empty](media/image13.png)

2.  Select **Label**.

![e3_create_menu](media/image14.png)

3.  On **Provide basic details for this label**, enter the values below, and then select **Next**. Leave **Label priority** as **Highest**.

| **Field**              | **Value**                                              |
|------------------------|--------------------------------------------------------|
| Name                   | General                                                |
| Display name           | General                                                |
| Description for users  | Business data that is not meant for public consumption |
| Description for admins | Lab label for internal, non-restricted content         |

![image15](media/image15.png)

4.  On **Define the scope for this label**, leave **Files & other data assets**, **Emails** and **Meetings** selected, and then select **Next**.

![gen_scope](media/image16.png)

5.  On **Choose protection settings for the types of items you selected**, leave every option cleared (no access control, no content marking), and then select **Next**.

![gen_items](media/image17.png)

6.  On **Auto-labeling for files and emails**, leave the toggle off, and then select **Next**.

![gen_auto](media/image18.png)

7.  On **Define protection settings for groups and sites**, leave every option cleared, and then select **Next**.

![gen_groups](media/image19.png)

8.  On **Review your settings and finish**, confirm **Access control** and **Content marking** show **None**, and then select **Create label**.

![gen_review](media/image20.png)

9.  On **Your sensitivity label was created**, keep **Don’t create a policy yet** selected, and then select **Done**. You publish the labels in Exercise 4.

![image21](media/image21.png)

## Task 2: Create the Confidential label group

A label group organizes related labels. Unlike a label, a group has no protection settings and cannot be applied to files, emails or sites.

1.  On the **Sensitivity labels** page, select **Create**, and then select **Label group**.

![cg_menu](media/image22.png)

2.  On the label group details page, enter the values below, and then select **Next**.

| **Field**             | **Value**                                                                         |
|-----------------------|-----------------------------------------------------------------------------------|
| Name                  | Confidential                                                                      |
| Display name          | Confidential                                                                      |
| Description for users | Sensitive business data that could cause harm if shared with unauthorized people. |

![image23](media/image23.png)

3.  On **Review your settings and finish**, select **Create label group**.

![cg_review](media/image24.png)

4.  On **Your label group was created successfully**, select **Don’t create a label yet**, and then select **Done**. You create the label for this group in the next task.

![image25](media/image25.png)

## Task 3: Create the Finance label with access control

The Finance label encrypts content so that only the users and groups you assign can open it.

1.  Select **Create**, and then select **Label**.

2.  On the label details page, enter the values below, and then select **Next**.

| **Field**             | **Value**                                                             |
|-----------------------|-----------------------------------------------------------------------|
| Name                  | Finance                                                               |
| Display name          | Finance                                                               |
| Description for users | Confidential finance data. Only the Finance Team can open or edit it. |

3.  On **Define the scope for this label**, leave **Files & other data assets**, **Emails** and **Meetings** selected, and then select **Next**.

4.  On **Choose protection settings for the types of items you selected**, select **Control access**, and then select **Next**.

![fin_items](media/image26.png)

5.  On the access control page, leave **User access to content expires** set to **Never** and **Allow offline access** set to **Always**. Under **Assign permissions to specific users and groups**, select **Assign permissions**. Add the **Contoso** group, all users in your organization, and your lab user account, and give each one the **Co-Author** permission. Then select **Next**.

![image27](media/image27.png)

6.  On **Auto-labeling for files and emails** and on **Define protection settings for groups and sites**, leave the defaults and select **Next** on each page.

7.  On **Review your settings and finish**, confirm **Access control** is listed, and then select **Create label**.

![fin_review](media/image28.png)

8.  On the final page, keep **Don’t create a policy yet** selected, and then select **Done**.

> **Troubleshooting:** If **Create label** returns a **Client Error** that says “Rights Management is not active for the tenant”, select **OK** and see the Troubleshooting section at the end of this lab. Fix it and then select **Create label** again.

## Task 4: Move the Finance label into the Confidential group

1.  On the **Sensitivity labels** page, select the check box next to **Finance**, and then select **⋮** (more actions) next to it.

2.  Select **Move into group**. The menu looks like the one below.

![move_menu](media/image29.png)

3.  In **Move the label into a group**, select **Confidential** from the list, and then confirm. The label becomes the highest-priority label in that group.

4.  Expand **Confidential** by selecting its arrow. **Finance** now appears inside the group and is shown as **Confidential/Finance** when you publish.

![after_move](media/image30.png)

> **Tip:** Use **⋮ \> Move up**, **Move down** or **Assign priority** to reorder labels. Labels lower in the list have higher sensitivity.

## Task 5: Edit a label

1.  Select the check box next to **General**, select **⋮**, and then select **Edit label**.

![edit_menu](media/image31.png)

2.  On **Edit sensitivity label**, update **Description for users** to Business data that is not meant for public consumption. The **Name** field cannot be changed. Select **Next** on each page until you reach the review page.

![image32](media/image32.png)

3.  Select **Save label**.

![edit_review](media/image33.png)

4.  Select **Refresh**. The **Last modified** time for **General** updates.

![image34](media/image34.png)

> **Checkpoint:** You have a General label and a Confidential group that contains a Finance label with access control. Nothing is published yet.

# Exercise 4: Publish labels with a label policy

Labels are not visible to users until you publish them. A label publishing policy chooses which labels appear, who sees them, and which settings apply.

## Task 1: Open the label publishing policies page

1.  In the **Information Protection** navigation, expand **Policies** and select **Label publishing policies**.

![pub_nav](media/image35.png)

2.  Select **Publish label**.

![pub_page](media/image36.png)

## Task 2: Create the policy

1.  On **Choose sensitivity labels to publish**, select **Choose sensitivity labels to publish**.

![image37](media/image37.png)

2.  Select **General** and **Confidential/Finance**, and then select **Add**. Publishing a label inside a group also publishes its parent group automatically.

![pub_pick](media/image38.png)

3.  Confirm both labels are listed, and then select **Next**.

![pub_chosen](media/image39.png)

4.  On **Admin units**, leave the default and select **Next**.

5.  On **Publish to users and groups**, leave **Exchange email** with **All users & groups** selected, and then select **Next**.

![pub_users](media/image40.png)

6.  On **Policy settings**, leave the four options selected: **Users must provide a justification to remove a label or lower its classification**, **Require users to apply a label to their emails and documents**, **Require users to apply a label to their Fabric and Power BI content**, and **Provide users with a link to a custom help page**.

![image41](media/image41.png)

7.  In the help page box, enter your tenant’s URL, for example `https://<your-tenant>-admin.sharepoint.com`, and then select **Next**.

![image42](media/image42.png)

8.  On **Name your policy**, enter Contoso Pilot Labels as the **Name**. A description is optional. Select **Next**.

![pub_name](media/image43.png)

9.  On **Review and finish**, confirm the labels, users and settings, and then select **Submit**.

![pub_review](media/image44.png)

10. On **New policy created**, select **Done**.

![pub_done](media/image45.png)

> **Note:** It can take up to 24 hours for the labels to appear in users’ apps. Because **Require users to apply a label** is on, users must pick a label before they save a document or send an email.
>
> **Checkpoint:** The **Contoso Pilot Labels** policy is created and published.

# Exercise 5: Apply and test labels in Office

This exercise proves the pilot works. Office asks you to label a document before you edit it; the label marking appears on the page; you can switch labels; and the label also shows in Outlook and OneDrive. You use the lab account for every task.

> **Note:** Labels can take up to 24 hours to appear in users’ apps. If the Sensitivity button or the labeling prompt does not appear, wait, and then sign out and back in to Office.

## Task 1: Respond to the mandatory labeling prompt

The Contoso Pilot Labels policy requires users to apply a label, and it does not set a default label. Word therefore asks you to choose one before you can edit.

1.  In Microsoft Edge, go to <https://www.office.com>, sign in with the lab account if prompted, and open **Word**.

2.  Select **Create blank document**.

![e5 word new](media/image46.png)

3.  A yellow banner reads **Add sensitivity label – Your organization requires you to label this document before editing**. Select **Select label**.

![e5 mandatory banner](media/image47.png)

4.  In the **Add sensitivity label** dialog, notice that no label is chosen and **OK** is unavailable until you pick one.

![e5 mandatory dialog](media/image48.png)

5.  Open the list, select **General**, and then select **OK**.

![e5 mandatory general](media/image49.png)

6.  Type a line of text in the document, for example, Hello. This is a new document for Contoso.

7.  Scroll to the bottom of the page. The footer shows the label marking, **Contoso – General**.

![e5 footer general](media/image50.png)

> **Checkpoint:** The document is labeled General and the General marking appears in the footer.

## Task 2: Switch to the encrypted Finance label

Only published labels appear in the Sensitivity menu. The Confidential label group cannot be applied to a document, so you choose the Finance label inside it.

1.  On the **Home** tab, select **Sensitivity**. The menu lists **General** and **Confidential**.

2.  Point to **Confidential**. The **Finance** label opens beside it.

![e5 menu](media/image51.png)

3.  Select **Finance**.

4.  Scroll to the bottom of the page. The footer now shows **Contoso – Confidential**.

![e5 footer confidential](media/image52.png)

5.  Name the file **Falcon-Plan** so it is saved to your OneDrive as Falcon-Plan.docx.

## Task 4: Label an email in Outlook

1.  Open Outlook on the web from the app launcher, and then select **New mail**.

2.  In the top-right corner of the new message, find the **Sensitivity** shield button.

![e5 outlook new](media/image53.png)

3.  Select the arrow next to the shield. The menu lists **General** and **Confidential**.

![e5 outlook menu](media/image54.png)

4.  Select **General**. Pause on the label to see its description for users.

5.  Address the message to your own lab account, enter a subject and a short message, and then select **Send**.

6.  Open the message from your **Inbox**. The label **General** appears beside the subject at the top of the message.

![e5 outlook received](media/image55.png)

> OneDrive integration all behave as planned.

# Exercise 6: Remove and delete sensitivity labels

Removing a label from a policy is the safe, reversible way to retire it. Deleting a label is permanent. In this exercise, you remove the Finance label from the Contoso Pilot Labels policy, and then you create and delete a throwaway label to see the difference.

| **Action**               | **Users can still pick it?**  | **Existing labeled content**                                                                                                                      | **Reversible?**                |
|--------------------------|-------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------|
| Remove from label policy | No (for users in that policy) | Keeps the label and any protection; Office still shows the label name                                                                             | Yes, add it back to the policy |
| Delete the label         | No (no one)                   | Keeps encryption, but apps no longer show the label name, and label-based features (DLP conditions, auto-labeling, reporting) stop recognizing it | No                             |

## Task 1: Remove a label from the policy

1.  In the Microsoft Purview portal, select **Information Protection** \> **Policies** \> **Label publishing policies**.

2.  Select the check box next to **Contoso Pilot Labels**, select **⋮** (more actions), and then select **Edit policy**.

![e6 edit policy](media/image56.png)

3.  On **Choose sensitivity labels to publish**, the policy lists General, Confidential and Confidential/Finance. Select **Edit**. Clear **Finance**.

![e6 edit labels](media/image57.png)

5.  Select **Next** through the remaining pages without changes, and then select **Submit**.

6.  After replication (up to 24 hours), sign in to Word and open the **Sensitivity** menu. **Finance** no longer appears. Falcon-Plan.docx still shows the label you applied earlier.

## Task 2: Create a test label to delete

1.  In the Information Protection navigation, select **Sensitivity labels**. The list shows General and Confidential.

![e6 create start](media/image58.png)

2.  Select **Create**, and then select **Label**.

3.  On **Provide basic details for this label**, enter **Test – Delete Me** as the Name, the Display name and the Description for users. Leave **Label priority** as **Highest**, and then select **Next**.

![e6 create details](media/image59.png)

4.  On **Define the scope for this label**, leave the defaults selected (Files & other data assets, Emails and Meetings). If the **Extend labels beyond Microsoft 365** pop-up appears, close it. Select **Next**.

![e6 create scope](media/image60.png)

5.  On each remaining page, leave every option cleared and select **Next**.

6.  On **Review your settings and finish**, confirm that **Access control** and **Content marking** show **None**, and then select **Create label**.

![e6 create review](media/image61.png)

8.  Confirm **Test – Delete Me** now appears in the list.

![e6 create list](media/image62.png)

> **Note:** Do not publish this label. It exists only so that you can delete it in the next task.

## Task 3: Delete the test label

1.  Select the check box next to **Test – Delete Me**, select **⋮** (more actions), and then select **Delete label**. You can also select **Delete label** on the toolbar.

![e6 delete menu](media/image63.png)

2.  In the **Delete ‘Test – Delete Me’** dialog, select **Delete**. The dialog warns that you cannot undo the delete. Select **Delete** and not the highlighted **Cancel** button.

![e6 delete confirm](media/image64.png)

3.  Select **Refresh**. Confirm that the label no longer appears in the list.

![image65](media/image65.png)

> **Checkpoint:** You can explain why Contoso should normally remove labels from policies rather than delete them.

**Lab Summary**

- A label group only organizes labels. It has no protection settings and can’t be applied to content.

- Labels aren’t visible to users until they’re published in a policy.

- Publishing a label inside a group also publishes its parent group.

- Encrypted labels require SharePoint/OneDrive integration to be enabled and Rights Management to be active.

- Normally you should remove labels from policies rather than delete them, because removal is reversible and deletion is not.

**Troubleshooting:** The lab mentions a “Rights Management is not active for the tenant” error when creating the Finance label, and refers to a Troubleshooting section at the end, but that section isn’t in the document you uploaded
