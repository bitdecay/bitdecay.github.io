---
title: "Understanding sekurlsa::logonpasswords Null Results"
date: 2025-10-24 13:24:11
categories: [Active Directory]
tags: [Active Directory]
---

I was setting up a lab where I’ll demonstrate credential dumping using the infamous Mimikatz. I thought this should be easy: disable the AV and dump the credentials of local users. So, I logged into a fresh installed Windows 10 lab using the target local admin account and disabled the AV. I proceeded to drop a mimkatz.exe file into the desktop folder of the targeted user. I am logged into the VM (windows 10) as a local administrator and fired up a CMD in admin mode. Then I used the sekurlsa::logonpasswords command to extract credentials from LSASS memory. As expected, I can see the hashes of the logged-on user, but the password value was (null).

![password null](/images/2025/10-24-password-null.png)

I have never run into this situation before in pentesting labs. Maybe I never noticed because the NTML/AES hashes were enough to escalate or move laterally in the networks. Or there are other ways to get cleartext passwords in AD pentest and through mimikatz. However, in this lab, I was expecting the cleartext password, but it wasn't there. What I thought would take a minute of mimikatz demo now required some more digging to figure out why I wasn't getting the expected results. That’s when I was introduced to Wdigest. And let me share with you what I learned. 


### So what is this wdigest?
WDigest is a Digest Authentication protocol, and it stores plaintext passwords in memory for locally authenticated accounts. Because of the work it does, it became a target of hackers. However, it is disabled by default in Windows 10 version 1703 and later versions of Windows for this abvious security risk. When explicitly configured though, it forces the Local Security Authority Subsystem Service (LSASS.exe) to store a user’s plain text password in memory after successful authentication. 

So I checked the wdigest in the registry because that’s where its configuration is. As you can see, the WDigest, by default in the latest Windows 10, doesn’t have the necessary registry key (UseLogonCredential) set up that stores plaintext passwords in memory.

`reg query HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest`

![wdigest](/images/2025/10-24-wdigest.png)

To accomplish my initial goal, I added UseLogonCredential with the value as 1 to WDigest to enable the plaintext caching. You can set it manually or run this command:

`reg add HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest /v UseLogonCredential /t REG_DWORD /d 1 /f`

After the command is completed, check the wdigest registry with the below command to see if the registry key was added. If it was successful, you will see `UseLogonCredential REG_DWORD 0x1` as the last key.

`reg query HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest`

![uselogoncreds](/images/2025/10-24-uselogoncredentials.png)

Now log off and log back on (or restart) to make the setting effective. Now when you dump credentials with `sekurlsa::logonpasswords`, you should see the target user’s cleartext password.

![plaintext password](/images/2025/10-24-plaintext.png)

If you want to test this on Linux, you can use [lsassy](https://github.com/login-securite/lsassy) tool.

![uselogoncreds2](/images/2025/11-5-lsassy.png)

Looking closely at the sekurlsa::logonpasswords output, it seems this module gets credentials from features like msv, wdigest, tspkg, kerberos, credman, etc. It became clear to me why the cleartext password showed up under wdigest, and why we looked for cleartext password there.
Interestingly, the session type for the dumped entry was Interactive, meaning that it was a local computer logon. This is interesting information because the way sekurlsa::logonpasswords handles interactive, network-only (e.g., SMB printers), and remote-interactive logons (e.g., RDP session) are different. WDigest hooks into the interactive credential path, so UseLogonCredential affects interactive sessions. That’s why we needed to log off/log back on (or restart) after changing the registry to see the change take effect, which makes the process local authentication. 

Here are a couple of more examples:
1. Network logon with Lewisville account through smbclient is not cached because SMB authentication is performed using NTML or Kerberos, which don't transmit the user's cleartext password when logging in. Thus, lsassy couldn't find the cleartext passwords of Lewisville.

![network logon](/images/2025/11-5-lsassy-smb.png)

2. RDP session is a remote-interactive logon. That's why the cleartext credential was cached.

![network logon](/images/2025/11-5-lsassy-rdp.png)


>Note that, the cached cleartext passwords are removed from the system when the users log off. So if you get cleartext passwords through wdigest during your attack, then luck must be on your side that day.
{: .prompt-tip }

I hope you learned something from this post.
