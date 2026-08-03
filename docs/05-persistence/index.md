# Module 5: Persistence Catalog

Every persistence mechanism here is written once, tagged with its ATT&CK sub-technique, and linked to from wherever it's relevant — a playbook, an artifact page, a Defender detection. Nothing about persistence gets explained twice in this guide; this module is the single source of truth.

## Structure every entry follows

**Mechanism → ATT&CK ID → where the evidence lives → detection query/approach → cleanup**

## Module status: complete

**Endpoint persistence**

- [x] [Registry Run / RunOnce Keys](registry-run-keys.md) (T1547.001)
- [x] [Scheduled Tasks](scheduled-tasks.md) (T1053.005)
- [x] [Windows Services](windows-services.md) (T1543.003)
- [x] [WMI Event Subscriptions](wmi-subscriptions.md) (T1546.003)
- [x] [COM Hijacking & DLL Search-Order Hijacking](com-dll-hijacking.md) (T1546.015 / T1574.001)
- [x] [AppInit_DLLs & Image File Execution Options](appinit-ifeo.md) (T1546.010 / T1546.012)
- [x] [LSA Provider / Security Support Provider Abuse](lsa-ssp.md) (T1547.005)
- [x] [BITS Jobs](bits-jobs.md) (T1197)
- [x] [Winlogon Shell/Userinit Modification](winlogon-helper.md) (T1547.004)

**Cloud/hybrid identity persistence**

- [x] [Malicious OAuth Application Consent Grants](oauth-consent-grants.md) (T1098.003)
- [x] [Backdoor App Registrations & Service Principal Credentials](backdoor-app-registrations.md) (T1098.001)
- [x] [Mailbox Forwarding Rules & Delegate Access](mailbox-forwarding-rules.md) (T1114.003)
- [x] [Federation Trust Abuse ("Golden SAML")](golden-saml.md) (T1606.002)
- [x] [Break-Glass / Emergency-Access Account Abuse](break-glass-abuse.md) (T1078.004)

Cloud persistence deserves equal billing with endpoint persistence in this catalog — an attacker who gets kicked off a host but left a mail-forwarding rule or an OAuth grant behind hasn't actually been evicted. See [Module 6's hybrid runbook](../06-cloud-identity/index.md#hybrid-account-compromise-runbook-available-now) for the remediation order that accounts for this.
