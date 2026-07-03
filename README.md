# Sentiment and Thematic Analysis of Google Maps Reviews

## Introduction
Every day, educational institutions receive a continuous stream of feedback on public platforms like Google Maps. While standard star ratings provide a quick snapshot of overall performance, they often fail to capture the "why" behind a student's experience. When an institution relies solely on an average rating, it misses actionable details regarding lab facilities, faculty performance, and campus life. 

This project bridges that gap by deploying a modern, AI-driven data engineering pipeline. It transforms unstructured, raw public review data into a structured and actionable intelligence dashboard for an educational institution. By automating data collection and leveraging advanced Natural Language Processing (NLP) models, this self-sustaining ecosystem empowers administration to move from reactive damage control to proactive institutional improvement.

## Insights
The final analytical dashboard yielded several critical operational insights for the educational institution:

* **Overall Institutional Health**: The institution maintains a highly favorable reputation, registering an average rating of 4.45 across 1,981 total reviews. The overall sentiment breakdown is exceptionally strong at 84.6% Positive versus 6.5% Negative.
* **Demographic Engagement**: Digital feedback is heavily male-driven, with 1,212 reviews originating from males compared to 706 from females. However, female reviewers tend to award marginally higher average star ratings than their male counterparts.
* **Thematic Strengths & Weaknesses**: 
    * "Faculty & Teaching" is a massive institutional asset, generating 448 positive mentions with negligible negative feedback. 
    * Other physical and cultural pillars like "Infrastructure & Facilities" and "Environment & Discipline" also demonstrate strong performance. 
    * While "General Feedback" is mostly positive, it contains the highest absolute volume of negative sentiment (111 mentions), indicating a need to investigate broader, unspecified grievances.
* **Academic Feedback Distribution**: Undergraduate programs dominate the feedback loop, specifically the BSc (303 reviews) and BCom (106 reviews) branches. At a departmental level, Biotechnology leads the feedback volume with 81 distinct reviews. Across all levels, core academic delivery remains a stable asset with consistently high positive sentiment.
* **Temporal Trends**: The volume of engagement shows a clear escalation starting in 2021, eventually peaking with a massive, concentrated spike of 378 reviews in April 2023. 

## Process of Analysis

### Scraping
The data pipeline begins with an automated N8N orchestration layer. The N8N workflow triggers the Apify platform, specifically interacting with a "Google Maps Scraper" actor. This scraper navigates the dynamic web interface of Google Maps, handling scrolling and pagination to extract raw review objects in bulk, including text, ratings, timestamps, and reviewer metadata as a JSON file.

### Storage
Because web scraping APIs typically return bulk data as a single, nested JSON array, the N8N workflow utilizes a "Split Out" node to isolate and flatten the array into distinct, individual JSON objects. These records are then securely routed to ingest the required data into Google Sheets. This connection is bridged using a dedicated Google Cloud Platform (GCP) Service Account configured with "Editor" access.

### Data Cleaning or Interpretation
The core processing and heavy computational lifting occur within the Kaggle platform, leveraging free-tier NVIDIA T4 GPUs and the GCP service account. The raw data is enriched through a multi-step NLP cascade:
* **Name Transliteration**: The `indic_transliteration` library cleans and standardizes names, translating any non-English regional Indian scripts into English phonetics to prepare them for demographic profiling.
* **Gender Tagging**: A custom gender classifier was developed by fine-tuning a Meta Llama 3 (8B) model via Hugging Face on a curated dataset of 42,000 labeled Indian names. This model effectively parses cultural nuances to tag reviewers as 'Male', 'Female', or 'Unknown'. 
    * **Fine-Tuned Model Repository:** [Llama 3 Indian Gender Classifier](https://github.com/SHIVASHANKAR-V07/Llama_3_Indian_Gender_Classifier.git)
* **Sentiment Evaluation**: The `twitter-roberta-base-sentiment-latest` RoBERTa model is used to evaluate the review sentiment, understanding context, negation, and emojis, assigning probabilistic scores for Positive, Neutral, and Negative sentiments.
* **Thematic Analysis**: The text is passed to the Gemini 2.5 Flash-Lite model for thematic findings. Through strategic prompt engineering, the generative AI performs zero-shot categorization, reading the reviews and assigning specific topics and standardized operational themes.

### Visualization
The final enriched dataset is exported as a CSV and loaded into Microsoft Power BI Desktop. To support complex and interactive reporting, the flat file is structurally transformed by creating a data model for the final dataset (a Star Schema). The central Fact Table is linked to distinct dimension tables for Themes and Topics, supporting the creation of the final dashboard that presents key performance indicators, temporal trends, and thematic distributions.
