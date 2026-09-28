# X.1 System preparation

> **Drafting note (not part of the thesis text).** The house-style exemplar
> `lategan_computational_2025.pdf` is **not present** in the project folder, the reference
> repository, or the uploads directory. This draft follows the house-style specification set out
> in `WRITEUP_PROMPT.md` §2–§3 (chapter-opening introduction, prose-then-`Pipeline of
> Operations`, flowchart per pipeline, Harvard citations, first person singular, methods in past
> tense, audit numbers as construction validation). Please upload the Lategan PDF so the
> formatting can be matched rather than inferred.

---

I built the enzyme system from the crystallographic trimer of *Bacillus subtilis* chorismate
mutase, repaired its chain termini against a higher-resolution structure of the same protein,
placed the chorismate substrate by superposition onto the crystallographic transition-state
analogue, and assigned protonation states before parameterisation. Each stage wrote an audit
record to `00_admin/`, and acceptance at each stage was conditional on that record meeting a
stated threshold.

## X.1.1 Acquisition and provenance locking of raw inputs

I retrieved the protein scaffold as PDB entry 2CHT, the crystal structure of the monofunctional
chorismate mutase from *B. subtilis* in complex with a transition-state analogue
(Chook, Ke and Lipscomb, 1993). I retrieved PDB entry 1DBF, the 1.30 Å structure of the same
catalytic homotrimer, as the template for terminal repair (Ladner *et al.*, 2000). Both files
were downloaded from the RCSB on 2026-07-06, recorded in `00_admin/step01_provenance.txt`.

I took the chorismate substrate coordinates from the CP2K examples repository at commit
`d71583a411a0a7d8557e94a24b87a3ceb87ec34e`, as three pre-aligned copies. I registered each copy
to the crystallographic analogue site it was aligned against by nearest-centre matching, giving
`liga.mol2` to site K210, `ligb.mol2` to site J212 and `ligc.mol2` to site L211, at centre
offsets of 0.161, 0.160 and 0.203 Å respectively. Each ligand carries 24 atoms, consistent with
the chorismate dianion.

I locked the identity of all five raw inputs by SHA-256 checksum in
`00_admin/sha256_step01_inputs.txt`. I recomputed all five checksums against the committed
copies of the inputs while drafting; all five match (see the verification log below).

## X.1.2 Terminal repair of the A/B/C trimer

The 2CHT deposition resolves twelve chains, of which I retained A, B and C as the paper-frame
trimer. I repaired the unresolved chain termini by grafting the corresponding residues from
1DBF, superposed on common backbone atoms. The global backbone superposition RMSD was 0.8037 Å
over 1396 atoms, recorded in `00_admin/step02_alignment_report.tsv`.

Grafting left two peptide links outside the acceptable C–N distance. I closed both by minimal
rigid translation of the grafted segment to a canonical C–N distance, after which all 378
peptide links checked were within tolerance (`00_admin/step02_repair_decision.txt`). This
translation conditions the join geometrically only; the graft bond angles and dihedrals are
relaxed by the minimisation and equilibration described in X.2, and were re-checked there.

## X.1.3 Cleanup and structural validation

I stripped crystallographic hydrogens, resolved alternate-location groups to a single
occupancy, and validated the resulting heavy-atom scaffold. The cleanup removed 601 hydrogens by
element and resolved 7 alternate-location groups, leaving 3030 heavy atoms, zero hydrogens, zero
alternate locations and three C-terminal OXT atoms. Backbone continuity held across all 378
peptide links, no residue carried duplicate atom names, no non-standard residue names remained,
and no active-site residue was absent (`00_admin/step03_validation_audit.txt`).

## X.1.4 Substrate placement and active-site contact audit

I placed one chorismate into each of the three sites by superposing the ligand onto the
crystallographic transition-state analogue pose of its registered source site. The fits gave
analogue RMSDs of 0.1267, 0.0797 and 0.1518 Å for sites A, B and C
(`00_admin/step04_placement_audit.txt`). I renamed the CP2K duplicate atom names to unique
labels for tleap safety, preserving coordinates, atom types, charges and bonds.

