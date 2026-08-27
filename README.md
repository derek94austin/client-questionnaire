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

### Custom domain (optional)

To serve it at `intake.schumacherlane.com`:

1. Create a file named `CNAME` in this repo containing exactly:
   `intake.schumacherlane.com`
2. At your DNS provider, add a `CNAME` record pointing
   `intake` → `<your-username>.github.io`
3. GitHub issues the certificate automatically within a few minutes.

This leaves your WordPress site completely untouched.

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
