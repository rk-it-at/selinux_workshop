---
marp: true
title: SELinux Workshop
theme: rk-it
size: 16:9
paginate: true
footer: SELinux Workshop 03/2026
---

# SELinux Workshop
## Understand SELinux with hands-on exercises (RHEL 9/10)

<!-- _notes:
Welcome the group, confirm schedule and labs, explain this is hands-on.
-->

---

<!-- _class: right-bg-author -->
# About me

- Rene Koch
- Self-employed consultant for:
  - Red Hat Ansible (Automation Platform)
  - Red Hat Enterprise Linux
  - Red Hat Satellite
  - Red Hat Identity Management (IPA)

<!-- _notes:
Very short intro; emphasize practical focus and RHEL background.
-->

---

<!-- _class: right-bg-author -->
# About me

- Rene Koch
  - rkoch@rk-it.at
  - +43 660 / 464 0 464
  - https://www.linkedin.com/in/rk-it-at
  - https://github.com/rk-it-at
  - https://github.com/scrat14

<!-- _notes:
Very short intro; emphasize practical focus and RHEL background.
-->

---

# Goals for today

- Understand what SELinux is and why it exists
- Explain DAC vs MAC with clear examples
- Read and interpret SELinux AVC logs
- Use audit2allow, setenforce, and booleans safely
- Fix real-world web issues (custom webroot, PHP access, outbound connections)
- Know what we are not doing: writing custom policy modules

<!-- _notes:
Set expectations and remind that we will not write custom policy modules.
-->

---

# Target environment

- RHEL 9 or RHEL 10
- SELinux in **enforcing** mode
- **root** access to system

<!-- _notes:
Confirm versions and packages; ask participants to verify SELinux enforcing.
-->

---

# Agenda (8 hours)

- 09:00-09:30 Introduction, SELinux overview
- 09:30-10:30 DAC vs MAC, SELinux concepts
- 10:30-10:45 Break
- 10:45-12:00 SELinux policy, labels, and file contexts (labs)
- 12:00-13:00 Lunch
- 13:00-14:00 Logs and troubleshooting workflow (labs)
- 14:00-14:15 Break
- 14:15-15:30 Booleans and common service scenarios (labs)
- 15:30-16:45 Web troubleshooting workshop (labs)
- 16:45-17:00 Wrap-up, Q&A

<!-- _notes:
Walk through timing, highlight breaks and lab-heavy afternoon.
-->

---

# Ground rules

- Everyone works on their own VM
- Ask questions anytime
- If you get stuck, show your last command and the error
- We keep SELinux enforcing throughout

<!-- _notes:
Encourage questions and sharing errors; enforcing mode stays on.
-->

---

# Lab setup

- You need root access
- Use two terminals:
  - one for commands
  - one for logs:
    ```bash
    $ sudo journalctl -f
    ```
    or
    ```bash
    $ sudo tail -f /var/log/audit/audit.log
    ```

<!-- _notes:
Ask everyone to open a log terminal now.
-->

---
# Exercise
## LAB 1: Access and baseline checks (10 min)

<!-- _notes:
Introduce LAB 1: Access and baseline checks (10 min) and expected outcome.
-->
---

# LAB 1: Access and baseline checks (10 min)

- SSH into your VM
- Switch to root:
  ```bash
  $ sudo -i
  ```
  or
  ```bash
  $ su -
  ```
- Tail the audit log:
  ```bash
  $ tail -f /var/log/audit/audit.log
  ```

---

# LAB 1: Access and baseline checks (10 min)

- Check RHEL version:
  ```bash
  $ cat /etc/redhat-release
  Red Hat Enterprise Linux release 10.1 (Coughlan)
  ```
- Verify repos:
  ```bash
  $ dnf repolist
  repo id                               repo name
  rhel-10-for-x86_64-appstream-rpms     Red Hat Enterprise Linux 10 for x86_64 - AppStream (RPMs)
  rhel-10-for-x86_64-baseos-rpms        Red Hat Enterprise Linux 10 for x86_64 - BaseOS (RPMs)
  ```

<!-- _notes:
Make sure everyone can log in, become root, and confirm RHEL version and repos.
-->

---
# Chapter 1
## Introduction and SELinux overview

