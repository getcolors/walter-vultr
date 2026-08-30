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
./green converge-nix       # update declared Nix entries on all three logins
./green converge-asdf      # exact asdf versions and Corepack on all three logins
```

`colors.yml` is the only file normally edited. The Vultr API token, R2
credentials, and optional feature credentials are `COLORS_PAR_*` variables in
the ignored `.envrc.private`; never put credentials in desired state and never
export `COLORS_PAR_PROFILE`.

## Machine access

With no explicit provider machine key, Walter generates the dedicated keypair
`~/.ssh/walter-vultr`. The compute template uses it directly and registers its
public half as a Terraform-managed Vultr SSH key. A successful delete removes
the provider registration and then the local keypair; a later create generates
a fresh pair.

Vultr's image exposes root only for bootstrap. Walter uses that connection once
to adopt the stock UID/GID 1000 account as `ubuntu`, install the dedicated key and passwordless sudo,
then disables root and password SSH. Normal Ansible provisioning and
`ssh walter-vultr` always use `ubuntu`.

Two seat aliases provide isolated, non-sudo workspaces with the same environment:

```sh
ssh walter-vultr-rose
ssh walter-vultr-jack
```

`converge-nix` and `converge-asdf` use these managed aliases and require only a
running machine, not Vultr or R2 credentials. They operate on ubuntu, rose and
jack; the Nix command preserves unrelated profile entries, while the asdf
command applies the exact versions in `colors.yml`.

## Desired machine

The deployment selects Amsterdam (`ams`), Ubuntu 26.04 LTS (`os_id` 2760), and the
four-vCPU/eight-GB `vc2-4c-8gb` plan. State is stored in Cloudflare R2 under
`walter-vultr/walter-compute.tfstate`. Destruction remains protected by default.

A real create begins with GitHub's device flow, then provisions and configures
the machine. Emacs package installation runs in the background after create;
watch it with:

```sh
ssh walter-vultr tail -f ~/.local/state/walter/emacs-packages.log
```
