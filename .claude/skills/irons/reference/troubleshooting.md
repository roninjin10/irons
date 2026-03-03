# Troubleshooting

## TLS/SSL Connection Errors

**Symptoms:** "SSL syscall error" or "TLS handshake failed" inside a sandbox.

**Cause:** Egress rules are blocking the outbound connection.

**Fix:**

1. Check audit logs for DENIED entries:
   ```
   irons audit egress --vm my-sandbox --verdict blocked
   ```
2. Add the required domain:
   ```
   irons egress add --host example.com
   ```
3. Or temporarily switch to warn mode to discover all needed domains:
   ```
   irons egress mode warn
   irons audit egress --vm my-sandbox --follow
   ```

## Sandbox Stuck / Not Ready

**Symptoms:** Sandbox creation hangs or never reaches ready state.

**Fix:**

1. Check status:
   ```
   irons status my-sandbox
   ```
   - 🟢 running / 🟡 starting / 🟠 stopped / 🔴 error
2. If stuck in error, destroy and recreate:
   ```
   irons destroy my-sandbox --force
   irons create my-sandbox
   ```

## SSH Connection Refused

**Symptoms:** `ssh: connect to host ... port 2222: Connection refused`

**Causes:**
- Sandbox still starting — wait for ready status
- SSH key mismatch — ensure the key used matches what was specified at creation (defaults to first key in `~/.ssh`)

**Fix:**

1. Verify the sandbox is ready:
   ```
   irons status my-sandbox
   ```
2. If key mismatch, destroy and recreate with the correct key:
   ```
   irons destroy my-sandbox --force
   irons create my-sandbox --key ~/.ssh/id_ed25519.pub
   ```

## Authentication Errors

**Symptoms:** `401 Unauthorized` or "invalid API key" errors.

**Fix:**

1. Check that the API key is set:
   ```
   echo $IRONS_API_KEY
   ```
2. If missing, export it:
   ```
   export IRONS_API_KEY=your-api-key
   ```
3. For persistence, add the export to your shell profile, or run:
   ```
   irons login
   ```

## IP Address Connection Errors

**Symptoms:** Cannot connect to a service by raw IP address from a sandbox.

**Cause:** Egress rules are domain-based. The bridge inspects TLS SNI / HTTP Host headers for hostnames. Raw IP connections have no hostname to match.

**Fix:** Use CIDR rules for IP-based access:
```
irons egress add --cidr 203.0.113.0/24 --name "my-service"
```

For outbound SSH to specific IPs, CIDR rules are the only option (see [Sandbox Model](./sandbox-model.md#outbound-ssh)).

## Support

For unresolved issues, contact **support@iron.sh**.