<!-- _notes:
Start with the motivation and high-level view of SELinux.
-->

---

# What is SELinux?

- SELinux = Security-Enhanced Linux
- Mandatory Access Control (MAC) on top of DAC
- Enforces least privilege based on labels and policy
- Default in RHEL and trusted OS base for security compliance

<!-- _notes:
High-level definition and why it matters for containment.
-->

---

# History of SELinux

- Developed by the NSA as research into MAC for Linux
- First released to the open source community in 2000 (GPL)
- Merged into the mainline Linux kernel in the 2.6 series
- In RHEL, SELinux is enabled and enforcing by default on install

<!-- _notes:
Keep this short: origin, open source release, kernel integration, and default RHEL posture.
-->

---

# Compliance drivers (NIS2, DORA, benchmarks)

- Regulations like NIS2 and DORA push stronger security controls and auditability
- Many hardening baselines (CIS, DISA STIGs, internal policies) recommend SELinux enforcing
- Treat requirements as organization-specific: confirm with your policy or auditors

<!-- _notes:
Keep this non-legal; emphasize that compliance frameworks typically favor MAC/SELinux.
-->

---

# Why SELinux is useful (1/5)

- Enforces least privilege with labels and policy rules (deny by default)
- Confines services so a compromise has limited reach
- Complements DAC rather than replacing it

<!-- _notes:
Frame SELinux as containment and damage reduction, not a silver bullet.
-->

---

# Why SELinux is useful (2/5) Impact: Critical

- Log4Shell `CVE-2021-44228` (link: `https://access.redhat.com/security/cve/CVE-2021-44228`)
- Critical issues often allow remote, unauthenticated arbitrary code execution
- SELinux confines the vulnerable service, limiting the attack’s reach (e.g., sensitive files or further payloads)
- Reference: Red Hat SELinux hardening blog (link: `https://www.redhat.com/en/blog/selinux-and-rhel-technical-exploration-security-hardening`)

<!-- _notes:
Reference: Red Hat blog on SELinux hardening impact examples.
-->

---

# Why SELinux is useful (3/5) Impact: Important

- Looney Tunables `CVE-2023-4911` (link: `https://access.redhat.com/security/cve/CVE-2023-4911`)
- Important issues can enable privilege escalation after an exploit
- SELinux policies can confine the elevated process and protect critical system components
- Reference: Red Hat SELinux hardening blog (link: `https://www.redhat.com/en/blog/selinux-and-rhel-technical-exploration-security-hardening`)

<!-- _notes:
Reference: Red Hat blog on SELinux hardening impact examples.
-->

---

# Why SELinux is useful (4/5) Impact: Moderate

- Grafana issue `CVE-2023-3128` (link: `https://access.redhat.com/security/cve/CVE-2023-3128`)
- Moderate issues are often harder to exploit or require unlikely configurations
- SELinux adds a defense layer that helps prevent escalation of impact
- Reference: Red Hat SELinux hardening blog (link: `https://www.redhat.com/en/blog/selinux-and-rhel-technical-exploration-security-hardening`)

<!-- _notes:
Reference: Red Hat blog on SELinux hardening impact examples.
-->

---

# Why SELinux is useful (5/5) Impact: Low

- vim crash `CVE-2023-48232` (link: `https://access.redhat.com/security/cve/CVE-2023-48232`)
- Low-severity issues usually have limited consequences
- SELinux may not directly prevent the issue, but consistent enforcement helps contain theoretical attacks
- Reference: Red Hat SELinux hardening blog (link: `https://www.redhat.com/en/blog/selinux-and-rhel-technical-exploration-security-hardening`)

<!-- _notes:
Reference: Red Hat blog on SELinux hardening impact examples.
-->

---
# Chapter 2
## DAC vs MAC and SELinux concepts

<!-- _notes:
Contrast DAC and MAC, then move into labels and types.
-->
---

# DAC recap (traditional Linux)

- Owner/group/other permissions
- Discretionary: user can change permissions
- Example: `chmod 777` opens everything
- Root can do almost anything

<!-- _notes:
Quick reminder; show why DAC alone is insufficient.
-->

---

# MAC vs DAC

- DAC: access based on identity (uid/gid)
- MAC: access based on labels and policy rules
- SELinux can still deny access even if DAC allows
- Goal: limit damage when a process is compromised

