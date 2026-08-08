---
title: "Roadmap 569022: Purview Label Inheritance from Email Attachments — Does It Touch You?"
date: 2026-08-08
description: "Outlook will start applying or recommending sensitivity labels based on what's attached to an email. Here's a read-only check for whether your tenant has anything that would trigger it."
tags: ["outlook", "purview", "sensitivity-labels", "roadmap"]
---

**Roadmap ID:** [569022](https://www.microsoft.com/en-us/microsoft-365/roadmap?filters=&searchterms=569022)
**Status:** In development
**Product:** Outlook — Worldwide (Standard Multi-Tenant), GCC, GCC High, DoD
**Rollout:** General availability expected September 2026

## What's changing

Per Microsoft's roadmap entry, Outlook will automatically apply or
recommend a sensitivity label on an email based on the classification of
files attached to it — the label follows the attachment, not just whatever
the compose window already had set. The initial description centers on
Mac, with iOS/Android inheritance tracked as a related, separately-numbered
item (569021).

## Why it's worth checking before it ships

This only does anything in a tenant that already has Microsoft Purview
sensitivity labels published and in active use. If your org has never
touched Purview Information Protection, this roadmap item is a no-op for
you. If it has, the practical question is: which labels and policies are
scoped in a way that could fire off attachment classification in Outlook,
and are any of them stricter than what your users currently expect from
a plain compose window.

## Read-only check

Requires the **ExchangeOnlineManagement** module (`Install-Module
ExchangeOnlineManagement`) and Compliance Administrator or Global Reader
access. This only reads configuration — it does not create, modify, or
apply anything.

```powershell
Connect-IPPSSession -UserPrincipalName admin@yourtenant.onmicrosoft.com

# All sensitivity labels currently published in the tenant
Get-Label | Select-Object Name, DisplayName, Priority, Disabled

# Policies that publish labels, and which locations/users they're scoped to
Get-LabelPolicy | Select-Object Name, Labels, ExchangeLocation, Disabled

# Narrow to policies that actually reach Exchange/Outlook,
# since that's the surface this roadmap item touches
Get-LabelPolicy | Where-Object { $_.ExchangeLocation -and -not $_.Disabled } |
    Select-Object Name, Labels, ExchangeLocation
```

If the last command returns nothing, no published label policy currently
reaches Exchange/Outlook, and this change has nothing to attach to in your
tenant yet. If it returns policies, note which labels they carry —
those are the ones that could start showing up as automatic or recommended
labels on mail with attachments once this rolls out, and worth a look at
before it does rather than after a user asks why an email got relabeled.

*Script uses `Get-*` cmdlets only. Read it before you run it.*
