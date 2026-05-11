---
description: >-
  This note documents a simple but effective workflow for subdomain enumeration
  and DNS analysis during reconnaissance
icon: bug
---

# Subdomain Enumeration Workflow

### Step 1: Subdomain Enumeration <a href="#step-1-subdomain-enumeration" id="step-1-subdomain-enumeration"></a>

Use multiple tools to maximize coverage.

### subfinder <a href="#subfinder" id="subfinder"></a>

```bash
subfinder -d target.com -all -recursive -o subs.txt
```

### assetfinder <a href="#assetfinder" id="assetfinder"></a>

```bash
echo target.com | assetfinder -subs-only > assetfinder.txt
```

### findomain <a href="#findomain" id="findomain"></a>

```bash
findomain -t target.com --quiet | tee findomain.txt
```

### Step 2: Combine and Deduplicate Results <a href="#step-2-combine-and-deduplicate-results" id="step-2-combine-and-deduplicate-results"></a>

Merge all outputs and remove duplicates.

```bash
cat *.txt > all-subs.txt
sort -u all-subs.txt > final-subdomains.txt
```

### Step 3: DNS Analysis with dnsx <a href="#step-3-dns-analysis-with-dnsx" id="step-3-dns-analysis-with-dnsx"></a>

Scan subdomains for DNS misconfigurations such as **SERVFAIL** and **REFUSED** responses.

```bash
dnsx -json -rcode servfail,refused -trace -t 500 -ns -r resolvers.txt -l final-subdomains.txt -o dnsx-output.json
```

### Step 4: Extract Takeover Candidates <a href="#step-4-extract-takeover-candidates" id="step-4-extract-takeover-candidates"></a>

Parse the dnsx JSON output to extract hosts and their nameserver chains.

```bash
cat dnsx-output.json | jq -r '"\(.host) \(.trace.chain[-2].ns[])"' | sort -u > dns-takeover-candidates.txt
```

### Output Files <a href="#output-files" id="output-files"></a>

*   ```
    final-subdomains.txt
    ```

    — clean list of unique subdomains
*   ```
    dnsx-output.json
    ```

    — DNS trace and response data
*   ```
    dns-takeover-candidates.txt
    ```

    — potential DNS takeover targets

### Notes <a href="#notes" id="notes"></a>

1. Always use multiple **resolvers** for accuracy
2. **SERVFAIL/REFUSED** responses can indicate misconfigured or dangling DNS records
3. **Manually verify** takeover candidates before reporting
