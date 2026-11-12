## A5: Exercise 1: PubMed EFetch and XML Parsing

### 1. EFetch Query and Metadata

`get_pubmed_data(pmid)` sends an HTTP GET request to NCBI's `efetch.fcgi` endpoint. It sets `db=pubmed`, passes the supplied PMID as `id`, and requests XML with `retmode=xml`. The function returns the raw XML string and can be reused with other PMIDs.

For PMID `34807729`, the response contains a `PubmedArticle` inside `PubmedArticleSet`. I found the following metadata in the XML:

| Category | What I found |
| --- | --- |
| Citation and article identifiers | `MedlineCitation/PMID` contains `34807729`. `PubmedData/ArticleIdList` lists the PMID and DOI `10.1089/omi.2021.0155`. |
| Bibliographic metadata | `MedlineCitation/Article` contains the title, journal, publication date, pagination, authors, language, and publication types. |
| Controlled vocabulary and keywords | `MeshHeadingList` includes MeSH descriptors such as `COVID-19` and `Cytokines`. The record also contains a `ChemicalList` and a separate `KeywordList`. |
| Funding information | The publication types include `Research Support, Non-U.S. Gov't`. The XML has no `GrantList`, so no structured grant identifier is available in this record. |
| Abstract structure | `Article/Abstract` contains one unlabeled `AbstractText` element with no inline child tags. |

### 2. Abstract Extraction and Validation

Cell 2 printed the full extracted abstracts and showed the following XML structures:

| PMID | `AbstractText` blocks | Section labels in XML order | Inline child tags |
| --- | --- | --- | --- |
| 34807729 | 1 | None | None |
| 32299786 | 5 | `BACKGROUND`, `AIM`, `METHOD`, `RESULTS`, `CONCLUSION` | None |

For PMID `34807729`, I compared the entire extracted abstract with the text of the original XML block after trimming leading and trailing whitespace. The output was `Full abstract matches the original XML block: True`, confirming that the parser retrieved the complete abstract.

For PMID `32299786`, the output included all five sections in the same order as the XML. Each block has a `Label` and an `NlmCategory` attribute. For example, the display label `AIM` corresponds to the normalized category `OBJECTIVE`.

The parser uses `findall()` to read every `AbstractText` element in order. It adds the `Label` as a heading when one is present and joins the sections with blank lines. It uses `itertext()` to preserve text before, inside, and after any inline tags.

### 3. Pipeline Results and Term Audit

The pipeline sends one EFetch request per PMID and waits 0.4 seconds before each request to stay within the limit of three requests per second without an API key. It searches only the extracted abstract for the exact terms `chemokines` and `COVID-19`, ignoring capitalization.

| PMID | Mentions `chemokines` | Mentions `COVID-19` | Status |
| --- | --- | --- | --- |
| 34807729 | Yes | Yes | OK |
| 21943025 | No | No | OK |
| 33980688 | Yes | Yes | OK |
| 32299786 | Yes | No | OK |
| 20725509 | No | No | OK |
| 41916485 | N/A | N/A | No abstract |
| 17245822 | No | No | OK |
| 31842501 | No | No | OK |
| 35064122 | No | Yes | OK |

Eight records had abstracts available for evaluation. Among those eight, three mentioned `chemokines`, three mentioned `COVID-19`, and two mentioned both.

### 4. Data Quality and Defensive Programming

PMID `41916485` is a valid PubMed record classified as a letter, but it has no abstract. Calling `.text` on the result of `find()` could raise an `AttributeError` because `find()` returns `None` when an element is absent. The parser uses `findall()`, which returns an empty list in this case. The pipeline then reports `No abstract` and marks both terms as `N/A` because there was no abstract to evaluate.

PMID `31842501` contains inline `<i>` tags within its abstract. Reading only `.text` would miss text inside and after those tags. The parser uses `itertext()` to collect the text in its original order.

The pipeline also verifies that EFetch returned the requested PMID. It catches request failures, malformed XML, and decoding errors for each record so processing can continue with the next PMID.


## Exercise 2: RESTful API

### Part 1: API Query Architecture

**Direct API query.** `GET /users/{username}` returns `public_repos`, the number of public repositories, and `created_at`, the account creation timestamp. The default response format is JSON.

**Aggregated API query.** `GET /users/ramcdougal` gives the public repository count, but the profile has no total-stars field. To calculate that total, the script calls `GET /users/ramcdougal/repos` with `type=owner` and sums `stargazers_count` across the returned repositories. The `per_page` parameter sets the page size. After each response, the script follows the `rel="next"` URL in the HTTP `Link` header. It stops when the response has no next-page link.

### Part 2: Results and Verification

| Metric | Result |
| --- | --- |
| Public repositories | 45 |
| Account creation timestamp (UTC) | `2014-02-13T03:10:45Z` |
| Total stars across public repositories | 29 |

I tested pagination with `per_page=10`. The API returned 5 pages containing 45 repository records with 45 unique repository IDs. The count matched `public_repos` from the profile. A second check with `per_page=100` returned 45 repositories in one page and the same total of 29 stars.

### Part 3: Rate Limits

The primary rate limit for unauthenticated GitHub REST API requests is 60 requests per hour per originating IP address.

`X-RateLimit-Limit` reports the request allowance for the current window. `X-RateLimit-Remaining` reports how many requests are still available. In the five-page check, the final response showed a limit of 60 and 53 requests remaining. The remaining count can differ between runs.

A production pipeline should read these headers after every response. When `X-RateLimit-Remaining` reaches zero, it should save its progress and pause requests until the time given by `X-RateLimit-Reset`, a Unix timestamp in seconds. After that time, it can resume from the next page or retry the request that was rate-limited.

### References

- [GitHub REST API: Users](https://docs.github.com/en/rest/users/users)
- [GitHub REST API: Repositories](https://docs.github.com/en/rest/repos/repos)
- [GitHub REST API: Pagination](https://docs.github.com/en/rest/using-the-rest-api/using-pagination-in-the-rest-api)
- [GitHub REST API: Rate Limits](https://docs.github.com/en/rest/using-the-rest-api/rate-limits-for-the-rest-api)
