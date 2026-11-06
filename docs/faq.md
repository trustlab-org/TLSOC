# TLSOC — Frequently Asked Questions

## General

### What is TLSOC?

TLSOC is a free and open-source Security Operations Platform. It collects logs
from Linux servers without agents, transports them over TLS through Apache
Kafka, normalizes them into Elastic Common Schema (ECS) events, indexes them into
Elasticsearch for Kibana dashboards, and generates daily HTML/PDF executive
reports. It is built from independent repositories that compose into one platform
— see the [ecosystem overview](../README.md#ecosystem-repositories).

### Is TLSOC a SIEM?

TLSOC delivers core Security Information and Event Management (SIEM)
capabilities — collection, normalization, storage, search, and visualization of
security events — and adds deployment automation, agentless onboarding, a parser
framework, and executive reporting around them. It is broader than a single SIEM
application, but it is not a full enterprise SOC suite (no case management or
endpoint telemetry). The precise positioning and trade-offs are laid out in
[Why TLSOC?](why-tlsoc.md).

### Why not just use plain ELK, or Elastic Security?

Short version: plain ELK is the foundation TLSOC builds on — TLSOC adds the
deployment, onboarding, parsing, and reporting layers you would otherwise build
by hand. Elastic Security is an excellent platform designed around Elastic
Agent; TLSOC is a different architecture optimized for agentless, self-hosted
environments. The full, honest comparison is in
[Why TLSOC?](why-tlsoc.md#why-not-plain-elk).

### Is it really agentless?

Yes. Monitored servers forward logs using stock `rsyslog` with the `omkafka`
module — both from the distribution's package repository. Nothing custom is
installed or left running on source machines beyond an rsyslog configuration file.

### What license does TLSOC use?

Apache License 2.0, across every repository. You may use, modify, and redistribute
it freely, commercially or not.

### Which operating systems are supported?

The core stack targets Ubuntu 20.04/22.04/24.04 (fresh VM or bare metal
recommended). The engine runs on any modern Linux with Python 3.8+; monitored
servers can be any Linux with rsyslog. The engine's Web UI console additionally
runs on Windows and macOS for rule development.

## Deployment

### Do I need all the repositories?

No — they compose, but each stands alone:

- **TLSOC Docker Deploy** alone gives you a TLS-secured ELK + Kafka stack.
- **TLSOC Engine** works with any Kafka that carries its JSON envelope, and any
  consumer can read its NDJSON output.
- **TLSOC Reporting** works against any Elasticsearch 8.x cluster holding
  ECS-style parsed logs.

### Can everything run on one server?

Yes — the reference deployment runs the Docker stack, the engine, Redis, and
reporting on a single Ubuntu server. The engine scales with CPU cores; additional
machines can join the same Kafka consumer group later without configuration
changes beyond `group_id` reuse.

### How does TLSOC scale?

Kafka topic partitions are the unit of parallelism. Each log source has its own
topic (`tlsoc.logs.<source>`, created by its first line), so a busy source can
be given more partitions (TLSOCDockerDeploy `kafka/topics.conf`, applied
automatically) and a burst from one source does not delay
the others. Set the engine's `runtime.workers` (`auto` = one per CPU core); all
workers share one consumer group; add cores or machines to scale.

### Is data lost if a component goes down?

The pipeline is designed for at-least-once delivery: rsyslog buffers and retries
when Kafka is unreachable; the engine commits Kafka offsets only after events are
durably written to disk; unparseable lines go to per-source dead-letter files
instead of being dropped. Duplicates are possible after a crash; silent loss is
the failure mode the design rejects.

## Parsing and data

### What log sources are supported out of the box?

The engine ships rules for Apache, Nginx, Linux auth (SSH/sudo), Postfix,
Roundcube, Squid, ModSecurity, Suricata, OpenVAS, Nessus, and file-integrity
monitoring, among others. Any other source can be added as a YAML rule — no code.

### Do I need to know regex or ECS to add a log source?

No. The engine documents an AI "master prompt": paste raw log samples into any AI
chatbot with the prompt from `foss-soc-engine/docs/writing-rules.md`, and it returns
a finished, ECS-compliant YAML rule. The `ecs_helper.py` tool then validates the
field names, and CI's golden-sample exam locks the behavior in.

### Why ECS?

One schema across all sources means one set of dashboards, detections, and report
queries. `source.ip` is `source.ip` whether the event came from Nginx, Postfix,
or a firewall.

## Reporting

### Can I preview reports without a live cluster?

Yes — the reporting framework has an offline backend that runs the same queries
over a local NDJSON sample file, producing output identical to a live run.

### Can I add a report type or data source without coding?

Yes. A datasource is a YAML file (index pattern + field map); a report is a YAML
file (queries + narrative rules + layout). The framework multiplies them: every
report renders for every datasource with a matching profile.

## Contributing

### Where do I report a bug or request a feature?

In the repository that owns the component, using its issue templates. If you are
unsure which repository, open the issue in
[tlsoc](https://github.com/sankettaware16/tlsoc/issues) and it will be routed.

### How do I report a security vulnerability?

Privately, never via public issues — see [SECURITY.md](../SECURITY.md).
