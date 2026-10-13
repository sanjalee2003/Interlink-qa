# CV upload test files (InterLink QA)

All content is synthetic. "Nimal Perera" is a fictional candidate.

| # | File | Size (bytes) | Purpose | Expected (check against the code) |
|---|------|-------------:|---------|-----------------------------------|
| 01 | valid-cv.pdf | 2,124 | Valid PDF with text | Accepted, CV analysed |
| 02 | valid-cv.docx | 37,019 | Valid DOCX | Accepted, CV analysed |
| 03 | cv-exactly-5MiB.pdf | 5,242,880 | Boundary: exactly 5 MiB (5 x 1024 x 1024) | Accepted if limit is 5 MiB and inclusive |
| 04 | cv-5MiB-plus-1-byte.pdf | 5,242,881 | Boundary: 1 byte over 5 MiB | Rejected with a size message |
| 05 | cv-exactly-5000000-bytes.pdf | 5,000,000 | Boundary if "5 MB" means 5,000,000 | Accepted if limit is 5,000,000 |
| 06 | cv-5000000-plus-1-byte.pdf | 5,000,001 | 1 byte over 5,000,000 | Rejected only if limit is 5,000,000 |
| 07 | cv-image.png | 69,106 | Unsupported type (image) | Rejected |
| 08 | cv-program.exe | 1,024 | Unsupported type (harmless, not runnable) | Rejected |
| 09 | cv-notes.txt | 613 | Unsupported type (text) | Rejected |
| 10 | blank-cv.pdf | 1,308 | Valid PDF, no content | Clear message, no crash |
| 11 | scanned-cv.pdf | 83,159 | Image-only PDF, no text layer | Known limitation (no OCR): log as observation |
| 12 | exe-renamed-to.pdf | 1,024 | Wrong content, .pdf extension | Rejected if backend checks content |
| 13 | text-renamed-to.pdf | 30 | Plain text, .pdf extension | Rejected if backend checks content |
| 14 | empty-0-bytes.pdf | 0 | Empty file | Rejected |
| 15 | valid-cv-UPPERCASE.PDF | 2,124 | Extension case | Accepted (".PDF" is still a PDF) |

The padded PDFs (03-06) are real, valid PDFs with the same text as 01, so a rejection
can only be caused by size. Use 03/04 or 05/06 depending on which limit the code uses.

Test each file twice: through the UI (frontend check) and directly against the
upload API (backend check). A file blocked only by the frontend is a finding.