<!-- _notes:
Emphasize MAC can deny even when DAC allows.
-->

---

# SELinux key concepts

- Subjects (processes) and objects (files, sockets)
- Security contexts: user:role:type:level
- Type Enforcement (TE) is the core decision model
- Policy defines allowed interactions between types

<!-- _notes:
Define subject/object/types; stress TE as core model.
-->

---

# Context examples

- `system_u:system_r:httpd_t:s0`
- `unconfined_u:unconfined_r:unconfined_t:s0`
- `system_u:object_r:httpd_sys_content_t:s0`

<!-- _notes:
Point out the type field is most important for decisions.
-->

---

# Modes

- Enforcing: policy is applied
- Permissive: only logs, no denial
- Disabled: SELinux off (do not use)

<!-- _notes:
Explain when permissive is acceptable and why disabled is not.
-->

---

# Tools we will use

- `getenforce`, `setenforce`
- `ls -Z`, `ps -eZ`, `id -Z`
- `semanage fcontext`, `restorecon`
- `ausearch`, `sealert`, `audit2allow`
- `getsebool`, `setsebool`

<!-- _notes:
Preview the toolchain; mention we will practice each one.
-->

---
# Exercise
## LAB 2: Inspect contexts (15 min)

<!-- _notes:
Introduce LAB 2: Inspect contexts (15 min) and expected outcome.
-->
---

# LAB 2: Inspect contexts (15 min)

- Check current mode: `getenforce`
- List file contexts: `ls -Z /var/www`
- Inspect process contexts: `ps -eZ | head`
- Check your user context: `id -Z`

<!-- _notes:
Give 10–12 minutes, then regroup for observations.
-->

---

# LAB 2: Expected observations

- `httpd` runs as `httpd_t`
- web content is labeled `httpd_sys_content_t`
- your shell is `unconfined_t`

<!-- _notes:
Verify everyone sees httpd_t and httpd_sys_content_t.
-->

---
<!-- _class: break-slide -->
![bg cover](assets/break.png)

<!-- _notes:
Time-box the break and announce restart time.
-->
---
# Chapter 3
## Policy, labels, and file contexts

<!-- _notes:
Transition into labeling and persistent contexts.
-->
---

# File contexts and labels

- Every file has a label
- Label determines which domains can access it
- Label mismatch causes denials even with correct DAC perms

<!-- _notes:
Reinforce labels drive access, not just permissions.
-->

---

# Changing labels the right way

- Temporary: `chcon`
- Persistent: `semanage fcontext` + `restorecon`
- Always prefer persistent rules

<!-- _notes:
Explain why chcon is temporary; semanage is persistent.
-->

---
# Exercise
## LAB 3: Fix a mislabeled webroot (30 min)

<!-- _notes:
Introduce LAB 3: Fix a mislabeled webroot (30 min) and expected outcome.
-->
---

# LAB 3: Fix a mislabeled webroot (30 min)

- Create a new directory `/srv/webroot`
- Add a test page
- Configure httpd to use this path
- Observe denial
- Fix with proper context

<!-- _notes:
Let them break it first; troubleshooting starts here.
-->

---

# LAB 3: Commands (example)

- `sudo mkdir -p /srv/webroot`
- `echo ok | sudo tee /srv/webroot/index.html`
- `sudo cp /etc/httpd/conf/httpd.conf /etc/httpd/conf/httpd.conf.bak`
- Edit `DocumentRoot` and `<Directory>` to `/srv/webroot`
- `sudo systemctl restart httpd`
- Browse: `curl http://localhost`

<!-- _notes:
Offer as guidance, but let them edit config themselves.
-->

---

# LAB 3: Fix context

- `sudo semanage fcontext -a -t httpd_sys_content_t '/srv/webroot(/.*)?'`
- `sudo restorecon -Rv /srv/webroot`
- Retry: `curl http://localhost`

<!-- _notes:
Show semanage + restorecon pattern.
-->

---
<!-- _class: break-slide -->
![bg cover](assets/lunch.png)

<!-- _notes:
Confirm return time and remind to keep terminals open.
-->
---
# Chapter 4
## Logs and troubleshooting workflow

