---
layout: distill
output:
  distill::distill_article:
    toc: true
    toc_depth: 2
title: Predicting when LLMs fail
date: 03-01-2025
date_published: 0
description: "An overview of different paradigms to do so and connections with related areas"

authors:  
  - name: Lorenzo Pacchiardi
    url: "uq_assessors"
    affiliations:
      name: University of Cambridge

bibliography: 2025-UQ_assessor.bib


toc:
  - name: Intro
  - name: Paradigms to predict where LLMs fail
    subsections:
      - name: Intrinsic uncertainty quantification
      - name: Assessors - anticipating LLM performance
      - name: Model-agnostic rejectors
      - name: Bonus - Modifying the LLM Itself
  - name: Tangential directions
    subsections:
      - name: Extracting other information from model internals
      - name: Using LLMs for forecasting
      - name: Reward models
      - name: How humans interpret LLM uncertainty
      - name: Robustness of LLMs to input perturbation
  - name: Further reading

---



# Intro 

Large Language Models (LLMs) are ML systems. As for any ML system, we may want to predict whether they will fail or succeed on specific task instances (e.g., a specific question from a QA dataset).

There are several interrelated paradigms to do so, differing in whether they are specific to a model or not and whether they rely on the model’s outputs or internals. The aim of this post is to frame  how these different approaches relate to one another. Therefore, I overview below three main paradigms, and point towards closely related research fields that tackle different questions.

*Disclaimer:* *I do not claim this overview to cover all existing research strands related to predicting LLM failures or to comprehensively capture all works in each of the considered ones. I welcome any feedback.*

TL;DR: in brief, I identified three paradigms that can predict where LLMs fail. The third one is likely very hard to apply to LLMs in practice. The table below summarises their properties:

- whether additional training is required
- whether the learned approach is model-specific
- whether the input must be passed through the model to predict whether the latter will fail or not
<table border="1">
  <thead>
    <tr>
      <th>CATEGORY</th>
      <th>TRAINING</th>
      <th>MODEL-SPECIFIC</th>
      <th>PASSING INPUT THROUGH THE MODEL</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Intrinsic UQ</td>
      <td>In some cases (eg whitebox methods)</td>
      <td>Yes</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>Assessor</td>
      <td>Yes</td>
      <td>Yes (they can be trained to work for a set of models, based on model features alongside instance features)</td>
      <td>No</td>
    </tr>
    <tr>
      <td>Model-agnostic rejector</td>
      <td>Yes, but once per data distribution</td>
      <td>No</td>
      <td>No</td>
    </tr>
  </tbody>
</table>

# Paradigms to predict where LLMs fail

## Intrinsic uncertainty quantification

Uncertainty quantification (UQ) has been extensively researched in “traditional” deep learning. It involves a suite of techniques aimed at extracting an uncertainty quantification from a trained model (such as reporting the logits of a trained multi-class classifier network) or training those models to embed a form of uncertainty (such as Bayesian Neural Networks, or BNNs). Most methods falling under the term “uncertainty quantification” in the traditional deep learning literature involve producing an output with the model and accompanying it with an uncertainty estimate obtained in some way.

Some of these methods have been adapted or evolved into approaches applicable to LLMs; however, other traditional methods are hardly applicable to them. In fact, with respect to traditional deep learning, LLMs have two main differences that limit the applicability of existing methods and spurred the creation of novel ones:

- first, training a LLM is extremely costly. As such, it is hard to apply uncertainty quantification methods that require bespoke training approaches (such as BNNs).
- second, LLMs do not perform regression or classification, but rather language generation in an auto-regressive manner: to answer a specific question, a LLM recursively generates new tokens to produce a suitable completion. Even if token-level logits (and thus probabilities) are immediately available, it is not trivial to obtain an answer-level uncertainty estimate from them. This therefore spurred the creation of novel approaches specific to LLMs.

Shorinwa et al., 2024, <d-cite key="shorinwa2024survey"></d-cite> provides a nice overview of existing UQ methods for LLMs and their connection with classical ones. Beyond token-level uncertainty quantification, a second class of methods involve having the models verbalise their uncertainty, either after or before an answer has been produced <d-cite key="kadavath2022language"></d-cite><d-cite key="lin2022teaching"></d-cite>. A third category is that of “semantic similarity” approaches; for instance, Kuhn et al., 2023, <d-cite key="kuhn2023semantic"></d-cite> generates different answers, clusters them according to meaning and considers the uncertainty over clusters. Finally, a set of “white-box” works rely on training additional modules that, starting from model activations at a given internal layer, predicts the probability of a produced answer being correct <d-cite key="kadavath2022language"></d-cite><d-cite key="ferrando2024entity"></d-cite>.

