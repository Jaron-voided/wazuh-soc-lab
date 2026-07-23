# Phase 1 — Building the Range

Four machines on one bridged LAN: an attacker, two victims, and the host that will become the SIEM.
Bridged networking is the point — every machine gets a real address on the network, so the SIEM is
reachable the way it would be in a real environment and agents can actually find it.

## Target layout

| Machine | Role | Firmware | Notes |
|---|---|---|---|
| Kali Linux | Attacker | n/a | Persistent live USB on a separate laptop, not a VM |
| Ubuntu Server 24.04 LTS | SIEM host | EFI | 6–8 GB RAM, 2–4 vCPU, 50 GB dynamic disk |
| Windows 11 Enterprise (eval) | Modern victim | EFI | 8 GB RAM, 2+ vCPU |
| Metasploitable 2 | Legacy victim | BIOS | 512 MB–1 GB RAM, 1 vCPU |

All adapters set to **Bridged**. Storage lives on a dedicated volume rather than the system drive.

## Build notes

**Storage first.** Create the lab volume and point the hypervisor's default machine folder at it
*before* creating any VM — otherwise VMs silently land on the system drive
([I5](issues-log.md#i5--virtualbox-silently-placing-vms-on-the-system-drive)). Confirm the volume
actually formatted ([I4](issues-log.md#i4--lab-volume-inaccessible-reported-as-raw)).

**Metasploitable 2** ships as an existing virtual disk, so it's attached rather than installed.
Two settings decide whether it boots at all: the disk must be on an **IDE** controller, because the
guest kernel predates common SATA support
([I7](issues-log.md#i7--metasploitable-disk-attached-to-the-wrong-controller)), and **EFI must stay
disabled**, because it uses a legacy BIOS bootloader
([I8](issues-log.md#i8--efi-enabled-on-a-legacy-image)).

Smoke test: boot it, note its address, then from Kali confirm it responds to a ping and that a port
scan returns the expected wide-open service list.

**Windows 11** installs from the evaluation ISO with EFI enabled. Note that Microsoft publishes both
an ISO and a prebuilt appliance image — they take different import paths and aren't interchangeable
([I3](issues-log.md#i3--windows-11-iso-wont-appear-under-import-appliance)).

**Ubuntu Server 24.04 LTS** — the **Server** edition specifically
([I11](issues-log.md#i11--wrong-ubuntu-edition-installed)), and 24.04 specifically, because that's
what the SIEM vendor supports; the download page defaults to something newer
([I6](issues-log.md#i6--ubuntu-release-chosen-against-the-vendor-support-matrix)). Enable OpenSSH
during installation — the console is not usable for real work
([I16](issues-log.md#i16--console-unusable-for-real-work)). Size the disk at 50 GB dynamic
([I12](issues-log.md#i12--virtual-disk-filled-to-zero-bytes-free)).

**Kali** runs from a persistent live USB on a separate physical laptop, which keeps the attacker off
the same hypervisor as the targets. At boot, select the persistence option — the plain live option
discards changes. Verify persistence by creating a file and rebooting before relying on it.

## Operating notes

- Snapshot each victim **before** attacking it. The SIEM host is infrastructure — never a target,
  never rolled back mid-investigation.
- Prefer graceful shutdown over power-off; abrupt power loss can corrupt the indexer's data.
- Attacks target specific lab addresses only, never a subnet range — the lab is bridged onto a real
  network.

**Next:** [Phase 2 — SIEM deployment](02-siem-deployment.md)
