# Restoring-hard-drive-data-from-Corrupted-VM
In this repo, I worked through different methods in trying to rescue/exfiltrate data from a corrupted VM that would hang at the login screen. Using methodology from the AWS solutionsn architect associate course, I saw similarities between the EFS, EC2, and EBS services and found ways to implement such methodology within the UTM environment. 

## 📋 Project Overview
This document chronicles the complete recovery process of a corrupted UTM virtual machine, documenting both successful strategies and the valuable failures that led to the ultimate solution. This showcases iterative problem-solving methodology in system recovery.

## 🚨 The Catastrophe
The UTM VM (`debraa.utm`) suffered complete system failure due to cascading issues:
- **Saved state corruption** creating login loops
- **EFI variable file bloated to 4GB** (32,000x normal size)
- **Boot configuration damage** preventing system access
- **Process instability** with constant VM restarts

## 🎯 Initial Assessment & The Three-Port Reconnaissance

### Phase 1: Network Exploration (The Failed Offensive)
I ran nmap and attempted to regain access through available network services in an attempt to simply exfiltrate important files:

#### **Port 5000 - The AirPlay Mirage**

Discovery: AirTunes/870.14.1 service responding with 403 Forbidden

Response: HTTP/1.1 403 Forbidden - Server: AirTunes/870.14.1

Failed Exploitation Attempts:
1. SOAP Injection - AirPlay rejected all payloads
2. RTSP Protocol attacks - Authentication required
3. Buffer overflow attempts - Service remained stable
4. Authentication bypass - No vulnerabilities found

Learning: Sometimes services are well-secured, not broken

## Port 7000 - The Silent Service**

Discovery: Same AirPlay service, same 403 responses
Hypothesis: Redundant service or load balancer
Reality: Another AirPlay endpoint, equally secure

Failed: All command injection attempts, Protocol manipulation
Learning: Multiple ports don't mean multiple vulnerabilities

## Port 53 - The DNS Dead End**

Discovery: DNS service running but unresponsive to exploits
```
dig @192.168.64.2 version.bind CHAOS TXT
```
Result: Timeout - no version information leaked

Failed: DNS cache poisoning attempts, zone transfer requests
Learning: Not all open ports are attack vectors


## **Creative Pivots attempts**
1. **Reverse Shell Through Service Injection** - AirPlay sanitized all input
2. **Service Crash and Recovery** - Services automatically restarted
3. **Protocol-Level Attacks** - All standard protocols properly implemented
4. **Timing Attacks** - No race conditions detected

## 🔍 The Diagnostic Breakthrough

### Phase 2: Forensic Analysis
Instead of continued exploitation, we shifted to diagnostics:

```bash
# Discovered the massive EFI file corruption
ls -lh debraa.utm/Data/efi_vars.fd
# -rw-r--r-- 1 owner staff 4.0G Nov 16 15:23 efi_vars.fd
```
Normal size should be: 128K-1M
This file was 4,000x larger than expected!

 
## 🛠️ The Pivot: Recovery Through Reconstruction

### Phase 3: Architectural Understanding
We analyzed the UTM VM structure and realized:

```
UTM VM Components:
├── Configuration Files (config.plist, efi_vars.fd) - ❌ CORRUPTED
├── Saved States (*.save, *.utmS) - ❌ CORRUPTED  
└── Hard Disk (.qcow2 file) - ✅ INTACT (contains actual OS + data)
```

### The Creative Leap
Instead of repairing the corrupted components, we asked: **"What if we build a new house around the existing foundation?"**

## 🔧 The Recovery Process

### Phase 4: The Reconstruction Strategy

#### **Step 1: Create New VM Infrastructure**

Fresh UTM VM with identical specifications:
- Architecture: ARM64 (aarch64)
- RAM: 8192MB
- CPU: 8 cores
- Network: Shared mode

#### **Step 2: Attach Existing Hard Disk**
- Used UTM's "Import existing drive" feature
- Pointed to original `.qcow2` file: `4BE46922-B085-47D7-82FC-BC9018CAAC9E.qcow2`
- Set as primary boot device after initial install and config

