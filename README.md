# Windows-Server-2022-Windows-10-Setup

This guide provides step-by-step instructions for setting up a lab environment using Windows Server 2022 as a domain controller and Windows 10 as a client. The configuration is based on a set of technical specifications commonly used in training and educational settings.

**Disclaimer:** This configuration, especially the simplified password policy, is designed **STRICTLY FOR CONTROLLED LAB ENVIRONMENTS ONLY**. Do not use these settings in a production environment as they are insecure.

**Important Note:** Throughout this guide, replace `SN` with your station number.

## Table of Contents
1.  [Prerequisites](#prerequisites)
2.  [Part 1: Virtual Environment Setup](#part-1-virtual-environment-setup)
3.  [Part 2: Windows Server 2022 Configuration](#part-2-windows-server-2022-configuration)
    *   [2.1 Server Installation & Basic Configuration](#21-server-installation--basic-configuration)
    *   [2.2 Active Directory Domain Services (AD DS) Installation](#22-active-directory-domain-services-ad-ds-installation)
    *   [2.3 Changing the Domain Password Policy (Critical Step)](#23-changing-the-domain-password-policy-critical-step)
    *   [2.4 Creating Active Directory Users & Groups](#24-creating-active-directory-users--groups)
    *   [2.5 DHCP Server Configuration](#25-dhcp-server-configuration)
    *   [2.6 DNS Server Configuration](#26-dns-server-configuration)
    *   [2.7 FTP Server Configuration](#27-ftp-server-configuration)
4.  [Part 3: Windows 10 Client Configuration](#part-3-windows-10-client-configuration)
5.  [Part 4: Testing & Verification](#part-4-testing--verification)

---

### Prerequisites

Before you begin, ensure you have:
*   A Host PC (e.g., Windows 11) capable of running virtual machines.
*   Virtualization software such as VMware Workstation, VirtualBox, or Hyper-V.
*   A Windows Server 2022 ISO image file.
*   A Windows 10 ISO image file.

### Part 1: Virtual Environment Setup

1.  **Create a Virtual Network:**
    *   In your virtualization software, create a new virtual switch.
    *   Set the network type to **Host-Only** or **Private Network**. This will isolate communication between the server and client from external networks, creating a secure lab environment.
    *   Ensure the built-in DHCP server for this network in your virtualization software is **disabled**, as we will configure it on our Windows Server.

### Part 2: Windows Server 2022 Configuration

#### 2.1 Server Installation & Basic Configuration

1.  **Create the Server Virtual Machine:**
    *   Create a new virtual machine using the Windows Server 2022 ISO image.
    *   Connect this VM to the virtual network you created earlier.
    *   Complete the Windows Server installation process.

2.  **Initial Server Settings:**
    *   **Rename the Computer:**
        *   Open PowerShell as an Administrator and type:
          ```powershell
          Rename-Computer -NewName "WINsrvSN" -Restart
          ```
    *   **Set a Static IP Address:**
        *   Go to `Control Panel` > `Network and Sharing Center` > `Change adapter settings`.
        *   Right-click your network adapter and select `Properties`.
        *   Select `Internet Protocol Version 4 (TCP/IPv4)` and click `Properties`.
        *   Enter the following settings:
            *   **IP address:** `192.168.SN.100`
            *   **Subnet mask:** `255.255.255.0`
            *   **Preferred DNS server:** `192.168.SN.100` (pointing to itself)

#### 2.2 Active Directory Domain Services (AD DS) Installation

1.  **Install the Role:**
    *   Open **Server Manager**, click `Manage` > `Add Roles and Features`.
    *   Follow the wizard until you reach `Server Roles`, and select `Active Directory Domain Services`.
    *   Accept the suggested additional features and complete the installation.

2.  **Promote to a Domain Controller:**
    *   Once the installation is finished, click the notification flag icon in Server Manager and select `Promote this server to a domain controller`.
    *   Select `Add a new forest`.
    *   **Root domain name:** `test1.com`
    *   Set the **Directory Services Restore Mode (DSRM)** password to `lkmb123`.
    *   Proceed with the default options until the installation begins. The server will restart automatically.

#### 2.3 Changing the Domain Password Policy (Critical Step)

By default, Windows Server enforces complex passwords. We need to disable this to use simple passwords like `1234`.

1.  **Open Group Policy Management:**
    *   In **Server Manager**, go to `Tools` > `Group Policy Management`.

2.  **Edit the Default Domain Policy:**
    *   Expand `Forest: test1.com` > `Domains` > `test1.com`.
    *   Right-click on **Default Domain Policy** and select `Edit...`.

3.  **Disable Complexity Requirements:**
    *   Navigate to: `Computer Configuration` > `Policies` > `Windows Settings` > `Security Settings` > `Account Policies` > `Password Policy`.
    *   In the right pane, double-click `Password must meet complexity requirements`.
    *   Select **Disabled** and click `OK`.
    *   (Optional) Double-click `Minimum password length` and set it to `0` if needed.

4.  **Enforce the Policy:**
    *   Open **Command Prompt** or **PowerShell** as an Administrator and run the following command to update the policy immediately:
      ```cmd
      gpupdate /force
      ```

#### 2.4 Creating Active Directory Users & Groups

1.  Open **Active Directory Users and Computers** from the `Tools` menu in Server Manager.
2.  Expand the `test1.com` domain.
3.  Create new groups if needed (e.g., `ftp`) by right-clicking `Users` > `New` > `Group`.
4.  Create the following user accounts by right-clicking `Users` > `New` > `User`. Be sure to add them to the correct groups.

| Account | Password | Group Membership |
| :--- | :--- | :--- |
| LocalAdmin | `skmtest1` | Administrators |
| Manager | `1234` | Administrators |
| Supervisor| `1234` | Power Users |
| User1 | `1234` | Domain Users |
| User2 | `1234` | Domain Users |
| User10 | `1234` | Domain Users |
| Webadmin | `123test1` | ftp |

#### 2.5 DHCP Server Configuration

1.  **Install the DHCP Role:**
    *   Use `Add Roles and Features` in Server Manager to install the `DHCP Server` role.
2.  **Configure a Scope:**
    *   Complete the post-installation DHCP configuration via the notification in Server Manager.
    *   Open `DHCP` from the `Tools` menu.
    *   Right-click `IPv4` and select `New Scope...`.
    *   **Scope Name:** `LAN`
    *   **IP Address Range:** `192.168.SN.110` to `192.168.SN.130`
    *   **Subnet Mask:** `255.255.255.0`
    *   **Router (Default Gateway):** Leave blank.
    *   **DNS Servers:** Ensure it's set to `192.168.SN.100`.
    *   Activate the scope when the wizard finishes.

#### 2.6 DNS Server Configuration

This role is installed automatically with AD DS. Verify its configuration:
1.  Open `DNS` from the `Tools` menu.
2.  Expand `Forward Lookup Zones` > `test1.com`.
3.  Ensure a Host (A) record exists for `WINsrvSN` pointing to `192.168.SN.100`.

#### 2.7 FTP Server Configuration

1.  **Install IIS & FTP Role:**
    *   Use `Add Roles and Features` to install the `Web Server (IIS)` role.
    *   Under `Role Services`, ensure you select `FTP Server` and `FTP Service`.

2.  **Set up the FTP Site:**
    *   Create a folder `C:\ftp_test1`.
    *   Open `Internet Information Services (IIS) Manager` from `Tools`.
    *   Right-click `Sites` > `Add FTP Site...`.
    *   **FTP site name:** `ftp test1`
    *   **Physical path:** `C:\ftp_test1`
    *   **Binding:** Set the IP Address to `192.168.SN.100` on Port `21`.
    *   **SSL:** Select `No SSL`.
    *   **Authentication:** `Basic`
    *   **Authorization:** `Specified users`, enter `webadmin`.
    *   **Permissions:** `Read` and `Write`.
    *   Click `Finish`.

### Part 3: Windows 10 Client Configuration

1.  **Create the Client Virtual Machine:**
    *   Create a new virtual machine using the Windows 10 ISO image.
    *   Connect this VM to the same virtual network as the server.
    *   Ensure its network settings are configured to **obtain an IP address automatically (DHCP)**.

2.  **Rename Computer & Join the Domain:**
    *   Once Windows 10 is installed, open `Settings` > `System` > `About`.
    *   Click `Rename this PC (advanced)`.
    *   Under the `Computer Name` tab, click `Change...`.
    *   **Computer name:** `hostSN`
    *   **Member of:** Select `Domain` and type `test1.com`.
    *   Click `OK`. Enter the credentials of a domain account with privileges (e.g., `test1\Manager` with password `1234`).
    *   Restart the PC when prompted.

### Part 4: Testing & Verification

1.  **Domain Logon Test:**
    *   At the Windows 10 login screen, log in using a domain user account. For example:
        *   **Username:** `test1\User1`
        *   **Password:** `1234`

2.  **DHCP & DNS Test:**
    *   Open **Command Prompt** on the Windows 10 client.
    *   Type `ipconfig /all`. Verify the client received an IP in the `192.168.SN.110` - `192.168.SN.130` range and the DNS Server is `192.168.SN.100`.
    *   Type `ping WINsrvSN`. You should get a reply from `192.168.SN.100`.

3.  **FTP Access Test:**
    *   Open **File Explorer** on the Windows 10 client.
    *   In the address bar, type `ftp://192.168.SN.100` and press Enter.
    *   When prompted, log in as:
        *   **Username:** `webadmin`
        *   **Password:** `123test1`
    *   Confirm you can see the folder's contents and try creating a new file to test write permissions.

If all the above tests are successful, your lab environment has been configured correctly.
