# Issues Log

Every problem worth recording from the build, with the symptom that surfaced it, the actual cause,
and what resolved it. Kept in ticket form because the troubleshooting is the transferable part —
the happy path is just a sequence of commands.

A few entries resolve to **"no action, documented"**. Those are the interesting ones.

**Permanent constraints (accepted, not defects):**

- **No hardware virtualization passthrough.** The Windows host runs Hyper-V-backed services, so
  VirtualBox guests run without AMD-V. VMs and containers are slow to start. Disabling the
  hypervisor would fix it at the cost of breaking other services on the host, so it stays.
  Expect multi-minute container startup times.
- **IPv6 instability over the wireless bridge.** Causes sustained transfers (Docker pulls, large
  downloads) to stall. Worked around by forcing IPv4; see I14 and I21.

---

## Phase 1 — Host and VM build

### I1 — VirtualBox installer warns about missing Python bindings

**Symptom:** Installer warns that Python core / win32api are unavailable.
**Cause:** Only affects the optional Python scripting SDK.
**Resolution:** No action. The GUI and all VM functionality are unaffected.

### I2 — Windows 11 VM boots to a black screen

**Symptom:** New VM shows a black screen instead of the installer.
**Cause:** Four candidates, checked in order — the guest is mid-reboot; the "press any key to boot
from CD/DVD" prompt expired unseen; wrong graphics controller; EFI not enabled.
**Resolution:** Set Graphics Controller to VBoxSVGA and enable EFI, then restart cleanly. Waiting
30–60 seconds first rules out the mid-reboot case at zero cost.

### I3 — Windows 11 ISO won't appear under Import Appliance

**Symptom:** The downloaded ISO isn't selectable in the import dialog.
**Cause:** Import Appliance accepts only `.ova` / `.ovf`. Microsoft publishes both an evaluation
**ISO** (manual install) and a prebuilt developer **.ova** (import) — they are different products.
**Resolution:** Either create a new VM and install from the ISO, or download the `.ova` and import
that. Don't mix the two paths.

### I4 — Lab volume inaccessible, reported as RAW

**Symptom:** "Volume does not contain a recognized file system."
**Cause:** The format step never completed when the volume was created.
**Resolution:** Format as NTFS via Disk Management. Verify the format finishes rather than assuming
volume creation implies a filesystem.

### I5 — VirtualBox silently placing VMs on the system drive

**Symptom:** Default Machine Folder pointed at a drive letter that didn't exist.
**Cause:** VirtualBox falls back to the system drive without warning when the configured path is
missing — so VMs land on C: while appearing correctly configured.
**Resolution:** Point Default Machine Folder at the real lab path before creating any VM, and audit
existing VMs under Settings → General.

### I6 — Ubuntu release chosen against the vendor support matrix

**Symptom:** The Ubuntu download page defaults to a release newer than Wazuh supports.
**Cause:** Latest-available is not the same as vendor-supported.
**Resolution:** Install 24.04 LTS specifically, from the archived release directory. Check the
vendor's support matrix before picking an OS version — this applies well beyond the lab.

### I7 — Metasploitable disk attached to the wrong controller

**Symptom:** Kernel panic, "no root device," or a black-screen hang at boot.
**Cause:** The image's kernel predates common SATA/AHCI support, so the disk isn't visible when
attached to a SATA controller.
**Resolution:** Attach the `.vmdk` under the **IDE** controller and ensure nothing remains on SATA.

### I8 — EFI enabled on a legacy image

**Symptom:** Metasploitable fails to boot.
**Cause:** It uses a legacy BIOS bootloader.
**Resolution:** Leave EFI disabled. Rule of thumb — legacy images boot BIOS, modern installs
(Windows 11, Ubuntu) boot EFI.

### I9 — New-VM wizard skipped the disk page

