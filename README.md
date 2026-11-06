# RNA Atlas for Liao lab members

Download `RNAAtlas-GitHub-source.zip` from the private lab repository and extract it on your Mac. If GitHub shows 404, ask the lab maintainer for repository access. GitHub's **Code → Download ZIP** downloads an outer repository ZIP; extract the `RNAAtlas-GitHub-source.zip` inside it as well.

**Open your personal browser page:** In the extracted `RNAAtlas-GitHub-source` folder, double-click `mac/start_personal.command`. A Terminal window opens. First enter **your NetID**. SSH then asks for your Discovery password; its input is hidden completely (no characters or `*` appear). Next enter the full path to a DartFS folder you own under `/dartfs/rc/lab/L/LiaoC/`, outside `Service_Account`. **The path field is visible text; never type a password there.** After the connection and access checks pass, your browser opens RNA Atlas automatically. Keep that Terminal window open. If the browser does not open, copy the **private URL printed in that Terminal window** into a browser on the same Mac. The personal app uses a different local port each time; do **not** open the administrator page at `http://127.0.0.1:8791/` or share your private URL. If Terminal reports an access error, resolve it with the lab maintainer rather than opening the administrator page.

RNA Atlas is a **personal browser app for macOS**. It opens on your own computer and prepares a Slurm script in your DartFS project. At launch, you enter your Discovery password **in Terminal** once; the app reuses that SSH connection to browse and prepare your project. You then open a Discovery shell through the same connection and submit the prepared script under your NetID. Your FASTQs, job logs, temporary workspace, and results stay in your own DartFS workspace. The lab maintains shared genome indexes and software under `Service_Account/RNAseq_atlas`; it is **not** where your projects run. The maintainer must confirm your account has safe read-only access to those assets before you use the app.

**Use the personal app for lab work.** Its header identifies your own Discovery account and its button says **Create Slurm job**. That button prepares one script, then displays the script location and exact command to run on Discovery; it does **not** submit the job. If a page instead says **service session** and **Submit Slurm job**, you are viewing the separately restricted administrator QA app. Its button submits immediately under the service account. Do not use that page for member projects or share its private URL or token.

This repository is intended to remain **private to the lab**. Do not put patient information, passwords, SSH keys, Kerberos tickets, or your research FASTQs in GitHub. The only reads intentionally included here are the small Candida example described below.

**First-run check:** Do not use this release for student research until the lab maintainer confirms shared-asset permissions: the current `Service_Account` directory may block ordinary lab accounts, while some assets beneath it are group-writable. Once access is safely configured, copy the included Candida example into your own workspace and complete one job under your own Discovery account. The maintainer has tested the pipeline with the service account, but not yet a member-owned job or long-job compute-node ticket renewal. If the example cannot be submitted or completed, stop and ask the lab maintainer to check permissions and credentials; do not move your data into `Service_Account`.

## 1. Check access before downloading

You need a Mac with Python 3.9 or newer, a Discovery account, access to Dartmouth's network or VPN, and a writable DartFS folder under `/dartfs/rc/lab/L/LiaoC/` that you own **outside** `Service_Account`. You also need read/traverse access to the lab's shared `Service_Account/RNAseq_atlas/shared/references` and `shared/runtime` assets. If an access check fails, ask the lab maintainer to verify those permissions; do not use someone else's credentials.

Connect to Dartmouth's campus network or VPN for [Discovery SSH access](https://rc.dartmouth.edu/hpc/intro-to-hpc/logging-into-the-cluster/). Use your own Dartmouth NetID and a DartFS workspace you own.

