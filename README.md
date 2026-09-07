## CogWorkLabs' shopify product scraper

**CogWorkLabs' shopify product scraper** turns a Shopify catalog into structured records that can be filtered, inspected, and exported without manually copying product information. The working flow reads product records, follows pagination, normalizes fields, and writes the selected data to machine-readable output. Where the source is an authorized Shopify store, the build uses Shopify's GraphQL Admin API rather than depending on brittle page markup. Shopify's product query exposes fields including titles, descriptions, vendors, product types, variants, media, SEO metadata, categories, tags, and availability. <a href="https://shopify.dev/docs/api/admin-graphql/latest/queries/products" target="_blank" rel="nofollow">Shopify's product query documentation</a> documents the same product connection used by this workflow.

<p align="center">
  <a href="https://t.me/Bitbash333" target="_blank" rel="nofollow">
    <img src="https://img.shields.io/badge/Chat_on-Telegram-2CA5E0?style=for-the-badge&amp;logo=telegram&amp;logoColor=white" alt="Chat on Telegram">
  </a>&nbsp;
  <a href="https://wa.me/923249868488?text=Hi%2C%20I%27m%20interested%20in%20CogWorkLabs." target="_blank" rel="nofollow">
    <img src="https://img.shields.io/badge/Chat-WhatsApp-25D366?style=for-the-badge&amp;logo=whatsapp&amp;logoColor=white" alt="Chat WhatsApp">
  </a>&nbsp;
  <a href="mailto:hello@cogworklabs.com" target="_blank" rel="nofollow">
    <img src="https://img.shields.io/badge/Email-hello@cogworklabs.com-EA4335?style=for-the-badge&amp;logo=gmail&amp;logoColor=white" alt="Email hello@cogworklabs.com">
  </a>&nbsp;
  <a href="https://www.cogworklabs.com" target="_blank" rel="nofollow">
    <img src="https://img.shields.io/badge/Visit-Website-007BFF?style=for-the-badge&amp;logo=google-chrome&amp;logoColor=white" alt="Visit Website">
  </a>
</p>

## What the workflow actually does

The useful part of the system is not simply retrieving a product record. It turns a catalog request into a repeatable sequence: authenticate against an authorized store, select the fields required for the export, apply an optional search filter, retrieve records in pages, flatten nested variants and media, normalize missing values, and write the final dataset. A request for products updated after a specific date can use Shopify's query syntax rather than downloading everything and filtering locally. Shopify documents field filters, Boolean operators, range comparisons, exclusion operators, and phrase searches in its <a href="https://shopify.dev/docs/api/usage/search-syntax" target="_blank" rel="nofollow">API search syntax reference</a>. That matters when a catalog contains thousands of records and the required subset is much smaller. The resulting dataset is designed for downstream spreadsheet work, catalog analysis, migration preparation, or another automation that consumes CSV or JSON.

## Core Features

| Feature | Description |
| --- | --- |
| Authorized catalog access | Manual copying creates inconsistent records and exposes credentials to unnecessary handling. The build uses scoped Shopify API access so the extraction process only requests the product data its workflow requires. Shopify documents `read_products` as the scope required for product access. <a href="https://shopify.dev/docs/api/admin-graphql/latest/objects/product" target="_blank" rel="nofollow">Product object reference</a> |
| Cursor-based pagination | A single request cannot represent an entire large catalog reliably. The scraper follows `hasNextPage` and `endCursor` values until the requested result set is exhausted, using Shopify's cursor-based pagination model. Shopify documents a maximum of 250 resources per connection request. <a href="https://shopify.dev/docs/api/usage/pagination-graphql" target="_blank" rel="nofollow">GraphQL pagination guide</a> |
| Field-level filtering | Downloading every record wastes processing when only a subset is required. Filters can target fields such as product type or update time before records reach the normalization stage, using Shopify's documented query grammar. |
| Variant and media normalization | Raw product responses contain nested structures that are awkward in spreadsheets. The transformation layer converts product-level fields and repeated variant or media values into predictable columns while preserving the relationship to the parent product. |
| CSV and JSON export | A raw API response is useful to software but inconvenient for operators. The export stage produces structured CSV for spreadsheet workflows and JSON for downstream scripts, preserving consistent field names between runs. |
| Failure-aware request handling | A temporary API throttle should not invalidate an otherwise successful extraction. Requests are handled with rate-limit awareness and retry behavior so a transient response can be retried instead of producing a partial silent export. Shopify's current GraphQL Admin API documentation describes calculated query-cost limits and `429 Too Many Requests` responses. |