**Symptom:** The wizard offers no disk step and unattended install is greyed out.
**Cause:** Expected behavior when no installation ISO is supplied.
**Resolution:** Let the wizard finish, then attach the existing disk via Settings → Storage.

### I10 — Attached disk appears unattached

**Symptom:** Storage details pane shows only dashes.
**Cause:** The empty optical drive slot was selected, not the disk.
**Resolution:** Click the `.vmdk` entry itself to see type, size, and location.

---

## Phase 2 — SIEM deployment

### I11 — Wrong Ubuntu edition installed

**Symptom:** Display-manager errors and the disk filling during installation.
**Cause:** The Desktop ISO was used instead of Server; the desktop environment is far larger than
the VM was sized for.
**Resolution:** Rebuild from the Server ISO — confirm the filename contains `live-server`.

### I12 — Virtual disk filled to zero bytes free

**Symptom:** Installation and updates fail with no space left.
**Cause:** Virtual disk sized too small.
**Resolution:** Recreate as a 50 GB **dynamically allocated** disk, which consumes real space only
as used — so oversizing costs nothing. To grow an existing disk instead: resize the medium, then
extend the logical volume and filesystem inside the guest.

### I13 — VBoxManage not recognized in PowerShell

**Symptom:** Command not found.
**Cause:** The VirtualBox install directory isn't on PATH.
**Resolution:** Invoke by full path, or add the directory to PATH.

### I14 — Docker image pulls stall or reset

**Symptom:** Pulls hang or fail with "connection reset by peer."
**Cause:** Docker defaulting to IPv6 over an unstable wireless bridge, compounded by parallel layer
downloads competing on the same flaky path.
**Resolution:** In order of impact — force IPv4 and single-stream downloads via
[`configs/docker-daemon.json`](../configs/docker-daemon.json); re-run the pull, which resumes from
banked progress; as a last resort switch the adapter to NAT only long enough to finish the
download, then switch back (cached layers survive the change).

### I15 — Host unreachable from the rest of the LAN

**Symptom:** The SIEM has a `10.0.2.x` address; nothing on the network can reach it and agents
can't connect.
**Cause:** The adapter was set to NAT rather than Bridged.
**Resolution:** Use Bridged as the permanent configuration and confirm the host holds a
`192.168.x.x` address. NAT is only ever a temporary measure for downloads. Docker's internal
`172.x` bridges are unrelated and expected.

### I16 — Console unusable for real work

**Symptom:** Fixed low resolution, no clipboard, tiny font.
**Cause:** The server console is not an interactive terminal in any useful sense.
**Resolution:** Enable OpenSSH during installation and do all subsequent work over SSH.

### I17 — Boot hangs at "Loading essential drivers"

**Symptom:** Boot appears frozen.
**Cause:** Usually a network timeout following an adapter change, or the installer ISO left mounted
in the virtual optical drive — not an actual freeze.
**Resolution:** Wait 2–3 minutes; remove the ISO from the optical drive.

### I18 — Dashboard unreachable immediately after startup

**Symptom:** TLS errors or a dead dashboard right after bringing the stack up.
**Cause:** The indexer must reach a healthy state before the dashboard will serve, and startup is
slow without hardware virtualization. This is initialization, not failure.
**Resolution:** Wait 3–8 minutes and watch the indexer logs until they settle. **Do not restart** —
restarting resets the initialization clock and hides the actual state.

---

## Phase 3 — Hardening, enrollment, and ingestion

### I19 — Password hash appeared truncated

**Symptom:** The generated bcrypt hash ends in a period, which reads like sentence punctuation.
**Cause:** bcrypt's alphabet includes `.` and `/`. The trailing character is part of the hash.
**Resolution:** Copy the full 60-character string. Sanity-check the length inside the quotes before
saving.

### I20 — Security tooling reports config directory not found

