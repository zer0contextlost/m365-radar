---
title: "Roadmap 484854: DLP Policy Tips Come to Outlook for Mac"
date: 2026-08-20
description: "Microsoft Purview DLP policy tips in Outlook for Mac are listed as Launched, with a July CY2026 disclosure date. Here's a read-only check for the Exchange DLP rules this touches."
tags: ["Microsoft Purview", "Data Loss Prevention"]
---

**Roadmap ID:** [484854](https://www.microsoft.com/en-us/microsoft-365/roadmap?filters=&searchterms=484854)
**Status:** Launched
**Product:** Microsoft Purview
**Rollout:** Public disclosure availability date: July CY2026. Public preview date not announced. Cloud instances listed: GCC, GCC High, DoD. Platform: Web. Release phase tags: General Availability, Preview.

## What's changing

Microsoft Purview DLP policy tips are reaching Outlook for Mac. Per the roadmap description, when a user composes an email containing sensitive information that violates organizational policy, they get a real-time notification directly in the mail tip UI, with two options: remove the non-compliant recipients, or override the policy if necessary. The stated intent is to alert users to potential data loss risks before the email is sent.

That is the full extent of what the entry says. It doesn't specify which sensitive information types trigger the tip, which DLP policy templates are supported, or whether there's a version floor for Outlook for Mac. The only cloud instances tagged are GCC, GCC High, and DoD, and the only platform tag is Web, so read the scoping tags with that in mind rather than assuming Worldwide coverage from this entry alone.

Status is "Launched," with a public disclosure availability date of July CY2026. The entry carries both General Availability and Preview release phase tags but gives no public preview date, so preview timing isn't confirmed either way from this data.

## Why it's worth checking

This only matters to tenants that already have DLP policies enabled against the Exchange workload, since policy tips are a downstream effect of DLP rules rather than a standalone setting. If your Purview DLP policies don't cover Exchange, or the rules that do exist have no user notification configured, Mac users won't see anything different.

The useful question is which of your existing Exchange DLP rules notify users versus enforce silently. A tenant with meaningful Mac Outlook adoption now has a client surface where those notification-bearing rules become visible to users, and it's worth knowing which policies that involves before the tickets arrive.

## Read-only check

Requires the ExchangeOnlineManagement module and a Security & Compliance PowerShell session via Connect-IPPSSession, with at minimum Compliance Data Administrator or Global Reader-equivalent read access to Purview DLP.

```powershell
# Requires: ExchangeOnlineManagement module
# Connect-IPPSSession uses an account with Compliance Data Administrator
# or Global Reader-equivalent read access to Microsoft Purview DLP.
Connect-IPPSSession

# Workload is a flags-style value ("Exchange, SharePoint, ..."), so match on
# it as text rather than treating it as an array.
$exchangePolicies = Get-DlpCompliancePolicy | Where-Object {
    $_.Workload -match "Exchange" -and $_.Enabled -eq $true
}

$results = foreach ($policy in $exchangePolicies) {
    foreach ($rule in Get-DlpComplianceRule -Policy $policy.Name) {
        [PSCustomObject]@{
            PolicyName       = $policy.Name
            PolicyMode       = $policy.Mode
            RuleName         = $rule.Name
            RuleDisabled     = $rule.Disabled
            NotifiesUser     = [bool]$rule.NotifyUser
            NotifyUserType   = $rule.NotifyUserType
            AllowsOverride   = $rule.NotifyAllowOverride
            HasPolicyTipText = [bool]$rule.NotifyPolicyTipCustomText
        }
    }
}

$results | Format-Table -AutoSize

# Optional: save a copy for review
$results | Export-Csv -Path "./dlp-exchange-policytip-audit.csv" -NoTypeInformation
```

An empty table means you have no enabled DLP policies scoped to Exchange, and this change activates nothing in your tenant. Otherwise, look at `NotifiesUser` and `NotifyUserType`: rules where notification is on and the type includes `PolicyTip` are the ones whose tips can now surface to Outlook for Mac users, and `AllowsOverride` tells you which of those permit the override option the roadmap entry describes. Rules with notification off keep enforcing silently with no user-visible change.

*Script uses read-only cmdlets only. Read it before you run it.*