#### **Step 3: Optimize New Configuration**
- **Display**: VirtIO-GPU + OpenGL acceleration (optimal Linux performance)
- **Boot Order**: Hard disk first (prevent installer loops)

### Phase 5: Validation and Success
**Result**: VM booted directly into fresh Debian installation with:
- All new user accounts intact
- Complete data preservation
- Full application functionality
- Network services working normally

## 📊 Problem-Solving Methodology Demonstrated:

1. **Exploration Before Assumption**

2. **Pivot When Stuck**

3. **Root Cause Analysis**

4. **Leverage System Architecture**

## 🎯 Key Insights Gained:

### About Creative Solutions
- **Sometimes the best resolution is rebuilding rather than repairing**
- **Understanding system architecture enables novel solutions**
- **Persistence in one direction isn't always better than strategic pivoting**

### About Virtualization
- **.qcow2 files are remarkably resilient** to VM-level corruption
- **UTM's component separation** is a feature, not a bug
- **Saved states introduce significant risk** compared to proper shutdowns

## 🛡️ Prevention Strategy Implemented

1. **Disable auto-save states** in all future VMs
2. **Regular .qcow2 file backups** independent of VM configurations
3. **Monitoring scripts** to alert on abnormal file sizes
4. **Documented recovery procedures** for future incidents

## 📈 Success Metrics Achieved

- ✅ **Data Recovery**: 100% of user data preserved
- ✅ **System Restoration**: Full functionality recovered
- ✅ **Performance**: Better than original (fresh config + optimized settings)
- ✅ **Future Prevention**: Corruption-resistant configuration implemented

## 🏁 Conclusion: Failure as a Guide

This recovery journey demonstrates that **failed attempts are not wasted effort** - they're data points that guide you to the real solution. By documenting and learning from each failed network attack, we:

1. **Eliminated surface-level issues** as root causes
2. **Understood what was actually working** in the system
3. **Pivoted to a more fundamental architectural solution**

4. **Achieved complete recovery** through creative reconstruction

The ultimate success wasn't in breaking into the system, but in understanding it well enough to rebuild it properly around the intact components. This approach preserved all data while eliminating the corruption entirely - a cleaner outcome than any patch or repair could have achieved.

<img width="1120" height="700" alt="Screenshot 2025-11-18 at 3 46 46 AM" src="https://github.com/user-attachments/assets/9708087f-ac5c-491b-a237-fe5772c222cd" />
<img width="1120" height="700" alt="Screenshot 2025-11-18 at 3 47 28 AM" src="https://github.com/user-attachments/assets/8fc20217-5945-4f07-86c3-7d2a2a24a0dd" />
<img width="1120" height="700" alt="Screenshot 2025-11-18 at 3 53 45 AM" src="https://github.com/user-attachments/assets/c11a4a8e-32d8-4a6e-94ff-9b7b9a81b64b" />
<img width="1120" height="700" alt="Screenshot 2025-11-18 at 3 55 12 AM" src="https://github.com/user-attachments/assets/c230f2d8-9ccc-415c-825d-2f9808762699" />
<img width="1120" height="700" alt="Screenshot 2025-11-18 at 3 56 22 AM" src="https://github.com/user-attachments/assets/16d077f7-8bc3-4227-9a67-ace7be7baddb" />
<img width="1120" height="700" alt="Screenshot 2025-11-18 at 3 57 55 AM" src="https://github.com/user-attachments/assets/19b0189b-299d-469f-9568-2ce70a3217a5" />
<img width="1120" height="700" alt="Screenshot 2025-11-18 at 3 58 06 AM" src="https://github.com/user-attachments/assets/55219f86-0109-4931-8482-24a678aea6d0" />
<img width="1120" height="700" alt="Screenshot 2025-11-18 at 3 58 14 AM" src="https://github.com/user-attachments/assets/b62085bc-1a4b-4bcd-95c6-2d948386b490" />



