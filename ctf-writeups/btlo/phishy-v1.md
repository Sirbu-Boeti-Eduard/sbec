---
description: A static-analysis walkthrough of a phishing kit, from cloned-page attribution and resource discovery to credential-capture logic, hashing, and failure analysis.
---

# BTLO: Phishy V1

<!-- Original publication date: 2023-12-31 -->

## Introduction

![Phishy V1 investigation cover image](../../assets/images/btlo-phishy-v1-001.png)

Hello, my name is Sirbu-Boeti Eduard-Cristian and in this write-up I am going to cover the “Phishy V1” investigation on Blue Team Labs Online.

[**Blue Team Labs Online \| Phishy V1**<br><span>blueteamlabs.online</span>](https://blueteamlabs.online "https://blueteamlabs.online")

| Investigation detail | Description |
| --- | --- |
| Platform | Blue Team Labs Online |
| Investigation | Phishy V1 |
| Difficulty | Easy |
| Primary evidence | A phishing landing page, its HTML and CSS, an exposed kit archive, and a PHP credential handler |
| Focus areas | Page provenance, resource discovery, form-action analysis, file hashing, credential exfiltration, redirect behavior, and implementation errors |
| Tools demonstrated | Isolated browser, text editor, page source inspection, and `sha256sum` |

### Scenario

The lab presents a phishing URL and asks the analyst to examine the website, identify how the page was assembled, and recover intelligence about the kit and its operator-facing configuration.

> **Lab-safety note:** This write-up documents a historical, authorized BTLO environment. Phishing infrastructure and kit files must be treated as untrusted. Perform analysis in an isolated VM, use static copies wherever possible, never submit real credentials, and do not browse, download, or interact with third-party infrastructure without authorization. Potentially harmful URLs are defanged below.

## Investigation approach

The investigation follows the phishing page from its visible presentation to its underlying behavior:

1. Record the supplied URL and inspect the landing page without entering credentials.
2. Review HTML comments, source references, and stylesheets for provenance and infrastructure clues.
3. Trace the form action to the server-side credential handler.
4. Preserve and hash the exposed kit archive using the file itself as input.
5. Review the PHP and JavaScript statically to identify the recipient, redirect, timestamp logic, and coding error.

The key distinction throughout the analysis is between **what the page displays**, **what the source code instructs the browser to do**, and **what the server-side handler attempts to do with submitted data**.

## Phase 1: Establish page provenance and behavior

### Question 1: Where was the decoy page mirrored from, and which tool was used?

Opening the supplied lab artifact leads to a Microsoft-themed credential page hosted beneath the defanged path:

```text
hxxp://securedocument[.]net/secure/L0GIN/protected/login/portal/index1.html?<timestamp>
```

![Microsoft-themed phishing landing page presented by the supplied URL](../../assets/images/btlo-phishy-v1-002.png)

The domain root displays a generic hosting page rather than content associated with Microsoft. This difference suggests that the credential page was added beneath a subdirectory of an otherwise unrelated server.

![Generic page displayed at the root of the phishing domain](../../assets/images/btlo-phishy-v1-003.png)

The root page source contains an HTML comment inserted by HTTrack. The comment records both the original source and the mirroring tool:

![HTML comment identifying the original page and HTTrack Website Copier](../../assets/images/btlo-phishy-v1-004.png)

```text
61[.]221[.]12[.]26/cgi-sys/defaultwebpage.cgi
HTTrack Website Copier
```

[HTTrack](https://www.httrack.com/html/index) is an offline website-copying utility. Its generated comments can reveal that a page was mirrored, but this does not by itself identify who performed the copy or who deployed the resulting phishing kit.

> **Finding:** The decoy was mirrored from `61[.]221[.]12[.]26/cgi-sys/defaultwebpage.cgi` using `HTTrack Website Copier`.

### Question 2: What is the full URL of the landing page’s background image?

The phishing page source links to a local stylesheet:

![Landing-page source referencing its local stylesheet and form handler](../../assets/images/btlo-phishy-v1-005.png)

Within the stylesheet, the `body` rule loads `axCBhIt.png` with a relative URL:

![CSS body rule referencing axCBhIt.png as the background image](../../assets/images/btlo-phishy-v1-006.png)

Because the image reference is relative, the browser resolves it against the stylesheet’s directory. The server directory listing confirms that the image is stored alongside the page resources:

![Directory listing showing the phishing page resources](../../assets/images/btlo-phishy-v1-007.png)

> **Finding:** `hxxp://securedocument[.]net/secure/L0GIN/protected/login/portal/axCBhIt.png`

### Question 3: Which PHP page processes submitted credentials?

The HTML form’s `action` attribute identifies the endpoint that receives the form data:

![HTML form action pointing to the jeff.php credential handler](../../assets/images/btlo-phishy-v1-008.png)

```html
<form action="jeff.php" method="post">
```

This is stronger evidence than the filename alone: the form explicitly instructs the browser to send the submitted fields to `jeff.php` with an HTTP POST request.

> **Finding:** `jeff.php`

## Phase 2: Preserve and identify the phishing kit

### Question 4: What is the SHA-256 value of the ZIP-format phishing kit?

Directory indexing exposes an archive named `0ff1cePh1sh.zip`:

![Open directory index exposing the 0ff1cePh1sh.zip phishing kit](../../assets/images/btlo-phishy-v1-009.png)

In an authorized lab, the archive should be saved inside the isolated analysis environment and hashed without opening or executing its contents:

```bash
sha256sum 0ff1cePh1sh.zip
```

The command calculates the following SHA-256 digest from the ZIP file’s contents:

```text
c778236f4a731411ab2f8494eb5229309713cc7ead44922b4f496a2032fa5b48
```

BTLO requests the final six characters of the digest.

> **Finding:** SHA-256 suffix `fa5b48`

## Phase 3: Analyze credential collection and post-submission logic

### Question 5: Which email address is configured to receive captured credentials?

Static review of `jeff.php` shows how the kit attempts to assemble a message from submitted values. The `$recipient` variable contains the destination mailbox:

![PHP handler showing the credential fields, recipient, headers, and redirect](../../assets/images/btlo-phishy-v1-011.png)

```text
boris[.]smets@tfl-uk[.]co
```

This address is a configured collection endpoint within the kit. It is an investigative indicator, not sufficient evidence to attribute the infrastructure or campaign to a specific person.

> **Finding:** `boris[.]smets@tfl-uk[.]co`

### Question 6: Which function produces the value appended to `index1.html`?

The supplied URL contains a long numeric query value after the page name:

![Landing-page URL containing a numeric query parameter](../../assets/images/btlo-phishy-v1-012.png)

The page source shows that JavaScript constructs this value during navigation:

![JavaScript appending Date.getTime output to the landing-page URL](../../assets/images/btlo-phishy-v1-013.png)

```javascript
window.location = 'index1.html?' + new Date().getTime();
```

[`Date.prototype.getTime()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/getTime) returns a timestamp measured in milliseconds since the Unix epoch. Appending a changing value to the URL can prevent a browser or intermediary from serving a previously cached response.

> **Finding:** `getTime()`

### Question 7: Which domain should appear after credentials are submitted?

The captured browser response shows that the post-submission flow did not complete as intended:

![Broken response displayed after the phishing form was submitted in the lab](../../assets/images/btlo-phishy-v1-014.png)

There is no need to repeat the submission to determine the intended destination. The bottom of `jeff.php` contains an explicit client-side redirect:

![PHP handler containing a JavaScript redirect to office.com](../../assets/images/btlo-phishy-v1-015.png)

```javascript
window.location = 'https://www.office.com/';
```

Redirecting a victim to a legitimate site after collection can make the failed sign-in appear less suspicious. In this kit, a separate implementation error prevents the workflow from operating correctly.

> **Finding:** `office[.]com`

### Question 8: Which variable-name mismatch breaks the phishing kit?

The HTML form defines the submitted field names as `userrr` and `passss`:

![HTML input elements using the names userrr and passss](../../assets/images/btlo-phishy-v1-016.png)

The PHP handler attempts to retrieve different POST keys, `user1` and `pass1`:

![PHP handler reading user1 and pass1 instead of the submitted field names](../../assets/images/btlo-phishy-v1-017.png)

```text
HTML sends:  userrr, passss
PHP reads:   user1, pass1
```

POST parameter names must match exactly. Because the client and server use different names, the handler cannot retrieve the values placed in the form fields. This source-level comparison explains the broken behavior without requiring further interaction with the page.

> **Finding:** `userrr` / `passss` do not match `user1` / `pass1`.

## Investigation outcome

| Investigative question | Finding | Evidence source |
| --- | --- | --- |
| Mirrored-page source | `61[.]221[.]12[.]26/cgi-sys/defaultwebpage.cgi` | HTTrack-generated HTML comment |
| Mirroring tool | HTTrack Website Copier | HTML comment and generated-page metadata |
| Background image | `.../portal/axCBhIt.png` | CSS `background` declaration and directory listing |
| Credential handler | `jeff.php` | HTML form `action` attribute |
| Kit hash | `c778236f4a731411ab2f8494eb5229309713cc7ead44922b4f496a2032fa5b48` | `sha256sum` calculated from `0ff1cePh1sh.zip` |
| Credential recipient | `boris[.]smets@tfl-uk[.]co` | PHP `$recipient` assignment |
| Query-value function | `getTime()` | JavaScript redirect expression |
| Intended redirect | `office[.]com` | JavaScript in `jeff.php` |
| Breaking error | Form sends `userrr`/`passss`; PHP reads `user1`/`pass1` | Comparison of HTML inputs and PHP POST keys |

## Investigation limitations

- The findings are based on retained lab screenshots rather than a preserved copy of every source file and HTTP response.
- No DNS history, WHOIS data, TLS certificate data, server logs, or hosting-provider records were preserved for infrastructure correlation.
- The configured mailbox, IP address, filenames, and code comments are indicators. None independently establishes actor identity or campaign attribution.
- The infrastructure may no longer exist or may have changed ownership; the defanged values should not be reactivated for testing.

## Key takeaways

- Start with source inspection. HTML comments, relative paths, form actions, and script blocks often reveal more than the rendered page.
- Separate infrastructure observations from attribution. A configured address or copied page identifies a lead, not necessarily an operator.
- Preserve collected artifacts and record their cryptographic hashes so later analysis can confirm that the evidence has not changed.
- Compare client-side field names with server-side parameter access when a form workflow fails.
- Prefer static analysis of preserved artifacts over repeated interaction with suspected phishing infrastructure.

## Additional resources

- [CISA phishing guidance](https://www.cisa.gov/sites/default/files/2025-03/Phishing%20Guidance%20-%20Stopping%20the%20Attack%20Cycle%20at%20Phase%20One%20508.pdf)
- [HTTrack documentation](https://www.httrack.com/html/index)
- [GNU Coreutils manual](https://www.gnu.org/software/coreutils/manual/)
- [MDN: `Date.prototype.getTime()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/getTime)
