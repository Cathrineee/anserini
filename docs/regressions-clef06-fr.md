# Anserini Regressions: CLEF2006 Monolingual French

This page documents BM25 regression experiments for monolingual French document retrieval as part of the [CLEF 2006 Multilingual Document Retrieval (Ad Hoc) Track](http://www.clef-initiative.eu/edition/clef2006).
Associated data can be found on the [CLEF test suites pages](http://www.clef-initiative.eu/dataset/corpus).

The exact configurations for these regressions are stored in [this YAML file](../src/main/resources/regression/clef06-fr.yaml).
Note that this page is automatically generated from [this template](../src/main/resources/docgen/templates/clef06-fr.template) as part of Anserini's regression pipeline, so do not modify this page directly; modify the template instead.

From one of our Waterloo servers (e.g., `orca`), the following command will perform the complete regression, end to end:

```
python src/main/python/run_regression.py --index --verify --search --regression clef06-fr
```

## Indexing

Typical indexing command:

```
target/appassembler/bin/IndexCollection \
  -collection JsonCollection \
  -input /path/to/clef06-fr \
  -index indexes/lucene-index.clef06-fr/ \
  -generator DefaultLuceneDocumentGenerator \
  -threads 16 -storePositions -storeDocvectors -storeRaw -language fr \
  >& logs/log.clef06-fr &
```

The collection comprises news articles from ATS (SDA) and Le Monde totaling 177,452 documents.
Since the original distribution is in a format that's slightly different from standard TREC collections, we used a [preprocessing script](../src/main/python/clir/document_preprocess.py) to convert the collection into Anserini's JSON line format (we also applied a bit of light data cleaning using a script that has been lost; if you have problems reproducing our results, get in touch directly).
The directory `/path/to/clef06-fr/` should point to the location of the processed collection.

For additional details, see explanation of [common indexing options](common-indexing-options.md).

## Retrieval

Topics and qrels are stored in [`src/main/resources/topics-and-qrels/`](../src/main/resources/topics-and-qrels/), from the [CLEF test suites pages](http://www.clef-initiative.eu/dataset/corpus):

+ [`topics.clef06fr.mono.fr.txt`](../src/main/resources/topics-and-qrels/topics.clef06fr.mono.fr.txt): CLEF 2006 ad hoc track topics in French
+ [`qrels.clef06fr.txt`](../src/main/resources/topics-and-qrels/qrels.clef06fr.txt): CLEF 2006 ad hoc track French relevance judgements

After indexing has completed, you should be able to perform retrieval as follows:

```
target/appassembler/bin/SearchCollection \
  -index indexes/lucene-index.clef06-fr/ \
  -topics src/main/resources/topics-and-qrels/topics.clef06fr.mono.fr.txt \
  -topicreader TsvString \
  -output runs/run.clef06-fr.bm25.topics.clef06fr.mono.fr.txt \
  -bm25 -language fr &
```

Evaluation can be performed using `trec_eval`:

```
tools/eval/trec_eval.9.0.4/trec_eval -m map -m P.20 -m ndcg_cut.20 src/main/resources/topics-and-qrels/qrels.clef06fr.txt runs/run.clef06-fr.bm25.topics.clef06fr.mono.fr.txt
```

## Effectiveness

With the above commands, you should be able to reproduce the following results:

| **MAP**                                                                                                      | **BM25**  |
|:-------------------------------------------------------------------------------------------------------------|-----------|
| [CLEF 2006 (Monolingual French)](../src/main/resources/topics-and-qrels/topics.clef06fr.mono.fr.txt)         | 0.3115    |
| **P20**                                                                                                      | **BM25**  |
| [CLEF 2006 (Monolingual French)](../src/main/resources/topics-and-qrels/topics.clef06fr.mono.fr.txt)         | 0.3184    |
| **nDCG@20**                                                                                                  | **BM25**  |
| [CLEF 2006 (Monolingual French)](../src/main/resources/topics-and-qrels/topics.clef06fr.mono.fr.txt)         | 0.4457    |
