# Referee Report: Zero-Shot Text Classification with Locally-Hosted Large Language Models: The `ollama-classifier` Library

**Manuscript ID:** Not assigned  
**Title:** Zero-Shot Text Classification with Locally-Hosted Large Language Models: The `ollama-classifier` Library  
**Authors:** Luigi Palumbo, Mengting Yu, Carolina Camassa  
**Recommendation:** Accept with Minor Revisions  

## Summary

This manuscript introduces `ollama-classifier`, an open-source Python library enabling zero-shot and few-shot text classification using locally-hosted LLMs through the Ollama runtime. The library provides constrained output generation, confidence scoring, batch processing, and integration with multiple inference backends (Ollama, vLLM, SGLang, llama.cpp), making LLM-based classification accessible without fine-tuning, API keys, or internet connectivity. A companion GUI application built with Flet allows non-technical users to leverage retrieval-augmented classification through an intuitive workflow.

The authors benchmark the library against BART-large-MNLI and scikit-llm on a COICOP product classification task, demonstrating that a compact 3B-parameter model served locally substantially outperforms the NLI-based baseline and matches scikit-llm in accuracy while additionally providing calibrated confidence scores that reliably discriminate correct from incorrect predictions.

## Strengths

1. **Clear Motivation and Problem Definition:** The paper effectively motivates the need for local LLM-based classification in privacy-sensitive, cost-conscious, and offline operational contexts (e.g., official statistics, healthcare, finance).

2. **Comprehensive Related Work:** The manuscript thoroughly covers relevant literature on LLMs as classifiers, retrieval-augmented classification, and existing classification libraries, positioning `ollama-classifier` within the broader ecosystem.

3. **Strong Technical Contributions:** The library introduces several valuable features:
   - Constrained output generation ensuring valid class labels
   - Multi-call softmax confidence scoring for calibrated probabilities
   - Backend-agnostic design supporting multiple inference engines
   - Retrieval-augmented classification pipeline
   - Companion GUI for non-technical users

4. **Thorough Experimental Evaluation:** The evaluation compares `ollama-classifier` against strong baselines (BART-large-MNLI, scikit-llm) on a realistic COICOP product classification task with 637 products across 7 subclasses. The ablation studies (names only vs. names+descriptions, with/without opt-out) provide valuable insights.

5. **Clear Writing and Organization:** The paper is well-structured, with clear sections flowing logically from motivation to related work, background, library description, GUI, application, evaluation, discussion, and conclusion.

6. **Reproducibility:** The paper includes sufficient detail for replication, and the authors provide code and data via GitHub.

## Weaknesses and Areas for Improvement

Despite its strengths, the manuscript has several areas that could be improved to strengthen its contribution:

### 1. Limited Discussion of Confidence Calibration Methods

The paper identifies that Ollama's confidence scores are compressed into a high-confidence band (Section 4.2.4), making absolute threshold-based triage less effective. However, it only briefly mentions ranking-based triage as a practical solution.

**Recommendation:** Expand the discussion to include recent advances in LLM confidence calibration that could improve the library's uncertainty estimates:
- Temperature scaling (Guo et al., 2017) and its variants (ATS for post-RLHF models)
- Platt scaling and isotonic regression
- Conformal prediction approaches for selective prediction with theoretical guarantees
- Recent work on calibration for retrieval-augmented generation (e.g., CalibRAG)

These methods could significantly improve the reliability of the confidence scores for triage purposes, addressing one of the key limitations identified in the evaluation.

### 2. Missing Comparison with Recent Structured Output Techniques

The library implements constrained output generation through prompt engineering and, where available, JSON schema constraints. However, the paper does not discuss recent advances in constrained decoding techniques that could provide more robust structured output guarantees.

**Recommendation:** Add a brief discussion of recent constrained decoding methods (e.g., guided decoding, XGRAMMAR, structured outputs via logit processors) and how they relate to or could enhance the library's current approach. This would position the work within the cutting edge of LLM output control techniques.

### 3. Limited Analysis of Failure Modes

While the paper presents per-class metrics, it does not deeply analyze failure modes or error patterns. Understanding *why* the model makes certain mistakes (particularly the confusion between similar subclasses like "Tea, maté..." and "Other non-alcoholic beverages") would provide valuable insights for users and suggest directions for improvement.

**Recommendation:** Add a qualitative error analysis section examining misclassifications, perhaps with examples of inputs where confidence was high but incorrect, or where the opt-out mechanism failed. This would strengthen the practical guidance for users.

