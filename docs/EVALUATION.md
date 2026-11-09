# Evaluation

How I measured PDFM's detection numbers.

## How I tested

- Real malware was only handled inside an offline REMnux VM. PDFM never opens the files, it just
  parses them.
- I split the data into development sets (used to tune the rules) and test sets (only scanned once,
  after the rules were done). The test-set numbers are the ones I report.
- "Flagged" means the verdict was suspicious or malicious. Phishing PDFs usually end up as
  suspicious, since a bad link alone isn't proof, so flagged is the main detection number.
- All numbers come from `tools/benchmark.py DIR --label malicious` (or `--label benign`).

## Datasets

**MalwareBazaar** (abuse.ch), downloaded on 2026-11-06:

- `mb`: the 100 most recent PDFs (development)
- `mb2`: 300 more PDFs from the `pdf` tag (development)
- `mb3`: the remaining 394 PDFs from that tag (test)

MalwareBazaar's lists change over time, so the same queries will return newer samples later.

**CIC-Evasive-PDFMal2022** (Canadian Institute for Cybersecurity): `Benign.zip` and `Malicious.zip`
from their download page. Only used for testing.

**GovDocs1** (digitalcorpora.org), PDFs only:

- zip files `000`-`009`, 2,471 PDFs (development)
- zip files `010`-`019`, 2,558 PDFs (test)

I also used 397 of my own PDFs (textbooks, slides, manuals, forms) as a quick sanity check.

## Results

| Dataset | Files | Flagged | Rated malicious |
|---|---|---|---|
| CIC malicious | 21,835 | 98.5% | 54.1% |
| CIC benign | 9,106 | 10.9% | 0.07% (6 files) |
| MalwareBazaar `mb3` | 394 | 54.3% | 18.5% |
| GovDocs1 test | 2,558 | 2.2% | 0% |
| My own PDFs | 397 | 3.8% | 0% |

On the development sets, adding phishing detection raised the flagged rate from 29% to 49% on `mb`
and from 26% to 38% on `mb2`.

CIC's benign set has a lot of interactive forms with scripts, which is why more of it gets flagged as
suspicious than GovDocs1 does.

## False positives on CIC

Six benign CIC files were rated malicious. I left the rules as they were, since CIC was a test set:

- `PDFwithjar.pdf` drops a Java JAR when opened, which honestly looks malicious.
- `RichContentorFlash (4).pdf` has embedded Flash plus auto-running JavaScript.
- Two IRS forms contain TIFF images in XFA and match my CVE-2010-0188 YARA rule. That rule is too
  broad and should check for a malformed TIFF.
- Two IRS forms show Adobe's "update your Reader" pop-up next to a link, which my phishing check
  mistakes for a lure.

30 CIC files (0.1%) crashed the scanner because of a sandbox bug. They're left out of the numbers,
and the bug has since been fixed.

## What it still misses

- Phishing links to hacked websites that look normal
- Links to real services like SharePoint or Dropbox
- PDFs with no links or scripts at all, like "call this number" scams

A few MalwareBazaar "malicious" files were ordinary books, so the real detection rate is probably a
bit higher than measured.
