
## **Windows Local Administrator Password Solutions(LAPS) With Intune**

What is Windows **LAPS**?

Windows Local Administrator Password Solution (Windows LAPS) is a Windows feature that automatically manages and backs up the password of a local administrator account on your Microsoft Entra joined or Windows Server Active Directory-joined devices.

**Configuring Windows LAPS**

There are a couple of ways to configure LAPS in the Intune admin portal including using the Endpoint security blade and Configuration sections using OMA-URI for LAPS.

**Method 1**: Using Intune Endpoint security

**Step 1**: You must first enable the LAPS feature at the Microsoft Entra tenant level
Sign in to the Microsoft Entra admin center as at least a Cloud Device Administrator.
Browse to **Identity** > **Devices** > **Overview** > D**evice settings**
Select Yes for the Enable Local Administrator Password Solution (LAPS) setting, then select **Save**.
 
**Step 2**: Login to the Intune admin portal with the right access.
Select the **Endpoint security** > **Account Protection** > **Create Policy** > **Windows platform** > **Windows LAPS**

![image](https://github.com/user-attachments/assets/aa6806e2-088e-41af-9938-c809f2c09e91)
_Figure 1_

Fill out the Configuration in accordance with your organization’s security policies. Assign this to a device group.

![Screenshot 2024-11-16 142619](https://github.com/user-attachments/assets/7c9e5f9b-89f6-4b8f-b4ac-6053681d9cc3)
_Figure 2_

### Method 2: Windows LAPS CSP

The Local Administrator Password Solution (LAPS) configuration service provider (CSP) is used by the enterprise to manage back up of local administrator account passwords. The link [LAPS CSP](https://learn.microsoft.com/en-us/windows/client-management/mdm/laps-csp#policiespasswordcomplexity/) contains a list of all OMA-URI policies that can be configured for LAPS. There are more customizations in using the OMA-URI settings over the Endpoint security in Method 1 above. The following policies are configured and assigned to device groups not user groups.

**Step 1**: Login to Intune admin portal > **Devices** >**Configuration** > **Create**<br />
Platform : Windows 10 or later<br />
Profile: Templates > Select **Custom**<br />
Name: Name the policy<br />
Click Next to the Configuration and select **Add** as shown in Figure 3 below

![image](https://github.com/user-attachments/assets/784029f2-18e9-4451-b801-deaac4cdbfc9)
_Figure 3_

Set up the OMA-URI settings as required following the guidelines highlighted below. The settings are arranged in order of dependency starting with the independent settings. Note that some of the settings works only on Windows 11 24H2.

![Screenshot 2024-11-17 095525](https://github.com/user-attachments/assets/58d4d420-b29e-4ad5-b3d2-4e02434e5f56)
_Figure 4_

![image](https://github.com/user-attachments/assets/e73a89bc-dea9-4de8-a4f4-2041e372e39d) <br />
_Figure 5_

![image](https://github.com/user-attachments/assets/08dba39b-cd41-4320-9f91-a0dd289c85a5) <br />
_Figure 6_ : OMA-URI settings

## **Result**: <br />
To verify if the device received the policy, go to **Event Viewer** on the local device > **Applications and Services Logs** > **Microsoft** > **Windows** > **LAPS** > Double-click on **Operational**<br />
Find **Event ID 10022** to see all the settings you pushed through Intune. See image below. 

![image](https://github.com/user-attachments/assets/66bc2fba-9244-4e5b-9ff9-1a4a61997853) <br />
_Figure 7_:Event Viewer

Also on the local device, check “Computer Management” > users & groups to see the new _LapsAdmin_ user account created. You can also verify in the Entra portal under the Devices > Local administrator password recovery.

#### More details about the OMA-URI settings - [LAPS CSP](https://learn.microsoft.com/en-us/windows/client-management/mdm/laps-csp#policiespasswordcomplexity/)
The following settings are arranged based on the dependency status. You may want to configure the _Independent settings_ first.

**Independent settings**<br />
**1. BackupDirectory:** Used to configure which directory the local admin account password is saved.

OMA-URI<br />
	**./Device/Vendor/MSFT/LAPS/Policies/BackupDirectory**
 
Applicable OS: Windows 10 and later<br />
Format: integer, value range of 0-2<br />
0=Disabled (password won't be backed up)<br />
1=Backup the password to Microsoft Entra ID only <br />
2=Backup the password to Active Directory only.<br />
Default value: 0<br />

**2. PasswordComplexity:** Used to configure password complexity of the managed local administrator account.

OMA-URI:<br />
  **./Device/Vendor/MSFT/LAPS/Policies/PasswordComplexity**		
  
Format: integer, range of 1-8 with each number representing complexity level<br />
Default value is 4 = Large letters + small letters + numbers + special characters and recommended by Microsoft<br />
7=Passphrase (short words)<br />
Dependency: 	None

**3. AutomaticAccountManagementEnabled:** Use this setting to specify whether automatic account management is enabled. If this setting is enabled, the [AutomaticAccountManagementTarget] will be automatically managed.

OMA-URI<br />
	**./Device/Vendor/MSFT/LAPS/Policies/AutomaticAccountManagementEnabled**

Applicable OS:  Windows 11, version 24H2 [10.0.26100] and later<br />
Format: bool<br />
Default value: False<br />

**Dependent settings**

**4. PasswordAgeDays:** Maximum password age of the managed local administrator account.

OMA-URI: <br />
  **./Device/Vendor/MSFT/LAPS/Policies/PasswordAgeDays**<br />
Applicable OS: Windows 10 and later<br />
Format: integer, range of [1-365] days<br />
Default value: 30 days<br />
Minimum 1 day when backing up to on-prem Active Directory<br />
Minimum 7 days when backing up to Azure AD<br />
Dependency: [BackupDirectoryAADMode/BackupDirectoryADMode]

**5. PassphraseLength**: Used to configure the number of passphrase words.

OMA-URI:<br />
  **./Device/Vendor/MSFT/LAPS/Policies/PassphraseLength**	<br />
Applicable OS: Windows 11, version 24H2 [10.0.26100] and later<br />
Format: integer, value range of 3-10 words<br />
Default value: 6 words<br />
Dependency: [PasswordComplexity]<br />
Dependency Allowed Value: [6-8]<br />

**Settings Dependent on [AutomaticAccountManagementEnabled]:**

**6. AutomaticAccountManagementTarget**: Used this setting to configure which account is automatically managed

OMA-URI:<br />
**./Device/Vendor/MSFT/LAPS/Policies/AutomaticAccountManagementTarget**

Applicable OS: Windows 11, version 24H2 [10.0.26100] and later<br />
Format: integer, value range of 0-1<br />
0=The builtin administrator account will be managed.<br />
1=A new account created by Windows LAPS will be managed.<br />
Default value: 1<br />
Dependency: [AutomaticAccountManagementEnabled], must be set to true<br />

**7. AutomaticAccountManagementEnableAccount**: Use this setting to configure whether the automatically managed account is enabled or disabled.

OMA-URI<br />
**./Device/Vendor/MSFT/LAPS/Policies/AutomaticAccountManagementEnableAccount**<br />
If this setting is enabled, the target account will be enabled<br />
Applicable OS: Windows 11, version 24H2 [10.0.26100] and later<br />
Format: bool<br />
Default: false, meaning the target account will be disabled<br />
True: Target account will be enabled<br />
Dependency: [AutomaticAccountManagementEnabled], must be set to true

**8. AutomaticAccountManagementNameOrPrefix**: Use this setting to configure the name or prefix of the managed local administrator account.

OMA-URI:
  **./Device/Vendor/MSFT/LAPS/Policies/AutomaticAccountManagementNameOrPrefix**<br />
If specified, the value will be used as the name or name prefix of the managed account.If not specified, this setting will default to "WLapsAdmin".

Applicable OS: Windows 11, version 24H2 [10.0.26100] and later<br />
Format: string<br />
Dependency: [AutomaticAccountManagementEnabled], must be set to true

**9. AutomaticAccountManagementRandomizeName**:  Use this setting to configure whether the name of the automatically managed account uses a random numeric suffix each time the password is rotated.<br />
If this setting is enabled, the name of the target account will be random. <br />
If not specified, this setting defaults to False.

OMA-URI
	**./Device/Vendor/MSFT/LAPS/Policies/AutomaticAccountManagementRandomizeName**
 
Applicable OS:  Windows 11, version 24H2 [10.0.26100] and later<br />
Format: bool<br />
Default value: False - The name of the target account won't use a random numeric suffix.<br />
True: The name of the target account will use a random numeric suffix.<br />
Dependency: [AutomaticAccountManagementEnabled], must be set to true


