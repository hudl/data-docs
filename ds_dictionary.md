# Decision Science Dictionary
This dictionary is meant to be a helpful, quick, and lightweight introduction to the phrases and tools used by Decision Science.

### Phrases
| Phrase | Definition | Examples |
|:-----|:----------:|-----:|
|Data Model| A collection of related tables that are joined together and curated to answer a particular business question.| Subscriptions Model, Trials Model|
|Statistical Model| An algorithm or mathematical operation used on raw data to diagnose relationships between variables and/or make predictions.| Ordinary Least Squares Regression, Logistic Regression, Random Forest|
|Predictive Modeling| The practice of implementing advanced statistical models to make recommendations or answer business questions. | Project Lux|
|Project Lux| A predictive modeling project focused on determining the most important factors in subscription renewal.| |
|Self Service| A state of data maturity in which Hudlies without technical skills can easily query data without the help of a data analyst. | |

### Tools
| Tool | Use | Link |
|:-----|:---:|-----:|
| Amazon Redshift | Amazon Redshift is a cloud-based data warehouse in which Hudl stores most of its data. | [Amazon Redshift website](https://aws.amazon.com/redshift/) |
| Test Environment | DAs use the test environment to validate changes to data models before those changes are deployed to production. |  |
| re:dash | re:dash is an open-source SQL-based business intelligence tool. We use re:dash to query Hudl's database | [re:dash](https://fulla.thorhudl.com/queries/new) |
| Metabase | Metabase is an open-source point-and-click business intelligence tool. Decision Science still maintains our Hudl's Metabase instance, but we hope to sunset it sometime in the near future. | [Hudl Metabase Instance](https://metabase.hudltools.com/) |
| Looker | Looker is a proprietary point-and-click business intelligence tool. In order to achieve Self Service, DS populates Looker with trusted, curated data models. | [Hudl Looker Instance](https://hudl.looker.com/browse) |
| Data Build Tool (dbt) | DS uses dbt to model and transform data that's already in the data warehouse. |  [dbt](https://docs.getdbt.com/) |
