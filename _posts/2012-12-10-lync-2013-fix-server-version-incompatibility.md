---
layout: post
title: Lync 2013 fix server version incompatibility
excerpt_separator: <!--more-->
tags:
    - office
---

Today, I installed Office 2013 and found out Lync 2013 is not signing in to BT Office Communicator Server 2007. <!--more--> I started getting a message saying 

> “Cannot sign in because the server version is incompatible with Microsoft Lync. Contact your support team with this information”

After scratching my head for many hours, I finally found how to disable the server check.

Make sure you run the Command Prompt as “Administrator” and execute the command below:

    Reg Add "HKEY_LOCAL_MACHINESOFTWAREPoliciesMicrosoftOffice15.0Lync" /V "DisableServerCheck" /D 1 /T REG_DWORD /F
