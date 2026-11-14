# pi-sandbox

Run [pi](https://pi.dev) (pi-coding-agent) inside a [microsandbox](https://microsandbox.dev) (msb) microVM, against a model served by a Docker container on this host.

## How it works

- The directory you pass is mounted read-write into the sandbox as `/workspace`, and pi starts there.
- A reusable base image (`pi-sandbox:v2`, built from `Dockerfile.pi`) has pi, pnpm, and uv preinstalled. If it is missing from the msb image store, the script builds it with docker and loads it into msb (one-time).
- The model container (default `127.0.0.1:8090`) is queried for its model list, and a pi `models.json` is generated automatically pointing at `http://host.microsandbox.internal:8090/v1` (the in-sandbox alias of the host) and copied into the sandbox at `/root/.pi/agent/models.json`. No API key is needed (llama.cpp ignores it).
- Default networking: host + public internet (`--net host,public`).
- `--no-internet` blocks all egress except the model endpoint and DNS (`--no-net --net-rule allow@host:tcp:8090 --net-rule allow@dns`), so pi still reaches the model but nothing else.

## Usage

```sh
# interactive, working on the current directory
./pi-sandbox.py

# interactive, working on a specific host directory
./pi-sandbox.py /path/to/project

# one-shot
./pi-sandbox.py /path/to/project --prompt "summarize this repo"

# offline sandbox (model only, no internet)
./pi-sandbox.py /path/to/project --no-internet

# extra pi args after --
./pi-sandbox.py /path/to/project -- -p "run the tests" --continue

# just show what would run
./pi-sandbox.py /path/to/project --dry-run
```

## Options

| Option | Default | Description |
|---|---|---|
| `DIR` | `.` | Host directory pi works in (mounted as `/workspace`) |
| `--model-host` | `127.0.0.1` | Model container address as seen from the host |
| `--model-port` | `8090` | Published port of the model container |
| `--model-id` | first served model | Model id served by the container |
| `--context-window` | `32768` | Context window in tokens |
| `--reasoning` | off | Mark the model as reasoning-capable |
| `--no-internet` | off | Block external network except model endpoint + DNS |
| `--prompt / -p` | off | One-shot mode instead of interactive |
| `--image` | `pi-sandbox:v2` | msb base image tag |
| `--cpus` | `2` | Sandbox vCPUs |
| `--memory` | `2G` | Sandbox memory |
| `--root-disk` | `4G` | Sandbox root disk |
| `--name` | `pi-<dir basename>` | Sandbox name |
| `--timeout` | off | msb run timeout in seconds |
| `--dry-run` | off | Print the msb command without running it |

Requires: `msb` on the host, a running model container exposing port 8090, and `docker` (only if the base image has not been built yet).
