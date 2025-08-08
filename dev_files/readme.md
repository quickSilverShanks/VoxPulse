# VoxPulse Development Notes

## Data Definition

Original source: [Amazon US Customer Reviews Dataset](https://www.kaggle.com/datasets/cynthiarempel/amazon-us-customer-reviews-dataset)

| Feature Name       | Description                                                                        |
|--------------------|------------------------------------------------------------------------------------|
| marketplace        | two letter country code of the marketplace where the review was written              |
| customer_id        | random identifier that can be used to aggregate reviews written by a single author |
| review_id          | the unique ID of the review                                                        |
| product_id         | the unique product ID the review pertains to; used to group multilingual reviews   |
| product_parent     | random identifier that can be used to aggregate reviews for the same product       |
| product_title      | title of the product                                                               |
| product_category   | broad product category used to group reviews and organize the dataset              |
| star_rating        | the 1–5 star rating of the review                                                  |
| helpful_votes      | number of helpful votes                                                            |
| total_votes        | number of total votes the review received                                          |
| vine               | review was written as part of the Vine program                                     |
| verified_purchase  | the review is on a verified purchase                                               |
| review_headline    | the title of the review                                                            |
| review_body        | the review text                                                                    |
| review_date        | the date the review was written                                                    |



## Data Preparation

Key observations from data:
- The original source has 37 files pertaining to reviews for different product categories and is 54.41 GB in total.
- Removing rows with no votes and skipping bad lines in raw data leaves us with ~42% of the rows that can be used.
- In usable data, there are product categories with as low as 7000 reviews; a total of 6 categories have less than 200,000 reviews; 18 categories have more than 1 mil. reviews. Based on this alone, 200,000 seems like a good value for confidence weight, `m`.

To generate the train-val-test an other required splits datasets below has been taken care of:
- Considering the size of data, it should first be explored in Kaggle itself, downloading only necessary data of much smaller size (10.7 GB zipped).
- Reviews with no user vote is not worth considering.
- The model must be able to predict helpfulness of products it hasn't seen in training. So, 4(2+2) product categories with good range of usefulness in the data must be kept separately for validation(2) and test(2) purposes only. This will allow us to have in-sample and out-of-sample validation/test scores.
- Keep only necessary columns: "product_category", "product_title", "product_parent", "review_headline", "review_body", "review_date", "star_rating", "total_votes", "helpful_votes", "verified_purchase", "vine".
- **[IMPORTANT]** Only 6 categories have less than 200,000 reviews and one has only 7000 reviews. From these 6 categories, a fixed percent of data must be kept aside for validation.
- To do a mockup of new inbound data in MLOps pipeline, separate certain percentage of data from 5 categories with >1mil. rows and 4 categories with 0.2-1mil. rows. This can be used to test multiple iterations of MLOps functionality from data ingestion to new champion model deployment.

