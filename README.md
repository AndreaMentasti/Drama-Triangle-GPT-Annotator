# Drama-Triangle-GPT-Annotator
> Annotate text (tweets, news, transcripts) using GPT prompts to detect characters and roles.

## Why `{DT GPT Annotator}`?

Distributing a Python package for GPT-based annotation using the Drama Triangle framework would provide researchers and practitioners with a scalable, transparent, and reproducible way to analyze political and social narratives. Political and social debates often take place in messy formats like tweets, interviews, or news articles, where multiple voices compete and traditional tools struggle to capture the underlying dynamics. The Drama Triangle offers a parsimonious yet expressive schema, collapsing complex identities into the archetypal roles of **hero**, **villain**, and **victim**, which appear consistently across stories and media. By wrapping this logic into a package, users can systematically detect characters and their roles across large corpora, turning raw text into structured outputs that are immediately usable for statistical analysis. This makes it possible to study not only what topics are being discussed, but also how they are framed and who is assigned credit or blame. Such a tool would lower entry barriers for narrative analysis, enable comparability across topics and contexts, and open up new paths for understanding virality and polarization in political communication.

In practice, this package provides an interface to the OpenAI API for text classification tasks. Following *Gehring & Grigoletto (2025)*, it enables annotation of text snippets using the Drama Triangle framework, identifying characters and assigning them to predefined roles. 
- **Input:** small snippets of text (tweets, newspaper lines, TV transcript segments) + **prompts** describing instructions for the GPT calls.
- **Output:** a dataset with the original text and added columns indicating **which characters are present** and **their roles** (e.g., hero/victim/villain).

📄: This repository works as a guide for practitioners and researchers who want to analyze narratives in a different way than just counting keywords or identifying topics. By focusing on how characters are framed as heroes, villains, or victims, it enables systematic exploration of more complex dynamics in text data.

This repository outlines the key steps for implementing the framework: **1)** defining the topic of interest, **2)** selecting the extraction methodology, **3)** specifying the set of characters, and **4)** applying prompts for annotation. 
Each step is documented with guidance and examples, so that users can adapt the framework to their own texts and research questions.

---

## 1) Topic selection
🔍 A well-defined topic is a prerequisite for a fruitful narrative analysis. The clearer the topic, the more straightforward the identification of relevant characters and the exploration of the research question. When deciding which topic to select, the researcher must consider the research question, data availabilty, and the available resources. It is also important to consider the trade-off between specificity and generalizability. A very specific topic can make it difficult to define clear characters or identify enough relevant narratives, while a topic that is too broad may make it hard to narrow down the analysis to a manageable set of characters.

E.g. in *Gehring & Grigoletto (2025)*, we analyze the political economy of climate change. Starting from the relevant literature we found two dominant discussions: scientific evidence on climate change and policy responses. Since our focus lies with the domain of political economy, we concentrate on climate change policies, deliberately excluding debates on the scientific reality and predictability of climate change.

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

