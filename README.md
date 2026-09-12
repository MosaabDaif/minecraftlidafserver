# lidafserver

NixOS configuration for a laptop repurposed as a headless Minecraft
(NeoForge / Create Aeronautics Skybound) server.

## What's here

- `configuration.nix` — the full system config: boot, networking, lid/sleep
  overrides (this runs on a laptop with the lid closed), the `lidafserver`
  admin account, and the `minecraft` systemd service.
- `hardware-configuration.nix` — **not tracked in this repo**. It's
  machine-specific (disk UUIDs, filesystem layout) and won't apply to any
  other box. Generate your own with `sudo nixos-generate-config` and place
  it next to `configuration.nix` before rebuilding.

## First-time setup on a new machine

1. Clone this repo into `/etc/nixos` (or symlink `configuration.nix` there).
2. Run `sudo nixos-generate-config` to produce a local
   `hardware-configuration.nix`.
3. **Add an SSH public key** in `configuration.nix` under
   `users.users.lidafserver.openssh.authorizedKeys.keys`. Password auth is
   disabled, so skipping this locks you out over SSH — keep local console
   access until a key is confirmed working.
4. `sudo nixos-rebuild switch`.
5. The `minecraft` service will fail to start until the server files exist
   — that's expected. Deploy the NeoForge server pack to
   `/var/lib/minecraft` (owned by the `minecraft` user), accept the EULA,
   and set JVM heap args in `user_jvm_args.txt` before starting it.

## Known gaps / next steps

- **RCON** isn't configured yet. Without it there's no way to run `/op`,
  `/stop`, etc. against the running server. Add
  `enable-rcon=true` / `rcon.password` in `server.properties` and
  `pkgs.mcrcon` to `environment.systemPackages` — keep port `25575` off
  `networking.firewall.allowedTCPPorts`, it's localhost-only.
- **Backups** aren't automated. The world directory under
  `/var/lib/minecraft` is the only unrecoverable state; a systemd timer
  running `tar` to a second disk or remote host is the minimum viable setup.
- **Remote access for players**: if the network is behind CGNAT (common on
  mobile/4G broadband and some ISPs), port-forwarding won't work no matter
  how it's configured. Check with `curl -4 ifconfig.me` against the
  router's WAN IP before relying on port forwarding + DDNS. Tailscale is
  the simplest fallback if so.
