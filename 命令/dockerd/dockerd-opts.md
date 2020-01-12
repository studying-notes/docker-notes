# `dockerd` 参数选项

这里列出了 `dockerd` 全部的参数选项，但也有可能因为宿主机平台不同而缺少一些选项。

| 名称 | 描述 |
| ---- | ---- |
| `--add-runtime` runtime | Register an additional OCI compatible runtime (default []) |
| `--allow-nondistributable-artifacts` list | Push nondistributable artifacts to specified registries (default []) |
| `--api-cors-header` string | Set CORS headers in the Engine API |
| `--authorization-plugin` list | Authorization plugins to load (default []) |
| `--bip` string | Specify network bridge IP |
| `-b`, `--bridge` string | Attach containers to a network bridge |
| `--cgroup-parent` string | Set parent cgroup for all containers |
| `--cluster-advertise` string | Address or interface name to advertise |
| `--cluster-store` string | URL of the distributed storage backend |
| `--cluster-store-opt` map | Set cluster store options (default map[]) |
| `--config-file` string | Daemon configuration file (default "/etc/docker/daemon.json") |
| `--containerd` string | Path to containerd socket |
| `--cpu-rt-period` int | Limit the CPU real-time period in microseconds |
| `--cpu-rt-runtime` int | Limit the CPU real-time runtime in microseconds |
| `--data-root` string | Root directory of persistent Docker state (default "/var/lib/docker") |
| `-D`, `--debug` | Enable debug mode |
| `--default-gateway` ip | Container default gateway IPv4 address |
| `--default-gateway-v6` ip | Container default gateway IPv6 address |
| `--default-address-pool` | Set the default address pool for local node networks |
| `--default-runtime` string | Default OCI runtime for containers (default "runc") |
| `--default-ulimit` ulimit | Default ulimits for containers (default []) |
| `--dns` list | DNS server to use (default []) |
| `--dns-opt` list | DNS options to use (default []) |
| `--dns-search` list | DNS search domains to use (default []) |
| `--exec-opt` list | Runtime execution options (default []) |
| `--exec-root` string | Root directory for execution state files (default "/var/run/docker") |
| `--experimental` | Enable experimental features |
| `--fixed-cidr` string | IPv4 subnet for fixed IPs |
| `--fixed-cidr-v6` string | IPv6 subnet for fixed IPs |
| `-G`, `--group` string | Group for the unix socket (default "docker") |
| `--help` | Print usage |
| `-H`, `--host` list | Daemon socket(s) to connect to (default []) |
| `--icc` | Enable inter-container communication (default true) |
| `--init` | Run an init in the container to forward signals and reap processes |
| `--init-path` string | Path to the docker-init binary |
| `--insecure-registry` list | Enable insecure registry communication (default []) |
| `--ip` ip | Default IP when binding container ports (default 0.0.0.0) |
| `--ip-forward` | Enable `net.ipv4.ip`_forward (default true) |
| `--ip-masq` | Enable IP masquerading (default true) |
| `--iptables` | Enable addition of iptables rules (default true) |
| `--ipv6` | Enable IPv6 networking |
| `--label` list | Set `key=value` labels to the daemon (default []) |
| `--live-restore` | Enable live restore of docker when containers are still running |
| `--log-driver` string | Default driver for container logs (default "json-file") |
| `-l`, `--log-level` string | Set the logging level ("debug", "info", "warn", "error", "fatal") (default "info") |
| `--log-opt` map | Default log driver options for containers (default map[]) |
| `--max-concurrent-downloads` int | Set the max concurrent downloads for each pull (default 3) |
| `--max-concurrent-uploads` int | Set the max concurrent uploads for each push (default 5) |
| `--metrics-addr` string | Set default address and port to serve the metrics api on |
| `--mtu` int | Set the containers network MTU |
| `--node-generic-resources` list | Advertise user-defined resource |
| `--no-new-privileges` | Set no-new-privileges by default for new containers |
| `--oom-score-adjust` int | Set the oom_score_adj for the daemon (default -500) |
| `-p`, `--pidfile` string | Path to use for daemon PID file (default "/var/run/docker.pid") |
| `--raw-logs` | Full timestamps without ANSI coloring |
| `--registry-mirror` list | Preferred Docker registry mirror (default []) |
| `--seccomp-profile` string | Path to seccomp profile |
| `--selinux-enabled` | Enable selinux support |
| `--shutdown-timeout` int | Set the default shutdown timeout (default 15) |
| `-s`, `--storage-driver` string | Storage driver to use |
| `--storage-opt` list | Storage driver options (default []) |
| `--swarm-default-advertise-addr` string | Set default address or interface for swarm advertised address |
| `--tls` | Use TLS; implied by `--tlsverify` |
| `--tlscacert` string | Trust certs signed only by this CA (default "~/.docker/ca.pem") |
| `--tlscert` string | Path to TLS certificate file (default "~/.docker/cert.pem") |
| `--tlskey` string | Path to TLS key file (default ~/.docker/key.pem") |
| `--tlsverify` | Use TLS and verify the remote |
| `--userland-proxy` | Use userland proxy for loopback traffic (default true) |
| `--userland-proxy-path` string | Path to the userland proxy binary |
| `--userns-remap` string | User/Group setting for user namespaces |
| `-v`, `--version` | Print version information and quit |
