# Anserini Regressions: TREC 2022 Deep Learning Track (Passage)

**Model**: uniCOIL (with doc2query-T5 expansions) zero-shot

This page describes baseline experiments, integrated into Anserini's regression testing framework, on the TREC 2022 Deep Learning Track passage ranking task using the MS MARCO V2 passage collection.
Here, we cover experiments with the uniCOIL model trained on the MS MARCO V1 passage ranking test collection, applied in a zero-shot manner, with doc2query-T5 expansions.

The uniCOIL model is described in the following paper:

> Jimmy Lin and Xueguang Ma. [A Few Brief Notes on DeepImpact, COIL, and a Conceptual Framework for Information Retrieval Techniques.](https://arxiv.org/abs/2106.14807) _arXiv:2106.14807_.

Note that the NIST relevance judgments provide far more relevant passages per topic, unlike the "sparse" judgments provided by Microsoft (these are sometimes called "dense" judgments to emphasize this contrast).
For additional instructions on working with MS MARCO V2 passage collection, refer to [this page](experiments-msmarco-v2.md).

The exact configurations for these regressions are stored in [this YAML file](../src/main/resources/regression/dl22-passage-unicoil-0shot.yaml).
Note that this page is automatically generated from [this template](../src/main/resources/docgen/templates/dl22-passage-unicoil-0shot.template) as part of Anserini's regression pipeline, so do not modify this page directly; modify the template instead.

From one of our Waterloo servers (e.g., `orca`), the following command will perform the complete regression, end to end:

```bash
python src/main/python/run_regression.py --index --verify --search --regression dl22-passage-unicoil-0shot
```

We make available a version of the MS MARCO passage corpus that has already been processed with uniCOIL, i.e., we have applied doc2query-T5 expansions, performed model inference on every document, and stored the output sparse vectors.
Thus, no neural inference is involved.

From any machine, the following command will download the corpus and perform the complete regression, end to end:

```bash
python src/main/python/run_regression.py --download --index --verify --search --regression dl22-passage-unicoil-0shot
```

The `run_regression.py` script automates the following steps, but if you want to perform each step manually, simply copy/paste from the commands below and you'll obtain the same regression results.

## Corpus Download

Download, unpack, and prepare the corpus:

```bash
# Download
wget https://rgw.cs.uwaterloo.ca/JIMMYLIN-bucket0/data/msmarco_v2_passage_unicoil_0shot.tar -P collections/

# Unpack
tar -xvf collections/msmarco_v2_passage_unicoil_0shot.tar -C collections/

# Rename (indexer is expecting corpus under a slightly different name)
mv collections/msmarco_v2_passage_unicoil_0shot collections/msmarco-v2-passage-unicoil-0shot
```

To confirm, `msmarco_v2_passage_unicoil_0shot.tar` is 41 GB and has an MD5 checksum of `1949a00bfd5e1f1a230a04bbc1f01539`.
With the corpus downloaded, the following command will perform the remaining steps below:

```bash
python src/main/python/run_regression.py --index --verify --search --regression dl22-passage-unicoil-0shot \
  --corpus-path collections/msmarco-v2-passage-unicoil-0shot
```

## Indexing

Sample indexing command:

```bash
target/appassembler/bin/IndexCollection \
  -collection JsonVectorCollection \
  -input /path/to/msmarco-v2-passage-unicoil-0shot \
  -index indexes/lucene-index.msmarco-v2-passage-unicoil-0shot/ \
  -generator DefaultLuceneDocumentGenerator \
  -threads 24 -impact -pretokenized -storeRaw \
  >& logs/log.msmarco-v2-passage-unicoil-0shot &
```

The path `/path/to/msmarco-v2-passage-unicoil-0shot/` should point to the corpus downloaded above.

The important indexing options to note here are `-impact -pretokenized`: the first tells Anserini not to encode BM25 doclengths into Lucene's norms (which is the default) and the second option says not to apply any additional tokenization on the uniCOIL tokens.
Upon completion, we should have an index with 138,364,198 documents.

For additional details, see explanation of [common indexing options](common-indexing-options.md).

## Retrieval

Topics and qrels are stored in [`src/main/resources/topics-and-qrels/`](../src/main/resources/topics-and-qrels/).
The regression experiments here evaluate on the 76 topics for which NIST has provided judgments as part of the TREC 2022 Deep Learning Track.

<!-- update link once data becomes public
The original data can be found [here](https://trec.nist.gov/data/deep2022.html).
-->

After indexing has completed, you should be able to perform retrieval as follows:

```bash
target/appassembler/bin/SearchCollection \
  -index indexes/lucene-index.msmarco-v2-passage-unicoil-0shot/ \
  -topics src/main/resources/topics-and-qrels/topics.dl22.unicoil.0shot.tsv.gz \
  -topicreader TsvInt \
  -output runs/run.msmarco-v2-passage-unicoil-0shot.unicoil-0shot.topics.dl22.unicoil.0shot.txt \
  -impact -pretokenized &

target/appassembler/bin/SearchCollection \
  -index indexes/lucene-index.msmarco-v2-passage-unicoil-0shot/ \
  -topics src/main/resources/topics-and-qrels/topics.dl22.unicoil.0shot.tsv.gz \
  -topicreader TsvInt \
  -output runs/run.msmarco-v2-passage-unicoil-0shot.unicoil-0shot+rm3.topics.dl22.unicoil.0shot.txt \
  -impact -pretokenized -rm3 -collection JsonVectorCollection &

target/appassembler/bin/SearchCollection \
  -index indexes/lucene-index.msmarco-v2-passage-unicoil-0shot/ \
  -topics src/main/resources/topics-and-qrels/topics.dl22.unicoil.0shot.tsv.gz \
  -topicreader TsvInt \
  -output runs/run.msmarco-v2-passage-unicoil-0shot.unicoil-0shot+rocchio.topics.dl22.unicoil.0shot.txt \
  -impact -pretokenized -rocchio -collection JsonVectorCollection &
```

Evaluation can be performed using `trec_eval`:

```bash
tools/eval/trec_eval.9.0.4/trec_eval -c -M 100 -m map -l 2 src/main/resources/topics-and-qrels/qrels.dl22-passage.txt runs/run.msmarco-v2-passage-unicoil-0shot.unicoil-0shot.topics.dl22.unicoil.0shot.txt
tools/eval/trec_eval.9.0.4/trec_eval -c -M 100 -m recip_rank -l 2 src/main/resources/topics-and-qrels/qrels.dl22-passage.txt runs/run.msmarco-v2-passage-unicoil-0shot.unicoil-0shot.topics.dl22.unicoil.0shot.txt
tools/eval/trec_eval.9.0.4/trec_eval -c -m ndcg_cut.10 src/main/resources/topics-and-qrels/qrels.dl22-passage.txt runs/run.msmarco-v2-passage-unicoil-0shot.unicoil-0shot.topics.dl22.unicoil.0shot.txt
tools/eval/trec_eval.9.0.4/trec_eval -c -m recall.100 -l 2 src/main/resources/topics-and-qrels/qrels.dl22-passage.txt runs/run.msmarco-v2-passage-unicoil-0shot.unicoil-0shot.topics.dl22.unicoil.0shot.txt
tools/eval/trec_eval.9.0.4/trec_eval -c -m recall.1000 -l 2 src/main/resources/topics-and-qrels/qrels.dl22-passage.txt runs/run.msmarco-v2-passage-unicoil-0shot.unicoil-0shot.topics.dl22.unicoil.0shot.txt

tools/eval/trec_eval.9.0.4/trec_eval -c -M 100 -m map -l 2 src/main/resources/topics-and-qrels/qrels.dl22-passage.txt runs/run.msmarco-v2-passage-unicoil-0shot.unicoil-0shot+rm3.topics.dl22.unicoil.0shot.txt
tools/eval/trec_eval.9.0.4/trec_eval -c -M 100 -m recip_rank -l 2 src/main/resources/topics-and-qrels/qrels.dl22-passage.txt runs/run.msmarco-v2-passage-unicoil-0shot.unicoil-0shot+rm3.topics.dl22.unicoil.0shot.txt
tools/eval/trec_eval.9.0.4/trec_eval -c -m ndcg_cut.10 src/main/resources/topics-and-qrels/qrels.dl22-passage.txt runs/run.msmarco-v2-passage-unicoil-0shot.unicoil-0shot+rm3.topics.dl22.unicoil.0shot.txt
tools/eval/trec_eval.9.0.4/trec_eval -c -m recall.100 -l 2 src/main/resources/topics-and-qrels/qrels.dl22-passage.txt runs/run.msmarco-v2-passage-unicoil-0shot.unicoil-0shot+rm3.topics.dl22.unicoil.0shot.txt
tools/eval/trec_eval.9.0.4/trec_eval -c -m recall.1000 -l 2 src/main/resources/topics-and-qrels/qrels.dl22-passage.txt runs/run.msmarco-v2-passage-unicoil-0shot.unicoil-0shot+rm3.topics.dl22.unicoil.0shot.txt

tools/eval/trec_eval.9.0.4/trec_eval -c -M 100 -m map -l 2 src/main/resources/topics-and-qrels/qrels.dl22-passage.txt runs/run.msmarco-v2-passage-unicoil-0shot.unicoil-0shot+rocchio.topics.dl22.unicoil.0shot.txt
tools/eval/trec_eval.9.0.4/trec_eval -c -M 100 -m recip_rank -l 2 src/main/resources/topics-and-qrels/qrels.dl22-passage.txt runs/run.msmarco-v2-passage-unicoil-0shot.unicoil-0shot+rocchio.topics.dl22.unicoil.0shot.txt
tools/eval/trec_eval.9.0.4/trec_eval -c -m ndcg_cut.10 src/main/resources/topics-and-qrels/qrels.dl22-passage.txt runs/run.msmarco-v2-passage-unicoil-0shot.unicoil-0shot+rocchio.topics.dl22.unicoil.0shot.txt
tools/eval/trec_eval.9.0.4/trec_eval -c -m recall.100 -l 2 src/main/resources/topics-and-qrels/qrels.dl22-passage.txt runs/run.msmarco-v2-passage-unicoil-0shot.unicoil-0shot+rocchio.topics.dl22.unicoil.0shot.txt
tools/eval/trec_eval.9.0.4/trec_eval -c -m recall.1000 -l 2 src/main/resources/topics-and-qrels/qrels.dl22-passage.txt runs/run.msmarco-v2-passage-unicoil-0shot.unicoil-0shot+rocchio.topics.dl22.unicoil.0shot.txt
```

Note that the TREC 2022 passage qrels are not publicly available (yet).
However, if you are a participant, you can download them from the NIST "active participants" site.
Place the qrels file in `src/main/resources/topics-and-qrels/qrels.dl22-passage.txt` for the above evaluation commands to work.

## Effectiveness

With the above commands, you should be able to reproduce the following results:

| **MAP@100**                                                                                                  | **uniCOIL (with doc2query-T5) zero-shot**| **+RM3**  | **+Rocchio**|
|:-------------------------------------------------------------------------------------------------------------|-----------|-----------|-----------|
| [DL22 (Passage)](https://microsoft.github.io/msmarco/TREC-Deep-Learning)                                     | 0.1200    | 0.1292    | 0.1380    |
| **MRR@100**                                                                                                  | **uniCOIL (with doc2query-T5) zero-shot**| **+RM3**  | **+Rocchio**|
| [DL22 (Passage)](https://microsoft.github.io/msmarco/TREC-Deep-Learning)                                     | 0.5831    | 0.5634    | 0.5577    |
| **nDCG@10**                                                                                                  | **uniCOIL (with doc2query-T5) zero-shot**| **+RM3**  | **+Rocchio**|
| [DL22 (Passage)](https://microsoft.github.io/msmarco/TREC-Deep-Learning)                                     | 0.4566    | 0.4471    | 0.4799    |
| **R@100**                                                                                                    | **uniCOIL (with doc2query-T5) zero-shot**| **+RM3**  | **+Rocchio**|
| [DL22 (Passage)](https://microsoft.github.io/msmarco/TREC-Deep-Learning)                                     | 0.2996    | 0.2993    | 0.3113    |
| **R@1000**                                                                                                   | **uniCOIL (with doc2query-T5) zero-shot**| **+RM3**  | **+Rocchio**|
| [DL22 (Passage)](https://microsoft.github.io/msmarco/TREC-Deep-Learning)                                     | 0.5543    | 0.5661    | 0.5847    |

**IMPORTANT**: These runs are evaluated prior to dedup, so the scores will be slightly lower than the official scores (e.g., computed by NIST), which includes dedup.

The uniCOIL condition corresponds to the `p_unicoil_exp` run submitted to the TREC 2022 Deep Learning Track as a "baseline".
As of [`91ec67`](https://github.com/castorini/anserini/commit/91ec6749bfef206e210bcc1df8cd4060e7d7aaff), this correspondence was _exact_.
That is, modulo the runtag and the number of hits, the output runfile should be identical.
This can be confirmed as follows:

```bash
# Trim out the runtag:
cut -d ' ' -f 1-5 runs/p_unicoil_exp > runs/p_unicoil_exp.submitted.cut

# Trim out the runtag and retain only top 100 hits per query:
cut -d ' ' -f 1-5 runs/run.msmarco-v2-passage-unicoil-0shot.dl22.unicoil-0shot | grep -E '^[^ ]+ Q0 [^ ]+ (\d|\d\d|100) ' > runs/p_unicoil_exp.new.cut

# Verify the two runfiles are identical:
diff runs/p_unicoil_exp.submitted.cut runs/p_unicoil_exp.new.cut
```

The "uniCOIL + Rocchio" condition corresponds to the `p_unicoil_exp_rocchio` run submitted to the TREC 2022 Deep Learning Track as a "baseline".
However, due to [`a60e84`](https://github.com/castorini/anserini/commit/a60e842e9b47eca0ad5266659081fe1180c96b7f), the results are slightly different (because the underlying implementation changed).

To reproduce the official NIST scores, we need to run dedup.
Official submissions start off with only 100 hits per query, which is (slightly) reduced after the dedup process.
Thus, we have to take our runs (which contain 1000 hits per query) and trim down to 100 hits per query prior to running dedup.
These commands are shown below:

```bash
# Trim from 1000 hits to 100 hits
python tools/scripts/trim_run_to_top_k.pl --k 100 --input runs/run.msmarco-v2-passage-unicoil-0shot.dl22.unicoil-0shot --output runs/run.msmarco-v2-passage-unicoil-0shot.dl22.unicoil-0shot.hits100
python tools/scripts/trim_run_to_top_k.pl --k 100 --input runs/run.msmarco-v2-passage-unicoil-0shot.dl22.unicoil-0shot+rm3 --output runs/run.msmarco-v2-passage-unicoil-0shot.dl22.unicoil-0shot+rm3.hits100
python tools/scripts/trim_run_to_top_k.pl --k 100 --input runs/run.msmarco-v2-passage-unicoil-0shot.dl22.unicoil-0shot+rocchio --output runs/run.msmarco-v2-passage-unicoil-0shot.dl22.unicoil-0shot+rocchio.hits100

# Run dedup
python tools/scripts/dedup.py tools/topics-and-qrels/msmarco-v2-passage-neardupes.txt.gz \
  runs/run.msmarco-v2-passage-unicoil-0shot.dl22.unicoil-0shot.hits100 \
  runs/run.msmarco-v2-passage-unicoil-0shot.dl22.unicoil-0shot+rm3.hits100 \
  runs/run.msmarco-v2-passage-unicoil-0shot.dl22.unicoil-0shot+rocchio.hits100

# Evaluate
tools/eval/trec_eval.9.0.4/trec_eval -c -m ndcg_cut.10 src/main/resources/topics-and-qrels/qrels.dl22-passage.txt runs/run.msmarco-v2-passage-unicoil-0shot.dl22.unicoil-0shot.hits100.dedup
tools/eval/trec_eval.9.0.4/trec_eval -c -m ndcg_cut.10 src/main/resources/topics-and-qrels/qrels.dl22-passage.txt runs/run.msmarco-v2-passage-unicoil-0shot.dl22.unicoil-0shot+rm3.hits100.dedup
tools/eval/trec_eval.9.0.4/trec_eval -c -m ndcg_cut.10 src/main/resources/topics-and-qrels/qrels.dl22-passage.txt runs/run.msmarco-v2-passage-unicoil-0shot.dl22.unicoil-0shot+rocchio.hits100.dedup

tools/eval/trec_eval.9.0.4/trec_eval -c -m map -l 2 src/main/resources/topics-and-qrels/qrels.dl22-passage.txt runs/run.msmarco-v2-passage-unicoil-0shot.dl22.unicoil-0shot.hits100.dedup
tools/eval/trec_eval.9.0.4/trec_eval -c -m map -l 2 src/main/resources/topics-and-qrels/qrels.dl22-passage.txt runs/run.msmarco-v2-passage-unicoil-0shot.dl22.unicoil-0shot+rm3.hits100.dedup
tools/eval/trec_eval.9.0.4/trec_eval -c -m map -l 2 src/main/resources/topics-and-qrels/qrels.dl22-passage.txt runs/run.msmarco-v2-passage-unicoil-0shot.dl22.unicoil-0shot+rocchio.hits100.dedup

tools/eval/trec_eval.9.0.4/trec_eval -c -m recall.100 -l 2 src/main/resources/topics-and-qrels/qrels.dl22-passage.txt runs/run.msmarco-v2-passage-unicoil-0shot.dl22.unicoil-0shot.hits100.dedup
tools/eval/trec_eval.9.0.4/trec_eval -c -m recall.100 -l 2 src/main/resources/topics-and-qrels/qrels.dl22-passage.txt runs/run.msmarco-v2-passage-unicoil-0shot.dl22.unicoil-0shot+rm3.hits100.dedup
tools/eval/trec_eval.9.0.4/trec_eval -c -m recall.100 -l 2 src/main/resources/topics-and-qrels/qrels.dl22-passage.txt runs/run.msmarco-v2-passage-unicoil-0shot.dl22.unicoil-0shot+rocchio.hits100.dedup
```

With the above commands, we should arrive at the following sores:

|              | **uniCOIL (with doc2query-T5) zero-shot** | **+RM3**  | **+Rocchio**|
|:-------------|-------------------------------------------|-----------|-------------|
| **nDCG@10**  | 0.4475                                    | 0.4422    | 0.4716      |
| **MAP@100**  | 0.1097                                    | 0.1198    | 0.1277      |
| **R@100**    | 0.2790                                    | 0.2787    | 0.2915      |
