# Drama-Triangle-GPT-Annotator
> Annotate text (tweets, news, transcripts) using GPT prompts to detect characters and roles.

> **Status:** planning the project & collecting requirements.  
> **Goal:** a light, reproducible pipeline for topic/character extraction and role labeling.

---

## ✨ What this tool will do
- **Input:** small snippets of text (tweets, newspaper lines, TV transcript segments) + **prompts** describing what to annotate.
- **Output:** a dataset with the original text and added columns indicating **which characters are present** and **their roles** (e.g., critic/supporter/neutral).

---

## 1) Topic selection
A topic is the thematic lens you apply when annotating.  
Examples:
- *Climate policy debate in Italy (2023–2025)*
- *Media coverage of carbon taxes*
- *NGO vs. Government narratives on emissions*

**Checklist**
- [ ] Define scope (what’s in/out)  
- [ ] Time window & language  
- [ ] Sources (Twitter/X, newspapers, TV)  
- [ ] Example positives/negatives

---

## 2) Character schema
Characters are the entities you care about (people, orgs, parties, outlets).

**Schema draft**
- **`character_id`**: stable key (e.g., `it_min_env`, `Greenpeace`, `John_Smith`)  
- **`display_name`**: human-readable label  
- **`aliases`**: list of names/handles (e.g., `@matteoXYZ`, “Ministry of Environment”)  
- **`type`**: `person | org | party | outlet | other`  
- **`notes`**: disambiguation tips for GPT (“If ‘Minister’ + ‘Environment’ in Italy → this character”)  

> Store the schema in `data/characters.csv` or `data/characters.yaml` to keep prompts and results consistent.

---

## 3) Extraction methodologies
Different levels of ambition; choose one or combine:

### A. Presence detection (boolean)
> “Is character **X** present in this text, considering aliases and context?”

### B. Role classification
> “If character **X** is present, what is their role relative to the **topic**?  
> Options: `critic | supporter | neutral | none`.”

### C. Open-class discovery (optional)
> “List any **new characters** not in the schema (with short description).”

**Quality controls**
- deterministic prompts & temperature  
- evaluation sets with human labels  
- agreement metrics (precision/recall; Cohen’s κ)

---

## 4) Prompt design (draft templates)

### Presence (per character)


---

## 5) Input formats
Supported source types (planned):
- **CSV**: `text`, optional `id`, `source`, `timestamp`
- **JSONL**: one record per line with a `text` field
- **Parquet**: same columns as CSV

Put samples in `examples/`.

---

## 6) Output schema (planned)
A tidy table, one row per input text.

| column              | type     | description                                   |
|---------------------|----------|-----------------------------------------------|
| `id`                | string   | source id (optional; auto-generated if empty) |
| `text`              | string   | original snippet                              |
| `topic`             | string   | topic label used                              |
| `character_id`      | string   | from character schema                         |
| `is_present`        | boolean  | presence flag                                 |
| `role`              | string   | critic/supporter/neutral/none                 |
| `model`             | string   | model name used                               |
| `prompt_version`    | string   | hash/tag of prompt                            |
| `run_at`            | datetime | timestamp                                     |

---

## 7) Reproducibility
- **Version prompts** (store in `prompts/` with semantic version).  
- **Log models & params** (temperature, max tokens).  
- **Pin dependencies** (lockfile).  
- **Save intermediate responses** (cache).  

---

## 8) Roadmap
- [ ] Define topic & character schema  
- [ ] Create prompt set (presence + role)  
- [ ] Sample dataset & evaluation split  
- [ ] Minimal CLI for batch annotation  
- [ ] Metrics & error analysis notebook  
- [ ] First public release

---

## FAQ
**Q:** Does it store my texts on external servers?  
**A:** It will call an LLM provider to generate annotations. We’ll document options, caching, and redaction.

**Q:** Can it detect new characters automatically?  
**A:** Planned as an optional discovery step.

---

## Contributing
Discussions and issue templates will appear soon. For now, feel free to open an issue with ideas or edge cases.

## License
MIT (to be confirmed).

