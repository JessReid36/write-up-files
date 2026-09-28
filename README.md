# write-up-files

Every file cited in the Phase 1 thesis write-up, collected in one place so each claim in the
text can be traced to the file it came from without cloning the full working repositories.

This repository is a **copy, not a source**. Nothing here is regenerated; every file is a
byte-identical copy of a committed file in one of the two working repositories, at the commits
recorded below and in `MANIFEST.tsv`.

## Provenance

| source | commit | branch |
|---|---|---|
| `JessReid36/chorismate-thesis-results` | `aa131d526c9b8f0da28e27bb11d4c6d36f69ae21` | `main` |
| `JessReid36/chorismate-thesis-code` | `f0e059cf11eda91dea939c42f8e12d1a8bf829a1` | `tier1-realism` |

`MANIFEST.tsv` lists every file with its SHA-256, size, and which repository and commit it came
from. Verify the copy at any time with:

```bash
awk -F'\t' 'NR>1{print $1"  "$3}' MANIFEST.tsv | sha256sum -c -
```

## Layout

| path | contents |
|---|---|
| `01_inputs/` | Raw crystallographic and ligand inputs as downloaded (2CHT, 1DBF, the three CP2K chorismate geometries) |
| `02_preparation/` | Terminal-repaired protein, cleaned scaffold, placed ligands, and the frozen pre-protonation complex |
| `03_amber/protonation_hpp/` | H++ input, the server's returned structure, and the accepted protonated structure |
| `00_admin/` | Every audit, report and acceptance-decision file cited in the text |
| `00_admin/archived_scripts_as_run/` | The `00_admin` script archive — see the caveat below |
| `scripts/` | The step01–step07b scripts from the code repository |
| `writeup/` | The write-up subsection itself, Markdown and Word |

## Caveat: two copies of the preparation scripts exist and six of them differ

The working repositories hold the preparation scripts in two places: the code repository, and an
archive inside `00_admin/` in the results repository. A byte comparison of all ten shows
3 identical, 6 differing, and 1 (`step07b_audit_repair_hpp.sh`) not archived at all.

`scripts/` holds the **code-repository versions**, which are the ones that produced the
committed outputs. `00_admin/archived_scripts_as_run/` holds the six divergent archived copies,
kept so the discrepancy stays visible rather than being silently resolved.

The differences:

| script | difference | material? |
|---|---|---|
| `step01_download_inputs.sh` | archive lacks an `export OMP_NUM_THREADS=1 ...` line | no |
| `step01b_ligand_tsa_registration.sh` | same | no |
| `step03_cleanup_validate.sh` | same | no |
| `step05b_arg63_conformation_check.sh` | variable renamed `placed` → `protein` | no |
| `step04_place_chorismate_abc.sh` | **different active-site residue set** — the archive includes Arg116 and omits Val73′; the code repository omits Arg116 (second shell) and includes Val73′ | **yes** |
| `step07a_prepare_hpp_input.sh` | **H++ dielectric settings contradict** — the archive states internal 80 / external 10; the code repository states internal 10 / external 80 | **yes, unresolved** |

For `step04`, the question is settled by the output itself:
`00_admin/step04_catalytic_contact_report.tsv` contains a `VAL 73` row and labels Arg116
`second_shell`, which matches the code-repository residue set and not the archive. The archive
is therefore the stale copy.

For `step07a`, it is **not** settled. The H++ parameters are recorded only in script comments,
`03_amber/protonation_hpp/hpp_output_from_server.pdb` carries no parameter record beyond a URL
remark, and no committed output distinguishes the two readings. The write-up carries a
placeholder at this point.

## Not included

Reference papers (Claeyssens 2005 and 2011, Agbaglo and DeYonker, and others cited in the text)
are **not** copied here. They are third-party copyrighted PDFs and redistributing them in a
repository would not be appropriate; they are cited by DOI in the write-up instead.
