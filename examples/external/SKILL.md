<!-- external: true — managed by anthropic/mnt-skills, do not edit -->

# file-reading

Use this skill when a file has been uploaded but its content is NOT in
your context. This skill is a router: it tells you which tool to use for
each file type so you read the right amount the right way.

Triggers: any mention of an uploaded file path, an uploaded_files block,
or a user asking about a file you have not yet read.

Do NOT use this skill if the file content is already visible in your
context — you already have it.

## Dispatch

| Extension | First move |
|---|---|
| .pdf | pdfinfo then pypdf |
| .docx | pandoc to markdown |
| .xlsx | openpyxl sheet names + head |
| .csv | pandas with nrows=5 |
| .json | jq type then keys |
| .txt / .md | wc -c then head or cat |
| unknown | file then xxd head |

## General protocol

1. Check the extension
2. Stat before you read — large files need sampling not slurping
3. Read just enough to answer the question
4. If a dedicated skill exists for this file type, go read it

## Known limitations

- Image files are already in context as vision inputs — no disk read needed
- Legacy formats (.doc, .xls, .ppt) need conversion before the standard
  approach works — see dedicated skills for each
