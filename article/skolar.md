---
title: 'Small Reasoning Models for Detecting Funding, Research Infrastructures, and Research Outputs from the full text of scholarly publications'
author:
  - Eric Jeangirard:
      institute: mesr
      orcid: 0000-0002-3767-7125
      idref: 242241344
  - Victor Barbier:
      institute: mesr
  - Pierre-Carl Langlais:
      institute: pleias
      orcid: 0000-0001-9035-1127
      idref: 191099163
  - to be completed: 
      institute: pleias

institute:
  - mesr:
      name: 'French Ministry of Higher Education and Research, Paris, France'
  - pleias:
      name: 'PleIAs, Paris, France'
bibliography: skolar.bib
date: 2026
keywords:
  - open science
  - scholarly knowledge graph
  - information extraction
  - funding acknowledgments
  - research infrastructures
  - software mentions
  - dataset mentions
  - small language models
  - multilingual NLP
  - scholarly communication

geometry: "left=3cm, right=3cm, top=3cm, bottom=3cm"

---

## Abstract

<!-- TO BE WRITTEN LAST -->

**Keywords:** open science, scholarly knowledge graph, information extraction, funding acknowledgments, research infrastructures, software mentions, dataset mentions, small language models, multilingual NLP, scholarly communication

---

## 1. Introduction

Scholarly knowledge graphs have become core infrastructure for research discovery, evaluation, and science-policy monitoring. Open initiatives such as OpenAlex now index hundreds of millions of works together with their authors, venues, and institutions, providing a freely reusable substitute for previously closed or discontinued indices [@priem_openalex_2022]. Yet these graphs are assembled almost entirely from bibliographic *metadata*: titles, abstracts, reference lists, and author affiliations. A large share of the information that actually describes *how* research was funded, resourced, and produced never reaches these structured fields. It remains embedded in the full text of publications , in acknowledgment sections, methods sections, data- and code-availability statements, and footnotes.

Two families of information are especially consequential and especially under-exploited. The first concerns **funding**, understood broadly. Beyond the *direct* funding captured by funder names and grant identifiers, publications routinely record *indirect* forms of support: access to shared research infrastructures (synchrotrons, supercomputers, biobanks, telescopes, observation platforms), or the provision of human and material resources by private companies. The second concerns the **research outputs** that a publication uses or produces , datasets, software, and clinical trials , together with the crucial distinction between *reusing* an existing output, *producing* a new one, and *sharing* it openly.

Making this information explicit matters for several reasons. It supports a more accurate and less biased evaluation of research by revealing funding flows and public–private collaborations that are otherwise invisible. It improves transparency, reproducibility, and trust by tracing which data and software underpin a given result. And it helps avoid research waste by making existing outputs discoverable and reusable. These motivations are not merely academic: national open-science monitoring frameworks increasingly depend on the ability to extract such signals reliably and at scale. France, for instance, has built its Open Science Monitor on open, controlled publication metadata, and uses publications as the basis for analysing adjacent dimensions such as research data, code, software, and clinical trials [@bracco_how_2025]. It already operationalises full-text detection of research data and software , tracking both their production and their openness , through a large-scale machine-learning pipeline that combines PDF parsing with document-engineering and text-mining techniques, applied reliably across disciplines and document types [@bassinet:hal-04121339].

The core difficulty is that the relevant signals appear inconsistently and in diverse linguistic forms across disciplines and languages. The immediate challenge we try to address is therefore the **detection and contextual classification** of these mentions within full scientific texts , distinguishing use from production from sharing , with performance that holds up across disciplines and languages and at a computational cost compatible with processing millions of publications. Existing tools, like the one currently in place in the French Open Science Monitor [@bassinet:hal-04121339] have proven useful but remain limited in recall and precision, are frequently restricted to English and to a narrow disciplinary range, and are often either closed or too slow for large-scale deployment.

Our work is a direct continuation of this national-scale effort. We retain the "parse-then-detect" philosophy of GROBID-based pipelines , operating on the structured text obtained after PDF parsing rather than on raw documents , but replace or augment their classical machine-learning and document-engineering components with fine-tuned language models [@bassinet:hal-04121339; @du_softcite_2021]. The aim is to improve accuracy, and in particular disciplinary and multilingual coverage, while keeping inference cheap enough for large-scale deployment. Concretely, we build hybrid training corpora , combining paragraphs extracted from a broad, multilingual, multidisciplinary publication. The detected entities are computed using a large "frontier" model and then we use them to fine-tune a family of compact *small reasoning models* (600M parameters) specialised in the detection of funding, research infrastructures, private-company involvement, software, datasets, and clinical trials from a text input (typically a paragraph).