## From store data to usable records

Consider a catalog containing products with different numbers of variants. The extraction begins with a product connection and asks only for the fields required by the export. If the first response reports another page, its end cursor becomes the starting point for the next request. A product with three variants therefore remains one product record in the logical model while its variant values are expanded into consistent output rows. Missing optional values are represented explicitly rather than shifting later columns. Media URLs remain associated with the relevant product instead of becoming unrelated rows. This distinction prevents a common scraping failure: producing a visually complete dataset that cannot be reliably joined back to its source products. Shopify's product model includes product options, variants, media, SEO data, categories, and collections, so the transformation layer has to preserve those relationships rather than treating the response as a flat HTML table. <a href="https://shopify.dev/docs/api/admin-graphql/latest/objects/Product" target="_blank" rel="nofollow">Shopify's Product reference</a> defines those fields and relationships.

## Authentication and data boundaries

The scraper is built around authorized access, not anonymous access to stores that have not granted permission. Shopify's authentication model associates an access token with approved scopes, and each GraphQL Admin API request carries that token in the `X-Shopify-Access-Token` header. <a href="https://shopify.dev/docs/apps/build/authentication-authorization" target="_blank" rel="nofollow">Shopify's authentication documentation</a> explains how authentication identifies the calling app and how scopes determine which resources it may access. For a product-only workflow, the practical boundary is important: the extraction code should request product access rather than broad permissions that the job does not require. Credentials belong in environment configuration or a secrets manager, never in the repository. Shopify also explicitly recommends keeping client secrets out of source control in its <a href="https://shopify.dev/docs/apps/build/authentication-authorization/manage-credentials" target="_blank" rel="nofollow">credential management documentation</a>. This keeps the repository focused on the extraction logic while store authorization remains an operational concern.

## Filtering before extraction

Filtering is most useful when the requested catalog slice can be expressed in Shopify's search language. For example, a run can request products where `product_type` matches a chosen category or where `updated_at` is newer than a defined timestamp. The filter is evaluated as part of the product query rather than after the entire catalog has been transferred. Shopify's search grammar supports field searches, ranges, Boolean operators, exclusions, and grouped expressions. It also warns that some range searches can become slow on sufficiently large collections when the searched field and sort key do not align, so query design matters as the catalog grows. <a href="https://shopify.dev/docs/api/usage/search-syntax" target="_blank" rel="nofollow">Shopify's search syntax documentation</a> provides the supported grammar and performance considerations. This makes the workflow useful for recurring extracts where the operator wants a changed-products slice instead of a complete catalog every time.

<a href="https://tally.so/r/b5QYLL?platform=GitHub&amp;format=Product+repo&amp;brand=CogWorkLabs&amp;niche=automation&amp;page=Shopify+Product+Scraper+for+Admin+API&amp;date=2026-09-07" target="_blank" rel="nofollow">
  <img src="media/cdh-src-b7812fd594864354.gif" alt="CogWorkLabs — get a free demo">
</a>

## Handling API limits without losing a run

API limits are part of the extraction design, not an exception added after deployment. Shopify's GraphQL Admin API uses calculated query cost rather than a simple requests-per-minute counter. The standard documented limit is 100 cost points per second, with higher limits for some Shopify plans. Shopify can also return `429 Too Many Requests` when a throttle is applied. <a href="https://shopify.dev/docs/api/usage/limits" target="_blank" rel="nofollow">The API limits reference</a> recommends limiting calls, caching where appropriate, and retrying responsibly. The scraper therefore keeps requests bounded, requests only needed fields, observes pagination state, and treats throttling as a recoverable condition. A failed page must not be mistaken for an empty page. The output should only be considered complete after the pagination loop reaches a confirmed terminal state. That distinction is what separates a reproducible export from a CSV that merely looks finished.

## Project structure

The repository separates API access, transformation, export, configuration, and tests so a change to output formatting does not require rewriting request handling. The structure also gives operators a clear place to configure environment-specific values without putting credentials into tracked files.

