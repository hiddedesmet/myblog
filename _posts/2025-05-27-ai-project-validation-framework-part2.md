---
layout: post
title: "Is AI the right solution? Part 2: Examples and ethical risks"
description: "Part 2 of our AI project validation series. See the decision framework in action with examples and explore key ethical risks like bias, privacy, and workforce impact."
date: 2025-06-02 07:00:00 +0300
author: hidde
image: '/images/ai-ethics.png' # Consider a different image for visual variety, e.g., related to ethics or examples
tags: [AI, IASA, Ethics, Series]
featured: false
toc: true
---

Welcome to Part 2 of our series on validating AI projects! In [Part 1: The decision Framework]({{ site.baseurl }}/ai-project-validation-framework-part1), we introduced a structured decision tree to help assess the viability of AI initiatives. Now, let's explore practical applications of this framework and dive into the crucial ethical considerations that every AI project must address.

## Applying the Framework: Generic examples

Here’s how this decision tree framework can be applied to common types of AI projects:

### Example 1: AI for process optimization (e.g., Manufacturing, Logistics, Back-office)

*   **Strategic alignment:** Does optimizing a specific business process (e.g., reducing production defects, streamlining supply chain logistics, automating data entry) align with strategic goals like cost reduction, improved quality, or operational efficiency?
*   **Pillars evaluation:**
    *   **Objective:** To reduce process cycle time by X%, decrease error rates by Y%, or save Z operational costs.
    *   **Audience/impact:** Affects [Number] internal operators/teams, potentially saving [Number] hours per week or reducing material waste by [Percentage/Quantity].
    *   **Training & data:** Requires historical process data, sensor logs, quality control records, or transaction data. Data collection, cleansing, and labeling might take [Timeframe] and cost [$Amount]. Model training complexity is [Low/Medium/High].
    *   **Operations:** Estimated ongoing operational cost of [$Amount] per month/year for the AI system (cloud resources, monitoring, retraining).
*   **Business impact:** Primarily cost reduction or efficiency improvement. Could also lead to improved quality or compliance.
*   **Impact quantification:** Estimated annual savings of [$Amount] due to reduced labor, fewer errors, less material waste, or faster throughput.
*   **Feasibility & effort:** Assessed as [Low/Medium/High] effort based on data complexity, model requirements, integration with existing systems, and change management needs.
*   **ROI assessment:** If high impact (significant savings/efficiency gains) and manageable effort, it could be a "Quick win" or "Strategic bet."

### Example 2: AI for enhanced customer experience (e.g., Personalization, support chatbots)

*   **Strategic alignment:** Does improving customer personalization, support responsiveness, or self-service capabilities align with strategic goals like increasing customer satisfaction, retention, or lifetime value?
*   **Pillars evaluation:**
    *   **Objective:** To increase customer satisfaction scores (CSAT/NPS) by X points, reduce customer churn by Y%, or increase conversion rates by Z%.
    *   **Audience/impact:** Affects [Number/Segment] of customers. Potential to improve engagement for [Percentage]% of the user base.
    *   **Training & data:** Requires customer interaction data (website clicks, purchase history, support transcripts), CRM data, and customer feedback. Data privacy and governance are key. Training might take [Timeframe] and cost [$Amount].
    *   **Operations:** Estimated ongoing operational cost of [$Amount] per month/year.
*   **Business impact:** Primarily revenue increase (through retention, upselling, new customer acquisition) or improved customer satisfaction and loyalty.
*   **Impact quantification:** Estimated annual revenue increase of [$Amount] from improved metrics, or the financial value of reduced churn / increased customer lifetime value.
*   **Feasibility & effort:** Assessed as [Low/Medium/High] effort, considering data integration, model sophistication, UI/UX development, and ethical AI considerations.
*   **ROI assessment:** If high impact (significant revenue uplift or satisfaction boost) and the effort is proportionate, it could be a "Strategic bet." Ensure ethical implications are thoroughly reviewed.

## Ethical considerations and risks

Beyond the financial and operational aspects, AI projects carry significant ethical responsibilities and potential risks that must be proactively addressed. This section will focus on three key areas:

1.  **Identifying common ethical implications**: This includes understanding issues like bias, fairness, and the need for transparency in AI systems.
2.  **Ensuring equitable and just access and outcomes**: This involves considering how AI impacts different groups and striving for fairness in its application.
3.  **Accounting for environmental impact**: Recognizing that AI systems have non-trivial environmental footprints that need to be considered.

Neglecting these areas can lead to reputational damage, legal issues, and, most importantly, harm to individuals or groups.

### 1. Identifying common ethical implications: bias, fairness, and transparency

AI systems learn from data, and if that data reflects existing societal biases, the AI can perpetuate and even amplify them. This is a critical consideration in any AI project.

