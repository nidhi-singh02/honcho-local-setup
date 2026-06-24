# Honcho local setup for Hermes

This repo contains the small config layer I use for running Honcho locally on a Mac while Hermes runs on a Raspberry Pi.

It is not a Honcho fork. You still use the official Honcho repo. These files only show the local `.env` values and the Docker Compose change that made the Pi-to-Mac setup work.

## What this setup assumes

- Honcho runs on your Mac with Docker Compose.
- Ollama runs on the same Mac.
- Hermes runs on a Raspberry Pi.
- The Pi talks to Honcho through your Mac's LAN IP.
- Honcho's Docker containers talk to Ollama through `host.docker.internal`.

## Files

- `.env.local`: copy-ready Honcho env values for Ollama + Qwen 3 Embedding 4B.
- `docker-compose.override.yml`: exposes Honcho API on the Mac LAN and loads `.env.local` into the API and deriver containers.

## Use it

Clone Honcho first:

```bash
git clone https://github.com/plastic-labs/honcho.git
cd honcho
cp docker-compose.yml.example docker-compose.yml
```

Copy these setup files into the Honcho repo:

```bash
cp /path/to/honcho-local-setup/.env.local .
cp /path/to/honcho-local-setup/docker-compose.override.yml .
```

Pull the models:

```bash
ollama pull qwen3-embedding:4b
ollama pull qwen3.5:27b
```

Start Honcho:

```bash
docker compose up -d --build
```

## Configure Hermes on the Raspberry Pi

Use your Mac's LAN IP, not `localhost`, not `0.0.0.0`, and not `host.docker.internal`.

Example:

```text
http://192.168.1.25:8000
```

`host.docker.internal` is only for Honcho's Docker containers calling Ollama on the Mac.

## Verify

From the Mac:

```bash
curl http://localhost:8000/health
docker compose logs --tail=50 api deriver
```

From inside the Honcho API container:

```bash
docker compose exec api curl -sS http://host.docker.internal:11434/v1/models
```

From the Raspberry Pi:

```bash
curl http://YOUR_MAC_LAN_IP:8000/health
```

If the Pi cannot reach that URL, fix networking before debugging Honcho memory.

## Security note

This demo uses:

```env
AUTH_USE_AUTH=false
```

That is okay for a trusted local network demo. If you expose Honcho outside your own machine or LAN, turn auth on and use a real JWT setup.

Do not paste real API keys into this public file.