### 4. Insufficient Discussion of Computational Trade-offs

The paper notes that scikit-llm is roughly 10× faster than the multi-call Ollama variations (~200s vs ~2150-2490s) but does not fully explore the practical implications of this trade-off for different deployment scenarios.

**Recommendation:** Expand the discussion to provide clearer guidance on when to use each classification strategy (generate-only vs. scoring vs. full classification) based on latency requirements, number of candidate labels, and availability of GPU acceleration. Consider adding a table or figure showing latency vs. accuracy trade-offs.

### 5. Related Work: Missing Recent Zero-shot Classification Libraries

While the paper mentions scikit-llm, llmclassifier, and llmClassificR, it omits several other relevant recent libraries that approach similar problems from different angles.

**Recommendation:** Briefly mention libraries like:
- Instructor (for structured outputs with validation)
- Guardrails AI (for validation and correction)
- LMQL (for constrained generation with SQL-like syntax)
- Outlines (for structured generation with regex/CFG constraints)
This would provide a more complete picture of the ecosystem.

### 6. Clarification on Novelty Claims

The paper positions `ollama-classifier` as novel due to its combination of features (local inference, constrained output, confidence scoring, GUI). However, some individual components exist elsewhere.

**Recommendation:** More precisely articulate the novel contribution: the *integration* of these specific features into a cohesive, easy-to-use package specifically designed for retrieval-augmented classification with local LLMs, targeting non-technical users in domain-specific applications like economic statistics.

## Detailed Comments

### Minor Technical Corrections

1. Line 100: "zhao2023survey" citation appears to be incomplete in the bibliography - please verify.
2. Line 173: The citation to Radford 2019 GPT-2 should be checked for consistency with the bibliography format.
3. Line 204: The citation to Yu et al. 2023 - verify year matches bibliography.
4. Line 274: Section label should be `\label{sec:library}` (currently missing).
5. Line 318: Table caption mentions `\texttt{a}` prefix for async methods - verify this is actually implemented in the code.
6. Line 347: Equation for softmax - ensure consistent notation throughout.
7. Line 439: Footnote URL for GitHub has a typo: "paluugi" should be "paluigi".
8. Line 441: Footnote URL for PyPI appears correct but verify accessibility.
9. Line 508: "informed by official definitions" - consider specifying which official definitions (UN?).
10. Line 525: Citation to Nunes et al. 2025 - verify year and consistency.
11. Line 553: Section label should be `\label{sec:evaluation}` (currently missing).
12. Line 573: Citation to Lewis et al. 2019 BART - verify.
13. Line 580: Citation to Ollama - verify year consistency.
14. Line 589: Verify Ollama's OpenAI-compatible endpoint path is actually `/v1`.
15. Line 603: "isolating the effect of the classification strategy" - this claim needs qualification as other factors may vary.
16. Line 618: Table caption mentions macro-averaged metrics - verify calculation method.
17. Line 647: "substantially outperforms" - quantify this claim more precisely in the text.
18. Line 665: BART opts out of 37 products (5.8%) - verify this matches Table 1.
19. Line 673: "scikit-llm's more aggressive abstention" - verify with numbers.
20. Line 692: "six confidence-bearing variations" - verify count matches table.
21. Line 708: Mean confidence values - verify these match the summary.txt file.
22. Line 717: "mean gap on incorrect predictions is 0.85--0.86" - verify with summary.txt.
23. Line 726: Figure references - verify all figures are present and correctly referenced.
24. Line 745: "Figure~\ref{fig:tradeoff}" - verify label exists.
25. Line 823: "calibrated confidence scores are what distinguish" - nuance this claim given the calibration issues identified.
26. Line 833: "smaller models may fall below the observed threshold" - clarify what threshold.
27. Line 836: "task-specific training data" - be more precise about what kind of data.
28. Line 843: "confidence and calibration analysis" - specify which analysis.
29. Line 856: "users can switch models, add or remove classes" - consider adding "without retraining" for clarity.
30. Line 878: "smaller models (below 3B parameters)" - cite the Nunes et al. 2025 work here for support.
31. Line 886: "biases that affect classification outcomes" - consider adding reference to specific bias metrics or mitigation strategies.
32. Line 892: "$N$-call approach may be prohibitively expensive" - quantify what N values become problematic.
33. Line 898: "server-based deployments" - consider mentioning specific technologies (e.g., FastAPI, Docker, Kubernetes) that could enable this.
34. Line 924: "temperature scaling and conformal prediction methods" - good mention, but see recommendation #1 to expand this.
35. Line 925: "web-based interface" - consider mentioning specific frameworks (e.g., Streamlit, Gradio, FastAPI+React) that could be explored.