*   **Automation and bias:**
    1.  AI systems are designed, built, and trained by humans.
    2.  Humans inherently possess biases and subjective points of view, often unconsciously.
    3.  Automation through AI can accelerate these biases at scale, leading to unfair or discriminatory outcomes, even when developers have the best intentions.

    For example, if an AI model is trained to generate images of historical figures and is predominantly shown images of one demographic for a particular role, it might exclusively produce results reflecting that bias. Consider an AI asked to depict the "Founding Fathers of America." If the training data lacks diversity, the AI might only generate images of white men, inadvertently erasing the contributions and existence of other individuals who were part of that historical context but are underrepresented in common datasets.

    ![AI-generated image of Founding Fathers showing bias](/images/founding%20fathers%202.png)
    *Example of potential bias in AI-generated imagery if not carefully managed.*

    ![AI-generated image of Founding Fathers more accurate](/images/founding%20fathers.png)
    *Striving for more inclusive and accurate AI outputs requires diverse data and conscious design.*

*   **Privacy considerations:** AI systems often require vast amounts of data for training and validation, raising significant privacy concerns.
    *   **Data de-identification:** Can we truly ensure that all data used is adequately de-identified to protect individuals?
    *   **Production data for retraining:** What are the implications of using inputs and outputs from production environments to further train and iterate on AI models? How is consent managed for this ongoing use?
    *   **Biometrics and facial recognition:** The ease with which AI can process biometrics and perform facial recognition necessitates stringent safeguards and clear policies to prevent misuse.
    *   **Data repurposing:** When data collected for one specific purpose is stored and later reused for AI training or other applications without explicit, informed consent for these new uses, it erodes trust and can violate privacy rights.
    *   **Data longevity:** How long should data be stored, especially sensitive data? What happens when data is stored longer than an individual is alive? Are there clear data disposal policies?
    *   **"Click-through" consent:** Does a user genuinely provide informed consent for their data to be used in AI training if they simply "click through" a generic "I agree" checkbox, often without fully understanding the implications? The validity and ethics of such consent mechanisms are highly debatable.

*   **Automation and workforce impact:** The drive to automate tasks using AI has profound implications for the workforce.
    *   **Cost of replacement vs. augmentation:** While AI can automate, the cost to develop, fit, and run models that *completely* replace a human worker can be substantial. Often, AI is better suited to augment human capabilities.
    *   **The new essential skills:** It's in every worker's best interest to develop AI-related skills, much like email and word processing skills became standard requirements in the past.
    *   **Upskilling initiatives:** Recognizing this shift, some governments are taking proactive steps. For example, Singapore is investing in paying its citizens to upskill them in AI, aiming for a middle ground between dystopian job displacement and universal basic income.
    *   **Automating repetitive tasks:** AI systems excel at automating repetitive tasks with a high degree of accuracy. This is beneficial for efficiency but directly impacts roles primarily focused on such tasks.
    *   **Worker displacement:** Consequently, this can lead to worker displacement. A notable example is Duolingo, which reportedly laid off 10% of its contractor workforce, citing a greater reliance on AI for content creation and translation. This highlights the real-world impact on employment.

*   **Transparency and explainability:** Understanding how AI systems arrive at their decisions is crucial for trust and accountability.
    *   **Lack of incentive for disclosure:** Many private companies are not inherently incentivized to explain the inner workings of their proprietary algorithms. This "black box" nature can make it difficult to assess fairness, identify biases, or understand why a particular decision was made.
    *   **Outliers in openness:** Some companies are moving towards greater transparency. For instance, X (formerly Twitter) open-sourced its feed algorithm, which utilizes machine learning. Similarly, GitHub has announced plans to open-source parts of VS Code and Copilot's AI components, as detailed in their blog post ([referencing the VS Code blog on open-sourcing AI in the editor](https://code.visualstudio.com/blogs/2025/05/19/openSourceAIEditor)). These initiatives, however, are currently more the exception than the rule. 
    *   **Public distrust:** A lack of transparency can breed significant public distrust. The concerns surrounding TikTok's machine learning algorithm in the US serve as a prominent example.
    *   **Impact of opaque algorithms:** The societal impact of non-transparent algorithms can be severe. For example, there are studies and reports suggesting that algorithms like Instagram's can negatively affect mental health, potentially tripling depression rates in teenage girls, by curating content in ways that are not clear or controllable by the user.
    *   **Regulatory moves (EU AI Act):** Recognizing these challenges, regulations like the EU AI Act are emerging. This act will mandate a degree of transparency for AI systems classified as "high-risk." For such systems, users (and regulators) must be provided with clear instructions on the system's capabilities, limitations, and potential risks.
    *   **Scope of regulation:** It's important to note, however, that most AI applications will likely not fall under the "high-risk" category as defined by the EU AI Act. The majority will be considered "low-risk" or "minimal risk," and thus, the regulatory requirements will be less stringent. However, adhering to ethical guidelines and ensuring transparency will remain best practices regardless of regulatory classification.

    ![EU ACT](/images/eu-act.png)
    *The EU AI Act aims to regulate high-risk AI systems, emphasizing transparency and user awareness.*

---
*Having explored practical examples and critical ethical risks, join us for [Part 3: Metrics, Piloting, and Key Takeaways]({{ site.baseurl }}/ai-project-validation-framework-part3) where we'll discuss defining success and the importance of iterative pilot projects. Available on Monday, June 9, 2025!*
