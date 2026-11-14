# Resume RAG Enhancement Feature Specification

## 1. Goal

Enhance the existing Resume RAG application to return unique, relevance-ranked resumes with optional candidate-fit summaries, and present results clearly in the existing web UI.

The intended pipeline is:

```text
Recruiter query
    ↓
BM25 search + vector search
    ↓
Merge candidates
    ↓
Deduplicate by resume ID
    ↓
Re-rank candidates
    ↓
Select final results
    ↓
Optionally summarize candidate fit
    ↓
Return API response
    ↓
Render ranked results in the UI
```

## 2. Scope and implementation rules

- Extend the existing backend and frontend projects; do not create replacement applications.
- Preserve resume ingestion and its existing API contract.
- Reuse current retrieval services, routes, validation, and UI components where possible.
- Keep result ordering stable and return each resume no more than once.
- Ground LLM-generated summaries only in supplied resume content.
- Preserve usable search results if an optional summary or re-ranking step fails, and make degradation visible through warnings.
- Avoid exposing private contact information in search result cards. Use the existing candidate-detail flow when more information is needed.
- Implement and validate **one phase at a time**. After each phase, provide its Postman request and expected response, then wait for user approval before continuing.

## 3. Repository baseline

The existing backend already has retrieval endpoints for readiness, hybrid search, re-ranking, summarization, and end-to-end search. Its retrieval service also contains merge/deduplication, re-ranking, optional summaries, and fallback handling.

The web application already has a search page and candidate feature. Verify existing behavior before changing it, and update only gaps. The phases below are verification and enhancement gates—not a requirement to duplicate functionality that already works.

## 4. Phase 1 — Re-ranking

### Goal

Ensure the standalone re-ranking API orders only the supplied candidates according to their relevance to the query.

### Endpoint

```http
POST /v1/search/rerank
Content-Type: application/json
```

### Postman request body

```json
{
  "query": "Senior QA architect with RAG, DeepEval, MCP, and enterprise automation experience",
  "candidates": [
    {
      "resumeId": "691db80aa895776f97b6eca6",
      "snippet": "Test Architect with 13 years of experience in RAG, DeepEval, MCP, and enterprise test automation."
    },
    {
      "resumeId": "691db80aa895776f97b6eca7",
      "snippet": "Software engineer experienced in frontend development and UI testing."
    }
  ],
  "topK": 2
}
```

### Expected response shape

Scores, reasons, and the model name are generated at runtime and may vary.

```json
{
  "results": [
    {
      "resumeId": "691db80aa895776f97b6eca6",
      "rank": 1,
      "relevanceScore": 0.96,
      "reason": "The resume directly matches the requested QA architecture, RAG, and evaluation experience."
    },
    {
      "resumeId": "691db80aa895776f97b6eca7",
      "rank": 2,
      "relevanceScore": 0.31,
      "reason": "The resume has testing experience but little evidence of the requested GenAI skills."
    }
  ],
  "model": "configured Groq model"
}
```

### Acceptance criteria

- Every returned `resumeId` was supplied in the request.
- No candidate is returned more than once.
- Results are ordered by `rank`, starting at 1.
- Invalid input or an unavailable LLM returns an explicit API error.
- Test against the configured LLM provider; sample scores are not fixed values.

### Phase gate

Test this request in Postman and inspect the response. Stop for user testing and approval.

## 5. Phase 2 — Candidate deduplication

### Goal

Ensure that the same resume retrieved by both BM25 and vector search appears only once in the merged candidate list.

### Deduplication rules

- Use the canonical `resumeId` as the deduplication key.
- Merge source labels for a repeated candidate (for example, `["bm25", "vector"]`).
- Preserve available candidate metadata and scores from both sources.
- Do not deduplicate by name alone; different people may share a name.
- Deduplicate before sending candidates to the re-ranker.

### Postman verification

Use a query expected to return overlapping BM25 and vector candidates.

```http
POST /v1/search
Content-Type: application/json
```

```json
{
  "query": "Senior QA architect experienced in RAG and LLM evaluation",
  "options": {
    "bm25TopK": 20,
    "vectorTopK": 20,
    "rerankTopN": 10,
    "finalTopK": 10,
    "summarize": false
  }
}
```

### Expected response checks

Exact resumes depend on the indexed data and query. Verify that:

- `results` contains no repeated `resumeId`.
- A candidate found through both strategies has both values in `sources`.
- `rank` values are unique and ordered from 1.
- The response includes `degraded`, `warnings`, and `timings`.

### Phase gate

Confirm result IDs are unique in Postman. Stop for user testing and approval.

## 6. Phase 3 — Candidate-fit summarization

### Goal

Generate a concise, query-specific summary based only on the supplied candidate text.

### Endpoint

```http
POST /v1/search/summarize
Content-Type: application/json
```

### Postman request body

```json
{
  "query": "Senior QA architect with RAG and LLM evaluation experience",
  "candidate": {
    "resumeId": "691db80aa895776f97b6eca6",
    "snippet": "Test Architect with 13 years of experience. Led enterprise test automation and worked with RAG, DeepEval, MCP, and agentic QA systems."
  },
  "style": "short",
  "maxTokens": 150
}
```