**Symptom:** `securityadmin.sh` can't locate its configuration directory.
**Cause:** Two separate path assumptions, both wrong. On the host, the config directory sits under
the compose project directory, not the repository root. Inside the indexer container, the layout is
flattened — there is no `config` subdirectory where the documentation implies one.
**Resolution:** Run the tool from inside the container with the config directory set to the
installation directory itself. Verify the path exists before running rather than trusting the docs.

### I21 — Download appears to hang

**Symptom:** A silent `curl` download looks frozen.
**Cause:** Usually no failure at all — `-sO` suppresses progress output, so a working download shows
nothing. If genuinely stalled, it's the IPv6 issue from I14.
**Resolution:** Check the partial file size to distinguish the two. Force IPv4 with a progress bar
when it's real.

### I22 — SSH refuses to connect to the legacy host

**Symptom:** "No matching host key type found."
**Cause:** Modern OpenSSH removed SHA-1-based host key algorithms from its defaults; the legacy host
offers only those.
**Resolution:** Re-enable the deprecated algorithms **per connection**, scoped to that host only —
never globally in `~/.ssh/config`, which would silently weaken every connection made from the
client. See [`configs/ssh-legacy-host.md`](../configs/ssh-legacy-host.md).
**Escalated as a finding:** [FIND-001](../findings/FIND-001-deprecated-ssh-host-key-algorithms.md).

### I23 — Manager configuration edits not taking effect

**Symptom:** Config changes made on the host don't appear in the running manager.
**Cause:** The manager's live configuration lives in a persistent volume that is seeded only when
the container is first created. Editing the host-side template and cycling the stack doesn't
re-copy it.
**Resolution:** Always verify the change loaded by grepping the config *inside* the running
container, and confirm the expected listener appears in the manager log. If the edit didn't apply,
edit the file in the container directly and restart the manager service. The general lesson: never
assume a config change took effect — verify it at the layer that consumes it.

### I24 — Text editor fails on the legacy host

**Symptom:** "Error opening terminal: xterm-256color."
**Cause:** The host's terminfo database predates the terminal type the modern SSH client advertises.
**Resolution:** Use `vi`, or set `TERM=xterm` for the session.

### I25 — Syslog test not visible in Discover

**Symptom:** A test message sent with `logger` never appears in the dashboard, suggesting the
forwarding pipeline is broken.
**Cause:** It isn't. The message matches no decoder, and with full logging disabled, unmatched
events are never indexed. Absence in Discover says nothing about delivery.
**Resolution:** Prove delivery at the layer that can actually answer the question — a packet capture
on the host interface filtered to the syslog port, showing datagrams arriving from the legacy host.
Real attack traffic matches rules and alerts normally.
**Takeaway:** match the verification method to the claim being tested. The dashboard answers "did
this alert," not "did this arrive."

### I26 — In-container socket tools show no listener

**Symptom:** `netstat` / `ss` inside the container report nothing bound to the syslog port, implying
the listener doesn't exist.
**Cause:** Stripped container tooling reports incomplete socket state — a false negative.
**Resolution:** Trust two better sources instead: the manager log line confirming the listener
started, and host-side socket state showing the port forwarder bound. Don't accept a negative result
from a tool that can't see the whole picture.

### I27 — Indexer insecure file permissions warnings

**Symptom:** The indexer's security plugin repeatedly warns that files should be mode 0600.
**Cause:** The flagged files are executables in the plugin's own `bin/` and `tools/` directories.
The check doesn't distinguish executables from secrets.
**Resolution:** **No action, documented.** Applying the recommended permissions strips execute bits
and breaks the security tooling — including the utility required to apply security configuration at
all. The remediation is more damaging than the finding. Recorded here with the reasoning so the
decision is auditable rather than looking like something that was missed.

### I28 — Filebeat connection errors during startup

**Symptom:** "Connection refused" and "security not initialized" errors in the log shipper at
startup.
**Cause:** A startup race — the shipper comes up before the indexer's security plugin finishes
initializing.
**Resolution:** No action. Clears on its own once the indexer reaches a healthy state.
