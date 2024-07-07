# Langchain-Based-Document-Retrieval-Tool
A Langchain-based document retrieval tool to check data consistency among several financial documents.


## Background
Each quarter Singapore Exchange Group (SGX) needs to publish several financial reports. Data consistency among these files becomes an important topic to make sure the data accuracy.
In the past, financial anaylsts they manually eyeballing check these files, which is quite time-costing and the process is quite fragile.
With the help of this project, we could automatically extarct financial figures from these files.

## Input Data Source
https://investorrelations.sgx.com/financial-information/financial-results

## Overall Implementation

### Highlevel Design

<img src="https://github.com/xlsi/Langchain-Based-Document-Retrieval-Tool/blob/main/doc/document_extarction.png" width="2500" height="600">


Extraction is when you use LLMs to extract structured information from unstructured text.

#### Step 1: Load Data
- LangChain provides tools to load file in differnet format. In our use case, we load data from PDF files.

#### Step 2: Split Data
Document data is usually too long to fit into the prompt due to the context length limitation of LLMs. To process this text, there are normally these strategies:  
1. **Change LLM** Choose a different LLM that supports a larger context window.
2. **Brute Force** Chunk the document, and extract content from each chunk.
3. **RAG** Chunk the document, index the chunks, and only extract content from a subset of chunks that look "relevant".
In our use case, we use the second way.

#### Step 3: Define Schema
- We need to describe what information we want to extract from the text. We'll use Pydantic to define an example schema to extract information.
- In **most cases**, you should be extracting a list of entities rather than a single entity. Therefore, we use pydantic to nest models inside one another.

#### Step 4: Prompts for Extractor
- Create an information extractor

#### Step 5: Add Reference Examples
The quality of extractions can often be improved by providing reference examples to the LLM. There are 3 different approaches for extracting information using LLMs:
1. prompt based/parsing
2. function/tool calling
3. JSON mode
We tried both 1 and 2. But as for some models they do not support tool calling, it turns out that 1 is a more robust way to add examples. It is also proved effective by final output.
Therefore, we add some examples in our prompt in step 4. Please note that extraction quality is principally driven by providing good reference examples and good schema documentation.

#### Step 6: Extract
Now run your extarction!!!
- Use any LLM (Mistral AI is used in our use case. We've also tried Open AI, but there's no too much difference) to run the extraction on each chunk.
- After extracting data from across the chunks, merge the extractions together.

