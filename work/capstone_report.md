# Capstone Report

- **Author:** Meer Musabih
- **Lane:** Machine Learning – Content Refresh Prioritization
- **Repo:** https://github.com/MeerMusabih/Flyrank-AI-Internship
- **Date:** August 15, 2026

> Copy this file to `work/capstone_report.md`.

---

# 1. Problem framing

This project supports the decision of **which content pages should be reviewed first for a possible content refresh**.

The unit of analysis is an individual content page. The model produces a ranked priority score for each page based on observed search-performance and content signals. The output is a ranked review queue rather than an automatic refresh recommendation.

A human content editor reviews the highest-ranked pages, investigates the cause of declining performance, and decides whether a refresh is appropriate.

The cost of a wrong recommendation is that editors may spend time reviewing pages that are not actually good refresh candidates, or miss pages that deserve attention. Because review time is limited, prioritizing the most promising pages is valuable.

Machine learning helps because many page-level signals interact at the same time. A model can combine these observed signals into a stronger prioritization score than a simple manual rule.

---

# 2. Data safety

The analysis uses the FlyRank ML Internship warehouse release `flyrank_pseudonymized_warehouse_release_v20260703`.

The following columns were deliberately excluded from the model because they could leak information about the target or contain outcome-period information:

- trend_direction
- trend_pct
- impressions_last_30d
- clicks_last_30d
- sessions_last_30d
- impressions_prev_30d
- clicks_prev_30d
- sessions_prev_30d

The identifier columns `content_id` and `client_id` were also excluded from the feature set. Client IDs were only used for grouped train/test splitting to prevent pages from the same client appearing in both sets.

The final model used 32 features.

No client names, domains, URLs, search queries, or other identifying information appear in the notebook, exported artifacts, or the `work/` directory.

---

# 3. Baseline

The baseline reproduces the ML-07 action score.

The baseline assigns points based on:

- stale content (days since last update ≥180)
- visible pages (last 30 day impressions ≥500)
- lower average search position (>10)

This baseline was evaluated on the same held-out validation split as the Random Forest model.

| Metric | Baseline | Random Forest |
|--------|---------:|--------------:|
| Precision@20 | 0.40 | 0.70 |
| Precision@50 | 0.36 | 0.68 |

The overall declining-page rate in the dataset is approximately **54.2%**, providing useful context for interpreting the precision values.

The Random Forest consistently ranked declining pages more effectively than the rule-based baseline.

---

# 4. Model / analysis

The project uses a Random Forest classifier as a decision-support model.

The target is defined as:

> Pages whose `trend_direction` equals **down** are treated as the declining class.

The model uses 32 page-level features describing search visibility, engagement, ranking, content characteristics, and page age.

Examples include:

- impressions_90d
- avg_position
- ctr
- pageviews_90d
- scroll_rate
- word_count
- char_count
- content_age_days
- days_with_impressions
- days_with_sessions

The following fields were intentionally excluded:

- client_id
- content_id
- trend_direction
- trend_pct
- outcome-period performance columns
- provider_used
- model_used

The model estimates refresh priority rather than making refresh decisions automatically.

---

# 5. Evaluation

The data was split using **GroupShuffleSplit**, grouped by client ID.

This prevents pages from the same client appearing in both training and testing data.

Training:

- 23,837 pages
- 25 clients

Testing:

- 6,163 pages
- 7 clients

There was no client overlap between the two sets.

The primary evaluation metrics were Precision@20 and Precision@50 because the practical goal is ranking a limited number of pages for review.

Results:

| Metric | Baseline | Random Forest |
|--------|---------:|--------------:|
| Precision@20 | 0.40 | 0.70 |
| Precision@50 | 0.36 | 0.68 |

Some highly-ranked pages belonged to the "up" or "stable" classes. These errors indicate that observed page-level signals are not perfectly separable. The model should therefore be treated as decision support rather than an automatic decision maker.

---

# 6. Interpretation

The most influential features were:

1. impressions_90d
2. avg_position
3. days_with_impressions
4. content_age_days
5. char_count
6. word_count
7. ctr
8. pageviews_90d
9. scroll_rate
10. days_with_sessions

Overall, the model placed the greatest importance on search visibility, ranking performance, content age, and engagement-related signals.

One interesting observation is that content length measures (character count and word count) also contributed to the ranking model, although they were less influential than search visibility metrics.

The feature importance values describe how the Random Forest used observed information for prediction. They should not be interpreted as causal effects.

---

# 7. Recommendation

The final model produces a ranked review queue for content editors.

Editors should begin with the highest-ranked pages and then:

1. Review the page content.
2. Verify the observed decline.
3. Check search intent.
4. Consider technical or seasonal explanations.
5. Decide whether to refresh or continue monitoring.

The generated reason codes provide simple explanations such as:

- Strong search visibility
- Lower average position
- Lower CTR
- Lower engagement
- Older content

Confidence in the ranking is moderate because the model outperformed the baseline on the held-out validation data. However, recommendations should always be reviewed by humans before content changes are made.

The model does not demonstrate causality and does not predict Google's ranking algorithm.

---

# 8. Reproducibility

Repository:

```
git clone https://github.com/MeerMusabih/Flyrank-AI-Internship.git
cd Flyrank-AI-Internship
pip install -r requirements.txt
```

Run the notebook from top to bottom.

Random seed:

```
RANDOM_STATE = 42
```

The notebook exports the following artifacts to `work/outputs/`:

- model_comparison.csv
- ranked_review_queue_top50.csv
- top_feature_importance.csv
- feature_importance.png
- baseline_vs_random_forest.png

The notebook was designed to execute from a fresh clone using the provided dataset and requirements file.

---

## Claims checklist

- Uses measured and observed evidence only.
- Results are directional and intended for decision support.
- No causal claims are made.
- No client-identifying information is included.
- Reported metrics match the notebook outputs.
