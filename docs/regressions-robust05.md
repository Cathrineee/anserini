# Anserini Regressions: TREC 2005 Robust Track

**Models**: various bag-of-words approaches

This page describes regressions for the TREC 2005 Robust Track, which uses the [AQUAINT collection](https://tac.nist.gov//data/data_desc.html#AQUAINT).
The exact configurations for these regressions are stored in [this YAML file](../src/main/resources/regression/robust05.yaml).
Note that this page is automatically generated from [this template](../src/main/resources/docgen/templates/robust05.template) as part of Anserini's regression pipeline, so do not modify this page directly; modify the template instead.

From one of our Waterloo servers (e.g., `orca`), the following command will perform the complete regression, end to end:

```
python src/main/python/run_regression.py --index --verify --search --regression robust05
```

## Indexing

Typical indexing command:

```
target/appassembler/bin/IndexCollection \
  -collection TrecCollection \
  -input /path/to/robust05 \
  -index indexes/lucene-index.robust05/ \
  -generator DefaultLuceneDocumentGenerator \
  -threads 16 -storePositions -storeDocvectors -storeRaw \
  >& logs/log.robust05 &
```

The directory `/path/to/aquaint/` should be the root directory of the [AQUAINT collection](https://tac.nist.gov//data/data_desc.html#AQUAINT); under subdirectory `disk1/` there should be `NYT/` and under subdirectory `disk2/` there should be `APW/` and `XIE/`.

For additional details, see explanation of [common indexing options](common-indexing-options.md).

## Retrieval

Topics and qrels are stored in [`src/main/resources/topics-and-qrels/`](../src/main/resources/topics-and-qrels/), downloaded from NIST:

+ [`topics.robust05.txt`](../src/main/resources/topics-and-qrels/topics.robust05.txt): [topics for the TREC 2005 Robust Track (Hard Topics of Robust04)](http://trec.nist.gov/data/robust/05/05.50.topics.txt)
+ [`qrels.robust05.txt`](../src/main/resources/topics-and-qrels/qrels.robust05.txt): [qrels for the TREC 2005 Robust Track (Hard Topics of Robust04)](http://trec.nist.gov/data/robust/05/TREC2005.qrels.txt)

After indexing has completed, you should be able to perform retrieval as follows:

```
target/appassembler/bin/SearchCollection \
  -index indexes/lucene-index.robust05/ \
  -topics src/main/resources/topics-and-qrels/topics.robust05.txt \
  -topicreader Trec \
  -output runs/run.robust05.bm25.topics.robust05.txt \
  -bm25 &

target/appassembler/bin/SearchCollection \
  -index indexes/lucene-index.robust05/ \
  -topics src/main/resources/topics-and-qrels/topics.robust05.txt \
  -topicreader Trec \
  -output runs/run.robust05.bm25+rm3.topics.robust05.txt \
  -bm25 -rm3 &

target/appassembler/bin/SearchCollection \
  -index indexes/lucene-index.robust05/ \
  -topics src/main/resources/topics-and-qrels/topics.robust05.txt \
  -topicreader Trec \
  -output runs/run.robust05.bm25+ax.topics.robust05.txt \
  -bm25 -axiom -axiom.deterministic -rerankCutoff 20 &

target/appassembler/bin/SearchCollection \
  -index indexes/lucene-index.robust05/ \
  -topics src/main/resources/topics-and-qrels/topics.robust05.txt \
  -topicreader Trec \
  -output runs/run.robust05.ql.topics.robust05.txt \
  -qld &

target/appassembler/bin/SearchCollection \
  -index indexes/lucene-index.robust05/ \
  -topics src/main/resources/topics-and-qrels/topics.robust05.txt \
  -topicreader Trec \
  -output runs/run.robust05.ql+rm3.topics.robust05.txt \
  -qld -rm3 &

target/appassembler/bin/SearchCollection \
  -index indexes/lucene-index.robust05/ \
  -topics src/main/resources/topics-and-qrels/topics.robust05.txt \
  -topicreader Trec \
  -output runs/run.robust05.ql+ax.topics.robust05.txt \
  -qld -axiom -axiom.deterministic -rerankCutoff 20 &
```

Evaluation can be performed using `trec_eval`:

```
tools/eval/trec_eval.9.0.4/trec_eval -m map -m P.30 src/main/resources/topics-and-qrels/qrels.robust05.txt runs/run.robust05.bm25.topics.robust05.txt

tools/eval/trec_eval.9.0.4/trec_eval -m map -m P.30 src/main/resources/topics-and-qrels/qrels.robust05.txt runs/run.robust05.bm25+rm3.topics.robust05.txt

tools/eval/trec_eval.9.0.4/trec_eval -m map -m P.30 src/main/resources/topics-and-qrels/qrels.robust05.txt runs/run.robust05.bm25+ax.topics.robust05.txt

tools/eval/trec_eval.9.0.4/trec_eval -m map -m P.30 src/main/resources/topics-and-qrels/qrels.robust05.txt runs/run.robust05.ql.topics.robust05.txt

tools/eval/trec_eval.9.0.4/trec_eval -m map -m P.30 src/main/resources/topics-and-qrels/qrels.robust05.txt runs/run.robust05.ql+rm3.topics.robust05.txt

tools/eval/trec_eval.9.0.4/trec_eval -m map -m P.30 src/main/resources/topics-and-qrels/qrels.robust05.txt runs/run.robust05.ql+ax.topics.robust05.txt
```

## Effectiveness

With the above commands, you should be able to reproduce the following results:

| **MAP**                                                                                                      | **BM25**  | **+RM3**  | **+Ax**   | **QL**    | **+RM3**  | **+Ax**   |
|:-------------------------------------------------------------------------------------------------------------|-----------|-----------|-----------|-----------|-----------|-----------|
| [TREC 2005 Robust Track Topics](../src/main/resources/topics-and-qrels/topics.robust05.txt)                  | 0.2032    | 0.2624    | 0.2587    | 0.2028    | 0.2484    | 0.2476    |
| **P30**                                                                                                      | **BM25**  | **+RM3**  | **+Ax**   | **QL**    | **+RM3**  | **+Ax**   |
| [TREC 2005 Robust Track Topics](../src/main/resources/topics-and-qrels/topics.robust05.txt)                  | 0.3693    | 0.4200    | 0.4120    | 0.3653    | 0.4080    | 0.4113    |