The chorismate mutase active site is formed at the interface of two adjacent monomers, so a site
is completed by residues contributed by the neighbouring chain. I therefore measured every
catalytic contact against all chains and reported the observed closest chain, which tests the
expected same-chain or cross-chain assignment rather than assuming it. All twelve active-site
assignments were recovered with zero mismatches in all three sites
(`00_admin/step04_catalytic_contact_report.tsv`).

**Table X.1.** Closest heavy-atom distance (Å) between each active-site residue and the placed
chorismate, per site. Primed residues are contributed by the adjacent chain. Source:
`00_admin/step04_catalytic_contact_report.tsv`.

| Residue | Origin | Site A | Site B | Site C |
|---|---|---|---|---|
| Arg7 | same | 2.782 | 2.782 | 2.709 |
| Arg90 | same | 2.845 | 2.865 | 2.997 |
| Glu78 | same | 3.362 | 2.900 | 3.098 |
| Tyr108 | same | 2.956 | 2.586 | 2.785 |
| Leu115 | same | 3.548 | 3.764 | 3.301 |
| Phe57′ | cross | 3.620 | 3.684 | 3.390 |
| Ala59′ | cross | 3.358 | 3.222 | 2.964 |
| Lys60′ | cross | 3.024 | 3.129 | 3.080 |
| Arg63′ | cross | 2.899 | 3.060 | 6.380 |
| Val73′ | cross | 3.594 | 3.525 | 3.492 |
| Thr74′ | cross | 3.448 | 3.266 | 3.612 |
| Cys75′ | cross | 3.384 | 3.231 | 3.188 |
| Arg116 (second shell) | same | 5.501 | 6.834 | 6.396 |

The active-site set used here is the published QM-cluster set of Agbaglo and DeYonker
`[PLACEHOLDER — needs sourcing: the script comment in step04_place_chorismate_abc.sh attributes
this residue set to the Agbaglo/DeYonker supporting information, scheme S3. The main text PDF is
held as nihms1987271.pdf but the supporting information is not in the reference repository, so
the attribution is currently unverified. Upload the SI, or the set must be restated as this
work's own with the geometric justification in Table X.1.]`. Arg116 lies 5.501–6.834 Å from the
substrate across the three sites, against 2.586–3.764 Å for every residue in the active-site
set, and I therefore classified it as second shell and report it for provenance only.

`[DECISION REQUIRED — Arg116 is excluded from the active-site set here on the Agbaglo/DeYonker
definition, but Claeyssens et al. (2011, Table 3) include Arg116 among the residues contributing
to electrostatic transition-state stabilisation, at 1.0 ± 0.9 kcal/mol. The two definitions
disagree. Options: (a) retain the Agbaglo set and note that Arg116's contribution in Claeyssens
is within one standard deviation of zero; (b) adopt the Claeyssens set for consistency with the
paper this work is benchmarked against. This choice propagates to every later statement about
which residues define the site.]`

Site C shows Arg63′ at 6.380 Å where sites A and B show 2.899 and 3.060 Å. I established that
this is inherited crystallographic heterogeneity rather than an artefact of the repair
workflow. The raw 2CHT deposition shows Arg63′ guanidinium distances spanning 2.9–7.3 Å across
its twelve analogue sites, with seven of twelve swung away, and the source site for `ligc` was
already swung in the crystal. I then compared the repaired A/B/C Arg63 side chains against raw
2CHT directly: all three gave a side-chain RMSD of 0.000 Å over seven side-chain atoms, verdict
unchanged (`00_admin/step05b_arg63_conformation_report.tsv`). The repair therefore altered no
Arg63 side chain.

## X.1.5 Acceptance freeze and protonation-state assignment

I froze the accepted pre-protonation complex and its three ligands, checksummed in
`00_admin/sha256_step06_accepted_preprotonation.txt`, with the acceptance conditions recorded in
`00_admin/step06_acceptance_decision.txt`.

I submitted the protein-only structure, 1010 atoms per chain over residues 1–127 in each of the
three chains (`00_admin/step07a_hpp_input_report.tsv`), to the H++ server for protonation-state
assignment at pH 7.0
`[PLACEHOLDER — needs sourcing: the remaining H++ run parameters (salinity, internal and
external dielectric) are recorded only in script comments, and the two committed copies of that
comment disagree — 00_admin/step07a_prepare_hpp_input.sh states internal dielectric 80 and
external 10, while phase1_system_dev/step07a_prepare_hpp_input.sh and step07b_audit_repair_hpp.sh
both state internal 10 and external 80. The server output 03_amber/protonation_hpp/
hpp_output_from_server.pdb carries no parameter record, only a URL remark. Confirm the values
actually entered, and cite the H++ method paper — DOI required, see Sourcing gaps.]`

