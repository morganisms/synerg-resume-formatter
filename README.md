# Syner-G Resume Formatter

A single-file web app that puts any resume into a Syner-G Word template, or into a template of your own.

**Run it:** open the live version at https://morganisms.github.io/synerg-resume-formatter/, or download `index.html` and open it in a current browser. There's nothing to install, and files never leave your computer.

## How to use it

1. **Step one – add the destination template.** Choose **Syner-G Resume (photo)**, **Syner-G Resume (default)**, or **Select your own template** (a `.docx` or `.dotx` file; you can also drop it on the button).
2. **Step two – add the source resume.** Select or drop a PDF, Word or text file, or paste the text. The formatter reads the name, credentials, contact details and sections.
3. **Step three – push the button.** **Create Syner-G Resume (photo)** (or whichever format you chose) downloads a Word file named after the person and the format, for example `Alex Morgan - Resume (Syner-G default).docx`.

Below the steps, **Check the content** shows what was read, ready to correct, with a live **Preview** on the right. A second download button and **Start over** sit at the bottom.

## Built-in formats

| Format | Layout | What goes where |
|---|---|---|
| Syner-G Resume (photo) | Sidebar with photo placeholder | Contact details, education and certifications in the sidebar; credentials under the name; summary, core competencies (from skills), experience, publications and speaking engagements in the main column. |
| Syner-G Resume (default) | Single column, Syner-G footer | Name shown as first name plus last initial ("Alex M."), with credentials after it; the line under the name lists the two most recent job titles; summary, experience, education, certifications, skills, publications and honors. Contact details are left out, as the format has no place for them. |

Template sections with no matching content are removed, and resume sections with no matching template section are added at the end of the main column in the experience format. **See where each part of the resume goes** under step one shows this before you download.

Both templates are embedded in `index.html` (as inert base64 data blocks near the end of the file), so the buttons work with nothing to upload. The default template includes embedded fonts, which makes `index.html` about 6.5 MB; the generated Word files from that format are about 5 MB. To update a template, replace the text inside `<script type="text/plain" id="tpl-photo">` or `id="tpl-default"` with the output of `base64 -w0 new-template.dotx`.

## What the formatter reads from the resume

- **Name**: the person's name wherever it sits: the first line, a large title, a table at the top, the page header or a sidebar. All-caps names are converted to normal capitalization ("ALEX MORGAN" → "Alex Morgan"), and "Dr." is dropped.
- **Credentials**: degrees and certifications after the name ("Alex Morgan, Ph.D., PMP" or "Alex Morgan PhD"), or on their own line next to the name.
- **Contact details**: emails, phone numbers in any common format, LinkedIn and other web addresses, and locations. They are found on one line or many, with or without labels such as "Email:", with icons, and in the page header, footer or a contact sidebar. Each detail becomes its own line, without duplicates. LinkedIn links labeled only "LinkedIn" are turned into the real address.
- **Education**: school and degree are paired into one entry even when they're on separate plain lines. Dates can be at the start or end, and lines such as GPA or honors stay with their entry.

## Content syntax

Each section in *Check the content* is plain text, one item per line. Select one or more lines and click **Title / degree**, **Company / school**, **Bullet** or **Paragraph** above the box, or type the prefix yourself (the space after it is optional):

| Line starts with | Becomes | Example |
|---|---|---|
| `## ` | Entry heading: role, degree or certification, with dates at the end | `## Senior Program Manager \| Mar 2016 – Dec 2020` |
| `### ` | Second entry line, such as company and location | `### Helix Instruments, Emeryville, CA` |
| `- ` | Bullet | `- Took the platform to 510(k) clearance` |
| anything else | Paragraph | `Portfolio management, stage-gate governance` |

## Your own templates

Any Word resume or template works. The formatter reads its layout (including sidebars and text boxes), learns how each section and entry is formatted from the sample text, and fills it in place. Templates can also use placeholder tags:

Tags are case-insensitive and can sit in the body, header or footer.

| Tag | Fills with |
|---|---|
| `{{name}}` | Name |
| `{{contact}}` | All contact details on one line |
| `{{email}}`, `{{phone}}`, `{{location}}`, `{{linkedin}}`, `{{website}}` | One contact detail each |
| `{{summary}}`, `{{experience}}`, `{{education}}`, `{{skills}}`, `{{certifications}}`, `{{publications}}`, `{{projects}}`, `{{awards}}`, `{{volunteer}}`, `{{languages}}`, `{{interests}}` | That section's content, without its heading (put your own heading above the tag) |
| `{{section: Any Title}}` | The section with that exact title |
| `{{sections}}` | Every section not used by another tag, each with its heading |

A section tag works best alone in its own paragraph: that paragraph's formatting is used for the section's body text. Tags in a line with other text are filled inline.

## Tips and limits

- Text-based PDFs work best. Scanned PDFs have no text to read; use the Word version or paste the text.
- Letter-spaced text in PDFs (common in headings) is rejoined for standard section names; other letter-spaced lines may need a quick fix in the content step.
- A sidebar is a fixed-size box in Word. If the resume has more sidebar content than the template allows for, shorten it in the content step or resize the box in Word.
- Entry lines follow the template's order. If the template puts the company above the job title, the `###` line prints first, and the preview shows it that way.
- Section matching works by topic. To send a section somewhere else, rename it in the content step to match the template's section title.
- Older `.doc` files aren't supported. Save them as `.docx` first.
- Open the result in Word to check page breaks before sending it.

## Data

Files are read and built entirely in your browser; nothing is uploaded or stored. Only the light/dark choice is saved, in local storage. JSZip 3.10.1 loads from cdnjs with each page; PDF.js 4.10.38 loads from cdnjs only when a PDF is opened. Pasted text works without either.

## Security

- **Libraries are verified before they run.** JSZip loads with a Subresource Integrity hash. PDF.js 4 can't use one (it loads as a module), so the app downloads both PDF.js files, checks each against a pinned SHA-512 hash, and only then runs it. A file altered on the CDN is refused.
- **PDF.js is pinned to 4.10.38.** Versions before 4.2.67 are affected by CVE-2024-4367, which lets a crafted PDF run JavaScript in the page. PDFs are also opened with `isEvalSupported: false` as a second safeguard.
- **Content Security Policy.** Scripts run only from the page and cdnjs; the only network requests allowed are to cdnjs for PDF.js; no remote images, frames, plugins or form posts. Even if something went wrong, resume data has nowhere to be sent.
- **Untrusted files are handled defensively.** Uploads over 20 MB, Word files that would unzip to over 100 MB (zip bombs), and macro-enabled templates are refused. PDFs are read up to 20 pages, and each PDF's memory is released afterwards. Font names from templates are sanitized before use in the preview, and links written into the output must be well-formed web or email addresses.
- **Browser support.** PDF.js 4 needs a current browser (Chrome or Edge 119+, Firefox 121+, Safari 17.4+).
- **Updating a library.** Change `PDFJS_VERSION` and the two `PDFJS_SRI` hashes near the top of the script (or the JSZip `integrity` attribute). The hash is `sha512-` plus the base64 SHA-512 of the exact file, as listed on cdnjs.com. A wrong hash blocks PDF reading with a clear message rather than running unverified code.
