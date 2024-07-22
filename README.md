# RAGChecker: A Fine-grained Framework For Diagnosing RAG

RAGChecker is an advanced automatic evaluation framework designed to assess and diagnose Retrieval-Augmented Generation (RAG) systems. It provides a comprehensive suite of metrics and tools for in-depth analysis of RAG performance.

<p align="center">
  <img src="imgs/ragchecker_metrics.png" alt="RefChecker Metrics" 
  style="width:800px">
  <br>
  <b>Figure</b>: RAGChecker Metrics
</p>

## 🌟 Highlighted Features

- **Holistic Evaluation**: RAGChecker offers `Overall Metrics` for an assessment of the entire RAG pipeline.

- **Diagnostic Metrics**: `Diagnostic Retriever Metrics` for analyzing the retrieval component. `Diagnostic Generator Metrics` for evaluating the generation component. These metrics provide valuable insights for targeted improvements.

- **Fine-grained Evaluation**: Utilizes `claim-level entailment` operations for fine-grained evaluation.

- **Benchmark Dataset**: A comprehensive RAG benchmark dataset with 4k questions covering 10 domains (upcoming).

- **Meta-Evaluation**: A human-annotated preference dataset for evaluating the correlations of RAGChecker's results with human judgments.

RAGChecker empowers developers and researchers to thoroughly evaluate, diagnose, and enhance their RAG systems with precision and depth.


## 🚀 Quick Start

### Setup Environment

```bash
pip install ragchecker
python -m spacy download en_core_web_sm
```


### Run the Checking Pipeline with CLI

Please process your own data with the same format as [examples/checking_inputs.json](./examples/checking_inputs.json). The only required annotation for each query is the `ground truth answer (gt_answer)`.

```json
{
  "results": [
    {
      "query_id": "<query id>", # string
      "query": "<input query>", # string
      "gt_answer": "<ground truth answer>", # string
      "response": "<response generated by the RAG generator>", # string
      "retrieved_context": [ # a list of retrieved chunks by the retriever
        {
          "doc_id": "<doc id>", # string, optional
          "text": "<content of the chunk>" # string
        },
        ...
      ]
    },
    ...
  ]
}
```

If you are using AWS Bedrock version of Llama3 70B for the claim extractor and checker, use the following command to run the checking pipeline, the checking results as well as intermediate results will be saved to `--output_path`:


```bash
ragchecker-cli \
    --input_path=examples/checking_inputs.json \
    --output_path=examples/checking_outputs.json \
    --extractor_name=bedrock/meta.llama3-70b-instruct-v1:0 \
    --checker_name=bedrock/meta.llama3-70b-instruct-v1:0 \
    --batch_size_extractor=64 \
    --batch_size_checker=64 \
    --metrics all_metrics
```

Please refer to [RefChecker's guidance](https://github.com/amazon-science/RefChecker/tree/main?tab=readme-ov-file#choose-models-for-the-extractor-and-checker) for setting up the extractor and checker models.

It will output the values for the metrics like follows:

```json
Results for examples/checking_outputs.json:
{
  "overall_metrics": {
    "precision": 73.3,
    "recall": 62.5,
    "f1": 67.3
  },
  "retriever_metrics": {
    "claim_recall": 61.4,
    "context_precision": 87.5
  },
  "generator_metrics": {
    "context_utilization": 87.5,
    "noise_sensitivity_in_relevant": 22.5,
    "noise_sensitivity_in_irrelevant": 0.0,
    "hallucination": 4.2,
    "self_knowledge": 25.0,
    "faithfulness": 70.8
  }
}
```

### Run the Checking Pipeline with Python
```python
from ragchecker import RAGResults, RAGChecker
from ragchecker.metrics import all_metrics


# initialize ragresults from json/dict
with open("examples/checking_inputs.json") as fp:
    rag_results = RAGResults.from_json(fp.read())

# set-up the evaluator
evaluator = RAGChecker(
    extractor_name="bedrock/meta.llama3-70b-instruct-v1:0",
    checker_name="bedrock/meta.llama3-70b-instruct-v1:0",
    batch_size_extractor=32,
    batch_size_checker=32
)

# evaluate results with selected metrics or certain groups, e.g., retriever_metrics, generator_metrics, all_metrics
evaluator.evaluate(rag_results, all_metrics)
print(rag_results)

"""Output
RAGResults(
  2 RAG results,
  Metrics:
  {
    "overall_metrics": {
      "precision": 76.4,
      "recall": 62.5,
      "f1": 68.3
    },
    "retriever_metrics": {
      "claim_recall": 61.4,
      "context_precision": 87.5
    },
    "generator_metrics": {
      "context_utilization": 87.5,
      "noise_sensitivity_in_relevant": 19.1,
      "noise_sensitivity_in_irrelevant": 0.0,
      "hallucination": 4.5,
      "self_knowledge": 27.3,
      "faithfulness": 68.2
    }
  }
)
"""
```

## Meta-Evaluation

Please refer to [data/meta_evaluation](./data/meta_evaluation/README.md) on meta-evaluation for the effectiveness of RAGChecker.

## Work with LlamaIndex

RAGChecker now integrates with LlamaIndex, providing a powerful evaluation tool for RAG applications built with LlamaIndex. For detailed instructions on how to use RAGChecker with LlamaIndex, please refer to the [LlamaIndex documentation on RAGChecker integration](https://docs.llamaindex.ai/en/latest/examples/evaluation/RAGChecker/). This integration allows LlamaIndex users to leverage RAGChecker's comprehensive metrics to evaluate and improve their RAG systems.

## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This project is licensed under the Apache-2.0 License.