H++ returned 381 residues carrying 3153 added hydrogens, with all 3030 template heavy atoms
preserved and none added. Nine histidines were assigned, three as HIE and six as HIP. I used
only the protonation and orientation decisions from this output; the H++ topology and
coordinates were discarded and the assignments applied to the frozen template instead.

I accepted the protonated structure against thresholds on heavy-atom drift. Backbone drift was
0.000 Å over 1527 atoms against a 0.05 Å threshold, non-flippable side-chain drift 0.000 Å
against 0.2 Å, and active-site backbone drift 0.000 Å against 0.05 Å. No active-site heavy atom
moved more than 1.0 Å. Twenty-two heavy atoms drifted beyond 1.0 Å, maximum 2.2452 Å, and all
were confined to non-active-site flippable asparagine, glutamine and histidine side chains,
which is the expected signature of H++ terminal-amide and imidazole flips
(`00_admin/step07b_audit_summary.tsv`, `00_admin/step07b_acceptance_decision.txt`). No residue
name fell outside ff14SB, no heavy atom was missing or extra, all 378 peptide links were normal,
and no disulfide was detected.

## Pipeline of Operations

- `step01_download_inputs.sh` — retrieves 2CHT and 1DBF from the RCSB and the three chorismate
  reference geometries from the CP2K examples repository at a pinned commit. Outputs
  `01_inputs/structures/2cht_raw.pdb`, `01_inputs/structures/1dbf_raw.pdb`,
  `01_inputs/ligands/lig{a,b,c}.mol2`, `00_admin/step01_download_manifest.tsv`,
  `00_admin/sha256_step01_inputs.txt`, `00_admin/step01_provenance.txt`.
- `step01b_ligand_tsa_registration.sh` — matches each reference ligand to the crystallographic
  analogue site it was aligned against, by nearest centre. Outputs
  `00_admin/ligand_tsa_registration.tsv`.
- `step02_terminal_repair_abc.sh` — superposes 1DBF onto the 2CHT A/B/C trimer, grafts the
  unresolved terminal residues, and closes bad peptide links by rigid translation. Outputs
  `02_preparation/protein_only/abc_repaired.pdb`,
  `02_preparation/protein_only/abc_repaired_unadjusted.pdb`,
  `02_preparation/protein_only/1dbf_abc_aligned_to_2cht.pdb`,
  `00_admin/step02_alignment_report.tsv`, `00_admin/step02_join_adjustment_report.tsv`,
  `00_admin/step02_peptide_continuity_report.tsv`, `00_admin/step02_residue_source_report.tsv`,
  `00_admin/step02_repair_decision.txt`.
- `step03_cleanup_validate.sh` — strips hydrogens, resolves alternate locations, and validates
  the heavy-atom scaffold. Outputs `02_preparation/protein_only/abc_repaired_clean.pdb`,
  `00_admin/step03_cleanup_report.tsv`, `00_admin/step03_validation_audit.txt`.
- `step04_place_chorismate_abc.sh` — superposes each chorismate onto its registered analogue
  pose, renames duplicate atom names, and measures every active-site contact against all chains.
  Outputs `02_preparation/ligand_placement/abc_with_chorismate_unprotonated.pdb`,
  `02_preparation/ligand_placement/cha_{a,b,c}_placed.mol2`,
  `00_admin/step04_placement_audit.txt`, `00_admin/step04_catalytic_contact_report.tsv`,
  `00_admin/step04_placement_transform_report.tsv`, `00_admin/step04_placement_contact_report.txt`.
- `step05_arg63_provenance.sh` — surveys Arg63′ guanidinium placement across all twelve analogue
  sites of raw 2CHT. Outputs `00_admin/step05_arg63_provenance_report.tsv`.
- `step05b_arg63_conformation_check.sh` — compares repaired A/B/C Arg63 side chains against raw
  2CHT. Outputs `00_admin/step05b_arg63_conformation_report.tsv`.
