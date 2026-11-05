# RNA Atlas for Liao lab members

Download `RNAAtlas-GitHub-source.zip` from the private lab repository and extract it on your Mac. If GitHub shows 404, ask the lab maintainer for repository access. GitHub's **Code → Download ZIP** downloads an outer repository ZIP; extract the `RNAAtlas-GitHub-source.zip` inside it as well.

RNA Atlas is a **personal browser app for macOS**. It opens on your own computer and prepares a Slurm script in your DartFS project. **You then log in to Discovery with your own password and submit that script under your NetID.** Your FASTQs, job logs, temporary workspace, and results stay in your own DartFS workspace. The lab's `Service_Account/RNAseq_atlas` supplies shared, read-only genome indexes and software; it is **not** where your projects run.

**Use the personal app for lab work.** Its header identifies your own Discovery account and its button says **Create Slurm job**. That button prepares one script, then displays the script location and exact command to run on Discovery; it does **not** submit the job. If a page instead says **service session** and **Submit Slurm job**, you are viewing the separately restricted administrator QA app. Its button submits immediately under the service account. Do not use that page for member projects or share its private URL or token.

This repository is intended to remain **private to the lab**. Do not put patient information, passwords, SSH keys, Kerberos tickets, or your research FASTQs in GitHub. The only reads intentionally included here are the small Candida example described below.

## 1. Check access before downloading

You need a Mac with Python 3.9 or newer, a Discovery account, access to Dartmouth's network or VPN, and a writable DartFS folder under `/dartfs/rc/lab/L/LiaoC/` that you own **outside** `Service_Account`. You also need read/traverse access to the lab's shared `Service_Account/RNAseq_atlas/shared/references` and `shared/runtime_v2` assets. If an access check fails, ask the lab maintainer to verify those permissions; do not use someone else's credentials.

