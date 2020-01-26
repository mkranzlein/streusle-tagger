# Measuring Out-of-Domain Calibration Error Using DiMSUM

In order to assess whether the calibrated STREUSLE tagger retains its improved calibration on data in a different domain, we examine how it performs on the [DiMSUM](https://github.com/dimsum16/dimsum-data) test set. Unlike Streusle, DiMSUM does not include preposition supersenses in the labelset. It also uses different lexical categories / POS. As a result, the confidence scores that we get on the outputs of our STREUSLE model are not directly applicable. To remedy this, we need to do the following:

1. Create a consolidated, intermediate labelset.
2. Post-process STREUSLE confidence scores to convert them to consolidated labelset.
3. Rerun adaptive binning on the STREUSLE validation set with confidence scores from the consolidated labelset to get new bin boundaries.
4. Calibrate DiMSUM test using these new bin boundaries.
5. (TODO). See how well calibration with the consolidated labelset holds up on the STREUSLE test set.

## Updating DiMSUM
- Found some tokens in DiMSUM tagged as nouns or verbs with no supersense. Nathan manually corrected these in a spreadsheet and provided some updated lexcat information about other items. See `data/dimsum16/Nouns and Verbs without Supersenses.xlsx`.
- Exported "data/dimsum16/Nouns and Verbs without Supersenses.xlsx" to txt `data/dimsum16_dimsum16_test_updated.txt`
- Updated scripts/dimsum_to_jsonl.py to include labels in output and reran on `data/dimsum16_dimsum16_test_updated.txt` to get `dimsum16_test_updated_labeled.json`
- `dimsum16_test_updated_labeled.json` requires a little bit of reformatting in order to be read, which is handled by the dimsum confidence scores notebook. That notebook produces `dimsum16_test_updated_labeled_reformatted.json`


## STREUSLE lexcats to DiMSUM
- ADJ : ADJ
- ADV : ADV
- AUX : AUX
- CCONJ : CONJ
- DET : DET
- DISC : X
- INF : PART (per [UD guidelines](https://universaldependencies.org/docs/en/pos/all.html))
- INF.P : PART
- INTJ : INTJ
- N : NOUN
- NUM : NUM
- P : ADP
- POSS : PART (per [UD guidelines](https://universaldependencies.org/docs/en/pos/all.html))
- PP : ADP
- PRON : PRON
- PRON.POSS : PRON
- PUNCT : PUNCT
- SCONJ : SCONJ
- SYM : SYM
- V : VERB
- V.IAV : VERB
- V.LVC.cause : VERB
- V.LVC.full : VERB
- V.VID : VERB
- V.VPC.full : VERB
- V.VPC.semi : VERB
- X : X
- _ : X

## Creating a Consolidated Labelset and Converting Scores
- Mapped each STREUSLE lexcat to DiMSUM according to the dictionary above.
- Removed preposition supersenses
- Created dictionary containing new labels and the old STREUSLE indexes they correspond to `calibration/consolidated_labels.pickle`
- Generated each new STREUSLE confidence score as the sum of confidence scores from old labelset mapped in `calibration/consolidated_labels.pickle`
- Ran original confidence scores for DiMSUM and converted using similar process as STREUSLE (dropped <20 instances from dataset due to rare errors in mapping DiMSUM label to consolidated labelset)

## Calibration
- With STREUSLE and DiMSUM confidence scores using the same labelset, calibration follows the same process as the original experiements that used the STREUSLE validation and test sets.