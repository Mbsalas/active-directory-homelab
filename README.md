# Active Directory Home Lab: Building and Managing a Windows Server 2025 Domain

For this project I built a working Active Directory domain from the ground up using three virtual machines: one Windows Server 2025 domain controller and two Windows 11 clients. The server got a static IP and DNS, then the AD DS and DNS roles, then I promoted it into a new domain called homelab.local and joined both clients to it.

After that I set up the domain itself. I built a departmental structure using organizational units for Sales, HR, Legal, and Finance, and populated each with users. I worked through the common account management tasks an admin handles day to day, set up security groups and shared folders with the right permissions, and used Group Policy to manage drives, devices, and the firewall across every machine at once.

> **Full build log:** This README is a curated walkthrough. For the complete step-by-step with every screenshot, see [`docs/full-build-log.pdf`](docs/full-build-log.pdf).

## Prerequisites

To follow along you need a computer with enough RAM and disk space to run three VMs at once (I'd recommend at least 16 GB of RAM), plus the following, all free:

- **VMware Workstation Pro** — the software that runs the virtual machines. It's free for personal use and requires a free Broadcom account to download. [Get it here.](https://www.vmware.com/products/desktop-hypervisor/workstation-and-fusion)
- **Windows Server 2025 ISO** — the operating system for the domain controller. The 180-day evaluation is free from the Microsoft Evaluation Center. [Download here.](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2025)
- **Windows 11 ISO** — for the two client machines. The official disk image includes Pro. [Download here.](https://www.microsoft.com/software-download/windows11)

Once VMware is installed, I created three VMs from those ISOs: one server (roughly 60 GB disk, 3 GB RAM) and two Windows 11 clients (60 GB disk, 4 GB RAM each). The full step-by-step for creating and installing the VMs is in the [build log PDF](docs/full-build-log.pdf). This README picks up once all three machines are running.

## The Lab

Everything runs on a single host machine using VMware, on an isolated NAT network so the three machines can talk to each other but stay separate from my real network. The setup simulates a small company with one server and two employee workstations.

| Role | Machine | Operating System |
|------|---------|------------------|
| Domain Controller (homelab.local) | Server | Windows Server 2025 Standard |
| Employee Workstation | W1 | Windows 11 Pro |
| Employee Workstation | W2 | Windows 11 Pro |

## Tools Used

VMware Workstation Pro, Windows Server 2025, Windows 11 Pro, Active Directory, DNS, and Group Policy.

---

## Phase 1: Naming, Static IP, and Connectivity

Renamed all three machines and gave the server a static IP, since a domain controller should never have an address that can change.

![Renaming the server](images/01-rename-server.png)

![Static IP configuration](images/02-static-ip-config.png)

Initial connectivity tests between machines failed with 100% packet loss, even though all three were on the same NAT network.

![Ping test failed](images/03-ping-failed.png)

After troubleshooting, I traced it to Windows Defender Firewall blocking inbound ICMP echo requests. I turned the firewall off on each machine to confirm the cause (re-enabled properly via Group Policy in Phase 6).

![Firewall turned off on W1](images/04-firewall-off-w1.png)

Connectivity was restored across all three machines with 0% packet loss.

![Ping success](images/05-ping-success.png)

## Phase 2: DNS and Active Directory Domain Services

Installed the DNS Server role, created a forward lookup zone for homelab.local, and installed the AD DS role.

![DNS role install](images/06-dns-role-install.png)

![Forward lookup zone created](images/07-forward-lookup-zone.png)

![AD DS role install](images/08-adds-role-install.png)

## Phase 3: Promoting the Domain Controller

Promoted the server, creating a new forest with the root domain homelab.local.

![New forest](images/09-new-forest.png)

After the promotion and reboot, the login shows HOMELAB\Administrator and Server Manager reports the domain instead of a workgroup.

![Domain login](images/10-domain-login.png)

![Domain instead of workgroup](images/11-domain-not-workgroup.png)

## Phase 4: Joining the Clients

Pointed each client's DNS at the domain controller and joined it to homelab.local.

![Joining W1](images/12-join-w1.png)

Both clients appear in Active Directory Users and Computers.

![W1 in ADUC](images/13-w1-in-aduc.png)

![Both clients joined](images/14-both-clients-joined.png)

## Phase 5: Organizational Units and Users

Built a departmental OU structure (Sales, HR, Legal, Finance) and populated each with user accounts.

![Creating an OU](images/15-create-ou.png)

![All OUs created](images/16-all-ous.png)

![Creating a user](images/17-create-user.png)

![Two users per OU](images/18-two-users-hr.png)

## Phase 6: Account Administration

Worked through common account management tasks using the test users.

**Password reset:** reset a locked-out user's password and required a change at next logon.

![Password reset](images/19-reset-password.png)

**Disable account (offboarding):** disabled a departing user's account.

![Account disabled](images/20-account-disabled.png)

**Account expiration:** set an expiration date for a temporary user.

![Account expiration](images/21-account-expiration.png)

**Logon hours:** restricted a user from logging in on certain days.

![Logon hours](images/22-logon-hours.png)

**Workstation restriction:** limited a user to logging on from a single machine.

![Workstation restriction](images/23-workstation-restriction.png)

## Phase 7: Security Groups and Shared Folders

Used security groups to control access instead of assigning permissions to users individually, then created department shares.

![Security group](images/24-security-group.png)

![Shared folder](images/25-shared-folder.png)

![Mapped drive on client](images/26-mapped-drive-hr1.png)

For the Legal share I disabled inheritance and set the permissions manually, giving the Legal_users group modify rights on the folder.

![Folder permissions](images/27-ntfs-permissions.png)

## Phase 8: Group Policy

**Automatic drive mapping:** created a GPO that maps the Legal drive, using item-level targeting so only members of the Legal_users group receive it.

![GPO drive map](images/28-gpo-drive-map.png)

![Item-level targeting](images/29-item-level-targeting.png)

![Mapped drive appears at login](images/30-mapped-drive-shows.png)

**Block USB storage:** created and enforced a GPO denying access to removable storage devices, then verified it applied.

![Disable USB storage GPO](images/31-disable-usb-gpo.png)

![gpresult verification](images/32-gpresult-usb.png)

## Phase 9: Re-enabling the Firewall with Group Policy

Closed the loop on the Phase 1 firewall shortcut by re-enabling Windows Defender Firewall centrally through a GPO, with a scoped inbound rule to allow ping from the local subnet.

![Firewall GPO configuration](images/33-firewall-gpo-config.png)

![ICMP rule scoped to subnet](images/34-icmp-rule-scope.png)

Confirmed the firewall was back on and that connectivity still worked with the echo rule in place.

![Firewall back on](images/35-firewall-back-on.png)

![Ping success with firewall on](images/36-ping-success-final.png)

---

## What I Learned

Going through this end to end gave me a solid feel for how Active Directory actually fits together in a real setup. A few things stood out:

- Getting the basics right first matters. The naming, static IP, and DNS all have to be correct because everything after the domain depends on them.
- The firewall issue early on was a good reminder to troubleshoot in order and confirm the cause instead of guessing.
- Security groups make managing access far cleaner than setting permissions user by user.
- Group Policy lets you push one setting from the domain controller and have it apply to every machine automatically, whether that's mapping drives, blocking USB storage, or turning the firewall back on.
