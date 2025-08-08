# VoxPulse

<img alt="banner" src="https://github.com/user-attachments/assets/ede275e4-1af0-4f47-92cc-2e0d66ae9dc6">



## What is Voxpulse?

"**Vox**" (Latin for voice) + "**Pulse**" (capturing the heartbeat of customer opinions)

VoxPulse is an AI-powered review-ranking engine that listens to the voice of the customer and identifies the most helpful, balanced, and informative opinions.



## 🎯 What it does?

Using intelligent scoring that factors in clarity, specificity, sentiment/tone balance, credibility, and engagement cues (e.g., upvotes or verified status if available), VoxPulse brings the most valuable feedback to the forefront. This empowers users to make smarter decisions and enables businesses to quickly spot high-signal feedback for product improvements.

🔄 Potential Downstream Applications
- **Review Summarization**: Generate concise summaries using only the most insightful and trustworthy reviews.
- **Customer Support Insights**: Surface recurring complaints or suggestions from high-quality feedback.
- **Product Benchmarking**: Compare usefulness trends across competing products or categories.
- **E-Commerce Ranking Signals**: Integrate usefulness scores into search/sort algorithms on product pages.
- **User Trust Systems**: Prioritize showing reviews that are likely to influence purchasing decisions.
- **Marketing Feedback Loops**: Identify language and themes in helpful reviews to inform messaging.
- **Quality Control**: Flag low-quality or misleading reviews for moderation or de-prioritization.
- **Feature Roadmap Planning**: Detect high-value suggestions from helpful review patterns.
- **Voice-of-Customer Dashboards**: Feed structured, trustworthy review insights into analytics tools.



## 🧮 Quantifying a Reliable Usefulness Score

### The Dilemma
When predicting the usefulness of a review, a common metric is the ratio of helpful votes to total votes:
> usefulness_ratio = helpful_votes / total_votes

However, this ratio alone can be misleading. For example:
- A review with 1 helpful vote out of 1 total vote yields a 100% ratio
- Another with 10 helpful votes out of 11 yields ~91%

Despite the lower ratio, the second review is likely more genuinely helpful, as it's backed by more community feedback. This presents a **credibility vs. ratio dilemma**:
> How do we reward both high ratios and high engagement?

### The Solution: Bayesian Adjusted Usefulness
To address this, we use a **Bayesian smoothing technique** — <u>commonly applied in rating systems like IMDB</u> — to combine the observed ratio with a prior belief, weighted by vote count.

Formula:
```python
adjusted_usefulness = (helpful_votes + m * prior) / (total_votes + m)
```
- `helpful_votes`: Number of helpful votes
- `total_votes`: Total votes received
- `prior`: Average usefulness across all reviews
- `m`: Confidence weight (number of votes needed before we "trust" the ratio)

**Example**:

Let’s assume:
- `prior = 0.6` (average helpfulness in dataset)
- `m = 5`

Then:
- Review A (1/1):
<br>  `adjusted = (1 + 5*0.6) / (1 + 5) = 4 / 6 ≈ 0.667`
- Review B (10/11):
<br>  `adjusted = (10 + 5*0.6) / (11 + 5) = 13 / 16 ≈ 0.812`

This method rightly gives more weight to Review B due to stronger support.

### ✅ Benefits
The above methodology helps us create a more reliable target for training regression or ranking models.
- Smooths out inflated scores from low-vote reviews
- Reflects confidence in the usefulness estimate
- Prevents usefulness score from swinging a lot on addition of every new vote



## Modeling Methodologies - Predicting Usefulness Score

The usefulness score defined above ranges between 0 and 1, reflecting the adjusted helpfulness of a review. Below are four possible modeling approaches we have experimented with to predict this score, each with its own working principle, benefits, and trade-offs.