## Output
|    | name                                                                              | value           | period               | source_sentence                                                                                                                                                                                                                          |
|---:|:----------------------------------------------------------------------------------|:----------------|:---------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|  0 | revenue                                                                           | S$592.2 million | 1H FY2024            | Revenue increased 3.6% to S$592.2 million ( S$571.4 million), mainly driven by higher revenues from Currencies and Commodities2 and Platform and Others, partially offset by lower Equities – Cash2 and Equities – Derivatives2 revenue. |
|  1 | adjusted EBITDA                                                                   | S$344.6 million | 1H FY2024            | Adjusted EBITDA rose to S$344.6 million ( S$334.1 million) , while adjusted earnings per share increased to 23.5 cents (22.2 cents).                                                                                                     |
|  2 | adjusted net profit                                                               | S$251.4 million | 1H FY2024            | Singapore Exchange (SGX Group ) today reported 1H FY202 4 adjusted net profit of S$ 251.4 million (S$2 36.8 million).                                                                                                                    |
|  3 | FICC revenue                                                                      | $151.9 million  | not specified        | FICC revenue increased 28.1% to S$151.9 million ( S$118.5 million) and accounted for 25.6% (20.7%) of total revenue3.                                                                                                                    |
|  4 | Fixed Income revenue                                                              | $3.9 million    | not specified        | Fixed Income revenue decreased 8.4% to S$3.9 million ( S$4.3 million).                                                                                                                                                                   |
|  5 | Listing revenue                                                                   | $2.5 million    | not specified        | Listing revenue: S$2.5 million, down 1.4% from S$2.6 million.                                                                                                                                                                            |
|  6 | Corporate actions and other revenue                                               | $1.4 million    | not specified        | Corporate actions and other revenue: S$1.4 million, down 19.1% from S$1.7 million.                                                                                                                                                       |
|  7 | Currencies and Commodities revenue                                                | $148.0 million  | not specified        | Currencies and Commodities revenue increased 29.5% to S$148.0 million ( S$114.3 million).                                                                                                                                                |
|  8 | OTC FX revenue                                                                    | $40.9 million   | not specified        | OTC FX revenue was S$40.9 million ( S$36.2 million) and accounted for 27.7% (31.7%) of Currencies and Commodities revenue.                                                                                                               |
|  9 | Trading and clearing revenue                                                      | $111.3 million  | not specified        | Trading and clearing revenue: S$111.3 million, up 25.3% from S$88.8 million.                                                                                                                                                             |
| 10 | Treasury and other revenue                                                        | $36.7 million   | not specified        | Treasury and other revenue: S$36.7 million, up 44.3% from S$25.4 million.                                                                                                                                                                |
| 11 | Equities – Cash revenue                                                           | $159.6 million  | not specified        | Equities – Cash revenue declined by 5.6% to S$159.6 million ( S$169.1 million ) and accounted for 26.9 % (29.6 %) of total revenue.                                                                                                      |
| 12 | Listing revenue                                                                   | $14.6 million   | not specified        | Listing revenue: S$14.6 million, down 3.3% from S$15.1 million.                                                                                                                                                                          |
| 13 | Trading and clearing revenue                                                      | $77.2 million   | not specified        | Trading and clearing revenue: S$77.2 million, down 13.9 % from S$89.6 million.                                                                                                                                                           |
| 14 | Securities settlement, depository management, corporate actions and other revenue | $67.9 million   | not specified        | Securities settlement, depository management, corporate actions and other revenue: S$67.9 million, up 5.3% from S$64.5 million.                                                                                                          |
| 15 | trading and clearing revenue                                                      | $123.2 million  | 1H FY2024            | Trading and clearing revenue: S$123.2 million, down 15.3 % from S$145.4 million                                                                                                                                                          |
| 16 | treasury and other revenue                                                        | $37.5 million   | 1H FY2024            | Treasury and other revenue: S$37.5 million, up 37.7% from S$27.2 million                                                                                                                                                                 |
| 17 | market data revenue                                                               | $24.2 million   | 1H FY2024            | Market data revenue: S$24.2 million, up 9.9% from S$22.0 million                                                                                                                                                                         |
| 18 | connectivity revenue                                                              | $38.5 million   | 1H FY2024            | Connectivity revenue: S$38.5 million, up 8.7% from S$35.4 million                                                                                                                                                                        |
| 19 | indices and other revenue                                                         | $57.4 million   | 1H FY2024            | Indices and other revenue: S$57.4 million, up 6.8% from S$53.8 million                                                                                                                                                                   |
| 20 | net profit                                                                        | S$289.7 million | first half of FY2024 | 5% to S$289.7 million (S$279.9 million), which excludes amortisation of purchased intangible assets.                                                                                                                                     |
| 21 | total capital expenditure                                                         | S$18.5 million  | not specified        | Total capital expenditure was S$18.5 million ( S$17.8 million). These investments include upgrades to our Titan OTC trade reporting system, modernisation of our technology infrastructure and consolidation of our office spaces.       |
| 22 | expenses                                                                          | not specified   | FY2024               | Expense growth in FY2024 is likely to be similar to the 3% year -on-year expense growth rate observed in the first half of FY2024, lower than the previously guided mid -single digit percentage range.                                  |
| 23 | projected capital expenditure                                                     | S$70-75 million | FY2024               | Our projected capital expenditure for FY2024 is anticipated to be within S$70-75 million, lower than our previously guided S$75-80 million range.                                                                                        |


## Reference
https://python.langchain.com/v0.2/docs/how_to/#extraction  
https://medium.com/@chipanajose/how-extract-data-from-pdf-using-langchain-and-mistral-74b252fd88a0  
https://stackoverflow.com/questions/77810548/langchain-model-to-extract-key-information-from-pdf  
https://blog.langchain.dev/open-source-extraction-service/  
https://github.com/langchain-ai/langchain-extract?ref=blog.langchain.dev  
