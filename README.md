# Client Questionnaire — Schumacher Lane, PLLC

A single self-contained HTML file. No server, no database, no dependencies.
Everything a client types stays in their own browser until they choose to
save a file.

---

## ⚠️ Read this first

**This repository is public.** GitHub Pages does not publish sites from
private repositories on the GitHub Free plan (that requires GitHub Pro or
higher). That is fine — the *form* is meant to be public — but it means:

> **Never commit a client's saved answers to this repository.**

A saved questionnaire file contains Social Security numbers, dates of birth,
account numbers and a full financial picture. Git keeps everything forever;
deleting a file in a later commit does not remove it from history. The
`.gitignore` in this repo blocks the obvious filenames as a safety net, but
the real safeguard is the habit: **client answers go in the matter file, not
in git.**

What this repo versions is the **form** — the questions, wording, and layout.
Not the responses.

---

## Publishing

1. Push this repo to GitHub.
2. **Settings → Pages → Source: Deploy from a branch → `main` / `/ (root)`.**
3. Wait about a minute. The site appears at
   `https://<your-username>.github.io/<repo-name>/`.

HTTPS is automatic and required — leave **Enforce HTTPS** checked. This
matters: over plain HTTP the page could be modified in transit, and an
injected script would see everything the client types.

### Custom domain

The form is served at **https://questionnaire.schumacherlane.com/**.

This is set up already. For the record, or if it ever has to be rebuilt,
the order matters — DNS record first, GitHub second, or the site goes dark
in between:

1. At the DNS provider (HostGator cPanel → Zone Editor), add a `CNAME`
   record: `questionnaire.schumacherlane.com.` → `derek94austin.github.io.`
2. Once that resolves, **Settings → Pages → Custom domain**, enter
   `questionnaire.schumacherlane.com`, Save. GitHub writes the `CNAME`
   file in this repo itself — do not hand-edit it.
3. GitHub issues the certificate automatically, usually within minutes.
   Then tick **Enforce HTTPS**.

This leaves the main schumacherlane.com site completely untouched.

---

## How the form is used

**Client side.** They open the link, work through the sections, and press
**Save my answers**. That downloads a single `.json` file to their device —
nothing is transmitted. They send that file to the office. There is also an
opt-in *Keep my progress in this browser* checkbox for clients filling it
across several sittings; it is off by default and warns about shared
computers.

**Office side.** Open the same page (or the OFFICE build, which additionally
has attorney-only fields), press **Open saved file**, choose the client's
`.json`, and every answer is editable. Save again to produce a corrected
version.

**Keeping a history of a client's answers.** Each save is named
`Questionnaire-<Name>-<YYYY-MM-DD>.json`. Keeping successive saves in the
matter folder gives you a dated trail of what the client said and when,
without any infrastructure. The most recent file is the current one.

---

## What the form does for you

**Completeness check.** On the review page, every question that applies and is
still blank is listed, grouped by section, each one a button that jumps
straight to it. Alongside it are consistency warnings — answers that are each
fine on their own but do not add up (a vehicle marked financed with no
creditor named, a joint filing with an empty spouse page, no creditors at
all). "Do not leave blanks" is the instruction on page 1 of the paper packet;
this is that instruction, enforced.

**Saving and printing.** **Save my answers** downloads a single `.json` file
immediately — no password step. The file is plain text: it holds Social Security
numbers, dates of birth and account numbers, so it is ordinary client data and
belongs in the matter file, not in email any longer than it has to be.

Files saved by an earlier version of this form carry a `.slq` extension and are
encrypted. **Open saved file** still unlocks those with the password that was set
at the time — the form can read them, it just no longer creates them.

**Download PDF** in the header jumps to the review page and opens the print
dialog; choosing *Save as PDF* as the destination gives the client a copy for
their records. The printed version is the answers only — the completeness check
and the copy buttons are screen tools and are suppressed in print.

**Sending it in.** The review page ends with a **Send to the office** button.
On a phone it hands the finished file straight to the share sheet with the
file already attached — one tap, no hunting in the Downloads folder. On a
computer, where browsers forbid a web page from attaching anything to mail,
it downloads the file and opens a pre-addressed email telling the client
exactly which file to attach. If the file was password protected, both paths
remind the client to phone the password through separately.

The firm's address and phone live in one place — the `FIRM` object at the top
of `02_helpers.js`. Point it at a dedicated intake mailbox rather than a
personal one if you have one.

**Office view.** A toggle on the review page switches from the client's
read-through to the same answers regrouped in the order the schedules ask for
them — identity, Schedule A/B, D, E, F, G, I, J, Form 122, then the Statement
of Financial Affairs. Every value has a copy button, each section has a
"copy section" button, and a checkbox reveals what the client left empty.
It does not eliminate re-keying into Jubilee, but it makes it a great deal
faster and harder to get wrong.

---

## Statement of Financial Affairs

The SOFA section follows **Official Form 107**, Parts 1 through 12, questions 1
to 28, in the official order and wording, so answers transfer to the filed form
line for line. It is split across four pages of the questionnaire:

| Page | Covers | Questions |
|---|---|---|
| Income & Past Addresses | Parts 1–2 | 1–5 |
| Payments & Lawsuits | Parts 3–4 | 6–12 |
| Gifts & Transfers | Parts 5–7 | 13–19 |
| Accounts & Business | Parts 8–12 | 20–28 |

Each question is a No / Yes pair; answering **Yes** opens the detail table
underneath it, exactly as the official form does. Part 12 shows the perjury
warning and asks the non-attorney-preparer question; the client signs the real
Form 107 at the office.

**Dollar amounts.** Question 6 carries the §547(c)(9) threshold, **$8,575** for
cases filed on or after 01 April 2025. It is adjusted every three years — the
next adjustment is due 01 April 2028. Search `8,575` in `index.html`; it appears
in the question text and in the hint beneath it.

An earlier version of this questionnaire used the superseded Form 7 numbering
(questions 1–17). Files saved against that version still open: `migrateData()`
carries the sixteen answers that map one-to-one onto Form 107 and leaves the
rest in the file untouched, so nothing is lost even where a person has to
re-key it.

---

## Editing the form

Every question is one line in a schema, not hand-written HTML. Open
`index.html` and search for the section you want — for example
`Household goods and furnishings` or `EXP_ROWS`. The field helpers read:

```js
T('fieldId', 'Label', width)          // text
M('fieldId', 'Label', width)          // money
YN('fieldId', 'Question?', width)     // Yes / No
RAD('fieldId','Question?',['A','B'])  // pick one
TA('fieldId', 'Label')                // long answer
```

`width` is out of 12 columns. Adding a question is adding a line; removing
one is deleting a line. Conditional visibility is `showIf`, and any field
referenced by a `showIf` is wired to re-render automatically.

After editing, commit and push — GitHub Pages redeploys on its own, and the
commit history gives you a record of what the questionnaire looked like on
any given date.

---

## Two builds

| File | Attorney fields | Browser autosave | Use |
|---|---|---|---|
| `index.html` (WEB) | removed at build time | opt-in | published to clients |
| OFFICE build | present, behind *Attorney mode* | none | internal, kept off the web |

The attorney block is not merely hidden in the web build — it is deleted from
the file, so it cannot be revealed from the browser console.
