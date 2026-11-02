# AWS VPC Peering — Private Multi-VPC Connectivity Project

Design, build and validation of private connectivity between two AWS VPCs using a peering connection, symmetric routing, bastion-proxy SSH and least-privilege security groups. Every resource is tagged `Project=vpc-peering` and the build includes scripted teardown with a zero-resource verification.

- **Live project site (AWS):** `https://d1fhfadxllt04r.cloudfront.net` — S3 private bucket (`vpc-peering-site-371892040349`, ap-southeast-2) + CloudFront OAC (`E2PRSKN1IP4IM7`)
- **Live project site (GitHub Pages, optional):** `https://<YOUR_GITHUB_USERNAME>.github.io/vpc-peering-project/`
- **Region:** ap-southeast-2 · **Instance type:** t3.micro · **OS:** Amazon Linux 2023

## Topology

```
Client ──► IGW ──┐
                 ▼
 VPC A 10.10.0.0/16 ── peering ──► VPC B 10.20.0.0/16
  ├─ public  10.10.1.0/24  → bastion  (10.10.1.10 + public IPv4)
  ├─ private 10.10.2.0/24  → app-a    (10.10.2.10, no public IP)
  └─ VPC B private 10.20.1.0/24 → app-b (10.20.1.10, no public IP)
```

## Build & validate in 7 steps
Full detail in **[RUNBOOK.md](RUNBOOK.md)** — short version:

1. `export STUDENT="YourName"; bash scripts/01-build-vpc-peering.sh` *(CloudShell)* — provisions both VPCs, IGW, subnets, route tables, security groups, 3 instances, key pair, peering and peer routes; prints the resource inventory.
2. Download `peering-lab.pem` from CloudShell, write `~/.ssh/config` from `scripts/ssh-config`, test `ssh app-a`.
3. Run tests 1–4 (`scripts/02-test-commands.sh`): baseline fail → forward/reverse ping → ProxyJump SSH.
4. Negative test: revoke/restore vpc-peering-app-b ICMP rule — traffic stops then resumes.
5. Fill `report/report-skeleton.md` with real IDs + outputs → export PDF.
6. `bash scripts/03-cleanup.sh` — teardown + zero-resource verification.
7. Publish this repo to GitHub Pages (see `PUBLISH.md`) for the live project site.

## Repository layout

| Path | What it is |
|---|---|
| `index.html`, `css/`, `js/` | Project site: architecture, connectivity walkthrough, packet-flow simulation, test matrix |
| `scripts/01-build-vpc-peering.sh` | One-shot provisioning (CLI) |
| `scripts/ssh-config` | SSH config with ProxyJump to both private hosts |
| `scripts/02-test-commands.sh` | Validation test matrix commands |
| `scripts/03-cleanup.sh` | Tag-based teardown + verification |
| `report/report-skeleton.md` | Report draft (fill → export PDF) |
| `RUNBOOK.md` | Ordered build/evidence run sheet |

## Validation matrix (summary)

| # | Test | Expected |
|---|---|---|
| 1 | Baseline ping, pre-peering | 100% loss |
| 2 | Forward ping (app-a → app-b) | 0% loss |
| 3 | Reverse ping (app-b → app-a) | 0% loss |
| 4 | SSH via ProxyJump | shell on 10.20.1.10 |
| 5 | Isolation audit | no public IPv4 on apps |
| 6 | SG rule removed | replies stop, peering still Active |
| 7 | SG rule restored | replies resume |
