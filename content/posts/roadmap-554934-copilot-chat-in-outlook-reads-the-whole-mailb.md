---
title: "Roadmap 554934: Copilot Chat in Outlook Reads the Whole Mailbox for Unlicensed Users — Does It Touch You?"
date: 2026-08-08
description: "Roadmap 554934 gives Copilot Chat users without a Copilot license reasoning over inbox and calendar in Outlook. A read-only Graph check shows who that reaches."
tags: ["Outlook", "Microsoft Copilot (Microsoft 365)"]
---

**Roadmap ID:** [554934](https://www.microsoft.com/en-us/microsoft-365/roadmap?filters=&searchterms=554934)
**Status:** In development
**Product:** Outlook, Microsoft Copilot (Microsoft 365)
**Rollout:** General Availability targeted for September CY2026. No public preview date listed.

## What's changing

Microsoft is expanding Copilot Chat in Outlook to reason over a user's entire inbox, their calendar, and other enterprise data. The part that matters for licensing math is in the last clause of the description: this applies to Microsoft 365 Copilot Chat users, the ones without a Microsoft 365 Copilot license.

That is the whole description. It is one sentence, and it leaves open questions the roadmap entry does not answer, such as which "other enterprise data" sources are in scope or what admin controls ship alongside it. We are not going to guess at those here. What the entry does state clearly is the platform spread: Android, iOS, Desktop, Mac, and Web, so this covers new Outlook on effectively every surface it runs on.

The cloud instance tags are worth a second look. GCC, GCC High, and DoD are all listed, which puts government tenants in scope for the same rollout. The item was created on 5 February 2026, was last modified on 7 August 2026, and currently sits at "In development" with GA targeted for September CY2026.

## Why it's worth checking before it ships

Until now, a reasonable working assumption was that Copilot reasoning over mailbox and calendar content was gated behind the paid Microsoft 365 Copilot license, and that unlicensed users interacting with Copilot Chat were working with a narrower surface. This item changes that assumption for Outlook. If your data governance story for Copilot was "we only licensed the pilot group, so only the pilot group's mailboxes are in play," the population this feature reaches is everyone else.

The practical question for an admin is simple: how big is the gap between your enabled user count and your assigned Copilot license count? That gap is the audience for this change. A tenant with no Copilot SKUs at all has the largest possible exposure, since every Copilot Chat user there is an unlicensed one.

## Read-only check

Requires the Microsoft.Graph PowerShell module and read scopes only (`Organization.Read.All`, `User.Read.All`), which Global Reader satisfies.

```powershell
# Roadmap 554934: Copilot Chat in Outlook will reason over inbox, calendar,
# and enterprise data for users WITHOUT a Microsoft 365 Copilot license.
# This script reads license and user counts. It changes nothing.

Connect-MgGraph -Scopes "Organization.Read.All","User.Read.All" -NoWelcome

# 1. Copilot SKUs present in the tenant and how many seats are assigned
$copilotSkus = Get-MgSubscribedSku -All |
    Where-Object { $_.SkuPartNumber -like "*Copilot*" } |
    Select-Object SkuPartNumber,
        @{ n = 'Purchased'; e = { $_.PrepaidUnits.Enabled } },
        @{ n = 'Assigned';  e = { $_.ConsumedUnits } }

$copilotSkus | Format-Table -AutoSize

# 2. Enabled member users, i.e. the population that can open Copilot Chat
Get-MgUser -Filter "accountEnabled eq true and userType eq 'Member'" `
    -ConsistencyLevel eventual -CountVariable enabledUsers -Top 1 | Out-Null

$assigned = ($copilotSkus | Measure-Object -Property Assigned -Sum).Sum

[pscustomobject]@{
    EnabledMemberUsers      = $enabledUsers
    CopilotLicensesAssigned = [int]$assigned
    UnlicensedForCopilot    = $enabledUsers - [int]$assigned
} | Format-List

# What the output means:
# - UnlicensedForCopilot = 0: every enabled user already holds a Copilot
#   license, so this roadmap item adds nothing new in your tenant.
# - UnlicensedForCopilot > 0: that number is the population this change
#   reaches. At GA, those users' Copilot Chat in Outlook gains reasoning
#   over their inbox, calendar, and enterprise data.
# - An empty SKU table with a large user count means no Copilot licenses
#   exist at all, which puts the entire tenant in scope for this change.
```

The first table lists any Copilot SKUs you own and their assignment counts. The final object is the number that matters: `UnlicensedForCopilot` approximates how many of your users will pick up mailbox and calendar reasoning in Outlook through this change rather than through a license you chose to buy.

*Script uses read-only cmdlets only. Read it before you run it.*
