# Attack / Detection Validation Log

For each technique: run it, search for the telemetry, and record whether the detection fired. If it didn't, tune the rule, re-run, and record that too.

## Run 1: Kerberoasting (T1558.003)

**Command:**
```powershell
Invoke-AtomicTest T1558.003 -TestNumbers 1
```

**Expected telemetry:** Event ID 4769 (Kerberos service ticket request) with encryption type `0x17` (RC4), requested for a service account with an SPN set.

**Splunk search:**
```
index=winlogs EventCode=4769 Ticket_Encryption_Type=0x17
| stats count by Account_Name, Service_Name
```

**Expected outcome:** No rule in the SIEM lab covers this, so it should expose a gap. Validate [`detections/kerberoasting.spl`](detections/kerberoasting.spl) against the events you see, then re-run the test.

**Date run:** _fill in_
**Result:** _fill in_

---

## Run 2: Credential Dumping via Mimikatz (T1003.001)

**Command:**
```powershell
Invoke-AtomicTest T1003.001 -TestNumbers 1
```

**Expected telemetry:** Sysmon Event ID 10 (ProcessAccess) targeting `lsass.exe`.

**Splunk search:**
```
index=winlogs source="*Sysmon*" EventCode=10 TargetImage="*lsass.exe"
| table _time, SourceImage, TargetImage, GrantedAccess
```

**Prerequisite:** the SIEM lab's Sysmon config logs `ProcessAccess` (Event ID 10) for `lsass.exe`, so this telemetry should appear once that config is installed.

**Date run:** _fill in_
**Result:** _fill in_

---

## Run 3: Encoded PowerShell Command (T1059.001)

**Command:**
```powershell
Invoke-AtomicTest T1059.001 -TestNumbers 2
```

**Expected outcome:** should trigger the existing [`powershell-encoded-command.spl`](https://github.com/EbubeNnaemeka/siem-detection-lab/blob/main/detections/powershell-encoded-command.spl) rule from the SIEM lab.

**Date run:** _fill in_
**Result:** _fill in_
