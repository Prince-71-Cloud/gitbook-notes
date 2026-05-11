---
description: >-
  This note contains high-signal Google Dorks and UrlScan queries used for
  reconnaissance, sensitive data exposure, and OAuth attack surface mapping.
icon: bug
---

# Google Dorks & OAuth Recon Cheatsheet

### Sensitive Files & Data Exposure (Google Dorks) <a href="#sensitive-files-data-exposure-google-dorks" id="sensitive-files-data-exposure-google-dorks"></a>

```bash
site:domain.com ext:pdf
site:domain.com (ext:pdf OR ext:doc OR ext:docx OR ext:zip OR ext:bak OR ext:txt OR ext:ppt OR ext:pptx OR ext:xls OR ext:xlsx)
site:domain.com ext:xls
site:domain.com ext:xlsx
site:target.com intitle:"Index of /backup"
site:target.com inurl:"/wp-content/uploads/"
site:target.com intitle:"Index of /database"
site:target.com ext:git "target.com"
site:target.com inurl:"/.git/config"
site:target.com filetype:java "password"
site:target.com filetype:pem "PRIVATE KEY"
site:target.com filetype:ppk intext:"PuTTY"
site:target.com inurl:"id_rsa" filetype:key
site:target.com intext:"PHPSESSID="
site:target.com intext:"session_token="
site:s3.amazonaws.com "target.com"
site:github.com "target.com" ("password" OR "API_KEY")
site:docs.google.com "target.com confidential"
site:target.com filetype:xls "private"
site:target.com filetype:xls inurl:"contact"
site:target.com filetype:xls inurl:"email.xls"
site:target.com allinurl:"admin" mdb
site:target.com inurl:"email" filetype:mdb
site:target.com inurl:"backup" filetype:mdb
site:target.com inurl:"*db" filetype:mdb
site:target.com inurl:".env"
```

***

### Finding New HackerOne Programs <a href="#finding-new-hackerone-programs" id="finding-new-hackerone-programs"></a>

```bash
waybackurls hackerone.com | grep "embedded_submissions/new"
```

***

### OAuth Attack Surface Mapping (UrlScan) <a href="#oauth-attack-surface-mapping-urlscan" id="oauth-attack-surface-mapping-urlscan"></a>

```sh
domain:redacted.tld AND page.url:openid NOT domain:microsoftonline.com NOT domain:google.com
page.url:scope=api NOT domain:microsoftonline.com NOT domain:google.com
page.url:scope=full NOT domain:microsoftonline.com NOT domain:google.com
page.url:scope AND page.url:full NOT domain:microsoftonline.com NOT domain:google.com
page.url:scope AND page.url:access NOT domain:microsoftonline.com NOT domain:google.com
page.url:scope AND page.url:system NOT domain:microsoftonline.com NOT domain:google.com
page.url:scope AND page.url:backend NOT domain:microsoftonline.com NOT domain:google.com
page.url:scope AND page.url:internal NOT domain:microsoftonline.com NOT domain:google.com
page.url:scope AND page.url:intranet NOT domain:microsoftonline.com NOT domain:google.com
page.url:scope AND page.url:extranet NOT domain:microsoftonline.com NOT domain:google.com
page.url:scope AND page.url:admin NOT domain:microsoftonline.com NOT domain:google.com
page.url:scope AND page.url:database NOT domain:microsoftonline.com NOT domain:google.com
page.url:scope AND page.url:panel NOT domain:microsoftonline.com NOT domain:google.com
page.url:scope AND page.url:callback NOT domain:microsoftonline.com NOT domain:google.com
```

***

### OAuth High-Risk Scopes <a href="#oauth-high-risk-scopes" id="oauth-high-risk-scopes"></a>

```bash
page.url:scope AND page.url:delete NOT domain:microsoftonline.com NOT domain:google.com
page.url:scope AND page.url:write NOT domain:microsoftonline.com NOT domain:google.com
page.url:scope AND page.url:share NOT domain:microsoftonline.com NOT domain:google.com NOT domain:amazoncognito.com
page.url:scope AND page.url:upload NOT domain:microsoftonline.com NOT domain:google.com
page.url:scope AND page.url:file NOT domain:microsoftonline.com NOT domain:google.com
page.url:scope AND page.url:storage NOT domain:microsoftonline.com NOT domain:google.com
page.url:scope AND page.url:permission NOT domain:microsoftonline.com NOT domain:google.com
page.url:scope AND page.url:account NOT domain:microsoftonline.com NOT domain:google.com NOT domain:shopify.com
page.url:refresh_token NOT domain:microsoftonline.com
```

***

### OAuth Versioned Endpoints <a href="#oauth-versioned-endpoints" id="oauth-versioned-endpoints"></a>

```bash
page.url:scope AND page.url:v1
page.url:scope AND page.url:v2
page.url:scope AND page.url:v3
page.url:scope AND page.url:v4
```

***

### Finding Unique URLs (Google Dork Patterns) <a href="#finding-unique-urls-google-dork-patterns" id="finding-unique-urls-google-dork-patterns"></a>

```bash
site:domain.tld inurl:& inurl:keyword
site:domain.tld inurl:& inurl:= inurl:keyword
site:domain.tld inurl:& inurl? inurl:keyword
site:domain.tld inurl:& inurl? inurl:= inurl:keyword
```

***

### OAuth State Parameter Missing (CSRF Risk) <a href="#oauth-state-parameter-missing-csrf-risk" id="oauth-state-parameter-missing-csrf-risk"></a>

```bash
page.url:scope NOT page.url:state AND page.url:redirect_uri
page.url:scope NOT page.url:state AND page.url:auth
page.url:scope NOT page.url:state AND page.url:oauth
page.url:scope NOT page.url:state AND page.url:oauth2
page.url:scope NOT page.url:state AND page.url:authorize
page.url:scope NOT page.url:state AND page.url:client_id
page.url:scope NOT page.url:state AND page.url:response_type
```

***

### UrlScan OAuth Dork <a href="#urlscan-oauth-dork" id="urlscan-oauth-dork"></a>

```bash
page.url:response_type AND page.url:code NOT domain:google.com NOT domain:microsoftonline.com
```

***

### Learning from Real HackerOne Reports <a href="#learning-from-real-hackerone-reports" id="learning-from-real-hackerone-reports"></a>

```bash
site:hackerone.com "oauth"

# Open Redirect
site:hackerone.com "oauth" "open redirect"

# Misconfiguration
site:hackerone.com "oauth" "misconfiguration"

# Account Takeover
site:hackerone.com "oauth" "account takeover"

# Bypass
site:hackerone.com "oauth" "bypass"

# XSS
site:hackerone.com "oauth" "XSS"

# CSRF
site:hackerone.com "oauth" "CSRF"

# Chained Bugs
site:hackerone.com inurl:reports "oauth" "chain"
```

***

### Pro UrlScan Tips <a href="#pro-urlscan-tips" id="pro-urlscan-tips"></a>

```bash
domain:redacted.tld AND page.url:console
page.url:dashboard AND page.url:welcome
```

***

### Notes <a href="#notes" id="notes"></a>

* OAuth bugs often appear low-impact alone but become critical when chained
* Always manually verify UrlScan findings
* Reading disclosed reports dramatically improves real-world intuition
