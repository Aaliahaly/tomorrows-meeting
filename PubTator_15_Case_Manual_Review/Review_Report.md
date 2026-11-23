# Manual review of the original 15 PubTator cases

Reviewed November 22, 2026. Scope: the 15 original diagnostic cases only.

## Summary

The original run recovered the submitted UniProt accession in 3 of 15 cases. This review finds that the low rate is not caused by our pipeline, and that it mostly reflects the test set rather than PubTator.

- **Pipeline:** the parser output matches the saved raw PubTator Gene annotations exactly (mention text, Gene ID, and character offsets) in 15 of 15 cases.
- **Reproducibility:** the PubTator responses retrieved again for this audit (November 22, 2026) are byte-for-byte identical to the saved responses from October 24, 2026, in 15 of 15 cases.
- **Testable cases:** only 7 of the 15 abstracts name the submitted target individually. The other 8 discuss a family, homolog, paralog group, operon or complex, or do not name the target at all, so no text-mining tool could recover the exact accession from them.
- **PubTator on the 7 testable cases:** 3 exact matches, 2 wrong-species normalizations, and 2 missed mentions.

| Outcome | Cases | n |
|---|---|---|
| Exact accession match | FVAL_001 (PAH), FVAL_045 (Clec4d), FVAL_063 (Olfr1393) | 3 |
| Recognized, normalized to the wrong species | FVAL_038 (AARE), FVAL_046 (TNK2) | 2 |
| Explicitly named, not annotated | FVAL_013 (YncL), FVAL_019 (MytiLec-3) | 2 |
| Not testable from the abstract | FVAL_012, 015, 027, 035, 044, 065, 071, 072 | 8 |

## Methods

For each case I read the supplied title and abstract, inspected the saved raw PubTator response (`original_raw/`), and compared its Gene annotations with `original_reports/gene_mentions.csv`. I retrieved each PMID individually from the PubTator3 export API (`fresh_raw/`) and compared the fresh annotations with the saved ones. Classification of whether the text names the target is a manual title/abstract judgment. No new UniProt mapping job was run; accession mappings cited below are the saved results in `original_reports/grouped_genes.csv`.

## Findings

**Wrong-species normalization (2 cases).** In both cases PubTator recognized the target name but assigned a gene from a different organism, so the accession could not match.

- *FVAL_046, TNK2/ACK1.* All 11 target mentions (TNK2, ACK1, Tnk2, tyrosine kinase non receptor 2) were assigned NCBI Gene 51789, mouse Tnk2; the saved candidates include mouse O54967. The submitted accession Q07912 is human ACK1. The abstract is mainly about human prostate cancer and mentions mice only for a transgenic model, yet every mention, including the human-context ones, was normalized to mouse.
- *FVAL_038, AARE.* All 8 mentions were assigned NCBI Gene 327, human APEH (saved mapping P13798). The paper studies AARE in *Physcomitrella* and *Arabidopsis*, both tagged as species by PubTator, and the submitted A9SN79 is the *Physcomitrella* protein (alias PpAARE3). Normalization defaulted to human.

**Missed explicit mentions (2 cases).**

- *FVAL_013, YncL.* Named directly ("two SMPs, YohP and YncL"). The response contains no Gene annotation at all, so YohP in the title is missed too.
- *FVAL_019, MytiLec-3.* Named through coordinated shorthand ("MytiLec-2 and -3"). There is no Gene annotation for any MytiLec variant, including MytiLec-1, which is named repeatedly.

**Not testable from the abstract (8 cases).**

- *Homolog or family context:* YisK (FVAL_015; OAT1 and its homologs only), OspD (FVAL_044; PoyD homologs), BcPINE1 (FVAL_072; SsPINE1 named, the *Botrytis* homolog not).
- *Paralog group:* ACS7 (FVAL_065; four paralogous acetyl-CoA synthetases, none singled out).
- *Operon or complex level:* UcdC (FVAL_035; ucd operon), EMC2 (FVAL_071; the EMC complex).
- *Target not named:* ABL1 (FVAL_012), phage capsid protein F (FVAL_027).

In FVAL_035 and FVAL_071, the Gene annotations that do exist are for other proteins (aryl hydrocarbon receptor; SOAT1 and FDFT1).

**Other annotation issues seen in passing (non-target).**

- *FVAL_071:* "EMC deficiency" is tagged as a Disease (MESH:D015433), not as a gene or complex.
- *FVAL_046:* ATP5F1A and ATP5IF1, central to the abstract, have no Gene annotation.

## Interpretation and limitations

The 3/15 figure is a submitted-accession recovery rate on this test set. It is not a measure of PubTator sensitivity. On the 7 cases that can be tested, the failures fall into two patterns: species selection during normalization, and recognition of small or non-model-organism proteins (bacterial SMPs, mussel lectins). Seven cases are too few to estimate error rates.

## Recommendations

1. Remove the 8 non-testable cases from the PubTator benchmark, or relabel them as family/complex-level cases scored separately.
2. Before UniProt matching, check the normalized gene's taxon against the submitted accession's organism. Flag mismatches as species errors, not as absent recognition.
3. For explicit mentions that PubTator misses (e.g., YncL), add a fallback exact-string match against the supplied names.

## Files

- `Manual_Review_15_Cases.csv`: per-case findings, text excerpts, and an `Outcome_category` column matching the table above.
- `original_raw/`: saved PubTator responses from the original run (October 24, 2026).
- `fresh_raw/`: responses retrieved for this audit (November 22, 2026).
- `original_reports/`: the original mention, grouping, and match-decision reports.
- `Original_15_Case_Input.txt`: the supplied input text.