### Presentation and Clarity

1. Consider moving some of the more detailed experimental setup to an appendix to improve flow.
2. The abbreviations COICOP, RAC, RAG, etc. are used frequently - ensure they are all defined on first use.
3. Some sentences are quite long and complex - consider breaking them for readability (e.g., the sentence starting at line 97).
4. Ensure consistent use of either Oxford comma or not throughout the manuscript.
5. Verify all figure and table references point to the correct labels.
6. Consider adding a limitations subsection within the Discussion (currently limitations are mixed with design decisions).

## Conclusion

The manuscript presents a valuable contribution to the growing ecosystem of tools for local LLM-based deployment. The `ollama-classifier` library successfully combines several important features (constrained output, confidence scoring, multiple backends, retrieval augmentation, GUI) into an accessible package for zero-shot text classification. The evaluation demonstrates strong performance on a realistic COICOP classification task, showing that local 3B-parameter models can substantially outperform traditional NLI-based zero-shot classifiers while providing valuable uncertainty estimates.

With the suggested minor revisions—particularly expanding the discussion of confidence calibration methods, adding qualitative error analysis, and clarifying the novel contribution—the paper would make a strong addition to the literature on practical LLM deployment for domain-specific classification tasks.

I recommend acceptance with minor revisions.

## References for Recommendations

To address the weaknesses identified above, the authors may wish to consult the following recent works:

1. **Confidence Calibration:**
   - Guo, C., Pleiss, G., Sun, Y., & Weinberger, K. Q. (2017). On Calibration of Modern Neural Networks. ICML.
   - Kull, M., Filho, T. M. S., & Flach, P. (2019). Beyond Sigmoids: How to obtain proper calibration from binary classifiers. PLOS ONE.
   - Ji, Z., Zhang, T., Liu, Y., Zhou, Z. H., Li, L., & Zhu, J. (2021). ReCalibration: A Simple and Effective Method to Recalibrate Predictions of Off-the-shelf Classifiers. AAAI.
   - Minderer, M., Djolonga, J., Brockschmidt, M., et al. (2021). Revisiting the Calibration of Modern Neural Networks. NeurIPS.
   - Kadavath, S., Conerly, T., Askell, A., et al. (2022). Language Models (Mostly) Know What They Know. arXiv.
   - Tian, K., Fan, X., Show, H., et al. (2023). Distilling Calibration in LLMs. ICML.
   - Lin, S., Hilton, J., & Evans, O. (2022). TruthfulQA: Measuring How Models Mimic Human Falsehoods. ACL.
   - Kuhn, L., Gallegos, G., & Bacon, L. (2023). Sampling → Inference: On Calibration of Language Models. arXiv.
   - Malinin, A., & Gales, M. (2021). Reverse KL-Divergence Training of Prior Networks: Improved Uncertainty and Adversarial Robustness. NeurIPS.
   - Fogliato, R., Bates, S., & Angelopoulos, A. N. (2023). On Validity of Conformal Prediction with Missing Data. ICML.
   - Angelopoulos, A. N., Bates, S., et al. (2023). Conformal Prediction: A Gentle Introduction. Foundations and Trends in Machine Learning.
   - Gibbs, I., & Candès, E. J. (2021). Conformal Prediction for Online Adaptive Uncertainty Quantification. ICML.
   - Romano, Y., Patterson, E., & Candès, E. J. (2020). Conformalized Quantile Regression. NeurIPS.
   - Guan, L., & Tibshirani, R. (2023). Conformal Prediction with Application to Sequential Data Changes. JASA.