Before launching RNA Atlas, complete these steps in **Terminal on your own Mac**. Replace every `YOUR_NETID` below with your Dartmouth NetID (without angle brackets); use only your own account. Dartmouth requires the campus network or VPN for [Discovery SSH access](https://rc.dartmouth.edu/hpc/intro-to-hpc/logging-into-the-cluster/).

1. Connect to Dartmouth's campus network or VPN. Obtain a Kerberos ticket and confirm that it is current:

```bash
kinit YOUR_NETID@KIEWIT.DARTMOUTH.EDU
klist
```

`kinit` may ask for your Dartmouth password **in Terminal**; its characters will not appear as you type. `klist` must show an unexpired ticket for your own `YOUR_NETID@KIEWIT.DARTMOUTH.EDU` principal. If it does not, stop and resolve your Kerberos access before opening the app.

2. Verify the non-interactive SSH connection used **only to prepare and browse** your project. If SSH displays an unfamiliar host-key fingerprint, confirm it with [Dartmouth Research Computing](https://rc.dartmouth.edu/hpc/discovery-overview/) before accepting it; do not guess or bypass a host-key warning.

```bash
ssh -o BatchMode=yes -o StrictHostKeyChecking=yes -o GSSAPIAuthentication=yes -o GSSAPIDelegateCredentials=yes -o PubkeyAuthentication=no YOUR_NETID@discovery.dartmouth.edu 'id -un; klist'
```

This must finish without a password prompt, print your NetID, and show a valid remote ticket. If the host key is new, verify its fingerprint using Dartmouth's instructions in an interactive Terminal session before retrying this strict check. If it fails, check VPN and `klist`, then ask the lab maintainer or Dartmouth Research Computing for help. The browser's project-preparation connection uses `BatchMode=yes` and cannot prompt for a password. Renew an expired Mac ticket with `kinit` before a later browser session. **Never enter your password in the RNA Atlas browser page or send it to the lab**; the app does not collect it.

## 2. Download and launch

From the extracted `RNAAtlas-GitHub-source/` folder, double-click `mac/start_personal.command`. On first launch, the script installs its pinned local Python packages, asks for your NetID and your own DartFS workspace path, checks Discovery and shared-asset access, and opens the browser on `127.0.0.1` with a private URL token. Keep the launcher window open while using the app; closing it stops the browser server but does not cancel submitted Slurm jobs.

For a diagnostic check from Terminal instead of opening the page:

```bash
cd mac
scripts/setup_mac.sh
.venv-web/bin/python personal_launcher.py --check
```

The first launch creates `~/.rnaseq-atlas/personal.json` and a private browser token on **your Mac**. To change your NetID or DartFS workspace later, run `.venv-web/bin/python personal_launcher.py --configure`. Do not share the token, expose the localhost port, or commit `~/.rnaseq-atlas/`.

## 3. Prepare a project in your own DartFS workspace

Create one folder for each study directly under `<personal-workspace>/`. The folder's name becomes the project name; you do **not** need a folder literally named `projects`. Put inputs in its `datasets/` subfolder before opening it in RNA Atlas:

```text
<personal-workspace>/
└── <project_folder>/
    ├── datasets/
    │   ├── meta_data/sample_sheet.tsv
    │   └── raw_data/
    │       ├── S1/S1_1.fq.gz
    │       ├── S1/S1_2.fq.gz
    │       ├── S2/S2_1.fq.gz
    │       └── S2/S2_2.fq.gz
    ├── genotype_VCF/dna_genotypes.vcf.gz   # only for ASE
    └── results/                            # created by RNA Atlas
```

Use `sample_id` and `condition` columns in `sample_sheet.tsv`; the app offers a downloadable template. Make a `raw_data/<sample_id>/` folder for every sheet row. Name paired reads `<sample_id>_1.fq.gz` and `<sample_id>_2.fq.gz`; `.fastq.gz` also works. For ASE, add a pre-called **DNA genotype VCF** under `genotype_VCF/` before submission. If RNA and VCF sample names differ, also add `genotype_VCF/ase_genotype_map.tsv`. Do not use the RNA SNV output as a substitute for DNA genotypes.

In the browser, click **Open project folder**, choose the prepared folder, select a preinstalled reference, and click **Load data**. The app compares the sample sheet with paired-read filenames and samples up to 2,000 read pairs per sample. The default complete-pair validation checks the full FASTQ files when analysis begins. Resolve any missing, mismatched, corrupt, or unreadable inputs before submitting.

## 4. Choose one analysis and prepare its script

Choose **one** module per Slurm job. To run all four analyses, create and submit four separate jobs. Each job handles all samples, input checks, QC, and its selected analysis; you do not submit separate sample or preprocessing jobs. The browser shows the input, output, and key settings for the selected module:

| Module | Additional input | Main results |
| --- | --- | --- |
| STAR + DESeq2 differential expression | At least two conditions in the sample sheet | Gene counts, DE table, PCA/volcano plots, QC |
| STAR + bcftools RNA SNV discovery | Paired RNA reads and a reference | RNA-derived VCF and site summaries; not DNA genotypes |
| STAR + GATK ASE | Compatible DNA genotype VCF (with sample map if needed) | Per-allele RNA read counts at eligible DNA heterozygous sites |
| Trinity de novo transcriptome | Paired RNA reads; no genome required for assembly | Transcript FASTA, assembly statistics, read-representation summary |

Review trimming, library strandedness, any module-specific filters, and the selected reference. The displayed biological-ploidy recommendation comes from the reference manifest; it does **not** measure your sample's ploidy or detect aneuploidy. If your sample's known ploidy differs, consult the lab maintainer before SNV/ASE analysis. The standard resource form permits 1–32 CPUs, 8–256 GB RAM, and 1 hour–30 days of walltime; your Slurm account or available nodes may be more restrictive. Click **Review and create job → Validate project**, then **Create Slurm job**. **This creates the script but does not submit the job.**

The browser shows the exact script path and a one-line submission command. The script is at `<personal-workspace>/<project_folder>/results/<module>/<job-id>/rnaseq.sbatch`; the command uses the same job ID. In a **new Terminal session on your Mac**, log in with your own Discovery password, rather than forwarding the Mac ticket:

```bash
ssh -o GSSAPIAuthentication=no -o PubkeyAuthentication=no YOUR_NETID@discovery.dartmouth.edu
```

On Discovery, run `id -un` and `klist`. Confirm your NetID and a valid **Discovery-side** ticket. If `klist` reports no ticket, run `kinit YOUR_NETID@KIEWIT.DARTMOUTH.EDU` **there in your own Terminal** and check again. Then copy the exact `python3 .../scripts/submit_prepared.py <job-id>` command from RNA Atlas and run it on Discovery. This helper verifies the unchanged prepared script and configuration, calls `sbatch`, and records the Slurm job ID so the browser can track the queue. Do not run the `.sbatch` file with `bash`, and do not submit the same prepared job twice. You can type `exit` after Slurm accepts the job.

Each job writes to `<personal-workspace>/<project_folder>/results/<module>/<job-id>/`, including status and logs. Loading the same folder later retains earlier results. Result bundles up to 2 GB are available as ZIP downloads for three days after completion. Larger bundles and all underlying results remain on DartFS; the three-day browser window is **not** a DartFS deletion policy. Monitor your quota and follow your lab's retention rules.

Closing the browser or leaving Discovery after `sbatch` succeeds does **not** cancel the Slurm job. Renew the Mac ticket and click **Reconnect** if you need browser status updates later. Password-based login avoids the [specific macOS forwarded-ticket renewability problem](https://services.dartmouth.edu/TDClient/1806/Portal/KB/PrintArticle?ID=89203), but it does **not** by itself guarantee that a compute-node ticket will last through queue wait and runtime. The job checks for a ticket at startup; it does not renew one automatically. Until a password-submitted compute job and its credential lifetime have been verified on Discovery, do not assume long jobs are safe from DartFS ticket expiry.

## Try the included Candida example

Copy `examples/Calbicans_WT_YL_HL/` from the extracted archive into your **own** DartFS workspace as one project folder. It has four paired samples (WT01, WT02, YL01, YL02), 20,000 read pairs each, a ready sample sheet, and input checksums. Open that copied folder, choose the FungiDB Release 71 Candida SC5314 reference, and try STAR + DESeq2 with WT as the reference condition. Results go into **your copy's** `results/differential_expression/<job-id>/`, not the service account or GitHub. This example tests input/output plumbing only; the small subset has no biological statistical power and no DNA VCF for ASE. See `examples/Calbicans_WT_YL_HL/README.md` inside the extracted archive.

## What this download does not include

The repo contains `mac/` (browser app and personal launcher), `discovery/` (pipeline code that the launcher copies to your private workspace), the small Candida example, and `SOURCE_SHA256SUMS.txt`. It **does not** contain the large reference genomes/STAR indexes, software runtimes, your reads, completed results, or anyone's credentials. The launcher checks access to shared reference/runtime files before opening the page.

**Validation:** Local tests and isolated service-account QA cover DE, RNA SNV, DNA-informed ASE, and a small Trinity run. Use the included Candida example to test the personal submission workflow under your own Discovery account. Long-job ticket renewal and immutable per-job code releases remain to be implemented. See `mac/PERSONAL.md` inside the extracted archive for the maintainer's handoff audit. Do not use RNA-derived variants for clinical decisions.