The launcher starts a reusable SSH connection and asks for your Discovery password **in its Terminal window**, not in the browser. If SSH shows a new host-key fingerprint, verify it with [Dartmouth Research Computing](https://rc.dartmouth.edu/hpc/discovery-overview/) **before** accepting it. Never accept an unfamiliar or changed key just to continue. The launcher then checks that Discovery reports your NetID, has a valid **Discovery-side** Kerberos ticket, and can read your DartFS workspace and the shared app assets. A password login alone does not prove DartFS ticket availability. If this check fails, stop and resolve your Discovery-side access; the browser will not open on an unverified connection. **Never enter your password in the RNA Atlas browser page or send it to the lab**; the app does not collect or save it.

## 2. Download and launch

From the extracted `RNAAtlas-GitHub-source/` folder, double-click `mac/start_personal.command`. On first launch, the script installs its pinned local Python packages, asks for your NetID, then starts SSH's hidden password prompt in Terminal. **Nothing appears as you type the password**; press Return when finished. Only then does it ask for **your own full DartFS workspace path—not a password**. For example, `/dartfs/rc/lab/L/LiaoC/<your-folder>/RNAseq_atlas` is valid only if `<your-folder>` is assigned to you. It must not point inside `Service_Account` or to a folder on your Mac. The launcher retries an invalid path without prompting for a password again. After the access checks pass, it opens the browser on `127.0.0.1` with a private URL token and prints that URL in Terminal as a fallback. Keep the launcher window open while using the app; closing it stops the browser server but does not cancel submitted Slurm jobs. The reusable SSH connection may also end after eight idle hours or when the network changes.

For a diagnostic check from Terminal instead of opening the page:

```bash
cd mac
scripts/setup_mac.sh
.venv-web/bin/python personal_launcher.py --check
```

The first launch creates `~/.rnaseq-atlas/personal.json`, a private browser token, and a user-only SSH connection socket on **your Mac**. To change your NetID or DartFS workspace later, run `.venv-web/bin/python personal_launcher.py --configure`. If the page shows **Discovery disconnected**, double-click `mac/connect_discovery.command`, enter your password in its Terminal window, then click **Reconnect** in the browser. Do not share the token, expose the localhost port, or commit `~/.rnaseq-atlas/`.

## 3. Prepare a project in your own DartFS workspace

The launcher creates `<personal-workspace>/projects/` automatically. Create one folder for each study inside it; that folder's name becomes the project name. Put inputs in its `datasets/` subfolder before opening it in RNA Atlas. Relaunching the app refreshes `app/` and `scripts/` code without replacing existing project folders or results; `shared/` keeps the job records needed to track previous runs. If you used an older release that kept a project directly under `<personal-workspace>/`, move that project folder into `projects/` yourself before opening it. The launcher does not move or overwrite it:

```text
<personal-workspace>/
├── app/                                    # refreshed launcher code
├── scripts/                                # refreshed launcher code
├── shared/                                 # persistent job records
└── projects/                               # created by the launcher
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

Review trimming, library strandedness, any module-specific filters, and the selected reference. For **DE and Trinity**, leave the strand setting on **Check a small alignment** unless you have a protocol-confirmed choice: the job maps up to 50,000 read pairs from every sample with STAR and shows forward/reverse assigned-gene counts. It automatically selects forward or reverse only when all samples have strong, agreeing evidence. Balanced, low-count, intermediate, or disagreeing samples stop before full analysis with **Needs review**; the browser shows per-sample evidence and offers an evidence ZIP for three days. Check your library protocol, choose an explicit orientation, tick the manual-review confirmation, and create/submit a new job. An explicit choice that contradicts the check also needs confirmation. A balanced signal does **not** prove unstrandedness. This check does not block SNV or ASE, whose modules do not use stranded gene counts. The displayed biological-ploidy recommendation comes from the reference manifest; it does **not** measure your sample's ploidy or detect aneuploidy. If your sample's known ploidy differs, consult the lab maintainer before SNV/ASE analysis. The standard resource form permits 1–32 CPUs, 8–256 GB RAM, and 1 hour–30 days of walltime; your Slurm account or available nodes may be more restrictive. Click **Review and create job → Validate project**, then **Create Slurm job**. **This creates the script but does not submit the job.**

The browser shows the exact script path and a one-line submission command. The script is at `<personal-workspace>/projects/<project_folder>/results/<module>/<job-id>/rnaseq.sbatch`; the command uses the same job ID. In **Terminal on your Mac**, use the exact SSH command shown in the browser to open a Discovery shell through the existing connection. For your NetID, it has this form:

```bash
ssh -S ~/.rnaseq-atlas/discovery-YOUR_NETID.sock -o BatchMode=yes -o ControlMaster=no -o GSSAPIAuthentication=no -o PubkeyAuthentication=no -o PasswordAuthentication=no -o KbdInteractiveAuthentication=no YOUR_NETID@discovery.dartmouth.edu
```

This reuses the password-authenticated login and normally does **not** ask for a second password. If it fails, use `mac/connect_discovery.command` to reauthenticate; do not bypass the host-key check. On Discovery, run `id -un` and `klist` and confirm your NetID and a valid **Discovery-side** ticket. Then copy the exact `python3 .../scripts/submit_prepared.py <job-id>` command from RNA Atlas and run it on Discovery. This helper verifies the unchanged prepared script, configuration, and pipeline-code snapshot, calls `sbatch`, and records the Slurm job ID so the browser can track the queue. Do not run the `.sbatch` file with `bash`, and do not submit the same prepared job twice. You can type `exit` after Slurm accepts the job. Each job's read-only `_code/` snapshot is checksum-checked again on the compute node; do not edit or delete it while the job is queued or running. Relaunching the app does not replace code used by an already prepared job.

Each job writes to `<personal-workspace>/projects/<project_folder>/results/<module>/<job-id>/`, including status and logs. Loading the same folder later retains earlier results. Result bundles up to 2 GB are available as ZIP downloads for three days after completion. Larger bundles and all underlying results remain on DartFS; the three-day browser window is **not** a DartFS deletion policy. Monitor your quota and follow your lab's retention rules.

Closing the browser or leaving Discovery after `sbatch` succeeds does **not** cancel the Slurm job. Reconnect using `mac/connect_discovery.command` if you need browser status updates later. Password-based login avoids the [specific macOS forwarded-ticket renewability problem](https://services.dartmouth.edu/TDClient/1806/Portal/KB/PrintArticle?ID=89203), but it does **not** by itself guarantee that a compute-node ticket will last through queue wait and runtime. The job checks for a ticket at startup; it does not renew one automatically. Until a password-submitted compute job and its credential lifetime have been verified on Discovery, do not assume long jobs are safe from DartFS ticket expiry.

## Try the included Candida example

Copy `examples/Calbicans_WT_YL_HL/` from the extracted archive into your **own** DartFS workspace's `projects/` folder as one project folder. It has four paired samples (WT01, WT02, YL01, YL02), 20,000 read pairs each, a ready sample sheet, and input checksums. Open that copied folder, choose the FungiDB Release 71 Candida SC5314 reference, and try STAR + DESeq2 with WT as the reference condition. Results go into **your copy's** `results/differential_expression/<job-id>/`, not the service account or GitHub. This example tests input/output plumbing only; the small subset has no biological statistical power and no DNA VCF for ASE. See `examples/Calbicans_WT_YL_HL/README.md` inside the extracted archive.

## What this download does not include

The repo contains `mac/` (browser app and personal launcher), `discovery/` (pipeline code that the launcher copies to your private workspace), the small Candida example, and `SOURCE_SHA256SUMS.txt`. It **does not** contain the large reference genomes/STAR indexes, software runtimes, your reads, completed results, or anyone's credentials. The launcher checks access to shared reference/runtime files before opening the page.

**Validation:** Local tests and isolated service-account QA cover DE, RNA SNV, DNA-informed ASE, and a small Trinity run. Each personal job pins a checksum-verified pipeline-code snapshot. Use the included Candida example to test the personal submission workflow under your own Discovery account. A member-owned end-to-end job and long-job compute-node ticket lifetime have not yet been verified; do not assume those are safe without that check. See `mac/PERSONAL.md` inside the extracted archive for the maintainer's handoff audit. Do not use RNA-derived variants for clinical decisions.
