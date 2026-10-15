# Mirrulations Structure

This guide describes the Mirrulations S3 and on-disk layout and the
Regulations.gov JSON stored within it.

## Mental model

```text
Regulations.gov resources → Mirrulations raw mirror → derived files
```

- **Raw**: downloaded API JSON, HTML/HTM, or attachments.
- **Derived**: generated content, such as PDF text extraction.
- Never replace raw data with a derivative.
- A docket is a container, not one document or one regulatory stage.

For the federal rulemaking process, read the excellent official
[Reg Map PDF](https://www.reginfo.gov/public/reginfo/Regmap/REG_MAP_2020.pdf).

## Storage tree

```text
s3://mirrulations/
├── raw-data/<agency>/<docket-id>/
│   ├── text-<docket-id>/
│   │   ├── docket/<docket-id>.json
│   │   ├── documents/<document-id>.json
│   │   ├── documents/<document-id>_content.htm[l]
│   │   └── comments/<comment-id>.json
│   └── binary-<docket-id>/comments_attachments/
│       └── <comment-id>_attachment_<n>.<extension>
└── derived-data/<agency>/<docket-id>/
    └── <organization>/<project>/<file-type>/<derived-file>
```

Branches exist only when files exist. Attachment formats are arbitrary.

## JSON resource shapes

Regulations.gov v4 detail responses use a common envelope:

```json
{
  "data": {
    "id": "public-resource-id",
    "type": "dockets | documents | comments",
    "attributes": {},
    "links": {},
    "relationships": {}
  }
}
```

`links` and `relationships` are optional. Fields can be null. Preserve the
complete payload; use the following fields for identity and traversal.

### Docket JSON

```json
{
  "data": {
    "id": "CMS-2026-1255",
    "type": "dockets",
    "attributes": {
      "agencyId": "CMS",
      "docketType": "Rulemaking",
      "title": "...",
      "dkAbstract": "...",
      "rin": "0938-AV44",
      "objectId": "opaque-id"
    }
  }
}
```

The docket groups public records. Its `rin`, when present, links to the Unified
Agenda/Reginfo.gov regulatory action.

### Document JSON

```json
{
  "data": {
    "id": "CMS-2026-1255-0001",
    "type": "documents",
    "attributes": {
      "docketId": "CMS-2026-1255",
      "documentType": "Proposed Rule",
      "subtype": null,
      "title": "...",
      "frDocNum": "2026-07205",
      "commentStartDate": "...",
      "commentEndDate": "...",
      "effectiveDate": null,
      "objectId": "09000064b927bd06",
      "withdrawn": false,
      "fileFormats": []
    }
  }
}
```

ANPRMs, proposed rules, final rules, notices, analyses, meetings, and SORNs are
all documents. Classify them using type, subtype, title, dates, Federal Register
metadata, docket context, and content—not a search term alone.

### Comment JSON

```json
{
  "data": {
    "id": "CMS-2026-1255-0032",
    "type": "comments",
    "attributes": {
      "docketId": "CMS-2026-1255",
      "documentType": "Public Submission",
      "subtype": "Public Comment",
      "commentOnDocumentId": "CMS-2026-1255-0001",
      "commentOn": "09000064b927bd06",
      "comment": "...",
      "organization": "...",
      "fileFormats": []
    }
  }
}
```

The form comment may be substantive, empty, or say “See attached.” Attachments
are separate raw files.

### Attachments

Attachment metadata appears in API relationships/file inventories; attachment
bytes are stored separately. An opaque attachment ID is not a comment ID.

```text
comment JSON ──comment ID + ordinal──> raw attachment
raw attachment ──same identity──> derived extraction
```

## Exact joins

```text
docket.data.id = document.attributes.docketId
docket.data.id = comment.attributes.docketId
document.data.id = comment.attributes.commentOnDocumentId
document.attributes.objectId = comment.attributes.commentOn
```

Important API behavior:

```text
GET /comments?filter[commentOnId]=<document.attributes.objectId>
```

The search expects the opaque `objectId`, not the public document ID.

| Identifier | Scope |
| --- | --- |
| Docket ID | Regulations.gov container |
| Document/comment ID | Public API resource |
| `objectId` | Opaque API join/search identity |
| RIN | Regulatory action in Reginfo.gov |
| `frDocNum` | Federal Register publication |
| Tracking number | Submission workflow |

## Process and SORNs

Typical informal rulemaking can include:

```text
RFI/ANPRM → proposed rule → comments → final rule → effective/compliance dates
```

It can branch through direct or interim final rules, supplemental proposals,
reopenings, corrections, withdrawals, delays, and reconsideration. Regulations.gov
does not encode a workflow; corroborate cross-stage edges from RINs, Federal
Register numbers, titles, dates, and document text.

A System of Records Notice is a parallel Privacy Act process, not an APA
rulemaking stage. Initial and modified SORNs commonly have
`documentType: "Notice"` and `subtype: "System of Records"`; rescindments may
carry the state only in the title.

## Versions and failures

- Identical JSON payloads are deduplicated.
- Changed JSON receives `(1)`, `(2)`, and later revision suffixes.
- Failed downloads create `<stem>_UNAVAILABLE` tombstones containing
  `HTTP <status-code>`.
- HTM and HTML document bodies are both valid.
- Incomplete path identity can use an `unknown` fallback.

Use the full object key and content hash when provenance matters.

Raw Mirrulations files are immutable input. Repairs and analyses belong in a
derived namespace.
