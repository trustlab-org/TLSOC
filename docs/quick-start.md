# TLSOC Quick Start

This walkthrough deploys the complete TLSOC platform: the TLS-secured core stack,
agentless log forwarding from your servers, the parsing engine, and daily reporting.

Each step links to the owning repository's documentation for full detail — this page
is the map, not the territory.

## Prerequisites

- One central **Ubuntu server** (20.04 / 22.04 / 24.04) for the TLSOC stack —
  fresh VM or bare metal recommended, with Docker and Docker Compose.
- Any number of **Linux servers to monitor** (they only need stock `rsyslog`).
- Network reachability from monitored servers to the central server on port `9094`.
- Optional: a free [MaxMind](https://www.maxmind.com) license key for GeoIP/ASN
  enrichment.

## Step 1 — Deploy the core stack

On the central server, deploy TLS-secured Kafka + Logstash + Elasticsearch + Kibana:

```bash
cd /opt
sudo git clone https://github.com/sankettaware16/TLSOCDockerDeploy.git
cd TLSOCDockerDeploy
sudo chmod +x install.sh
sudo ./install.sh
```

The installer does everything: it generates the passwords and TLS certificates,
applies log retention (180 days by default, per the CERT-In Directions) and
index templates, starts the stack, and installs the *TLSOC - SOC Overview*
dashboard and health monitoring. At the end it prints the Kibana address and the
generated `elastic` password. There are no manual password or Kibana setup
steps. If the dashboard step did not finish (for example Kibana was still
starting), run `sudo ./dashboards.sh` — see
[TLSOC Docker Deploy — Dashboards and monitoring](https://github.com/sankettaware16/TLSOCDockerDeploy/blob/main/docs/dashboards.md).
Before relying on the default retention, check that your disk can hold it:
[Data retention and storage](https://github.com/sankettaware16/TLSOCDockerDeploy/blob/main/docs/retention-and-storage.md).

Kibana is then available at `https://<server-ip>:5601`.

## Step 2 — Onboard log sources

On **each server you want to monitor**, run the one-command onboarding script. It
installs `omkafka`, auto-discovers logs in `/var/log`, generates a hardened
rotation-safe rsyslog configuration, and verifies end-to-end delivery to Kafka:

```bash
curl -fsSL https://raw.githubusercontent.com/sankettaware16/TLSOCDockerDeploy/main/tlsoc-onboard.sh -o tlsoc-onboard.sh
sudo bash tlsoc-onboard.sh
```

There is no topic to choose: each log source goes to its own Kafka topic named
after it (`auth` → `tlsoc.logs.auth`), and the engine finds the parser by the
same name. How names, topics and parsers fit together:
[Log sources and Kafka topics](https://github.com/sankettaware16/TLSOCDockerDeploy/blob/main/docs/sources-and-topics.md).
Manual template and non-interactive rollout:
[TLSOC Docker Deploy — Onboarding](https://github.com/sankettaware16/TLSOCDockerDeploy/blob/main/docs/onboarding.md).

## Step 3 — Start the parsing engine

On the central server (or a dedicated parsing host), install
[foss-soc-engine](https://github.com/sankettaware16/foss-soc-engine):

```bash
git clone https://github.com/sankettaware16/foss-soc-engine.git
sudo mv foss-soc-engine /etc/
cd /etc/foss-soc-engine
export MAXMIND_LICENSE_KEY=<your-key>   # optional, enables GeoIP/ASN enrichment
./install.sh
```

Then:

1. Edit `config.yaml` — point `kafka.bootstrap_servers` at your stack. Nothing
   else: the engine reads every per-source topic (`tlsoc.logs.<source>`) and finds
   each source's parser by name.
2. Validate before going live: `python3 preflight.py` (checks config, rules, ECS
   fields, Kafka reachability, topics, Redis, and partition counts in one command).
3. Run as a service: `sudo ./setup_service.sh`, then
   `sudo systemctl status foss-soc`.

The engine also ships a browser console (Web UI) and an optional Kibana plugin for
managing rules without a terminal — see
[TLSOC Engine — documentation](https://github.com/sankettaware16/foss-soc-engine/tree/main/docs).

## Step 4 — Enable reporting

On a host that can reach Elasticsearch (zero configuration if it is the stack host):

```bash
cd /opt
sudo git clone https://github.com/sankettaware16/tlsoc-reporting-framework.git
sudo chown -R $USER: tlsoc-reporting-framework
cd tlsoc-reporting-framework
python3 -m venv .venv
.venv/bin/pip install -r requirements.txt
sudo apt install -y wkhtmltopdf

.venv/bin/python -m framework validate
.venv/bin/python -m framework generate-all
```

Schedule it daily and (optionally) enable email delivery:
[TLSOC Reporting — Automation](https://github.com/sankettaware16/tlsoc-reporting-framework/blob/main/docs/automation.md).

## Verifying the pipeline

| Check | Command / location |
|---|---|
| Every source at a glance (topic, rate, lag, parser) | `python3 sources.py` in the engine directory |
| Logs arriving in Kafka | `docker exec -it kafka /opt/kafka/bin/kafka-console-consumer.sh --bootstrap-server kafka:9092 --topic tlsoc.logs.<source>` |
| Engine parsing healthily | `logs/stats.json` (EPS, errors) or the Web UI Monitor |
| Unparsed lines | `logs/dlq/<source>.json` in the engine directory |
| Events in Elasticsearch | Kibana → Discover, index pattern `fosstlsoc-logs-*` |
| Reports generated | `output/html/` and `output/pdf/` in TLSOC Reporting |

## Where to go next

- [Architecture](architecture.md) — how the pieces fit together
- [Ecosystem Standards](ecosystem.md) — repository conventions
- [Roadmap](roadmap.md) — what is coming next
- [FAQ](faq.md) — common questions
