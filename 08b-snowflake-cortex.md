# Snowflake Cortex
Fully managed service that offers machine learning and AI solutions:
1. LLM Functions: SQL/Python functions that leverage large language models (LLMs) for understanding, querying, translating, summarizing, and generating free-form text.
2. ML Functions: SQL functions that perform predictive analysis using machine learning, to help you gain insights into your structured data, and accelerate everyday analytics.

## LLM Functions
The available functions:
- COMPLETE: Given a prompt, returns a response that completes the prompt.
- EMBED_TEXT_768: Given a piece of text, returns a vector embedding that represents that text.
- EXTRACT_ANSWER: Given a question and unstructured data, returns the answer to the question if it can be found in the data.
- SENTIMENT: Returns a sentiment score, from -1 to 1, representing the detected positive or negative sentiment of the given text.
- SUMMARIZE: Returns a summary of the given text.
- TRANSLATE: Translates given text from any supported language to any other.

The CORTEX_USER database role in the SNOWFLAKE database includes the privileges that allow users to call Snowflake Cortex LLM functions. 

By default, the CORTEX_USER role is granted to the PUBLIC role, so this allows all users in your account to use the Snowflake Cortex LLM functions.

Snowflake Cortex LLM functions incur compute cost based on the number of tokens processed. A token is the smallest unit of text processed by Snowflake Cortex LLM functions, approximately equal to four characters of text. 

Snowflake recommends executing queries that call a Snowflake Cortex LLM Function with a smaller warehouse (no larger than MEDIUM) because larger warehouses do not increase performance.

Vector embeddings use cases:
- find similar documents using vector similarity

## ML Functions




