<!-- _notes:
Set the troubleshooting mindset before showing logs.
-->
---

# SELinux logging basics

- Denials appear as AVC messages
- Sources:
  - `/var/log/audit/audit.log`
  - `journalctl -t setroubleshoot` (if installed)

<!-- _notes:
Show where AVCs appear; mention auditd and setroubleshoot.
-->

---

# Interpreting an AVC message

- What (class): file, dir, tcp_socket
- Source: process type (e.g., `httpd_t`)
- Target: object type (e.g., `default_t`)
- Permission: `read`, `write`, `name_connect`, etc.

<!-- _notes:
Walk line by line: source, target, permission.
-->

---
# Exercise
## LAB 4: Read AVC logs (20 min)

<!-- _notes:
Introduce LAB 4: Read AVC logs (20 min) and expected outcome.
-->
---

# LAB 4: Read AVC logs (20 min)

- Trigger a denial (from LAB 2 before fix)
- Find it:
  - `sudo ausearch -m AVC,USER_AVC -ts recent`
  - `sudo journalctl -t audit -n 50`
- Identify source type, target type, and permission

<!-- _notes:
Have them identify source/target/perms from one AVC.
-->

---

# Troubleshooting workflow

- 1) Verify mode and context
- 2) Check AVC logs
- 3) Decide: correct label or boolean?
- 4) Use policy change only as last resort

<!-- _notes:
Stress label or boolean first; policy last.
-->

---

# audit2allow and why we are careful

- `audit2allow` suggests policy changes based on logs
- It does not know your intent
- Use for diagnostics, not for blind fixes
- We will not create custom policies in this workshop

<!-- _notes:
Position as learning tool, not production fix.
-->

---

# setenforce

- `setenforce 0` (permissive) for short-term debugging
- Always switch back: `setenforce 1`
- We keep enforcing unless explicitly instructed

<!-- _notes:
Use briefly only; always return to enforcing.
-->

---
# Exercise
## LAB 5: setenforce practice (10 min)

<!-- _notes:
Introduce LAB 5: setenforce practice (10 min) and expected outcome.
-->
---

# LAB 5: setenforce practice (10 min)

- Check current mode
- Switch to permissive, reproduce denial, switch back
- Confirm logs are still generated

<!-- _notes:
Confirm denials still logged in permissive.
-->

---
<!-- _class: break-slide -->
![bg cover](assets/break.png)

<!-- _notes:
Time-box the break and announce restart time.
-->
---

# SELinux booleans

- Tunables for common service behaviors
- Examples: `httpd_can_network_connect`, `httpd_enable_homedirs`
- Persistent change with `-P`

<!-- _notes:
Explain booleans as safe, supported toggles.
-->

---
# Exercise
## LAB 6: Boolean discovery (20 min)

<!-- _notes:
Introduce LAB 6: Boolean discovery (20 min) and expected outcome.
-->
---

# LAB 6: Boolean discovery (20 min)

- `getsebool -a | grep httpd`
- Identify a boolean that enables a needed behavior
- Enable it and verify

<!-- _notes:
Let them search and discuss which boolean fits.
-->

---
# Chapter 6
## Web troubleshooting workshop

<!-- _notes:
Move into real-world web app fixes.
-->
---

# Practical scenario: PHP needs outbound access

- Symptom: PHP app cannot call external API
- AVC shows `name_connect` denied for `httpd_t`
- Fix: enable `httpd_can_network_connect`

<!-- _notes:
Connect to real apps; show typical AVC pattern.
-->

---
# Exercise
## LAB 7: Enable outbound connections (20 min)

<!-- _notes:
Introduce LAB 7: Enable outbound connections (20 min) and expected outcome.
-->
---

# LAB 7: Enable outbound connections (20 min)

- Create a simple PHP script that curls a URL
- Observe denial
- `sudo setsebool -P httpd_can_network_connect on`
- Verify success

<!-- _notes:
Use httpd_can_network_connect and verify success.
-->

---

# Practical scenario: User content served by httpd

- Use home directories as web content
- Need `httpd_enable_homedirs` and labels
- `public_content_t` may apply

<!-- _notes:
Show how homedirs differ and why extra steps required.
-->

---
# Exercise
## LAB 8: Serve user content (25 min)

