# walter-vultr

A remote development machine on Vultr, managed with
[walter](https://github.com/getcolors/walter).

```sh
./green build
./green create --dry-run
./green create
ssh walter-vultr
./green stop
./green start
```

`colors.yml` is the only file normally edited. The Vultr API token, R2
credentials, and optional feature credentials are `COLORS_PAR_*` variables in
the ignored `.envrc.private`; never put credentials in desired state and never
export `COLORS_PAR_PROFILE`.

## Machine access

`compute-keygen: true` creates the dedicated persistent keypair
`~/.ssh/walter-vultr`. OpenTofu apply runs under an isolated temporary
`ssh-agent` containing only that key. Walter registers its public half as a
Terraform-managed Vultr SSH key, so the account registration is removed with
the deployment while the local keypair survives for a later recreate.

## Desired machine

The deployment selects Amsterdam (`ams`), Ubuntu 24.04 (`os_id` 2284), and the
`vc2-2c-4gb` plan. State is stored in Cloudflare R2 under
`walter-vultr/walter-compute.tfstate`. Destruction remains protected by default.

A real create begins with GitHub's device flow, then provisions and configures
the machine. Emacs package installation runs in the background after create;
watch it with:

```sh
ssh walter-vultr tail -f ~/.local/state/walter/emacs-packages.log
```