- `step06_freeze_preprotonation.sh` — freezes and checksums the accepted pre-protonation
  complex. Outputs `02_preparation/accepted_preprotonation/abc_cha_preprotonation.pdb`,
  `02_preparation/accepted_preprotonation/cha_{a,b,c}.mol2`,
  `02_preparation/accepted_preprotonation/cha_{a,b,c}_renaming.tsv`,
  `00_admin/sha256_step06_accepted_preprotonation.txt`,
  `00_admin/step06_acceptance_decision.txt`, `00_admin/step06_combined_pdb_audit.txt`,
  `00_admin/step06_ligand_mol2_audit.tsv`.
- `step07a_prepare_hpp_input.sh` — writes the protein-only, TER-delimited structure for
  submission to H++. Outputs `03_amber/protonation_hpp/hpp_input_protein_only_with_TER.pdb`,
  `00_admin/step07a_hpp_input_report.tsv`.
- `step07b_audit_repair_hpp.sh` — audits the returned H++ structure against the frozen template,
  classifies heavy-atom drift, and records the acceptance decision. Outputs
  `03_amber/protonation_hpp/abc_protonated_hpp_accepted.pdb`,
  `00_admin/step07b_audit_summary.tsv`, `00_admin/step07b_drift_by_atom.tsv`,
  `00_admin/step07b_drift_by_residue.tsv`, `00_admin/step07b_heavy_atom_drift.tsv`,
  `00_admin/step07b_acceptance_decision.txt`.

`[FIGURE X.1 — flowchart of the system-preparation pipeline. Nodes: RCSB 2CHT and 1DBF, and the
CP2K ligand set, feeding step01/step01b; step02 terminal repair; step03 cleanup and validation;
step04 placement and contact audit, with step05/step05b as a side branch producing the Arg63′
provenance verdict; step06 acceptance freeze; step07a/step07b protonation and acceptance,
terminating at abc_protonated_hpp_accepted.pdb as the input to X.2. Annotate each acceptance
gate with its threshold: 0.8037 Å superposition RMSD, 0 bad peptide links of 378, 3030 heavy
atoms retained, 0.000 Å backbone drift. Render from the file list in this Pipeline of
Operations; no data file required.]`

---

# Sourcing gaps

1. **House-style exemplar absent.** `lategan_computational_2025.pdf` is not in the project
   folder, the reference repository, or uploads. This draft follows the written specification in
   `WRITEUP_PROMPT.md` §2–§3 instead. Upload the PDF to close.
2. **Ladner *et al.* (2000).** DOI `10.1107/S0907444900004625`, read from the `JRNL DOI` record
   of `01_inputs/structures/1dbf_raw.pdb`. The PDF is not in the reference repository. Needed to
   cite the terminal-repair template.
3. **H++ method paper.** No DOI held and none recoverable from any committed file. Search terms:
   *Anandakrishnan, Aguilar, Onufriev — H++ 3.0 automating pK prediction and the preparation of
   biomolecular structures — Nucleic Acids Research 2012*. Please supply the DOI.
4. **H++ run parameters.** Recorded only in script comments, which disagree (see Inconsistencies
   1). No server-side record exists in `hpp_output_from_server.pdb`.
5. **Agbaglo and DeYonker supporting information.** The active-site residue set is attributed to
   their scheme S3 by the script comment, but only the main text (`nihms1987271.pdf`) is held and
   I have not read it in this session. A DOI for the paper is also needed for the citation.
   Supply the SI and the DOI, or the set is restated as this work's own.
6. **Bartlett and Johnson (1985)**, the transition-state analogue bound in 2CHT. Cited by
   Claeyssens *et al.* (2011) as *J. Am. Chem. Soc.*, 1985, **107**, 7792–7793. No DOI held;
   I will not reconstruct one. Please supply if the analogue is to be cited by name.

# Inconsistencies found

1. **H++ dielectric settings contradict between committed copies.**
   `00_admin/step07a_prepare_hpp_input.sh` line 11 states internal dielectric 80, external 10.
   `phase1_system_dev/step07a_prepare_hpp_input.sh` line 11 and
   `phase1_system_dev/step07b_audit_repair_hpp.sh` lines 18–19 both state internal 10, external
   80. Two records against one, and the latter is the physically conventional assignment for a
   solute in water, but this is not established by any committed output. Placeholdered above.