| **Methodology** | **How it Works** | **Pros** | **Cons** |
| --------------- | ---------------- | -------- | -------- |
| **1. Regression with Clipping** <br> *(optionally with custom loss)* | Train a standard regression model to predict raw scores; clip outputs to \[0, 1] range. Optionally, define a custom loss to penalize errors more heavily if predictions are out-of-bounds. | - Simple to implement<br>- Works with any regression algorithm<br>- Can adapt to custom evaluation priorities | - Ignores bounded nature during training<br>- Predictions before clipping may be unrealistic |
| **2. Regression with logit(y)**                                      | Transform target `y` using logit: `log(y / (1 - y))`; train a regression model on transformed data; apply inverse logit at inference.                                                        | - Respects \[0, 1] bounds after transformation<br>- Handles skewed target distributions better                | - Fails if y = 0 or y = 1 exactly (requires smoothing)<br>- Transformation adds complexity   |
| **3. Beta Regression**                                               | Model target as a Beta-distributed variable parameterized by mean and precision; directly predicts bounded continuous values.                                                                | - Naturally handles (0, 1) range<br>- Good for skewed or U-shaped distributions<br>- Statistically grounded   | - More complex to implement<br>- Cannot handle exact 0 or 1 without adjustments              |
| **4. Classification with Buckets**                                   | Discretize the \[0, 1] range into 2+ buckets (zones); train a multiclass classifier to predict the correct zone.                                                                             | - Turns regression into simpler classification<br>- Can focus on critical ranges (e.g., high usefulness)      | - Loses granularity within each bucket<br>- Requires careful choice of bucket boundaries     |

The custom loss function proposed in method 1 can be calculate as follows:

$$
L_{\text{reg}} = (y_{\text{pred}} - y_{\text{true}})^2
$$

$$
L_{\text{penalty}} = \lambda \cdot \max(0, y_{\text{pred}} - 1)^2 + \lambda \cdot \max(0, 0 - y_{\text{pred}})^2
$$



## 🧠 Workflow

<details>
  <summary><b>TODO List</b></summary>

- [ ] Planning: Data | Target | Modeling Methodology
- [ ] Notebook: Data Profiling(EDA) | Preliminary Analysis
- [ ] Notebook: Data Preprocessing | Baseline Predictions (Optuna)
- [ ] FastAPI: API endpoint
- [ ] MLFlow: Experiment Tracking and Model Registry
- [ ] Prefect/Airflow: Workflow Orchestration
- [ ] Docker: Containerize
- [ ] Grafana/Evidently AI: Model Monitoring
- [ ] Add reproducibility instructions | Documentation(readme)
- [ ] Model Deployment with localstack and docker
- [ ] Add unit and integration tests
- [ ] Use linters/formatters
- [ ] Add makefile
- [ ] Use pre-commit hooks
- [ ] CI/CD pipeline
- [ ] Webscraping for new reviews
- [ ] Add Product Documentation and User Guide

</details>



## Technologies

- [Python](https://www.python.org) - this project has been developed using python and bash scripts
- [Jupyter Notebook](https://jupyter.org/) - to perform experiments and evaluation for initial data profiling and baseline predictions
- [Docker](https://www.docker.com/)  - to containerize application and required services
- [Streamlit](https://streamlit.io/) - to create a application UI for end-users interaction
- [Prefect](https://www.prefect.io/) - to perform workflow orchestration for training and monitoring flows
- [MLFlow](https://mlflow.org/) - for experiment tracking and model registry
- [Grafana](https://grafana.com/) – for monitoring model performance and data drift
- [PostgreSQL](https://www.postgresql.org/) - to store model data, application usage data and logs
- [Adminer](https://www.adminer.org/) - database management tool to view records in tables pertaining to user interactions



## Resources

### Datasets Used
- https://www.kaggle.com/datasets/cynthiarempel/amazon-us-customer-reviews-dataset
- https://www.kaggle.com/datasets/mohamedbakhet/amazon-books-reviews
- https://www.kaggle.com/datasets/anushabellam/amazon-reviews-dataset

### Useful Links
- https://github.com/DataTalksClub/mlops-zoomcamp
- https://github.com/yokalyan/mlops-churn
- https://github.com/AnnaGanisheva/churn-prediction
- https://www.reddit.com/r/explainlikeimfive/comments/ptvrh/eli5_imdbs_top_250_scoring_system_aka_a_true/



<br><br><br><br>
<hr>



<div align="center">
If you like this project, please consider giving it a ⭐️ <b>star</b> to help others discover it. 

[![GitHub Stars](https://img.shields.io/github/stars/quickSilverShanks/VoxPulse.svg?style=social)](https://github.com/quickSilverShanks/VoxPulse/stargazers)
[![GitHub Forks](https://img.shields.io/github/forks/quickSilverShanks/VoxPulse.svg?style=social)](https://github.com/quickSilverShanks/VoxPulse/network/members)
[![GitHub Issues](https://img.shields.io/github/issues/quickSilverShanks/VoxPulse.svg)](https://github.com/quickSilverShanks/VoxPulse/issues)
</div>
<hr>

<div align="center">
Please note this is not open for contributions yet as basic features are still being added in, but feel free to share improvement suggestions or 🍴 <b>fork</b> it and explore the code!
</div>