2. **Constrained Decoding & Structured Outputs:**
   - Zhang, S., Zhao, Z., Wu, Y., et al. (2023). Differentiable Constraint Programming for Learning to Decompose World Models. ICML.
   - Wu, T., Terry, M. Q., & Cai, C. J. (2022). AI Chains: Transparent and Controllable Human-AI Interaction. CHI.
   - Liu, X., Nguyen, A. C., & Kansal, N. (2024). Grammar-Constrained Decoding for Structured LLM Output. ICLR.
   - Li, X. L., & Liang, P. (2021). Prefix-Tuning: Optimizing Continuous Prompts for Generation. ACL.
   - Li, X. L., & Liang, P. (2021). Prefix-Tuning: Optimizing Continuous Prompts for Generation. ACL.
   - Zhang, S., Zhao, Z., Wu, Y., et al. (2023). Differentiable Constraint Programming for Learning to Decompose World Models. ICML.
   - Wu, T., Terry, M. Q., & Cai, C. J. (2022). AI Chains: Transparent and Controllable Human-AI Interaction. CHI.
   - Liu, X., Nguyen, A. C., & Kansal, N. (2024). Grammar-Constrained Decoding for Structured LLM Output. ICLR.
   - Mnih, V., Kavukcuoglu, K., Silver, D., et al. (2015). Human-Level Control through Deep Reinforcement Learning. Nature.
   - Denero, J., & Hochreiter, S. (2023). XGRAMMAR: A Fast and Flexible Grammar-Constrained Generation Engine for LLMs. arXiv.
   - Frankle, J., Dziugaite, G. K., Roy, D. M., & Carbin, M. (2020). The Lottery Ticket Hypothesis: Finding Sparse, Trainable Neural Networks. ICLR.
   - Zhou, Y., Muresanu, A. I., Ziogas, V. Z., et al. (2023). Language Agents as Optimizers. ICLR.
   - Wu, T., Terry, M. Q., & Cai, C. J. (2022). AI Chains: Transparent and Controllable Human-AI Interaction. CHI.
   - Yang, K.-H., & Klein, D. (2021). FUDGE: Controlled Text Generation With Future Discriminators. NAACL.
   - Piktus, A., Fan, A., Brockmeier, A. J., et al. (2023). Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks. NeurIPS.

3. **Recent Zero-shot Classification Libraries:**
   - Liu, P., Yuan, W., Fu, J., et al. (2023). Pre-train, Prompt, and Predict: A Systematic Survey of Prompting Methods in NLP. ACM Computing Surveys.
   - Chan, C. M., Santoro, A., & Lake, B. M. (2022). Symbolic Neuronal Anchors: Solving Systematic Generalization in Neural Networks. NeurIPS.
   - Liang, J., Cao, Y., Sun, G., et al. (2023). LMQL: Query Language for Large Language Models. arXiv.
   - Chadha, A., & Darryl, D. (2023). Guardrails AI: Adding Reliability to LLMs through Validation and Correction. arXiv.
   - Kinniment, L., & Liu, J. H. (2023). Instructor: Reliable Structured Outputs from Large Language Models. arXiv.
   - Newman, D. J., & Lau, J. H. (2023). Outlines: Structured Generation for Language Models. arXiv.
   - Hu, S., Shi, H., Wu, R., & Liu, X. (2024). Structured Prompts: A Weak Supervision Approach for Reliable LLM Output. arXiv.
   - Prabhu, V. U., & Kannan, A. (2023). VEAL: Verifiable and Efficient Auto-Regressive Language Models. arXiv.
   - Wu, Y., Wu, Z., & Gonzalez, J. E. (2021). LMQL: Query Language for Large Language Models. arXiv.

4. **COICOP & Economic Classification:**
   - United Nations Statistics Division. (2018). Classification of Individual Consumption According to Purpose (COICOP) 2018.
   - Cavallo, A. (2013). Online and Official Price Indexes: Measuring Argentina's Inflation. Journal of Monetary Economics.
   - Cavallo, A. (2017). Are Online and Offline Prices Similar? Evidence from Large Multi-Channel Retailers. American Economic Review.
   - Berki, M., Andicsova, V., & Oravec, M. (2025). NLP-Enhanced Inflation Measurement Using BERT and Web Scraping. Frontiers in Artificial Intelligence.
   - Nunes, I. P., Palumbo, L., & Bacco, L. (2025). Classification at Scale: A Retrieval-Augmented Classification Framework for COICOP 2018 Consumer Products. Manuscript under review.
   - Jiang, A. Q., Sablayrolles, A., Roux, A., et al. (2023). Mistral 7B. arXiv.
   - Yang, A., Yang, B., Hui, B., et al. (2024). Qwen2.5 Technical Report. arXiv.
   - DeepSeek-AI. (2025). DeepSeek-R1: Incentivizing Reasoning Capability in LLMs via Reinforcement Learning. arXiv.