```text
shopify-product-scraper/
├── src/
│   ├── api/
│   │   ├── client.py
│   │   ├── queries.py
│   │   ├── pagination.py
│   │   └── rate_limits.py
│   ├── transform/
│   │   ├── products.py
│   │   ├── variants.py
│   │   └── media.py
│   ├── export/
│   │   ├── csv_writer.py
│   │   └── json_writer.py
│   ├── config.py
│   └── main.py
├── tests/
│   ├── test_pagination.py
│   ├── test_transform.py
│   └── test_exports.py
├── .env.example
├── requirements.txt
└── README.md
```

## Running the finished build

The working project is intended to be run with the store credentials and extraction parameters supplied through its configuration. The operator does not need to modify the pagination logic or rewrite the GraphQL query for every run.

```bash
python -m src.main --format csv --output products.csv
```

- **STEP 1 — Get the Project**
Get **CogWorkLabs' shopify product scraper** as the working repository, configure its authorized store credentials, and keep secrets outside tracked files.
- **STEP 2 — Select the Input**
Open the configured run entry point and select the Shopify store plus the product fields, filter, and output format required for the extraction.
- **STEP 3 — Configure the Query**
Set the product search expression, page size, and destination filename. Leave cursor handling enabled so subsequent pages follow the previous response.
- **STEP 4 — Run and Export**
Start the extraction command. The process follows pagination, normalizes products and variants, then writes the completed dataset as CSV or JSON.

## Where the build fits

- Catalog operations can produce a structured product snapshot for spreadsheet review instead of manually copying titles, handles, variants, and media.
- Engineering teams can feed normalized JSON into another internal process when the source catalog needs to become an input to a separate automation.
- Merchandising workflows can isolate products by type or update time before export, reducing the amount of catalog data that needs downstream processing.
- Migration work can use the exported Shopify product dataset as a controlled intermediate representation before another system transforms or imports it.

## Practical limits and expected behavior

The build is constrained by the permissions and API behavior of the Shopify store it accesses. It does not bypass authentication, permissions, throttling, or platform restrictions. A GraphQL connection can request up to 250 resources in a page, after which cursor pagination or a bulk operation is required for larger result sets. Shopify documents bulk operations as the route for larger volumes when ordinary pagination is insufficient. The scraper also depends on the fields exposed by the API version and the scopes granted to its access token. That makes version-aware queries and explicit field selection preferable to relying on undocumented response shapes. For recurring exports, the safest operational pattern is to keep the query narrow, record the filter used for each run, and treat an export as successful only when every requested page has been processed and the final file has been written without transformation errors.

## How the pieces fit together

At the repository level, the system has a straightforward responsibility chain. The API layer authenticates and sends the GraphQL request. The pagination layer determines whether another page exists and carries the cursor forward. The transformation layer converts Shopify's nested product model into stable records. The export layer serializes those records into CSV or JSON. Configuration stays outside the extraction logic so a store-specific credential or output location can change without changing the core workflow. This separation also makes failures diagnosable: authentication errors belong to the access layer, throttling belongs to request handling, malformed product data belongs to transformation, and serialization problems belong to export. Shopify's GraphQL Admin API is explicitly designed around structured queries and typed product data, which makes this division a better fit than scraping rendered storefront HTML when authorized API access is available. The result is a small, inspectable automation whose main job is to move catalog data from a Shopify source into predictable records.

## FAQ

### Can the scraper collect variants and product images?

Yes. The product extraction can include variant fields and media associated with each product, then normalize those nested values into consistent output records. Shopify's Product model explicitly exposes variants and media as part of the product structure.

### How does the scraper handle large Shopify catalogs?

It uses cursor-based pagination and continues while Shopify reports another page. Shopify allows up to 250 resources in a connection request, so larger catalogs are divided into successive pages or handled through a bulk operation where appropriate.

### What happens when Shopify rate limits a request?

A throttle is treated as a recoverable API condition rather than an empty result. The request layer can pause and retry according to the platform's rate-limit behavior, while the run remains incomplete until all required pages have been processed.

### Can the output be filtered before it is exported?

Yes. Shopify's product query supports search expressions that can filter records before they are returned, including field comparisons, ranges, Boolean conditions, and exclusions. The resulting subset is then normalized and exported.