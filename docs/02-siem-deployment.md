# Phase 2 — Standing Up the SIEM

Wazuh 4.13.1 — indexer, manager, and dashboard — deployed as a single-node Docker stack on the
Ubuntu Server VM. This is the component the whole lab exists to exercise: it ingests endpoint
telemetry, correlates it against a rule set, and raises alerts.

All work done over SSH from the desktop rather than the VM console
([I16](issues-log.md#i16--console-unusable-for-real-work)).

## Sequence

**1. Kernel parameter first.** The indexer is OpenSearch underneath and will crash on first start
without a raised memory-map limit. Set it and persist it before anything else — see
[`configs/99-wazuh-sysctl.conf`](../configs/99-wazuh-sysctl.conf).

```bash
sudo sysctl -w vm.max_map_count=262144
cat /proc/sys/vm/max_map_count   # verify before continuing
```

**2. Install Docker**, add the working user to the `docker` group, and log out and back in for the
group membership to apply.

**3. Clone the vendor's Docker repository pinned to the 4.13.1 tag**, then generate the indexer
certificates with the provided generator compose file. Pinning matters — an unpinned deployment
drifts the moment upstream tags move.

**4. Pull and start.** Expect the pull to be the hard part
([I14](issues-log.md#i14--docker-image-pulls-stall-or-reset)). If layers stall, apply
[`configs/docker-daemon.json`](../configs/docker-daemon.json) to force IPv4 and serialize downloads,
then re-run the pull — it resumes from banked progress.

```bash
sudo docker compose pull
sudo docker compose up -d
sudo docker compose ps    # expect three containers: indexer, manager, dashboard
```

**5. Verify reachability, then wait.**

```bash
hostname -I                        # must be 192.168.x.x, not 10.0.2.x  (I15)
curl -k -I https://localhost:443   # expect a response once initialization completes
```

A dead dashboard immediately after startup is normal — the indexer has to reach a healthy state
before the dashboard serves, and startup is slow on this host. Wait and watch the indexer logs
rather than restarting, which only resets the clock
([I18](issues-log.md#i18--dashboard-unreachable-immediately-after-startup)).

## What this phase actually taught

The install is a documented sequence. The work was diagnostic:

- An OS edition chosen without checking the vendor support matrix, caught only after it filled the
  disk mid-install ([I6](issues-log.md#i6--ubuntu-release-chosen-against-the-vendor-support-matrix),
  [I11](issues-log.md#i11--wrong-ubuntu-edition-installed)).
- Repeated transfer failures that looked like a registry problem and were actually IPv6 over an
  unstable wireless bridge — diagnosed by changing one variable at a time
  ([I14](issues-log.md#i14--docker-image-pulls-stall-or-reset)).
- A NAT-versus-bridged mixup that governs whether anything on the network can reach the SIEM at all
  ([I15](issues-log.md#i15--host-unreachable-from-the-rest-of-the-lan)).

The stack ships with a published default password at this point. Rotating it is the first task of
Phase 3 — before a single endpoint is enrolled.

**Next:** [Phase 3 — Hardening, enrollment, and ingestion](03-endpoints-and-ingestion.md)
