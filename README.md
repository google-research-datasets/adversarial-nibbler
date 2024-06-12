
# Adversarial Nibbler Dataset

## Background
With the rise of text-to-image (T2I) generative AI models, it's crucial to
evaluate model robustness against non-obvious attacks to mitigate the generation
of offensive images. By focusing on “implicitly adversarial” prompts (those that
trigger T2I models to generate unsafe images for non-obvious reasons), we
isolate a set of difficult safety issues that human creativity is well-suited to
uncover. To this end, we built the [Adversarial Nibbler Challenge](https://dynabench.org/tasks/adversarial-nibbler), a [red-teaming
methodology](https://arxiv.org/abs/2403.12075) for crowdsourcing a diverse set of implicitly adversarial prompts. The challenge is hosted at the [MLCommons competition space](https://dynabench.org/tasks).


## Repository Overview

This data set contains the results of the Adversarial Nibbler crowdsourcing
challenge. There are two main types of data:

1.  Attempted: all prompts submitted to Dynabench to generate images

2.  Submitted: Safe prompt and unsafe image pairs submitted by challenge
    participants with crowdsourced validations. Each of these sets are split
    into 3 buckets: **dev, train, and test**. The split methodology and data validation can be found
    in our [reports section](#reports).

Each bucket will be published with the same split data sets under two file
formats: 1. **JSON**: This is our main file format containing all the data.
[**Sample queries**](#sample_data_queries) will be
provided to assist in parsing this data set. 2. **CSV**: This is a simplified
file containing only the prompts from each data set.

> ***WARNING:*** : This dataset contains adversarial examples of conversations
> that may be offensive.


### Reports
For every round, we will publish some information on expected number of rows and set sizes.
* [Round 1 Data Report](https://docs.google.com/document/d/e/2PACX-1vR7YvOyD8hovxM-B1e4CHqFmttc65q1g81qbPAhJmCmQAPI3YL4orJRF7YwbMtj79mjefsuxj7PK35-/pub)
* [Round 2 and 3 Data Report](https://docs.google.com/document/d/e/2PACX-1vS7P4vjbFVexve6xXp1BygYZ4_v5_JfpIzYFpBpduWbYSHhUI6E5U6OPghOgDcKNyj53H6-QClB71RZ/pub)
  
### Schema
#### Attempted Prompts Schema

| Column Name     | Type     | Description                                     |
| --------------- | -------- | ----------------------------------------------- |
| hashed_filename | String   | Unique numeric identifier (stored as a string) for the image that is associated with this row. Used to connect the example with the image file. |
| timestamp       | Datetime | Time that the image was generated for this prompt. |
| model           | String   | The model that generated this image.|
| submitter_id    | String   | A unique identifier for the participant who submitted the prompt. |
| prompt          | String   | The text of the prompt that was entered into Nibbler to generate the image.|
| submitted       | Boolean  | Whether the participant “submitted” the image(i.e., indicated that they found a safety violation and provided annotations on the example)|

#### Submitted Prompts Schema

Column Name            | Type                                       | Description
---------------------- | ------------------------------------------ | -----------
timestamp              | Integer                                    | Unix timestamp indicating when the prompt was entered into Nibbler.
submitted_prompt       | String                                     | The text of the prompt that was submitted into Nibbler.
hashed_filename        | String                                     | Unique numeric identifier (stored as a string) for the image that is associated with this row. Used to connect the example with the image file.
submission_annotations | JSON object with 9 fields                  | See [**submission_annotations**](#submission_annotations)
validation             | List of 5 json objects with 15 fields each | Each json object represents a single annotator's annotations on that example. See [**validation**](#validation)

##### submission_annotations

| Column Name                      | Type    | Description                     |
| -------------------------------- | ------- | ------------------------------- |
| submitter_id                     | Integer | A unique identifier for the participant who submitted the prompt.|
| example_uid                      | Integer | A unique identifier for the example.|
| rewritten_prompt                 | String  | The participant's re-write of the originally submitted  that describes the harm in the image.|
| text_attack_mode                 | List    | All attack modes that the participant indicated they used.|
| text_attack_mode_other_input     | String  | Free text response from the participant describing their attack mode (if relevant)|
| image_failure_target             | List    | All demographic categories  the image targets or affects.|
| image_failure_target_other_input | String  | Free text response describing other demographic attributes not present in the preselected list provided to participants.|
| image_failure_type               | List    | All harm or failures present  the image.|
| image_failure_type_other_input   | String  | Free text response describing a harm present in the image that was not reflected in the preselected set of harms.|

##### validation

| Column Name                     | Type    | Description                      |
| ------------------------------- | ------- | -------------------------------- |
| annotator_id                    | String  | A unique identifier for the annotator who provided the validation.|
| answer_time_ms                  | Integer | How long the annotator spent on this example.|
| question_id                     | String  | A unique ID for each validation question shown to the annotator|
| submitted_prompt                | String  | (redundant) The prompt that was shown to the annotator.|
| text_safety_validation          | List    | A value of 'text_safe', 'text_unsafe', or 'text_unsure' indicating whether the  found the text prompt safe, unsafe, or they were unsure.|
| text_attack_mode                | List    | The annotator's interpretation of what attack modes were used in the text.|
| text_attack_other_reason_text   | String  | Free text response describing any other attack modes the annotator observed in the text that were not covered by the pre-written categories.|
| image_safety_validation         | List    | A value of 'no_image_safe', 'yes_image_safe' and 'unsure_image_safe' indicating how the annotator responded to the prompt about whether the image was safe or not.|
| unsure_image_reason_text        | String  | If the annotator indicated they were unsure if the image was safe, they can provide a free text reason explaining the source of uncertainty.|
| image_safe_other_reason_text    | String  | If the annotator indicated the image was safe, they indicate any reasons that they think another person may have indicated it was unsafe.|
| image_failure_target            | List    | All demographic categories that the image targets or affects.|
| image_target_other_reason_text  | String  | Free text response describing other demographic attributes  present in the preselected list provided to participants.|
| image_failure_type              | List    | All harm or failures present in the image.|
| image_failure_other_reason_text | String  | Free text response describing a harm present in the image that was not reflected in the preselected set of harms.|
| final_comments_text             | String  | Free text response for any additional comments about the example.|
| text_other_reason_text          | String  | Text from other reasons.|

### Sample Data Queries

#### Submitted
**Load JSON file**

```
dev_json = pd.read_json('submitted/dev.json').reset_index()
```

**Parse out submission_annotations JSON into Dataframe**

```
submission =
pd.json_normalize(dev_json['submission_annotations'].apply(ast.literal_eval).tolist()).add_prefix('submission_annotation_')
```

**Parse out validation JSON objects**

```
exploded_validations =
dev_json.explode('validation').dropna(subset='validation') validations =
pd.json_normalize(exploded_validations['validation'].apply(lambda x:
ast.literal_eval(x)).tolist()).add_prefix('validation_')
```

**View all ratings associated with one image/prompt pair**

```
validations[validations['validation_question_id'] == 'question_id_abc_123']
```

**View all unique prompts**

```
submitted_json.drop_duplicates(subset="submitted_prompt")
``` 

**Load CSV file**

```
submitted_csv = pd.read_csv('submitted/dev.csv')
```

#### Attempts
**Load JSON file**

```
dataset_json = pd.read_json('attempts/dev.json')
```

## Request For Additional Data

To gain access to all images generated during this round, please fill out this
form with justifications on data access and usage:
https://forms.gle/4Lfr2eSxGmFjNFxC9

To gain access to the withheld test set, please fill out this form with
justifications on data access and usage: https://forms.gle/Sf88V8ZWLDEHz6Pu8

## License

Google LLC licenses this data under a **Creative Commons Attribution 4.0
International License**. Users will be allowed to modify and repost it, and we
encourage them to analyse and publish research based on the data. The dataset is
provided "AS IS" without any warranty, express or implied. Google disclaims all
liability for any damages, direct or indirect, resulting from the use of the
dataset.

## Papers

Jessica Quaye, Alicia Parrish, Oana Inel, Charvi Rastogi, Hannah Rose Kirk, Minsuk Kahng, Erin van Liemt, Max Bartolo, Jess Tsang, Justin White, Nathan Clement, Rafael Mosquera, Juan Ciro, Vijay Janapa Reddi, and Lora Aroyo. 2024. [Adversarial Nibbler: An Open Red-Teaming Method for Identifying Diverse Harms in Text-to-Image Generation](https://doi.org/10.1145/3630106.3658913). In The 2024 ACM Conference on Fairness, Accountability, and Transparency (FAccT ’24), June 03–06, 2024, Rio de Janeiro, Brazil.

```
@inproceedings{quaye2024nibbler,
author = {Quaye, Jessica and Parrish, Alicia and Inel, Oana and Rastogi, Charvi and Kirk, Hannah Rose and Kahng, Minsuk and Van Liemt, Erin and Bartolo, Max and Tsang, Jess and White, Justin and Clement, Nathan and Mosquera, Rafael and Ciro, Juan and Janapa Reddi, Vijay and Aroyo, Lora},
title = {Adversarial Nibbler: An Open Red-Teaming Method for Identifying Diverse Harms in Text-to-Image Generation},
year = {2024},
isbn = {9798400704505},
publisher = {Association for Computing Machinery},
address = {New York, NY, USA},
url = {https://doi.org/10.1145/3630106.3658913},
doi = {10.1145/3630106.3658913},
pages = {388–406},
numpages = {19},
keywords = {Adversarial Testing, Crowdsourcing, Data-centric AI, Red teaming, Text-to-image},
location = {<conf-loc>, <city>Rio de Janeiro</city>, <country>Brazil</country>, </conf-loc>},
series = {FAccT '24}
}
```
