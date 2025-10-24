---
title: "Understanding sekurlsa::logonpasswords Null Results"
date: 2025-10-24 13:24:11
categories: [Active Directory]
tags: [Active Directory]
---

I was setting up a lab where I’ll demonstrate credential dumping using the infamous Mimikatz. I thought this should be easy: disable the AV and dump the credentials of local users. So, I logged into a fresh installed Windows 10 lab using the target local admin account and disabled the AV. I proceeded to drop a mimkatz.exe file into the desktop folder of the targeted user. I used the sekurlsa::logonpasswords command to extract credentials from LSASS memory. As expected, I can see the hashes of the logged-on user, but the password value was (null).

![password null](/images/2025/10-24-password-null.png)

I have never run into this situation before in pentesting labs. Maybe I never noticed because the NTML/AES hashes were enough to escalate or move laterally in the networks. However, in this lab, I was expecting the cleartext password, but it wasn't there. What I thought would take two minutes now required some digging. That’s when I ran into WDigest.
 
WDigest is an authentication protocol, and it stores plaintext passwords in memory. However, WDigest is disabled by default in Windows 10 version 1703 and later versions of Windows. When explicitly configured though, it forces the Local Security Authority Subsystem Service (LSASS.exe) to store a user’s plain text password in memory after successful authentication.

So I checked the wdigest in the registry because that’s where its configuration is. As you can see, the WDigest, by default in the latest Windows 10, doesn’t have the necessary registry key (UseLogonCredential) set up that stores plaintext passwords in memory.

![wdigest](/images/2025/10-24-wdigest.png)


To accomplish my initial goal, I added UseLogonCredential with the value as 1 to WDigest to enable the plaintext caching. You can set it manually or run this command: 
`reg add HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest /v UseLogonCredential /t REG_DWORD /d 1 /f`

![uselogoncreds](/images/2025/10-24-uselogoncredentials.png)

After changing, log off and log back on (or restart) to make the setting effective. Now when you dump credentials with sekurlsa::logonpasswords, you should see the target user’s cleartext password.

![plaintext password](/images/2025/10-24-plaintext.png)

Looking closely at the sekurlsa::logonpasswords output, it seems this module gets credentials from features like msv, wdigest, tspkg, kerberos, and credman. It became clear to me why the cleartext password shows up under wdigest and why we looked for cleartext password there.
Interestingly, the session type for the dumped entry was Interactive, meaning that it was a local computer logon. This is interesting information because the way sekurlsa::logonpasswords handles interactive, network-only (e.g., SMB printers), and remote-interactive logons (e.g., RDP session) is different. WDigest hooks into the interactive credential path, so UseLogonCredential affects interactive sessions. That’s why we needed to log off/log back on (or restart) after changing the registry to see the change take effect.
Lastly, a (null) password can also occur for reasons other than WDigest being disabled. For example, if the user logged in via RDP and WDigest was not enabled for that session, cleartext credentials will not be cached the same way as an interactive logon.
