# RNA Atlas for Liao lab members

**Download this release:** Open [`RNAAtlas-GitHub-source.zip`](RNAAtlas-GitHub-source.zip) and download that file, then unzip it on your Mac. The detailed instructions below apply to the extracted `RNAAtlas-GitHub-source/` folder. Do not use GitHub's **Code → Download ZIP** button unless you also unzip the nested `RNAAtlas-GitHub-source.zip` inside it. You can check the downloaded archive against [`RNAAtlas-GitHub-source.zip.sha256`](RNAAtlas-GitHub-source.zip.sha256). The source code and small Candida example are inside the archive rather than individually browsable in this repository.

RNA Atlas is a **personal browser app for macOS**. It opens on your own computer, connects to Dartmouth Discovery as **you**, and submits Slurm jobs under **your NetID**. Your FASTQs, job logs, temporary workspace, and results stay in your own DartFS workspace. The lab's `Service_Account/RNAseq_atlas` supplies shared, read-only genome indexes and software; it is **not** where your projects run.

This repository is intended to remain **private to the lab**. Do not put patient information, passwords, SSH keys, Kerberos tickets, or your research FASTQs in GitHub. The only reads intentionally included here are the small Candida example described below.

## 1. Check access before downloading

You need a Mac with Python 3.9 or newer, a Discovery account, access to Dartmouth's network or VPN, and a writable DartFS folder under `/dartfs/rc/lab/L/LiaoC/` that you own **outside** `Service_Account`. You also need read/traverse access to the lab's shared `Service_Account/RNAseq_atlas/shared/references` and `shared/runtime_v2` assets. If an access check fails, ask the lab maintainer to verify those permissions; do not use someone else's credentials.

On your Mac, obtain your own Kerberos ticket and confirm your normal Discovery SSH login in Terminal, following Dartmouth's current instructions. For example, if Kerberos is configured on your Mac:

```bash
kinit <your-netid>@KIEWIT.DARTMOUTH.EDU
ssh <your-netid>@discovery.dartmouth.edu
```

Verify Discovery's SSH host identity through Dartmouth's instructions before accepting a new host key. Exit the SSH session before launching the app. **Never type your Discovery password into the RNA Atlas browser page**; the app does not collect it.

## 2. Download and launch

Download the release archive linked above, unzip it into a folder on your Mac, then double-click `mac/start_personal.command` inside the extracted folder. On first launch, the script installs its pinned local Python packages, asks for your NetID and your own DartFS workspace path, checks Discovery and shared-asset access, and opens the browser on `127.0.0.1` with a private URL token. Keep the launcher window open while using the app; closing it stops the browser server but does not cancel submitted Slurm jobs.

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

## 4. Choose one analysis and submit

Choose **one** module per Slurm job. The browser shows the input, output, and key settings for the selected module:

| Module | Additional input | Main results |
| --- | --- | --- |
| STAR + DESeq2 differential expression | At least two conditions in the sample sheet | Gene counts, DE table, PCA/volcano plots, QC |
| STAR + bcftools RNA SNV discovery | Paired RNA reads and a reference | RNA-derived VCF and site summaries; not DNA genotypes |
| STAR + GATK ASE | Compatible DNA genotype VCF (with sample map if needed) | Per-allele RNA read counts at eligible DNA heterozygous sites |
| Trinity de novo transcriptome | Paired RNA reads; no genome required for assembly | Transcript FASTA, assembly statistics, read-representation summary |

Review trimming, library strandedness, any module-specific filters, and the selected reference. The displayed biological-ploidy recommendation comes from the reference manifest; it does **not** measure your sample's ploidy or detect aneuploidy. If your sample's known ploidy differs, consult the lab maintainer before SNV/ASE analysis. The standard resource form permits 1–32 CPUs, 8–256 GB RAM, and 1 hour–30 days of walltime; your Slurm account or available nodes may be more restrictive. Click **Review and submit → Validate project**, then **Submit Slurm job**.

Each job writes to `<personal-workspace>/<project_folder>/results/<module>/<job-id>/`, including status and logs. Loading the same folder later retains earlier results. Result bundles up to 2 GB are available as ZIP downloads for three days after completion. Larger bundles and all underlying results remain on DartFS; the three-day browser window is **not** a DartFS deletion policy. Monitor your quota and follow your lab's retention rules.

## Try the included Candida example

Copy `examples/Calbicans_WT_YL_HL/` from the extracted archive into your **own** DartFS workspace as one project folder. It has four paired samples (WT01, WT02, YL01, YL02), 20,000 read pairs each, a ready sample sheet, and input checksums. Open that copied folder, choose the FungiDB Release 71 Candida SC5314 reference, and try STAR + DESeq2 with WT as the reference condition. Results go into **your copy's** `results/differential_expression/<job-id>/`, not the service account or GitHub. This example tests input/output plumbing only; the small subset has no biological statistical power and no DNA VCF for ASE. See `examples/Calbicans_WT_YL_HL/README.md` inside the archive.

## What this download does not include

The repo contains `mac/` (browser app and personal launcher), `discovery/` (pipeline code that the launcher copies to your private workspace), the small Candida example, and `SOURCE_SHA256SUMS.txt`. It **does not** contain the large reference genomes/STAR indexes, software runtimes, your reads, completed results, or anyone's credentials. The launcher checks access to shared reference/runtime files before opening the page.

**Current validation limit:** local tests and isolated service-account QA cover DE, RNA SNV, DNA-informed ASE, and a small Trinity run, but a full job under a lab member's personal identity has **not** yet been verified. Long-job ticket renewal and immutable per-job code releases also remain to be implemented. See `mac/PERSONAL.md` inside the archive for the maintainer's handoff audit. Do not assume this first package is production-validated for every member or use RNA-derived variants for clinical decisions.
