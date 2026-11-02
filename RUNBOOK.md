# RUNBOOK — actual AWS build, validation, teardown
Everything in `<<>>` is yours to fill in. Total hands-on time: **~40–60 min**. Do steps strictly in order.

## Step 0 — Pre-flight (5 min, browser)
1. AWS Console → check **Region: ap-southeast-2 (Sydney)**, top-right.
2. Verify t3.micro is **Free-tier eligible** on this account (or credits available).
3. Have a folder ready on your machine: `vpc-peering-project\img\` for screenshots.

## Step 1 — Provision everything (10 min, CloudShell)
CloudShell already has AWS CLI + your console credentials:
```bash
export STUDENT="<<YourName>>"
# upload or paste scripts/01-build-vpc-peering.sh, then:
bash 01-build-vpc-peering.sh
```
**During/after this script:**
- When it creates `peering-lab.pem`: CloudShell **Actions ▾ Files ▸ Download** → save to `~/.ssh/peering-lab.pem` (Windows: `C:\Users\<you>\.ssh\peering-lab.pem`), then `chmod 400 ~/.ssh/peering-lab.pem` (Git Bash) or set file permissions on Windows.
- Copy the **RESOURCE INVENTORY** block (IDs) into `report-skeleton.md` §2 and `index.html` results/inventory table.
- 📸 Screenshot: the inventory output → `img/01-inventory.png`.

## Step 2 — SSH access + baseline fail test (5 min, laptop)
1. Copy `scripts/ssh-config` → `~/.ssh/config`, replace `REPLACE_WITH_BASTION_PUBLIC_IP` with the public IP from Step 1.
2. From a terminal:
```bash
ssh app-a 'hostname; whoami'      # ProxyJump through bastion — must work
```
3. **While still on app-a**, run the baseline (note: run BEFORE peer routes ONLY if you skipped that part of script 01; if you ran the full script, peering+routes already exist — instead capture baseline by temporarily *removing* routes, OR just note script 01 does both. Recommended evidence alternative: run the ping right after instances are up but before you re-run script 02 if you split your run.)
```bash
ping -c 4 10.20.1.10                   # expected: 100% loss (no route/peering yet)
```
📸 `img/02-baseline-fail.png`

> ⚠️ Script 01 provisions peering + routes in one go. To capture a genuine baseline-fail, run script 01 **up to instances only** (comment out Phase 5a/5b and the peer-route block), take the baseline screenshot, then re-run the remainder or run this script again from that point. See "script 01 partial run" note in the script comments.

## Step 3 — Positive path tests (10 min, CloudShell + instances)
With peering + routes in place:
```bash
# on app-a:
ping -c 4 10.20.1.10     # → 0% loss        📸 img/03-forward.png
# on app-b (ssh app-b from laptop):
ping -c 4 10.10.2.10     # → 0% loss        📸 img/04-reverse.png
# from laptop:
ssh app-b 'hostname; whoami'           # ip-10-20-1-10    📸 img/05-ssh.png
# CloudShell — isolation check (no public IPs on apps):
bash 02-test-commands.sh                   # prints the table 📸 img/06-isolation.png
```
📸 Console: route tables (3×) + associations → `img/07-routes-console.png` · peering Active → `img/08-peering-active.png`

## Step 4 — Negative test: controlled failure + recovery (5 min)
```bash
# CloudShell (replace sg from your inventory):
aws ec2 revoke-security-group-ingress --group-id <<SG_APP_B>> \
  --ip-permissions "IpProtocol=icmp,FromPort=8,ToPort=0,IpRanges=[{CidrIp=10.10.2.10/32}]"
# app-a:
ping -c 4 10.20.1.10                       # replies STOP     📸 img/09-failure.png
# CloudShell:
aws ec2 authorize-security-group-ingress --group-id <<SG_APP_B>> \
  --ip-permissions "IpProtocol=icmp,FromPort=8,ToPort=0,IpRanges=[{CidrIp=10.10.2.10/32}]"
# app-a:
ping -c 4 10.20.1.10                       # replies RESUME   📸 img/10-recovery.png
```

## Step 5 — Fill the report (15 min, laptop)
Fill `report/report-skeleton.md`: section 2 IDs, section 4 "Observed" column, section 5 analysis, date/name. Export/save as PDF → `<<YourName>>_VPC_Peering_Report.pdf` (print-to-PDF from any editor).

## Step 6 — Teardown (5 min, CloudShell)
```bash
bash 03-cleanup.sh     # type YES at the prompt
```
📸 Final verification output ("no chargeable resources") → `img/11-cleanup.png`

## Step 7 — Publish (5 min, browser → PUBLISH.md)
Drag the `vpc-peering-project` folder (with `img/` screenshots) to GitHub → enable Pages → live link. Update `<YOUR_GITHUB_USERNAME>` in `index.html` (2 places) + `README.md` + report.

## Evidence bundle (what lands in the report)
| File | Shows |
|---|---|
| 01-inventory | all resource IDs from provisioning summary |
| 02-baseline-fail | routing absent → 100% loss |
| 03/04-forward-reverse | symmetric connectivity works |
| 05-ssh | shell on private app-b via bastion |
| 06-isolation | description of instances: no public IPv4 |
| 07/08-console | route tables + Active peering |
| 09/10-failure-recovery | SG policy governs after peering |
| 11-cleanup | tag-filtered zero resources |
