---
title: "Roadmap 557172: Microsoft Teams Cancels Appointment Booking in the Live Chat Widget"
date: 2026-08-09
description: "Microsoft has cancelled the planned Teams live chat widget appointment booking feature. Here is what the roadmap item said and a read-only check for related tenant settings."
tags: ["Microsoft Teams", "General Availability", "Cancelled"]
---

**Roadmap ID:** [557172](https://www.microsoft.com/en-us/microsoft-365/roadmap?filters=&searchterms=557172)  
**Status:** Cancelled  
**Product:** Microsoft Teams  
**Release phase:** General Availability, previously targeted for May CY2026  
**Cloud instances:** Worldwide (Standard Multi-Tenant), GCC  
**Platforms:** Desktop, Mac  
**Last modified:** August 7, 2026

## What's changing

The Teams live chat widget lets customers hold one-to-one conversations with your business directly from your website. This roadmap item, first published on February 18, 2026, would have added appointment scheduling to that widget, with general availability targeted for May CY2026 in the Worldwide and GCC clouds on the Desktop and Mac platforms.

On August 7, 2026, Microsoft updated the item to say it has decided not to move forward with the change at this time. No replacement or revised timeline was given.

## Why it's worth checking

If your organization planned customer booking flows around the live chat widget, that plan now needs a different path. Since the feature never shipped, there is no dedicated tenant setting to clean up. The closest configuration surface in Teams today is the Virtual Appointments policy, and reviewing it tells you what appointment-related settings your tenant currently carries while you evaluate alternatives.

## Read-only check

Global Reader is sufficient for the read cmdlets in the MicrosoftTeams PowerShell module.

```powershell
# Connect to Microsoft Teams (authentication only)
Connect-MicrosoftTeams | Out-Null

# The cancelled widget booking feature has no setting of its own.
# List Virtual Appointments policies, the nearest related configuration.
$policies = Get-CsTeamsVirtualAppointmentsPolicy

# Export for review
$policies | Export-Csv -Path ".\TeamsVirtualAppointmentsPolicies.csv" -NoTypeInformation

# Disconnect when done
Disconnect-MicrosoftTeams
```

Every tenant returns at least the Global policy, so expect one row. Additional custom policies mean someone in your tenant has already configured virtual appointment behavior, which is worth knowing before you pick a replacement booking approach.

*Script uses read-only cmdlets only. Read it before you run it.*
