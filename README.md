# Veeam-Infrastructure-Sizing-Calculator

Please visit https://m365admintools.com/veeam-sizing-calculator for more information and a running view of the tool.

A browser-based sizing and design tool for Veeam Backup & Replication. Enter your sites, hosts, and VM counts, and it produces a complete infrastructure design: proxy plan, VBR server specification, repository storage per site, WAN bandwidth and seed times, job design, and a 3-2-1 compliance table.

One HTML file. No installation, no server, no sign-up, no internet connection required. Nothing you type leaves the browser.

Built for Veeam architects, MSPs, and administrators who need a defensible sizing figure for a proposal, a budget, or a design document.

<!-- Add a screenshot of the calculator with the report pane here, then uncomment:
![Calculator](docs/images/calculator.png)
-->

<!-- Add a screenshot of the exported report here, then uncomment:
![Exported report](docs/images/exported-report.png)
-->

## Why three VM size tiers

Most sizing spreadsheets use one average VM size. That produces a wrong answer in almost every real environment, because an estate holding forty 40 GB application servers and two 5 TB file servers has no meaningful average. A single average overstates storage for the many small VMs and hides the one problem that actually breaks a deployment: a single large VM cannot be split across proxies, so its full backup read time sets the floor on the backup window no matter how much proxy capacity is added.

This calculator models Small, Medium, and Large tiers separately, each with its own average size and its own daily change rate, then sums the results. It reports the largest VM read time against the configured window and warns when the window cannot be met.

## Quick start

1. Download `Veeam-Sizing-Calculator.html` from this repository, or clone it.
2. Open the file in any modern browser by double-clicking it.
3. Enter the customer name and your name in the top bar.
4. Adjust the VM size tiers, global assumptions, and retention on the left.
5. Add or remove sites, and mark the site hosting the backup server as the VBR Hub.
6. The report on the right updates as you type.
7. Click **Export HTML** for a self-contained report file, or **Print / PDF** for a PDF.

There is nothing to install. The file can be opened from a USB drive, an email attachment, or a network share.

## Inputs

**VM size tiers**

| Field | Default | Notes |
|---|---|---|
| Small average size and change rate | 40 GB, 5% | Application and utility servers |
| Medium average size and change rate | 500 GB, 3% | General purpose servers |
| Large average size and change rate | 5000 GB, 2% | File, database, and archive servers |

**Global assumptions**

| Field | Default | Notes |
|---|---|---|
| Compression ratio | 50% | Validate against pilot backup session statistics |
| Backup window | 8 hours | Nightly incremental window |
| Concurrent tasks per host | 4 | Reduce on hosts already running under heavy production load |
| Repository type | Block cloning (ReFS/XFS) | Switch to Independent fulls for NTFS or deduplicating appliances |
| VMware proxy OS | Veeam Infrastructure Appliance (Linux JeOS) | Alternatives are self-deployed Linux or Windows Server |

**Retention (GFS)**

Daily, weekly, and monthly restore point counts. Defaults are 14 daily, 4 weekly, 3 monthly.

**Per site**

Site name, Hyper-V host count, ESXi host count, Small, Medium, and Large VM counts, WAN bandwidth in Mbps, and a VBR Hub flag. Sites can be added and removed. The tool ships with three sample sites so the report is populated on first open. Click **Reset** to return to those defaults.

## What the report contains

1. **Environment summary.** Totals, platform mix, tier sizes, retention, VBR instance count, and database recommendation.
2. **VM size profile.** Per site breakdown by tier with the blended change rate that drives incremental size and WAN transfer.
3. **Proxy deployment plan.** What to deploy, how many, where, and in which transport mode. Hyper-V On-Host proxies and VMware proxy VMs are handled separately, because the two platforms use different models and a single job cannot span both.
4. **VBR server.** Minimum and recommended specification for the hub.
5. **Repository sizing per site.** Compute and storage, with the block cloning figure and the independent fulls figure shown side by side.
6. **Bandwidth planning.** Daily incremental to the offsite target, copy time, initial seed time, and whether a throttling rule is needed.
7. **Job design.** Job list split by platform, by site, and with large VMs isolated into their own jobs.
8. **3-2-1 compliance.** How each rule is met by the proposed design.
9. **Overall summary.** Every headline number in one table, with risk flags.

**Automatic warnings**

- **Backup window risk.** The largest VM full read time exceeds the configured window. The report explains why adding proxies does not fix this and lists the options that do.
- **Single point of failure.** A site with only one host has no proxy redundancy.
- **Throttling required.** A site on 500 Mbps or less needs a global network traffic rule.

## Calculation basis

Stated so the numbers can be checked rather than taken on trust.

| Item | Basis |
|---|---|
| Proxy throughput | 180 MB/s per core for full backups, 80 MB/s per core for incrementals |
| Repository cores | One core per three proxy concurrent tasks, minimum two |
| Repository RAM | 4 GB per repository core, plus 0.5 GB per TB of repository storage for ReFS |
| Storage, block cloning | One active full plus the daily chain, with weekly and monthly GFS points stored as changed blocks, calculated per tier |
| Storage, independent fulls | One active full plus the daily chain, with every weekly and monthly GFS point stored as a complete full |
| Storage overhead | 25% added for free space, working space, and indexes, rounded up to the nearest 0.5 TB |
| VMware proxy count | Ceiling of eight concurrent tasks per proxy VM |
| VMware proxy RAM | 2 GB per task, with an 8 GB floor for the Veeam Infrastructure Appliance |
| WAN usable throughput | 80% of the stated link speed |
| Copy throughput | 25 MB/s on links of 250 Mbps or less, 100 MB/s above that |

Design guidance follows the Veeam Best Practice Guide at [bp.veeam.com/vbr](https://bp.veeam.com/vbr).

## Limitations

- This is a design and budgeting estimate. Change rate and compression are the two inputs that move the result the most, and both should be validated with a pilot backup before hardware is purchased.
- Enter **used** disk space, not provisioned. Check with `Get-VM | Get-VHD` on Hyper-V or a vSphere provisioned space report.
- The 3-2-1 section assumes an offsite copy to Veeam Data Cloud. If the design uses a different offsite target, the table wording needs to be adjusted after export.
- The following are not modelled: deduplicating appliance behaviour, tape, NAS and file share backup, agent-based workloads, VM replication, Veeam ONE, Veeam Recovery Orchestrator, cloud storage cost, and licence counts.
- The tool assumes a single VBR instance, which is appropriate below roughly 5,000 VMs.
- Input is not saved. Closing or refreshing the page returns the defaults, so export the report before closing.

## Privacy

The file contains no tracking, no analytics, and no external requests. All calculation happens in the browser. The exported report is generated locally and never uploaded.

## Related

- Free Microsoft 365, Active Directory, and Veeam tools at [m365admintools.com](https://m365admintools.com)

## Author

Charles Arconi, [m365admintools.com](https://m365admintools.com)

Not affiliated with, endorsed by, or supported by Veeam Software. Veeam is a trademark of Veeam Software Group GmbH.

## License

MIT. See [LICENSE](LICENSE).
