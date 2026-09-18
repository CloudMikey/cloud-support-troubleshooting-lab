# RCA — SSH: Connection Timed Out vs. Permission Denied (publickey)

## 1. Symptom
"I can't SSH into my EC2 instance." Two separate incidents, same instance, same key file — different error each time.

## 2. Hypotheses
- **Network-layer block** (Security Group, NACL, routing) — ruled in first since it's the most common cause and cheapest to check.
- **Instance not running / crashed** — ruled out; instance was confirmed running in the console before testing.
- **Wrong key file / wrong username** — ruled out; same key and `ubuntu` user used successfully earlier in the session.
- **Auth-layer rejection** (bad key permissions on the server, corrupted `authorized_keys`) — considered once the network-layer test came back clean.

## 3. Diagnostics
**Incident 1 — Connection timed out**
- Removed the port 22 inbound rule from the instance's Security Group (`launch-wizard-1`).
- Ran `ssh -i cloud-lab-key.pem ubuntu@<EC2_PUBLIC_IP>`.
- Result: `ssh: connect to host <EC2_PUBLIC_IP> port 22: Connection timed out`.
- This eliminates auth, DNS, and the instance itself as causes — the TCP handshake never completed, meaning the packet was silently dropped before reaching sshd. That signature is specific to a stateful firewall drop (Security Group), not a refused connection.

**Incident 2 — Permission denied (publickey)**
- Restored the port 22 rule, confirmed login worked.
- Ran `chmod 777 ~/.ssh/authorized_keys` on the instance.
- Ran `ssh -i cloud-lab-key.pem ubuntu@<EC2_PUBLIC_IP>`.
- Result: `ubuntu@<EC2_PUBLIC_IP>: Permission denied (publickey)`.
- This eliminates the network layer — the TCP/SSH handshake completed and the server actively responded with a rejection, rather than staying silent. `sshd` refuses to trust an `authorized_keys` file that's group- or world-writable, regardless of whether the key itself is valid.

## 4. Root Cause
Two independent causes producing two distinguishable errors:
1. Security Group had no inbound rule for port 22 → connection timed out at the network layer.
2. `authorized_keys` permissions were `777` (world-writable) → `sshd` rejected the otherwise-valid key at the auth layer.

## 5. Fix + Prevention
- Re-added the port 22 inbound rule scoped to `<HOME_IP>/32` (not `0.0.0.0/0`).
- Ran `chmod 600 ~/.ssh/authorized_keys` to restore the owner-only permissions `sshd` requires.
- Verified with a clean login on both fixes.
- **Prevention:** monitor Security Group changes via CloudTrail/EventBridge for unexpected rule deletions; a bootstrap script or `cron` check could periodically enforce `600` on `authorized_keys` to catch accidental `chmod` mistakes before they lock out access.

![Connection timed out](screenshots/01-error-connection-timeout.png)
![Permission denied](screenshots/02-error-permission-denied.png)
![Verified login](screenshots/03-verified-login-success.png)