As a tangential note, a recent work <d-cite key="pawitan2024confidence"></d-cite> has shown how the model confidence obtained by those different methods is not necessarily consistent: indeed, they find that the token-level probabilities and verbalised confidence are very poorly correlated.

The approaches above differ in terms of level access: from access to activations (white-box), to log-probabilities (token-level approaches), to necessity of multiple generation (as for some semantic similarity approaches), to simple generation for verbalised UQ. Across all of those categories, some works involve finetuning the model to improve its UQ ability <d-cite key="kadavath2022language"></d-cite>.

Nevertheless, all UQ approaches for LLMs require to produce model outputs (or even multiple versions), or at least to pass the input through the LLM to obtain its activations.

## Assessors: anticipating LLM performance

An alternative paradigm which has received less attention is that of “assessors” <d-cite key="hernandez2022training"></d-cite>: additional modules that are trained to predict the correctness of a main model on specific instances **without** relying on that model’s output or activations <d-footnote>The original nomenclature also included assessors  relying on LLM output, which would make the “white-box” unceratinty quantification methods also fall under this. For ease of understanding, I will then consider all assessors to not rely on LLM outputs</d-footnote>. In practice, assessors are classifiers that tackle the problem of predicting the success of the main model starting from a set of features of the input <d-cite key="schellaert2024analysing"></d-cite><d-cite key="zhou2022reject"></d-cite>, such as some embeddings. This makes the assessor model-specific. Alternatively, a single assessor can be trained to predict the success of multiple models, by relying on specific features to identify them <d-cite key="pacchiardi2024instances"></d-cite>, thus predicting success of a model on a specific instance from a pair (instance features, model features).

In both cases, the main point is that the model does not require to see the input at test time for the assessor to make a prediction of its performance. This has two advantages:

- first, it reduces the computational cost of running an input through an LLM if it will predictably fail.
- second, it makes the assessor robust to manipulation: if an LLM receives feedback from an assessor that uses its outputs, the LLM could learn to produce manipulative output, similarly to what happens with RLHF <d-cite key="wen2024mislead"></d-cite>.

The concept of assessors is closely related to that of routers <d-cite key="hu2024routerbench"></d-cite>, which are modules that predict which of a set of LLMs is more likely to succeed at a specific task instance.

## Model-agnostic rejectors

Model-agnostic rejectors operate on the assumption that a model is likely to fail when confronted with a data point significantly different from the training distribution. This concept is closely aligned with anomaly detection. To implement this, a predictor is trained on the training data to identify inputs that diverge notably from that distribution.

I am unaware of any work applying this idea to LLMs; I believe the main reason is that LLMs' training datasets are vast and often unknown, so it is basically impossible to train a rejector using information on that. Even more importantly, as LLMs can tackle a wide range of tasks, it would not make sense to train a rejector on the pre-training dataset.

Therefore, the practical usefulness of this approach may be limited to cases where LLMs undergo fine-tuning for specific use cases in niche domains. Even here, however, it may be better to rely on assessors or uncertainty quantification methods which, by exploiting information on the model, are likely to perform better. In fact, the main advantage of model-agnostic rejectors is that they can be applied to any model trained on a specific dataset; however, it is rare to have multiple LLMs trained (or even finetuned) on the same dataset, due to their large cost.



## Bonus: Modifying the LLM Itself

A recent study <d-cite key="cohen2024idk"></d-cite> introduced an innovative approach by integrating an "I don't know" token into the LLM's vocabulary during the pre-training phase. This addition  improved the model's calibration by enabling it to explicitly express uncertainty.

Such advancements echo techniques in uncertainty quantification (UQ) for traditional deep learning, where models are modified or trained in bespoke ways to better capture uncertainty.

# Tangential directions

## Extracting other information from model internals

There are other works investigating the use of model internals to extract additional insights. For instance, some studies <d-cite key="burger2024truth"></d-cite><d-cite key="azaria2023internal"></d-cite> focus on determining whether a model is engaging in deceptive behavior (commonly referred to as "lying").

This approach bears similarities to extracting uncertainty quantification for LLMs but instead targets different "functions" of a response beyond mere correctness. It could also prove valuable in identifying instances of "sandbagging" <d-cite key="weij2024sandbagging"></d-cite>, where models strategically underperform under specific conditions.

## Using LLMs for forecasting

Rather than focusing on quantifying uncertainty in LLM-generated answers, this approach leverages LLMs to provide uncertainty quantification for external world events. This is conceptually akin to applying intrinsic model uncertainty to simple QA quizzes but extended to inherently uncertain questions. Instead of addressing questions with definitive answers, this method deals with questions characterized by intrinsic unpredictability. Notable efforts in this domain include:

