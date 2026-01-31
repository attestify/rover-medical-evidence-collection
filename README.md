#  Rover Medical – Evidence Collection

The Rover Medical (Evidence Collectio ) uses Ansible in local-connection mode to simulate realistic remote evidence collection for the [Rover Medical System](https://github.com/attestify/rover-medical-system) demo, producing deterministic artifacts that can be ingested and assessed by assurance tooling.

## Overview

Its sole purpose or this repository is to collect evidence that can later be assessed by verification procedures (for example, [NAPE procedures](https://github.com/nape-not-another-policy-engine/nape-catalog)). There repo **_does not_** deploy infrastructure or configure systems.

This evidence collection approach is intentionally structured to look and behave like real-world remote collection. Event though it is simulated using Ansible’s local execution mode.

## Release-based Evolution

This repository supports incremental maturity across releases. Each new release adds new canned evidence files, new Ansible tasks, and does not require changes to workflows.

This mirrors how real assurance programs evolve over time.

The progression is a follows:

|Release| Scenario | Data Collected as Evidence|
|---|---|---|
| 1 | 1x thin slice - Getting one inconclusive | Pet Medicine App - Database Connection Config.|
| 2 | Expand thin slice to broader scope. | Full Configuration for the Pet Medicine App, and the Pet Medicine Database|
| 3 | Go beyond App & DB to the runtime infrastructure, AND include other common business risk concerns (3rd Party Assurance).| 1) Full Configuraiton for the App, Database, 2) Pet Medicine App testing results, 3) Rover Cloud configuration, and 4) Rover Cloud 3rd Party Provider Report (SOC1) |
| 4 | View how drift appears over time, and how clear it is to identify the drift. | 1) Full Configuraiton for the App, Database, 2) Pet Medicine App testing results, 3) Rover Cloud configuration, and 4) Rover Cloud 3rd Party Provider Report (SOC1) |

## Relationship to NAPE & Attestify

This repository only collects evidence. It does not evaluate correctness, determine pass or fail, or interpret anything else.

Such responsibilities belong to the downstream assurance tooling [NAPE](https://napecentral.com), which consumes the evidence, applies tests of detail, and procudes verification reports (pass, fail, inconclusive) that draw a determinsitic conclusion that "one is doing what they said they should do."

This separation is intentional and fundamental because that assurnace processes is a [x] step prrocess of,
1. Collecte data as evidence (what this repo does),
2. Compare this data against expectations (NAPE Verification Procedures),
3. Evaluate the verified conclusions asses the level of risk one is willing to take with the impirical knowledge of

    a. here is what we say we should be doing (Verification Proceudre), and

    b. here is how we are actually doing to what we said we should be doing (Verification Report)

## Architecture & Flow Diagram

Below is a conceptual flow diagram showing how simulated hosts, Ansible, evidence, and assurance fit together.

Even though the hosts are simulated the **collection mechanism is real**, the **evidence artifacts are real**, the **assurance logic is real**.

_**Only the source of the data is simulated.**_

This allows teams to demonstrate assurance concepts credibly, evolve evidence maturity across releases, and avoid the risks and complexity of live systems.

### Design Principles

1. **Deterministic** - same inputs produce the same outputs.
2. **Read-only** - no state mutation.
3. **Realistic** - mirrors real operational patterns.
4. **Composable** -  can be reused across demos and scenarios.
5. **Transparent** - easy to explain what is simulated and why.

```
┌───────────────────────────┐
│   Simulated Hosts         │
│                           │
│  rover-db-1               │
│  rover-app-1              │
│                           │
│  (Canned command output,  │
│   config files, metadata) │
└─────────────┬─────────────┘
              │
              │  (Ansible local connection)
              │
┌─────────────▼─────────────┐
│   Ansible Data            │
│   Collection              │
│                           │
│  - inventory.ini          │
│  - collect-evidence.yml   │
│                           │
│  Tasks simulate:          │
│   • systemctl cat         │
│   • auditctl -s           │
│   • config copy           │
└─────────────┬─────────────┘
              │
              │  (Filesystem artifacts)
              │
┌─────────────▼─────────────┐
│   Data as Evidence        │
│    Artifacts              │
│                           │
│  rover-medical-system/    │
│    evidence/<component>/  │
│                           │
│  Deterministic,           │
│  versioned evidence       │
└─────────────┬─────────────┘
              │
              │  (Ingest)
              │
┌─────────────▼─────────────┐
│   NAPE Assurance Engine   │
│                           │
│  - Interpret evidence     │
│  - Apply test of detail   │
│  - Produces clais         │
│                           │
│  PASS / FAIL /            │
│  INCONCLUSIVE             │
└───────────────────────────┘
```

### Why This Exists

The repos exists to paint the picture of, “**_What would we collect from a real system, and how would we collect it, if this were production?_**”

Autonomous Assurance for an Assured Release depends on data collected from servers, applications, operating systems, configuration files, and command outputs in real environments.

This repository demonstrates one means of gather data (in a real-world use case) while allowing for the Rover Medical System demo to be deterministic, repeatable runs, and have no dependency on live infrastructure.

### What This Repository Does

This repo is a **_read-only_** data collection by design. The repo intentionally **_does not_** configure systems, deploy applications, change state on any host, or require SSH or credentials.

This repository provides:
* Ansible inventories that model real hosts (database, application)
* Ansible playbooks that collect evidence
* A structure that supports a release-by-release maturity storyline
* Deterministic output files that can be ingested by assurance tooling

### How Evidence Collection Works In This Demo

This approach allows for demos and verification procedures to behave exactly as they would against real infrastructure.

Ansible is run in **_local connection mode_**, meaning that
* no SSH connections are made,
* no real remote hosts are accessed, and
* data, used as evidence, is copied from canned source files that represent what a real host would return.

The structure mirrors real operational collection or
* hosts are grouped (db, app),
* tasks run “per host”,
* outputs are written per host, and
* commands and files look realistic.



## Repository Structure

```
ansible/
  inventory.ini
  collect-evidence.yml
```

### Inventory - _ansible/inventory.ini_

The inventory defines logical hosts and roles were hostnames are symbolic and descriptive, _ansible_connection=local_ ensures no SSH is used, and host grouping matches how real environments are modeled.

```ini
[db]
rover-db-1 ansible_connection=local

[app]
rover-app-1 ansible_connection=local
```
### Evidenced Data -  Source and Destination

#### Source - Canned Data

Canned evidence lives in the demo repository, organized by release. These files represent command output, configuration files, and system state snapshots. All data is treated as if they came from real hosts.

```
rover-medical-sysem/
  release-1/
      ped-medicine-app/
      pet-medicine-db/
```

#### Destination - Aggregated Data

This mirrors real collection runs where evidence is grouped by release, runtime execution, and host identity. Downstream assurance tooling reads from this directory.

Evidence is written to ``rover-medical-system/evidence/<component-name>/<file name here>``

For example, the database configuration file (_my-config.ini_) for of the Ped Medicine Databse (_ped-med-db_) would be captured at the destination as: ```rover-medical-system/evidence/ped-med-db/my-config.ini ```

### The Playbook - _collect-evidence.yml_

The playbook performs the following steps:
1. Iterates over logical components (db, app, os)
2. Creates per-component evidence output directories
3. Copies canned evidence into runtime evidence locations
4. Produces deterministic, structured artifacts

Each task corresponds to something that would be:
* a remote command (systemctl cat, auditctl -s)
* a file retrieval (/etc/systemd/...),
* or a system inspection