### Expected response shape

Summary wording varies with the model and supplied text.

```json
{
  "resumeId": "691db80aa895776f97b6eca6",
  "summary": "Strong match for a senior QA architecture role, with extensive test automation experience and direct work with RAG, DeepEval, and MCP."
}
```

### Acceptance criteria

- The response contains the same `resumeId` as the request.
- The summary is relevant to the query and grounded in the supplied candidate text.
- The API validates `style` and `maxTokens`.
- A summarization failure does not discard otherwise usable search results; surface a warning.

### Phase gate

Test the request and inspect the summary. Stop for user testing and approval.

## 7. Phase 4 — End-to-end search integration

### Goal

Verify that a single request runs retrieval, merging, deduplication, re-ranking, final result selection, and optional summarization.

### Endpoint

```http
POST /v1/search
Content-Type: application/json
```

### Postman request body

```json
{
  "query": "Senior agentic QA architect with RAG, MCP, DeepEval, API automation, and Azure DevOps experience",
  "filters": {
    "minYearsExperience": 10
  },
  "options": {
    "bm25TopK": 20,
    "vectorTopK": 20,
    "rerankTopN": 10,
    "finalTopK": 5,
    "summarize": true,
    "summaryStyle": "short"
  }
}
```

### Expected response shape

Candidate and timing values are illustrative; actual results depend on the database and configured services.

```json
{
  "query": "Senior agentic QA architect with RAG, MCP, DeepEval, API automation, and Azure DevOps experience",
  "results": [
    {
      "rank": 1,
      "resumeId": "691db80aa895776f97b6eca6",
      "name": "Example Candidate",
      "role": "Test Architect",
      "company": "Example Company",
      "totalExperience": 13,
      "skills": ["RAG", "DeepEval", "MCP", "Azure DevOps"],
      "sources": ["bm25", "vector"],
      "summary": "Strong match for the requested QA architecture and GenAI experience."
    }
  ],
  "degraded": false,
  "warnings": [],
  "timings": {
    "embeddingMs": 420,
    "bm25Ms": 180,
    "vectorMs": 310,
    "rerankMs": 800,
    "summarizeMs": 600,
    "totalMs": 1450
  }
}
```

### Acceptance criteria

- BM25 and vector search contribute candidates when both are available.
- Duplicate resume IDs are removed before re-ranking.
- Final results are ordered and limited by `finalTopK`.
- Each result includes `rank`, `resumeId`, `sources`, and available resume metadata.
- When requested and available, each result includes a grounded `summary`.
- `degraded` and `warnings` describe fallback or optional-stage failures.
- `timings` reports retrieval and LLM stages.
- If both retrieval strategies fail, the API returns an explicit error rather than an empty success response.

### Phase gate

Test the request in Postman with summarization enabled, then with summarization disabled. Stop for user testing and approval.

## 8. Phase 5 — Search results UI

### Goal

Display the end-to-end API results in the existing search page, without removing ingestion or candidate-detail functionality.

### Result card content

Display available response fields clearly:

- Rank
- Candidate name
- Role and company
- Total experience
- Skills
- Retrieval source labels
- Candidate-fit summary, when returned
- Action to open the existing candidate-detail view

Do not display missing values as fabricated data. Do not expose email or phone in result cards unless explicitly authorized by existing product behavior.

### UI states

- Loading state while search is in progress
- Results state with ranked candidate cards
- Empty state when no resumes match
- Error state for request failures
- Degraded state when the API returns warnings
- Summary hidden or marked unavailable when summarization fails

### UI acceptance criteria

- Results appear in API rank order.
- A resume appears no more than once.
- Long skills and summaries remain readable on desktop and mobile.
- Search errors and degraded warnings are visible to the user.
- Existing ingestion navigation and candidate-detail navigation continue to work.
- The UI does not claim that re-ranking or summarization succeeded when the API reports a failure.

### Postman and browser checkpoint

Use the Phase 4 Postman request to confirm the API response. Then submit the same query through the UI and verify that displayed cards match the API response.

Stop for user testing and approval.

## 9. Validation and tests

Add or update focused tests for:

- Deduplication by `resumeId`, including source merging.
- Re-ranking output validation and unknown IDs.
- Summary validation and grounded input handling.
- End-to-end result ordering, final result limits, and optional summaries.
- Fallback behavior when BM25, vector search, re-ranking, or summarization fails.
- UI loading, empty, error, degraded, and results states.

Run the smallest relevant test/build command for each phase. Do not proceed to the next phase until its code checks pass and the user has tested the supplied Postman request.

## 10. Completion checklist

- [ ] Retrieved resumes are deduplicated by canonical resume ID.
- [ ] Candidates are re-ranked before final selection.
- [ ] Final results can include query-specific summaries.
- [ ] The end-to-end endpoint exposes warnings and timings.
- [ ] The existing UI renders available relevant result fields clearly.
- [ ] Existing ingestion and candidate-detail flows remain available.
- [ ] Each phase has passed its Postman and code validation gates.
- [ ] User approval has been received after each phase.