2. **`00_admin/` script archive is stale for `step04`.** The archived copy uses an active-site
   set containing Arg116 and omitting Val73′. The code-repository copy on `tier1-realism` uses
   the Agbaglo set, which omits Arg116 and includes Val73′. I established that
   `step04_catalytic_contact_report.tsv` contains Val73′ and classifies Arg116 as `second_shell`,
   so **the committed report was produced by the code-repository version and the archive is the
   outdated copy**. The methods text above therefore follows the code repository. The archive
   should be refreshed, because it is currently misleading as a provenance record.

3. **`00_admin/step06_acceptance_decision.txt` lists the superseded residue set.** Its
   active-site architecture paragraph names Arg63′, Lys60′, Thr74′, Cys75′, Phe57′ and Ala59′,
   omitting Val73′. This is the pre-Agbaglo list. The frozen structure is unaffected — only the
   decision text is out of date — but it should be corrected so it does not contradict Table X.1.

4. **Five further archived scripts differ from the code repository, immaterially.**
   `step01`, `step01b` and `step03` differ only by an added `export OMP_NUM_THREADS=1 ...` line;
   `step05b` differs only by a variable rename (`placed` → `protein`). None changes any output.
   `step07b_audit_repair_hpp.sh` is **not archived in `00_admin/` at all**, unlike its siblings.

5. **`WRITEUP_INDEX.md` branch guidance is stale.** It records `tier1-realism` at `e860bd7` as
   "convex-cutplane / fractional-charge exploration" and directs barrier work to
   `rebuild-cleanroom`. `tier1-realism` is now `f0e059c` and carries the Phase 1 ensemble
   pipeline. `WRITEUP_PROMPT.md` §1 repeats the `rebuild-cleanroom` instruction. Both should be
   updated before the barrier sections are drafted, or the wrong branch will be read.

# Verification log

- **Raw-input checksums re-derived.** I recomputed SHA-256 for all five committed raw inputs and
  compared against `00_admin/sha256_step01_inputs.txt`: `2cht_raw.pdb`, `1dbf_raw.pdb`,
  `liga.mol2`, `ligb.mol2`, `ligc.mol2` — **5 of 5 match**.
- **Crystallographic citations re-derived from the files.** DOIs `10.1073/pnas.90.18.8600`
  (2CHT) and `10.1107/S0907444900004625` (1DBF) were read from the `JRNL DOI` records of the
  committed PDB files, not from memory.
- **Residue set cross-checked against the committed report.** I confirmed that
  `step04_catalytic_contact_report.tsv` contains a `VAL 73` row and an `ARG 116` row labelled
  `second_shell`, matching the code-repository `active_site` and `second_shell` lists exactly,
  and not the archived script's list.
- **Archive-versus-code comparison.** I ran a byte comparison of all ten preparation scripts
  between `00_admin/` and `phase1_system_dev/` on `tier1-realism`: 3 identical, 6 differing
  (classified in Inconsistencies 1, 2 and 4), 1 not archived.
- **Table X.1 values** were read directly from `step04_catalytic_contact_report.tsv`; no value
  was rounded or interpolated.
- **No barrier is quoted in this subsection**, so no barrier re-derivation applies here.

# References

Chook, Y.M., Ke, H. and Lipscomb, W.N. (1993) 'Crystal structures of the monofunctional
chorismate mutase from *Bacillus subtilis* and its complex with a transition state analog',
*Proceedings of the National Academy of Sciences USA*, 90, pp. 8600–8603.
doi:10.1073/pnas.90.18.8600.

Claeyssens, F., Ranaghan, K.E., Lawan, N., Macrae, S.J., Manby, F.R., Harvey, J.N. and
Mulholland, A.J. (2011) 'Analysis of chorismate mutase catalysis by QM/MM modelling of
enzyme-catalysed and uncatalysed reactions', *Organic & Biomolecular Chemistry*, 9,
pp. 1578–1590. doi:10.1039/c0ob00691b.

Ladner, J.E., Reddy, P., Davis, A., Tordova, M., Howard, A.J. and Gilliland, G.L. (2000) 'The
1.30 Å resolution structure of the *Bacillus subtilis* chorismate mutase catalytic homotrimer',
*Acta Crystallographica Section D*, 56, pp. 673–683. doi:10.1107/S0907444900004625.
