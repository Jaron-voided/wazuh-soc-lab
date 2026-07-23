# Connecting to the legacy host over SSH

Modern OpenSSH removed SHA-1-based host key algorithms from its defaults. The legacy host offers
only those, so current clients refuse the connection
([I22](../docs/issues-log.md#i22--ssh-refuses-to-connect-to-the-legacy-host)).

Re-enable them **per connection**:

```bash
ssh -o HostKeyAlgorithms=+ssh-rsa \
    -o PubkeyAcceptedAlgorithms=+ssh-rsa \
    <user>@<legacy-host-ip>
```

If key exchange or cipher negotiation also fails, add the specific legacy algorithms the server
requires — one at a time, based on the actual error, rather than pasting a broad compatibility
block.

## Do not do this globally

The convenient fix is adding these options to `~/.ssh/config` without a `Host` restriction. That
silently downgrades **every** SSH connection the client makes, including to correctly configured
servers, turning a one-host problem into an every-connection problem.

If it needs to be persistent, scope it explicitly:

```
Host legacy-lab-host
    HostName <ip>
    HostKeyAlgorithms +ssh-rsa
    PubkeyAcceptedAlgorithms +ssh-rsa
```

Written up as a finding: [FIND-001](../findings/FIND-001-deprecated-ssh-host-key-algorithms.md).