- Dataset: The Autocast dataset <d-cite key="zou2022forecasting"></d-cite> compiles a collection of questions and aggregated human forecasts derived from forecasting tournaments, complemented by a news corpus. In these tournaments, forecasters tackle True/False, multiple-choice, and numerical questions. After the resolution of each question, forecasters are scored using Scoring Rules based on the forecasts they provided at earlier dates. An LLM can simulate forecasting for a specific date by processing news articles up to that date and attempting to answer the questions. The diverse question formats and the extensive numerical ranges make this a challenging benchmark. Consequently, Autocast serves as an outstanding real-world testbed for developing scalable uncertainty quantification methods tailored to LLMs.
- Experiments: Recent studies <d-cite key="schoenegger2024wisdom"></d-cite><d-cite key="halawi2024forecasting"></d-cite> have explored the use of agentic architectures, enabling LLMs to actively engage in forecasting tasks. These experiments push the boundaries of LLM capabilities in addressing uncertainty in dynamic, real-world scenarios.

## Reward models

Reward models can be considered as somehow related to uncertainty quantification: they are typically trained (for instance from human feedback) to evaluate a completion in terms of a validity specification. However, this validity specification usually includes absence of toxic content or alignment with user intentions, rather than correctness of an answer, even if the latter could plausibly be used.
Then, they are used as the reward function to finetune LLMs via RL. However, they can also be independently used to evaluate completions produced by a model; some works also show that they can be used to evaluate partial completions, thus stopping generation and then restarting it if a generation is likely to be unsuccessful wrt the final specification <d-cite key="nath2024goal"></d-cite>. 

However, in contrast to intrinsic uncertainty quantification and assessors, reward models are generally not specific to a single LLM. As such, they rely on generic features of the output that can be used to understand agreement with the considered validity specification. In contrast, assessors and uncertainty quantification approaches are trained considering a specific model or a set of models and explicitly or implicitly rely on their features.

## How humans interpret LLM uncertainty

Human perception and prediction of LLM performance represents another important research direction. Studies have revealed significant limitations in human ability to anticipate LLM behavior. A recent experiment <d-cite key="carlini2024gpt"></d-cite> has shown that humans perform only marginally better than random chance when predicting GPT-4's performance. Moreover, humans tend to exhibit overconfidence in predicting LLM performance on high-stakes tasks, likely due to anchoring on past interactions with these systems <d-cite key="vafa2024large"></d-cite>. A particularly concerning finding is that LLM success on challenging tasks within a domain does not necessarily translate to reliable performance on simpler tasks in the same domain <d-cite key="zhou2024larger"></d-cite>. This counterintuitive behavior challenges our natural assumptions about machine learning systems and highlights the importance of systematic evaluation approaches.

## Robustness of LLMs to input perturbation

The sensitivity of LLMs to minor prompt modifications has been well-documented in the literature <d-cite key="dhole2022augmenter"></d-cite><d-cite key="shen2024jailbreak"></d-cite>. Research has demonstrated that even subtle changes to input prompts can result in substantially different outputs, raising questions about the reliability and consistency of these systems. This sensitivity to input perturbations suggests a potential avenue for uncertainty estimation: by measuring a model's response variability to small input modifications, we could potentially identify cases where the model exhibits high uncertainty and should be rejected. While this approach has been theoretically proposed in recent literature reviews, particularly in Hendrickx et al.'s 2024 review <d-cite key="hendrickx2024reject"></d-cite>, its practical application to LLMs remains unexplored.


# Further reading

- A comprehensive introduction to uncertainty quantification in NLP can be found in a [2022 tutorial](https://sites.google.com/view/uncertainty-nlp) that covers fundamental concepts and key techniques in the field. While slightly dated, it provides valuable foundational knowledge.
- Hendrickx et al.'s 2024 review <d-cite key="hendrickx2024reject"></d-cite> on machine learning with reject option offers broader insights beyond the LLM context. The review introduces a useful taxonomy, distinguishing between novelty rejection (for inputs substantially different from training data) and ambiguity rejection (for cases where training data or learned algorithms show uncertainty). It proposes three architectural principles:
  - Separate rejectors operate independently of any predictor, aligning with the model-agnostic rejector paradigm discussed earlier. Notably, while assessors don't rely on model outputs, they differ from separate rejectors as they incorporate information about predictor performance.
  - Dependent rejectors utilize predictor outputs, often through confidence functions or input perturbation techniques, encompassing many of the UQ approaches outlined above.
  - Integrated rejectors combine rejection capability within the primary model, typically by introducing an additional "reject" class. This category includes both certain UQ methods and approaches involving LLM modification during training.