<!-- _notes:
Introduce LAB 8: Serve user content (25 min) and expected outcome.
-->
---

# LAB 8: Serve user content (25 min)

- Create `/home/student/public_html/index.html`
- Enable boolean for homedirs
- Set context with `httpd_sys_content_t` or `httpd_user_content_t`
- Verify via curl

<!-- _notes:
Emphasize both boolean and labels are needed.
-->

---

# Practical scenario: Custom webroot with uploads

- Separate static vs writable content
- Writable content must be labeled correctly
- Example: `httpd_sys_rw_content_t`

<!-- _notes:
Explain rw label split between static and writable.
-->

---
# Exercise
## LAB 9: Upload directory (25 min)

<!-- _notes:
Introduce LAB 9: Upload directory (25 min) and expected outcome.
-->
---

# LAB 9: Upload directory (25 min)

- Create `/srv/webroot/uploads`
- Set type: `httpd_sys_rw_content_t`
- Validate using a simple PHP upload or `touch` via httpd

<!-- _notes:
Warn about over-labeling; keep scope tight.
-->

---

# Common mistakes

- Using `chcon` without `semanage`
- Relabeling entire filesystem without understanding
- Turning SELinux off
- Copying audit2allow output into production

<!-- _notes:
Share real-world pitfalls and quick fixes.
-->

---

# Quick reference cheatsheet

- Check mode: `getenforce`
- Inspect labels: `ls -Z`, `ps -eZ`
- Fix labels: `semanage fcontext` + `restorecon`
- Logs: `ausearch -m AVC -ts recent`
- Booleans: `getsebool -a`, `setsebool -P`

<!-- _notes:
Tell them to keep this for after class.
-->

---
# Exercise
## Mini challenge (30 min)

<!-- _notes:
Introduce Mini challenge (30 min) and expected outcome.
-->
---

# Mini challenge (30 min)

- You get a broken httpd app
- Identify the SELinux issue(s)
- Fix without disabling SELinux
- Document your steps

<!-- _notes:
Let them work in pairs; ask for their reasoning.
-->

---
# Chapter 7
## Wrap-up and Q&A

<!-- _notes:
Summarize and invite questions.
-->
---

# Wrap-up

- SELinux enforces MAC on top of DAC
- Most issues are labeling or booleans
- Logs are your truth source
- Keep enforcing, use least privilege

<!-- _notes:
Reinforce key takeaways: labels, booleans, logs.
-->

---

# Q&A

- What scenarios do you want to try next?
- Ideas for advanced follow-up workshops?

<!-- _notes:
Invite follow-up topics and advanced ideas.
-->

---

# Link list

- Linux kernel SELinux docs: https://docs.kernel.org/admin-guide/LSM/SELinux.html
- Red Hat SELinux hardening blog: https://www.redhat.com/en/blog/selinux-and-rhel-technical-exploration-security-hardening
- RHEL SELinux docs: https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/using_selinux/index
- SELinux Notebook (SELinuxProject): https://github.com/SELinuxProject/selinux-notebook
- SELinux Project documentation: https://selinuxproject.github.io/documentation/

<!-- _notes:
Point participants to the notebook and upstream docs for follow-up reading.
-->

---

# Abbreviations

- API — Application Programming Interface
- AVC — Access Vector Cache (denial message)
- DAC — Discretionary Access Control
- MAC — Mandatory Access Control
- PHP — Hypertext Preprocessor
- RHEL — Red Hat Enterprise Linux
- SELinux — Security-Enhanced Linux
- SSH — Secure Shell
- TE — Type Enforcement
- VM — Virtual Machine

<!-- _notes:
These match abbreviations used throughout the labs.
-->

---

# Terminology

- AVC/audit log: denial records stored by auditd
- Boolean: runtime policy toggle for common behaviors
- Domain: type assigned to processes (e.g., `httpd_t`)
- Enforcing/Permissive: SELinux modes with/without blocking
- Label: context on files, sockets, and other objects
- Policy: rules that allow or deny type interactions
- Relabel/restorecon: apply default contexts from policy mappings
- Security context: label in the form `user:role:type:level`
- Type: TE label used for access decisions

<!-- _notes:
Keep definitions brief; expand only if time permits.
-->