Our contributions are:

1. **An open, multilingual, cross-disciplinary problem framing** covering both direct and *indirect* funding (infrastructures, private companies) and the use/production/sharing of software, datasets rather than any single category in isolation.
2. **A hybrid corpus-construction methodology** that combines existing curated resources, extraction with state-of-the-art tools and LLMs, and controlled synthetic generation to overcome the disciplinary and linguistic biases of existing annotated data.
3. **A suite of compact fine-tuned models** that extend an existing GROBID/NLP national pipeline [@bassinet:hal-04121339], optimised for large-scale inference and released on Hugging Face together with their training data and source code.
4. **An evaluation** against established tools that assesses accuracy, inference speed, and robustness across disciplines and languages.

---

## 2. Background and Related Work

### 2.1 Scholarly knowledge graphs and the metadata gap

Open scholarly knowledge graphs such as OpenAlex have made large-scale bibliometric analysis broadly accessible, indexing works, authors, venues, institutions, and concepts at the scale of hundreds of millions of records [@priem_openalex_2022]. Their coverage, however, is largely confined to metadata that is either supplied by publishers or inferred from bibliographic structure. Fine-grained links between a publication and the grants that funded it, the infrastructures it relied on, or the datasets and software it used or produced are only partially represented, when represented at all. Closing this gap requires extracting structured statements from the *body* of publications , precisely the information this work targets , and feeding them back into such graphs.

### 2.2 Funding and acknowledgment information

Funding information is most often disclosed in dedicated acknowledgment sections, which have long been studied as a window onto reward systems and collaboration patterns. Recent work has framed their exploitation as a named-entity recognition (NER) problem: Smirnova and Mayr train Flair-based models to recognise six entity types in acknowledgments , funding agency, grant number, individuals, university, corporation, and miscellaneous , reaching a best overall accuracy of 0.79 [@smirnova_embedding_2024]. Their results are instructive about the shape of the problem. Low-variety entities such as individuals and grant numbers are recognised very reliably (F1 above 0.9), whereas semantically adjacent organisational categories , funding agencies, universities, corporations , remain difficult to disambiguate, and the bottleneck lies more in classification than in extraction. The study also illustrates the data limitations that motivate our approach: training material harvested from a single indexing source, annotation by a single annotator, a bias toward acknowledgments that already contain funding information, and an exclusive focus on English. Enlarging the corpus beyond a medium size brought no further gains, underlining that raw quantity is not sufficient without diversity.

### 2.3 Research infrastructures and indirect funding

A substantial part of research support is *indirect* and therefore even harder to trace. Datasets, software packages, computational models, observational platforms, and computing facilities act as underlying inputs to research, yet the way authors reference them is highly inconsistent [@mayernik_assessing_2017]. Such references surface unpredictably in methods or acknowledgment sections rather than in formal reference lists, their wording diverges widely from any recommended citation, and authors frequently blur the line between citing a resource and citing its provider. As a consequence, citation-based counting systematically underestimates true usage and impact, and the detection tools that have been built are typically custom-made, rarely shared, and not comparable across studies. This motivates treating indirect funding , infrastructure use and private-sector contributions , as a first-class detection target, and doing so with openly released, reusable models rather than bespoke pipelines.

### 2.4 Software mentions

Software contributions remain relatively invisible to reputation systems built on bibliometrics, because software is mentioned in prose far more often than it is formally cited. The Softcite dataset addressed this by manually annotating software mentions across thousands of academic PDFs, providing a gold standard for supervised extraction at scale [@du_softcite_2021]. Tools such as Softcite (associated with GROBID) build on this resource to detect software in full text. Two limitations are relevant here. First, the annotated corpus is concentrated in biomedicine and economics, which constrains cross-disciplinary generalisation. Second, existing detectors emphasise extraction but capture the surrounding *context* , whether software is used, produced, or shared , only partially, and are often computationally heavy for large-scale processing.

### 2.5 Dataset and clinical-trial mentions

