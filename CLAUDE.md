# CLAUDE.md

## What this repository is

Desired state for `walter-vultr`: one Vultr development machine with OpenTofu
state in Cloudflare R2. `colors.yml` is source; `.colors/` is generated and must
never be edited or committed. Credentials live only in ignored
`.envrc.private` as `COLORS_PAR_*` variables.

The root `green` launcher is a copy of
`.agents/skills/package-walter-green/green`. Keep it synchronized after every
skill update. `skills-lock.json` records the real Package Skill installation.

## Commands

```sh
./green build
./green create --dry-run
./green create
./green stop
./green start
./green delete
```

Build and dry-run need no credentials. Real create/delete and power operations
use `COLORS_PAR_VULTR_API_KEY`; the R2 backend uses
`COLORS_PAR_R2_ACCESS_KEY_ID` and `COLORS_PAR_R2_SECRET_ACCESS_KEY`.

Never export `COLORS_PAR_PROFILE`. Keep `compute-prevent-destroy: true`; lift it
only for one explicitly authorized delete with
`COLORS_PAR_COMPUTE_PREVENT_DESTROY=false`. Never run a real create/delete
without explicit authorization.

## Dedicated machine key

`compute-keygen: true` creates `~/.ssh/walter-vultr`, an ed25519 keypair named
by the deployment profile. An existing file is adopted and never overwritten.
The keypair survives delete. Walter renders a Terraform-managed `vultr_ssh_key`
registration and feeds its ID to the instance; the registration is destroyed
with provider state. OpenTofu's remote-exec runs under a short-lived isolated
ssh-agent loaded with only this key, killed after apply. No personal key or
forwarded agent is involved.

The provider image initially exposes root, but only `walter-ansible-bootstrap`
may use it. That stage creates `ubuntu` with UID/GID 1000, the dedicated key and
passwordless sudo; writes the earliest sshd drop-in with `PermitRootLogin no`
and `PasswordAuthentication no`; validates sshd; and reloads it. Every normal
Ansible stage and `ssh walter-vultr` use ubuntu. Later creates probe ubuntu first
and do not depend on root access Walter has already closed.

## Power and state

`stop` and `start` call Vultr's HTTP API using the immutable instance UUID and
wait for the terminal state. `start` reads the public address live and refreshes
the managed SSH alias. Power state is not declared in OpenTofu, so an out-of-band
stop produces no drift and a later `create` does not start the machine.

After the first create, record `vultr-instance-id` in `colors.yml` so power verbs
still work if the R2 backend is unavailable. Remote state is keyed
`walter-vultr/walter-compute.tfstate`.

## Interactive create

A first create starts GitHub's device flow before any provider action. Run from
an environment without ambient `GITHUB_TOKEN`; otherwise `gh` may reuse that
token instead of minting the machine's own. A failed partial create keeps its
sandboxed token for retry and removes it only after the machine is seeded.

## Git

Work on the current branch. Do not commit or push unless explicitly authorized.
