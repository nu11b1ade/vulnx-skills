# CLI reference — VulnX v2.0.2

Read only for setup, advanced queries, or output handling. The API's current `filters` response is authoritative for searchable fields; the installed CLI's help is authoritative for flags.

## Install and authenticate

If installation is needed, prefer the [official release binaries](https://github.com/projectdiscovery/vulnx/releases/latest) for the OS/architecture and verify the asset against the release checksums. Alternatively, with Go 1.24+ and an available toolchain:

```sh
go install github.com/projectdiscovery/vulnx/v2/cmd/vulnx@v2.0.2
vulnx version --disable-update-check
```

The pinned command reproduces this baseline. Use `@latest` only when explicitly installing the newest release, then recheck help. Ensure the Go bin directory (normally `$(go env GOPATH)/bin`, or `GOBIN` if configured) is on PATH.

Anonymous lookups are supported subject to service limits. For authenticated access, inherit `PDCP_API_KEY` from the user's secret configuration or let the user run `vulnx auth` in their terminal. Avoid embedding keys in command arguments, logs, or files. Do not start interactive auth inside an unattended agent. Use `vulnx auth --test` only when checking an already configured key; authentication does not guarantee unlimited quota.

## Queries

Append `--json --silent --disable-update-check` to these data commands when executing for an agent.

| Intent | Command |
| --- | --- |
| Discover query fields | `vulnx filters` |
| Product or vendor broad match | `vulnx search --product log4j --limit 20` |
| Exact vendor + product fields | `vulnx search 'affected_products.vendor:apache && affected_products.product:log4j' --limit 20` |
| Multiple products | `vulnx search --product nginx,tomcat --severity critical,high --limit 20` |
| Score thresholds | `vulnx search --product tomcat --cvss-score '>=8' --epss-score '>0.5' --limit 20` |
| PoC availability | `vulnx search --product log4j --poc --limit 20` |
| Without a template flag | `vulnx search --product log4j --template=false --limit 20` |
| Boolean field logic | `vulnx search '(apache || nginx) && severity:critical' --limit 20` |
| Calendar month | `vulnx search 'cve_created_at:>=2026-10-01 && cve_created_at:<2026-11-01' --limit 20` |
| Recent publication age | `vulnx search --vuln-age '<30' --limit 20` |
| Selected fields | `vulnx search --product log4j --fields cve_id,severity,cvss_score --limit 20` |
| Next page | `vulnx search --product log4j --limit 20 --offset 20` |
| Facets | `vulnx analyze --fields severity --query 'is_kev:true' --facet-size 10` |

`--product` searches both vendor and product fields. Use explicit fields to narrow that behavior, then verify the returned affected-product records and version ranges. `--vendor` is also a broad discovery convenience; do not treat convenience flags as exact installed-version matching. Use reported identifiers (`cve_id` or `doc_id`) to deduplicate search results.

Use comparison operators for dates, such as `>=YYYY-MM-DD` and `<YYYY-MM-DD`; a bare year equality is not a date range. Resolve “last month” to a calendar range and “last 30 days” to an age filter using the current date. Default search order is newest `cve_created_at` first; `--sort-asc` and `--sort-desc` are mutually exclusive.

For complete exports, hold query/sort constant, fetch bounded pages, advance the offset by the actual number of returned records, and deduplicate. Stop on an empty page, the reported total, a repeated page, a service limit, or a user budget. An API index can change mid-pagination, so report any gap or inconsistency. The CLI accepts `--limit` from 0 to 10000; 0 means server default, not unlimited. Server-side caps can be lower.

## JSON and export

```sh
# Writes a NEW JSON file; --output takes precedence over JSON stdout.
vulnx search --product log4j --limit 20 --output log4j-page-1.json --silent --disable-update-check

# Multiple IDs produce an array after deduplication; one distinct ID produces an object.
vulnx id CVE-2021-44228,CVE-2021-45046 --json --silent --disable-update-check

# Connectivity diagnostics.
vulnx healthcheck --disable-update-check
```

Search JSON has `results`, `count`, `total`, and optional `facets`. Filters JSON is an array. Analyze JSON is a search response with facet buckets; do not treat its minimal sample result as the complete matching dataset. Single-ID JSON is the vulnerability itself, not an API `data` wrapper. Multiple-ID JSON is an array of successful records; missing records require explicit reconciliation.

Search/analyze reject output paths without a `.json` suffix. ID/filters accept other suffixes but still write JSON. Preserve the raw JSON when exporting. If CSV is requested, parse `results` for search or normalize an ID object/array, select consistent columns, and use a CSV library. Do not split JSON with text tools or assume a `.csv` output filename enables CSV. Never overwrite a user's existing export to resolve an output error.

## Baseline evidence

- [Release v2.0.2](https://github.com/projectdiscovery/vulnx/releases/tag/v2.0.2), published 2026-07-09; latest stable checked 2026-11-01.
- Release source commit: `76410ddde266b7103673813e14a79dbad13b4beb`.
- [Command implementations](https://github.com/projectdiscovery/vulnx/tree/v2.0.2/cmd/vulnx/clis): `search.go`, `id.go`, `analyze.go`, `filters.go`, `common.go`.
- [Response types](https://github.com/projectdiscovery/vulnx/blob/v2.0.2/types.go) and [client](https://github.com/projectdiscovery/vulnx/blob/v2.0.2/client.go).

Prefer release source/help over README examples when they conflict. This skill intentionally has no wrapper, MCP server, background process, or bundled VulnX binary.