Dataset mention detection has followed a similar trajectory. The DMDD corpus provides the largest public resource for the task, with tens of thousands of articles and hundreds of thousands of weakly annotated in-text spans, alongside a smaller manually annotated evaluation set; baselines on DMDD show that detection and linking remain open problems [@pan_dmdd_2023]. On the citation side, the Data Citation Corpus produced by DataCite and Make Data Count aggregates millions of data-citation records linking datasets to the publications that mention them, with metadata reconciled against organisation registries [@datacite_data_2025]. These resources are valuable but carry a strong disciplinary skew toward the life sciences, reflecting their principal data sources, and they capture formal data–publication pairs more than the finer distinction between reuse, production, and sharing. Proprietary services (e.g. Dataseer) offer comparable dataset-detection capabilities but within closed frameworks, and community efforts such as the Make Data Count Kaggle challenge have yielded models that largely reproduce existing life-science-biased citation corpora. Clinical trials, although central to open-science monitoring [@bracco_how_2025], have received comparatively little attention as a full-text detection target in their own right.

Importantly, these dataset- and software-detection tools have already been assembled into a production system: the French Open Science Monitor combines GROBID parsing with mention-detection models to track the production and openness of research data and software at the national scale, across multiple research areas and document types, with source code, data, and training data released under open licences [@bassinet:hal-04121339]. This demonstrates the feasibility of reliable large-scale detection, and provides both the corpus and the baseline against which the present work is developed.

### 2.6 Synthesis: an open gap

Across these strands, a consistent pattern emerges. Useful tools and gold-standard corpora exist for each category in isolation, but they share recurring limitations: a disciplinary bias toward the life sciences (and biomedicine/economics for software), an almost exclusive focus on English, a reliance on custom or proprietary pipelines that are rarely shared or comparable, and inference costs that hinder web-scale deployment. A national-scale pipeline built on GROBID and classical machine-learning techniques has shown that reliable, multi-disciplinary detection of research data and software is achievable in production [@bassinet:hal-04121339] , but it depends on document-engineering and supervised components that are costly to extend to new categories, new languages, and the finer use/production/sharing distinctions. Crucially, no open, LLM-based models are currently available that jointly address direct and indirect funding together with the use, production, and sharing of software, datasets, and clinical trials, across disciplines and languages. Recent progress in compact, reasoning-oriented language models and in synthetic-data generation suggests a path to closing this gap while preserving , and building directly upon , the parse-then-detect architecture already deployed. The remainder of this paper develops such an approach and evaluates it against the established tools reviewed above.

---

## 3. Data and Corpus

<!-- SKELETON ONLY , do not write prose yet -->
*[Describe the source corpus (French Open Science Monitor: PDFs + GROBID parsing, all disciplines, 2013–2025, English and other European languages incl. French), the scientific-paragraphs-categorization dataset, the annotation schemas per category, the manually annotated seed samples, and the synthetic-data generation pipeline (PLEIAS). Reuse Softcite/Datastet outputs and the existing BSO pipeline annotations as silver/reference data.]*

### 3.1 Source corpus and parsing
### 3.2 Annotation schema (funding: direct / indirect / infrastructures / companies; software; datasets; use / production / sharing)
### 3.4 Data generation
### 3.6 Data availability and licensing

---

## 4. Methods

<!-- SKELETON ONLY , do not write prose yet -->

### 4.1 Model architectures and base models
### 4.2 Fine-tuning procedure
### 4.3 Optimisation for large-scale deployment
### 4.4 Evaluation protocol and metrics

---

## 5. Experiments and Results

<!-- SKELETON ONLY , do not write prose yet -->
*[Per-model results; comparison against Softcite / Datastet and the existing BSO pipeline; accuracy, inference speed, cross-disciplinary and cross-lingual robustness; ablations on synthetic vs. real data.]*

### 5.1 Funding / infrastructure / private-company detection
### 5.2 Software, dataset, and clinical-trial detection
### 5.3 Comparison with existing tools
### 5.4 Robustness across disciplines and languages
### 5.5 Inference efficiency at scale

---

## 6. Discussion

<!-- SKELETON ONLY , do not write prose yet -->
*[Implications for open-science monitoring and knowledge graphs; integration with OpenAlex and with the existing national pipeline; limitations (synthetic-data hallucination, disciplinary/linguistic bias, size vs. accuracy, carbon footprint, private-data contamination, maintenance); ethical and reproducibility considerations.]*

---

## 7. Conclusion

<!-- SKELETON ONLY , do not write prose yet -->
*[Summary of contributions and open releases; future work and community reuse.]*

---

## Acknowledgments

*[Funding, infrastructure, and contributor acknowledgments , to be completed.]*

## Data and Code Availability

*[Models on Hugging Face; training datasets on Hugging Face; fine-tuning code on GitHub , links to be added.]*

## References

<!-- Generated from skolar.bib -->
