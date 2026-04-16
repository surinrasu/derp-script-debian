# Tailscale DERP on Debian

You need:

- A public IPv4 address with 80/tcp, 443/tcp, 3478/udp open
- Not behind a NAT, reverse proxy, or load balancer, etc.

0. Prepare

Install git and mise if you haven't:

```bash
sudo apt update -y && sudo apt install -y git curl
sudo install -dm 755 /etc/apt/keyrings
curl -fSs https://mise.jdx.dev/gpg-key.pub | sudo tee /etc/apt/keyrings/mise-archive-keyring.asc 1> /dev/null
echo "deb [signed-by=/etc/apt/keyrings/mise-archive-keyring.asc] https://mise.jdx.dev/deb stable main" | sudo tee /etc/apt/sources.list.d/mise.list
sudo apt update -y && sudo apt install -y mise
echo 'eval "$(mise activate bash)"' >> ~/.bashrc && source ~/.bashrc
```

Then clone this repo:

```bash
git clone https://github.com/surinrasu/derp-script-debian.git
cd derp-script-debian
```

And create a `mise.local.toml`:

```toml
[env]
DERP_HOST = "" // address
DERP_IP = "" // address
LE_EMAIL = "" // email
DERP_VERIFY_CLIENTS = "" // 0 or 1
TAILSCALE_VERSION = "" // version number or `latest`, same with `tailscaled`
```

1. Set up

```bash
mise trust && mise install
mise run setup
```

This will install deps, Certbot, derper and create a systemd service, set mirror for `apt` and `go` in advance if you need

2. Get a certificate

Try:

```bash
mise run cert-init-staging
```

If failed, check:

- Whether port 80 is actually accessible from the public internet
- Whether this machine is actually directly connected to the public internet
- Whether the address is the one by which the machine is currently visible externally

And then:

```bash
mise run cert-init
mise run cert-sync
mise run start
```

3. Generate DERP map

```bash
mise run policy
```

This will create a `derp-map.json` which should be copied to Tailscale policy

4. Install a timer for auto renew

```bash
mise run install-timer
```

That's all. And in case you need to do something manually, check:

```bash
mise run logs # Check logs

mise run openssl-check # Check certificate

mise run restart # Restart derper

mise run stop # Stop derper

mise run netcheck # Check Tailscale

mise run cert-renew # Renew cerificate immidately
```

If you change region or name in `mise.local.toml`, run:

```bash
mise run policy
```

If you changed `DERP_HOST`, `DERP_IP`, or most things about certificate:

```bash
mise run cert-init
mise run cert-sync
mise run restart
mise run policy
```

If you changed `DERP_VERIFY_CLIENTS`:

```bash
mise run write-service
mise run restart
```

If you changed `TAILSCALE_VERSION` or updated `derper`:

```bash
mise run build-derper
mise run install-derper
mise run restart
```

If you want to enable `--verify-clients`, you will need to install `tailscaled` and log in:

```bash
mise run tailscale-install
sudo tailscale up
```

Then edit:

```toml
DERP_VERIFY_CLIENTS = "1"
TAILSCALE_VERSION = "" // same to `tailscaled`
```

And run:

```bash
mise run build-derper
mise run install-derper
mise run write-service
mise run restart
```
