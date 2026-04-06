# Designing fraud-prevention systems that keep analysts in the loop

Welcome to the companion repository for the [Nerdearla Chile 2026](https://nerdearla.com/chile/) talk **[Fraud prevention, machine learning, and design patterns: keep your analysts in the loop](https://nerdearla.com/chile/schedule/prevencion-de-fraude-machine-learning-y-patrones-de-diseno-manten-a-tus-analistas-en-el-loop/)**.

This repository is a documentation-first extension of the talk. The slides introduce the main ideas in a short conference format; the README and the technical article expands those ideas with more freedom and extension. The central argument is that fraud prevention is not only a modeling problem. It is an **ML-enabled software-architecture problem** that must be solved with rules, machine learning, operational policy, platform support, and expert analysts working together.

You can explore the material in three complementary formats:

- [Rendered web article](PENDING-LINK)
- [Technical article (PDF)](https://github.com/jospablo777/fraud_ml_design_pattern/blob/main/communication/LaTeX/technical_article.pdf)
- [Slides / presentation (PDF)](https://github.com/jospablo777/fraud_ml_design_pattern/blob/main/Nerdearla-fraud_prevention_ML_and_architecture.pdf)

In practice, robust fraud systems are neither purely manual nor purely automatic. They are hybrid socio-technical systems. Known cases can often be handled with deterministic controls. Repetitive and high-volume traffic benefits from machine-learning based prioritization. Ambiguous cases still require human judgment, contextual reading, and structured feedback. The architecture proposed here treats those capabilities as complementary parts of one system rather than as competing alternatives.

## How to read this repository

Readers who want a quick walkthrough can start with this README and then continue to the technical article. Readers who want the full argument can begin with the slides, use the README as an orientation guide, and then move to the longer article for the complete discussion and bibliography.

A useful reading path is:

1. Read the short conceptual sections below.
2. Open the figures as you move through the text.
3. Continue to `technical_article.md` for the full treatment.


## Core thesis

Fraud prevention is a decision problem under uncertainty, asymmetry, and time pressure. Labels are delayed and incomplete, fraudsters adapt, some interventions must happen in real time, and the cost of acting incorrectly is not symmetric. A false negative can mean direct loss, abuse, or downstream operational damage. A false positive can mean customer friction, support load, and unnecessary interruption of legitimate activity. Under those conditions, no single component is enough.

Rules are precise and auditable, but brittle. Machine learning compresses complex signals and scales prioritization, but it does not carry the whole burden of policy or governance. Human analysts contribute contextual judgment, novelty detection, and operational learning, but their attention is scarce. The argument of this repository is that resilient fraud-prevention systems emerge when those components are designed as one architecture.

![Fig. 1. A conceptual overview of the pattern: effective fraud systems are built from the interaction of rules, machine learning, and expert analysts rather than any one component in isolation.](img/fraud_system_three_pillars.png)

**Fig. 1.** A conceptual overview of the pattern: effective fraud systems are built from the interaction of rules, machine learning, and expert analysts rather than any one component in isolation.

Figure 1 is the shortest summary of the repository. It introduces the basic proposition that the most useful architectural question is not which component to choose, but how to make rules, ML, and analysts work together coherently.

## From isolated models to ML-enabled fraud systems

The systems literature repeatedly shows that a production model never lives alone (Sculley et al., 2015). It sits inside a larger environment of data collection, feature generation, serving, monitoring, operational tooling, and organizational constraints (Lewis, Ozkaya, & Xu, 2021; Kästner, 2025). For fraud operations, that picture is still incomplete unless expert human work is made visible as part of the production system itself.

![Fig. 2. System anatomy for fraud operations: the model lives inside a wider production environment made of data, software, infrastructure, monitoring, and expert human work. Adapted from Sculley et al. (2015)](img/system_anatomy.png).

**Fig. 2.** System anatomy for fraud operations: the model lives inside a wider production environment made of data, software, infrastructure, monitoring, and expert human work. Adapted from Sculley et al. (2015)

Figure 2 extends the familiar hidden-technical-debt picture into a fraud setting. The model is one component inside a broader socio-technical arrangement. Data quality, serving infrastructure, process tools, and analyst workflows all shape the actual behavior of the system seen by the business.

The same logic applies at planning time. An ML-enabled fraud system should not be designed as a disconnected sequence where software is built first, the model is added later, and analyst workflow is improvised at the end. Business requirements, software architecture, ML development, human review, and platform concerns should be designed together as seen in Fig. 3.

![Fig. 3. Planning view of an ML-enabled fraud system: business context, software architecture, traditional software, ML development, human expertise, and platform concerns should be co-designed. Adapted from Lewis, Ozkaya & Xu (2021), Andersen, & Maalej (2024), and Kästner (2025)](img/ml_enabled_system_architecture.png)

**Fig. 3.** Planning view of an ML-enabled fraud system: business context, software architecture, traditional software, ML development, human expertise, and platform concerns should be co-designed. Adapted from Lewis, Ozkaya & Xu (2021), Andersen, & Maalej (2024), and Kästner (2025).

Fraud systems are not only about achieving predictive power; they are also about co-architecting the data-science workflow, the conventional software system, the human workbench, and the operational platform.

## The design principle that holds the pattern together

One of the key design ideas in the talk is that **score and policy should be separated** (Fig. 4). A model produces an estimate. A policy layer turns that estimate into operational action under business constraints, governance requirements, analyst capacity, and risk appetite.

![Fig. 4. Score-to-policy-to-action pipeline: the model produces a risk score, a policy layer translates that score into operational logic, and only then does the system take action.](img/score_policy_action.png)

**Fig. 4.** Score-to-policy-to-action pipeline: the model produces a risk score, a policy layer translates that score into operational logic, and only then does the system take action.

That separation improves controllability and traceability. It also prevents the common mistake of treating the model as if it were the whole decision system. In operational fraud work, business logic often needs to account for analyst saturation, regional restrictions, temporary campaigns, service levels, and guardrails that should not be hard-coded into the model itself.

Once score and policy are separated (Fig. 5), the overall runtime pattern becomes easier to explain.

![Fig. 5. Reference architecture of the ML-enabled HIL Triage pattern: business rules filter explicit cases, ML scores the non-trivial cases, and a decision policy routes them to approval, analyst review, or automatic mitigation.](img/hil_triage_reference_architecture.png)

**Fig. 5.** Reference architecture of the ML-enabled HIL Triage pattern: business rules filter explicit cases, ML scores the non-trivial cases, and a decision policy routes them to approval, analyst review, or automatic mitigation.

Rules handle explicit cases and guardrails. Machine learning compresses weak signals into a score or ranking. Policy maps that score to action. Ambiguous cases are routed to analysts. The architecture therefore combines automation and human judgment without pretending that either one can carry the full system alone.

## Analysts are part of the architecture

The human component is not an exception bucket added at the end of the pipeline. Analysts bring contextual judgment, novelty detection, escalation logic, and structured feedback. Their contribution is operational, not ornamental.

![Fig. 6. Bidirectional collaboration between analysts and ML: models provide prioritization, speed, and focus, while analysts contribute feedback, calibration, contextual judgment, and learning.](img/bidirectional_human_ml_collaboration.png)

**Fig. 6.** Bidirectional collaboration between analysts and ML: models provide prioritization, speed, and focus, while analysts contribute feedback, calibration, contextual judgment, and learning.

The relationship is symmetrical, as illustrated in Fig. 6. The model is useful because it helps analysts focus scarce attention on cases that matter most. Analysts are useful because they improve the system through adjudication, context, and feedback. This is closer to a closed-loop intelligence architecture than to a one-way automation pipeline.

That loop becomes even clearer when feedback is represented explicitly inside the architecture.

![Fig. 7. Closed-loop HIL triage with explicit feedback paths: analyst outcomes feed rule maintenance and model improvement, turning review into a learning mechanism rather than an operational dead end.](img/hil_triage_feedback_architecture.png)

**Fig. 7.** Closed-loop HIL triage with explicit feedback paths: analyst outcomes feed rule maintenance and model improvement, turning review into a learning mechanism rather than an operational dead end.

As seen in Fig. 7, structured feedback matters. A review queue should not end with a manual decision that disappears into narrative notes. Analyst outcomes can improve rules, thresholds, features, labels, and model behavior if they are captured in a disciplined way.

Figure 8 condenses that idea into an operational-learning view. It is simpler than the reference architecture, but explains the core improvement loop to readers who want the shortest possible visual summary.

![Fig. 8. Operative learning in ML-enabled HIL triage: downstream feedback should update both rules and models so that operations become a source of system learning.](img/operative_learning.png)

**Fig. 8.** Operative learning in ML-enabled HIL triage: downstream feedback should update both rules and models so that operations become a source of system learning.

The analyst workbench is where this architecture becomes concrete. Ranking, reason codes, SLA timers, action controls, and structured feedback inputs are not interface decoration. They are part of the system design.

![Fig. 9. Analyst workbench for HIL fraud triage: ranked cases, risk bands, reason codes, SLA timers, and structured feedback turn the architecture into an operational review system. Synthesized from Jalalvand et al. (2024), Ghadermazi et al. (2024), and Alves et al. (2025).](img/analyst_queue_ui_mockup.png)

**Fig. 9.** Analyst workbench for HIL fraud triage: ranked cases, risk bands, reason codes, SLA timers, and structured feedback turn the architecture into an operational review system. Synthesized from Jalalvand et al. (2024), Ghadermazi et al. (2024), and Alves et al. (2025).

This figure makes an important point for practitioners: queue design and analyst interaction design are architecture concerns. The ranking policy, the explanation surface, the way SLA pressure is shown, and the structure of decision logging all affect how the ML-enabled system behaves in practice. The workbench should also respect broader human-AI interaction guidance around explanation, control, override, and feedback capture (Amershi et al., 2019).

## Evaluate the whole system, not only the model

A recurring problem in production ML is that systems are judged with metrics that are too narrow for the actual operation. In fraud-prevention settings, accuracy alone is not enough. Queue quality, analyst load, policy behavior, calibration, and runtime health also matter.

![Fig. 10. Metrics and trade-offs for ML-enabled human-in-the-loop fraud triage: model quality, queue operations, runtime quality, and policy behavior should be evaluated together. Synthesized from Lewis et al. (2021), Jalalvand et al. (2024), Ghadermazi et al. (2024), Alves et al. (2025), Chen et al. (2022), and Tan et al. (2026).](img/metrics_and_tradeoffs_framework.png)

**Fig. 10.** Metrics and trade-offs for ML-enabled human-in-the-loop fraud triage: model quality, queue operations, runtime quality, and policy behavior should be evaluated together. Synthesized from Lewis et al. (2021), Jalalvand et al. (2024), Ghadermazi et al. (2024), Alves et al. (2025), Chen et al. (2022), and Tan et al. (2026).

Figure 10 is meant to correct a common modeling bias. The production objective is to reduce fraud loss while controlling customer friction and analyst overload. That objective spans more than one metric family, and it creates unavoidable trade-offs. A better fraud architecture therefore needs a broader scorecard.

## Connect architecture to MLOps and platform engineering

The pattern also has a lifecycle. Production fraud systems need monitoring, versioning, deployment discipline, data curation, retraining logic, policy revision, and observability. The same signals that help analysts work better should also help the organization improve data, models, and operational logic over time (Fig. 11).

![Fig. 11. MLOps feedback lifecycle for ML-enabled HIL fraud systems: runtime monitoring and analyst feedback support data curation, model improvement, and policy revision. Adapted from Lewis, Ozkaya, & Xu (2021), extended with platform-engineering concepts from Tan, Padmanabhan, & Mallya (2026), and specialized for fraud feedback loops using Kadam (2024).](img/mlops_feedback_lifecycle.png)

**Fig. 11.** MLOps feedback lifecycle for ML-enabled HIL fraud systems: runtime monitoring and analyst feedback support data curation, model improvement, and policy revision. Adapted from Lewis, Ozkaya, & Xu (2021), extended with platform-engineering concepts from Tan, Padmanabhan, & Mallya (2026), and specialized for fraud feedback loops using Kadam (2024).

Here, we connect the runtime architecture back to platform engineering. Fraud systems are not maintained only through model retraining. They also improve through relabeling, feature fixes, threshold changes, routing changes, and governance-aware revision of policy.

## Why describe this as a design pattern

The repository does not present a one-off diagram for one specific implementation. It proposes a reusable architectural response to a recurring problem: how to make fast fraud decisions without losing judgment where it matters. The pattern language view is useful because it makes explicit the **context**, the **forces in tension**, the **solution**, and the **consequences**. We make the design-pattern claim explicit in Fig. 12.

![Fig. 12. Pattern framing for ML-enabled human-in-the-loop fraud triage: context, forces, reusable solution, consequences, and related lineage. Synthesized from Lakshmanan, Robinson, & Munn (2020), Heiland et al. (2023), Cruz et al. (2023), and Järvenpää et al. (2024).](img/pattern_context_forces_consequences.png)

**Fig. 12.** Pattern framing for ML-enabled human-in-the-loop fraud triage: context, forces, reusable solution, consequences, and related lineage. Synthesized from Lakshmanan, Robinson, & Munn (2020), Heiland et al. (2023), Cruz et al. (2023), and Järvenpää et al. (2024).

The proposed architecture responds to recurring tensions: speed versus caution, fraud capture versus customer friction, autonomy versus control, and expertise versus capacity. It offers clear benefits, but it also creates costs in complexity, traceability, calibration, and organizational coordination. Treating those trade-offs openly is one reason the pattern framing is valuable.

## Why this matters for ML-enabled software architecture

The field of ML-enabled software architecture is still consolidating. The literature already documents hidden technical debt, architecture mismatch, monitorability, co-architecting, co-versioning, architecture evaluation, and broader architecting guidance for ML-enabled systems (Sculley et al., 2015; Lewis, Bellomo, & Ozkaya, 2021; Lewis, Ozkaya, & Xu, 2021; Nazir, Bucaioni, & Pelliccione, 2024). Yet many teams still approach production fraud detection either as a pure data-science problem or as a pure workflow problem. This repository argues that the most useful unit of design is the hybrid system.

That perspective is valuable beyond fraud. Any high-stakes setting where predictions must be converted into actions under uncertainty can benefit from the same separation between score and policy, the same concern for structured human feedback, and the same attention to operational queues, platform support, and architecture-level trade-offs.

## Continue with the technical article

The README is only the introduction. The longer discussion lives in [`technical_article.md`](technical_article.md), where the repository expands the background, the pattern framing, the runtime architecture, the human-review workflow, the evaluation logic, the MLOps lifecycle, and the open questions for practitioners and researchers.

## Selected references

The technical article contains the full bibliography. A few central references are listed here to orient the reader.

Amershi, S., Weld, D., Vorvoreanu, M., Fourney, A., Nushi, B., Collisson, P., Suh, J., Iqbal, S. T., Bennett, P. N., Inkpen, K., Teevan, J., Kikin-Gil, R., & Horvitz, E. (2019). *Guidelines for human-AI interaction*. In **Proceedings of the 2019 CHI Conference on Human Factors in Computing Systems**. Association for Computing Machinery. https://doi.org/10.1145/3290605.3300233

Alves, J. V., Leitão, D., Jesus, S., Sampaio, M. O. P., Liébana, J., Saleiro, P., Figueiredo, M. A. T., & Bizarro, P. (2025). *A benchmarking framework and dataset for learning to defer in human-AI decision-making*. **Scientific Data, 12**, 506. https://doi.org/10.1038/s41597-025-04664-y

Andersen, J., & Maalej, W. (2024). *Design Patterns for Machine Learning-Based Systems With Humans in the Loop*. **IEEE Software, 41**(4), 151-159. https://doi.org/10.1109/MS.2023.3340256

Chen, C., Murphy, N. R., Parisa, K., Sculley, D., & Underwood, T. (2022). *Reliable machine learning: Applying SRE principles to ML in production*. O’Reilly Media.

Cruz, P., Ulloa, G., San Martin, D., & Veloz, A. (2023). *Software Architecture Evaluation of a Machine Learning Enabled System: A Case Study*. In **2023 42nd IEEE International Conference of the Chilean Computer Science Society (SCCC)**. IEEE. https://doi.org/10.1109/SCCC59417.2023.10315755

Ghadermazi, J., Shah, A., & Jajodia, S. (2024). *A machine learning and optimization framework for efficient alert management in a cybersecurity operations center*. **Digital Threats: Research and Practice, 5**(2), Article 19. https://doi.org/10.1145/3644393

Heiland, L., Hauser, M., & Bogner, J. (2023). *Design Patterns for AI-based Systems: A multivocal literature review and pattern repository*. In **2023 IEEE/ACM 2nd International Conference on AI Engineering - Software Engineering for AI (CAIN)**. IEEE. https://doi.org/10.1109/CAIN58948.2023.00034

Jalalvand, F., Chhetri, M. B., Nepal, S., & Paris, C. (2024). *Alert prioritisation in security operations centres: A systematic survey on criteria and methods*. **ACM Computing Surveys, 57**(2), Article 42. https://doi.org/10.1145/3695462

Järvenpää, H., Lago, P., Bogner, J., Lewis, G. A., Muccini, H., & Ozkaya, I. (2024). *A synthesis of green architectural tactics for ML-enabled systems*. In **Proceedings of the 46th International Conference on Software Engineering: Software Engineering in Society (ICSE-SEIS '24)** (pp. 130-141). Association for Computing Machinery. https://doi.org/10.1145/3639475.3640111

Kadam, P. (2024). *Enhancing financial fraud detection with human-in-the-loop feedback and feedback propagation*. In **2024 International Conference on Machine Learning and Applications (ICMLA)** (pp. 1198-1203). IEEE. https://doi.org/10.1109/ICMLA61862.2024.00185

Kästner, C. (2025). *Machine learning in production: From models to products*. MIT Press.

Lakshmanan, V., Robinson, S., & Munn, M. (2020). *Machine learning design patterns: Solutions to common challenges in data preparation, model building, and MLOps*. O’Reilly Media.

Lewis, G. A., Bellomo, S., & Ozkaya, I. (2021). *Characterizing and detecting mismatch in machine-learning-enabled systems*. In **2021 IEEE/ACM 1st Workshop on AI Engineering - Software Engineering for AI (WAIN)** (pp. 133-140). IEEE. https://doi.org/10.1109/WAIN52551.2021.00028

Lewis, G. A., Ozkaya, I., & Xu, X. (2021). *Software architecture challenges for ML systems*. In **2021 IEEE International Conference on Software Maintenance and Evolution (ICSME)** (pp. 634-638). IEEE. https://doi.org/10.1109/ICSME52107.2021.00071

Nazir, R., Bucaioni, A., & Pelliccione, P. (2024). *Architecting ML-enabled systems: Challenges, best practices, and design decisions*. **Journal of Systems and Software, 207**, 111860. https://doi.org/10.1016/j.jss.2023.111860

Sculley, D., Holt, G., Golovin, D., Davydov, E., Phillips, T., Ebner, D., Chaudhary, V., Young, M., Crespo, J.-F., & Dennison, D. (2015). *Hidden technical debt in machine learning systems*. In **Advances in Neural Information Processing Systems**, 28.

Tan, B. T. W. H., Padmanabhan, S., & Mallya, V. (2026). *Machine learning platform engineering: Build an internal developer platform for ML and AI systems*. Manning.
