# Building an Open Knowledge Graph From The Full Text of Scholarly Publications
MESRE & Pleias

# Summary
We propose to use the full text of scientific publications to extract information about their funding (funders, grant IDs, but also mentions of “indirect” funding through human or material resources, especially from research infrastructure and from private companies) as well as the production or use of other research outputs: datasets, software, and clinical trials.
Our goal is to provide a set of tools that, from a scientific publication (more precisely, from the XML file obtained after parsing the PDF), can identify mentions of direct or indirect funding, software, datasets, and clinical trials.

We therefore propose, first, to build training datasets, mainly from the corpus produced by the French Open Science Monitor, for which we already have PDFs and Grobid parsing. That includes publications in all disciplines, from 2013 to 2025, mainly in English but also in other european languages especially French. These training datasets will be partly based on existing data and partly on synthetic data. Next, we propose to fine-tune Small Reasoning Models (350M–3B parameters) using these training datasets.

The resulting open fine-tuned models will detect mentions of funding, research infrastructure, private companies, software, datasets, and clinical trials from a text input (typically a paragraph). These models will be made available on Hugging Face. We will aim to obtain the smallest possible models to facilitate large-scale deployment. The datasets used to train these models will also be openly shared on Hugging Face (as much as possible, with certain datapoints possibly removed if they originate from closed-access publications with overly restrictive licenses). Finally, the fine-tuning source code will also be available on GitHub.
The resulting models can be deployed at small or large scales. 

We will deploy them at the national (French) level, but we also hope they will be used globally, for instance, by OpenAlex. Moreover, it is certain that other actors will want to improve these models (better accuracy, handling of specific cases, other languages, etc.). The training data can therefore be expanded and reused, as can the fine-tuning code.

# Data available 

- A first dataset is available at https://huggingface.co/datasets/dataesr/scientific-paragraphs-categorization
It contains 833k paragraphs, extracted from cc-by publications. The paragraphs are classified (acknowledgment, dataset, software, clinicaltrial).

- A list of main funders [ from OpenAlex ]
https://github.com/ourresearch/openalex-walden/blob/main/plans/awards/funder-ingestion-tracker.md

## Acknowledgments analysis

- A prompt has been tested woth `mistral-small` cf `prompt_acknowledgment.txt` 

