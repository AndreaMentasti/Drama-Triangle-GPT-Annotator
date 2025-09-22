# Drama-Triangle-GPT-Annotator
> Annotate text (tweets, news, transcripts) using GPT prompts to detect characters and roles.

## Why `{DT GPT Annotator}`?

Distributing a Python package for GPT-based annotation using the Drama Triangle framework provides researchers and practitioners with a scalable, transparent, and reproducible way to analyze political and social narratives. Political and social debates often take place in messy formats like tweets, interviews, or news articles, where multiple voices compete and traditional tools struggle to capture the underlying dynamics. The Drama Triangle offers a parsimonious yet expressive schema, collapsing complex identities into the archetypal roles of **hero**, **villain**, and **victim**, which appear consistently across stories and media. By wrapping this logic into a package, users can systematically detect characters and their roles across large corpora, turning raw text into structured outputs that are immediately usable for statistical analysis. This makes it possible to study not only what topics are being discussed, but also how they are framed and who is assigned credit or blame. Such a tool would lower entry barriers for narrative analysis, enable comparability across topics and contexts, and open up new paths for understanding virality and polarization in political communication.

In practice, this package provides an interface to the OpenAI API for text classification tasks. Following *Gehring & Grigoletto (2025)*, it enables annotation of text snippets using the Drama Triangle framework, identifying characters and assigning them to predefined roles. 
- **Input:** small snippets of text (tweets, newspaper lines, TV transcript segments) + **prompts** describing instructions for the GPT calls.
- **Output:** a dataset with the original text and added columns indicating **which characters are present** and **their roles** (e.g., hero/victim/villain).

📄: This repository works as a guide for practitioners and researchers who want to analyze narratives in a different way than just counting keywords or identifying topics. By focusing on how characters are framed as heroes, villains, or victims, it enables systematic exploration of more complex dynamics in text data.

This repository outlines the key steps for implementing the framework: **1)** select and define the topic, **2)** identify the source and extract data, **3)** identify relevant characters, **4)** prepare the prompt, and **5)** obtain predictions and assemble outputs. 
Each step is documented with guidance and examples, so that users can adapt the framework to their own texts and research questions.

---

## 1) Topic selection 🔍
A well-defined topic is a prerequisite for a fruitful narrative analysis. The clearer the topic, the more straightforward the identification of relevant characters and the exploration of the research question. When deciding which topic to select, the researcher must consider the research question, data availabilty, and the available resources. It is also important to consider the trade-off between specificity and generalizability. A very specific topic can make it difficult to define clear characters or identify enough relevant narratives, while a topic that is too broad may make it hard to narrow down the analysis to a manageable set of characters.

E.g. in *Gehring & Grigoletto (2025)*, we analyze the political economy of climate change. Starting from the relevant literature we found two dominant discussions: scientific evidence on climate change and policy responses. Since our focus lies with the domain of political economy, we concentrate on climate change policies, deliberately excluding debates on the scientific reality and predictability of climate change.

---

## 2) Identifying data sources and extracting data💻 
After selecting the topic, the next step consists in gathering data. Common sources include digitized newspapers, social media, transcribed TV/radio/YouTube, and open ended survey responses. When selecting the data source, it is important to focus on the media channels where narratives about the chosen topic are most prominent. Regarding data extraction, the chosen data source will determine which methodologies can be used to retrieve the relevant text snippets.

E.g., our focus is on narratives about climate change policies in the United States, collected from the social media platform Twitter . We specifically choose the U.S. due to the significant role Twitter plays in shaping and disseminating political narratives there. The data collection process involves querying the Twitter historical APIv2 with a set of keywords adapted from *Oehl, Schaffer, and Bernauer (2017)*.

---

## 3) Identify relevant characters 👥 
The third step represents a critical stage in the pipeline, centered on identifying relevant characters within the topic. Character selection is primarily guided by the research question and the researcher's analytical focus. However, it is crucial to balance the scope of selection with practical considerations: expanding the set of characters too broadly can complicate prediction accuracy and increase computational demands, while a more focused selection ensures greater reliability and interpretability of results.\
Lastly, characters can be identified through various methods, including literature review, exploratory tools (topic modeling, entity recognition, RELATIO), and domain reading. 

E.g., guided by the relevant literature, exploratory tools, and intensive domain reading, we pre-specify ten characters: five *human characters* (made of institutions and groups of individuals) and five *instrument characters* (policy tools and instruments):

Human Characters: `DEVELOPING ECONOMIES | US DEMOCRATS | US REPUBLICANS | CORPORATIONS | US PEOPLE`

Instrument Characters: `EMISSION PRICING | REGULATIONS | FOSSIL INDUSTRY | GREEN TECH | NUCLEAR TECH`

---

## 4) Prompt design 📝 

The fourth step involves designing an effective prompt to instruct the AI model. Prompt engineering is a crucial part of the pipeline, as the quality of the prompt directly affects the accuracy and consistency of the model's responses. We recommend designing the prompt with the assistance of the same model that will later annotate the text. In this case we suggest to use GPT-40 via OpenAI's ChatGPT to be consistent with the model used in this Python package. 

This package allows to `a) mark a text as relevant` and `b) detect characters and assign them to a role`. 

#### a) Mark a text as relevant
E.g., in the paper we are interested in using only the tweets that actually contributes to the political discourse regarding climate change policies, hence we are not interested in keeping tweets that are irrelevant, that assert climate change, or that deny it. In the prompt engineering phase, we provide detailed instructions to the LLM to assign a tweet to one of the four categories: 

`irrelevant | assert | deny | relevant`

Here four examples of tweets assigned to the above labels:

| Tweet text                                                                                                                                 | Label              |
|--------------------------------------------------------------------------------------------------------------------------------------------|--------------------|
| She gimme hot head... I call it global warming                                                                                             | ❌irrelevant      |
| Global Warming is really real                                                                                                              | ❌assert          |
| It's funny, I've a friend who is more right winged than I am, but he still believes there's such thing as Global Warming.  Go figure.      | ❌deny            |
| The fact that we aren’t doing enough to challenge climate change effects EVERYBODY. But it effects Black people the most                   | ✅relevant         |

Below is a sample prompt configuration for classifying tweets about US climate change discourse. It defines a system role and instructs GPT to return a JSON output with a single key `"r"`.

```json
{
  "SYSTEM_MESSAGE": "You are an average US citizen. The user will provide the content of a tweet posted from the US.\nYour task is to analyze the tweet within the context of US political discourse, particularly in relation to climate change. Respond in JSON format.\n\n1. Relevance Check: Analyze the tweet in the context of US climate change discussion and determine its relevance. Provide one of the following values:\n   - 0 (irrelevant): If the tweet does not discuss climate change in a meaningful way. For example, if it only includes a hashtag (like #climatechange) or a passing reference but does not engage in any discussion about climate change or related policies, it should be considered irrelevant.\n   - 1 (assert): If the tweet asserts the existence of climate change but does not engage with specific policies or actions related to it. This includes tweets that acknowledge climate change as an issue without going deeper into details.\n   - 2 (deny): If the tweet denies the existence or severity of man-made climate change, referring to it as a hoax, scam, or fraud, or using sarcasm or language that undermines the reality of climate change.\n   - 3 (relevant): If the tweet discusses climate change or related policies in a substantive way. This includes any tweet that debates, critiques, or supports policies or actions related to climate change, as well as conversations on how to combat or adapt to climate change.\n\nRespond in JSON format, returning the value in the key \"r\".\n"
}
```

#### b) Detect characters and role
E.g., once selected the characters, the next step is to instruct the LLM to identify them in the texts, and frame them into one of the three (+ one) roles: `Hero | Victim | Villain | Neutral`. Notice that amongst the labels we also decided to include the Neutral role to account for contexts in which the character doesn't fit in any of the other roles.

Below is a sample prompt for character and role recognition in the case of US climate change discourse.

```json
{
    "SYSTEM_MESSAGE": "You are an average US citizen. The user will provide the content of a tweet posted from the US between 2010 and 2021. \nYour task is to analyze it within the context of US political discourse, particularly in relation to climate change and related policies. Respond in JSON format.\n\n1. Character Analysis: Identify whether the tweet mentions specific characters. For each character mentioned, assess their contextual role using the following scale:\n   - Villain (1): The character is portrayed as contributing to problems, opposing positive change, negatively or engaging in harmful actions related to climate change. Look for language that blames, criticizes, or attributes a negative impact.\n   - Hero (2): The character is portrayed as leading efforts to combat climate change, promoting environmental policies, positively or acting in a morally commendable way. Look for praise, leadership roles, or proactive efforts.\n   - Victim (3): The character is portrayed as being unfairly attacked, facing challenges, or suffering due to external factors. Look for language that depicts them as unjustly targeted, enduring consequences, suffering or being the victim.\n   - No role (4): Choose this option if the tweet mentions the character but does not clearly assign one of the above described roles, or if the context is ambiguous or neutral.\n\n2. Character Definitions: Evaluate these characters in the context of the tweet.\n   - For \"Developing Countries and Emerging Economies\" (emerging and developing economies, poorer countries, or nations part of the BRICS group -Brazil, Russia, India, China, South Africa-, as well as any related government institutions, representatives, or citizens associated with these countries and regions), provide the assessment in the key \"a\":\n      - 0: No mention of the character.\n      - 1: Villain.\n      - 2: Hero.\n      - 3: Victim.\n      - 4: None of the roles applies.\n   \n   - For \"The US Democrats\" (politicians, members, and public figures associated with the US Democratic Party. This includes prominent individuals such as Joe Biden, Nancy Pelosi, Alexandria Ocasio-Cortez, Bernie Sanders, Barack Obama, and others who represent ideals and policies of the Democratic party), provide the assessment in the key \"b\":\n      - 0: No mention of the character.\n      - 1: Villain.\n      - 2: Hero.\n      - 3: Victim.\n      - 4: None of the roles applies.\n   \n   - For \"The US Republicans\" (politicians, members, and public figures associated with the US Republican Party. This includes prominent individuals such as Trump, Mitch McConnell, Ted Cruz, Ron DeSantis, and others who represent ideals and policies of the Republican Party), provide the assessment in the key \"c\":\n      - 0: No mention of the character.\n      - 1: Villain.\n      - 2: Hero.\n      - 3: Victim.\n      - 4: None of the roles applies.\n   \n   - For \"Corporations and Industry\" (large corporations, small and medium businesses, banks, and other private sector entities. This includes CEOs and leadership figures like Elon Musk and Jeff Bezos, representatives from industries such as energy, technology, finance, and manufacturing, as well as industry lobbying groups, corporate interests, and small or local business owners), provide the assessment in the key \"d\":\n      - 0: No mention of the character.\n      - 1: Villain.\n      - 2: Hero.\n      - 3: Victim.\n      - 4: None of the roles applies.\n   \n   - For \"The People of the US\" (the collective citizens, voters, workers, youth, and grassroots movements of the United States, often portrayed in contrast to political elites or corporate interests. This also includes references to the 'average American,' and general terms like the public, society, community action, and any collective expression of public will or activism, including movements like FridaysForFuture, Sunrise Movement, and Extinction Rebellion), provide the assessment in the key \"e\":\n      - 0: No mention of the character.\n      - 1: Villain.\n      - 2: Hero.\n      - 3: Victim.\n      - 4: None of the roles applies.\n   \n   - For \"Emission Pricing Tools\" (any market-based instruments and schemes designed to price carbon emissions and incentivize reductions. This includes tools such as carbon taxes, cap and trade systems, carbon pricing, emission trading, carbon markets, pollution credits, carbon credits, carbon fees, and carbon dividends. These tools are part of the broader market-led response to addressing climate change), provide the assessment in the key \"f\":\n      - 0: No mention of the character.\n      - 1: Villain.\n      - 2: Hero.\n      - 3: Victim.\n      - 4: None of the roles applies.\n   \n   - For \"Banning or Regulation Policies\" (government actions, policies and movements aimed at combating climate change through the banning, phasing out, or strict regulation of specific products or industries. This includes efforts such as banning fracking, phasing out fossil fuels, banning single-use plastics, and other regulatory measures designed to reduce environmental impact and promote sustainability. It also encompasses movements that challenge economic growth models or advocate for systemic changes, such as de-growth movements and anti-capitalist environmental initiatives), provide the assessment in the key \"g\":\n      - 0: No mention of the character.\n      - 1: Villain.\n      - 2: Hero.\n      - 3: Victim.\n      - 4: None of the roles applies.\n   \n   - For \"Fossil Fuels\" (any explicit reference to fossil fuels, including terms like coal, oil, natural gas, and related critical labels such as 'dirty energy.' This encompasses all forms of energy derived from fossil sources, as well as the technologies and infrastructure that rely on them, such as combustion engines, power plants, and industrial machinery.), provide the assessment in the key \"h\":\n      - 0: No mention of the character.\n      - 1: Villain.\n      - 2: Hero.\n      - 3: Victim.\n      - 4: None of the roles applies.\n   \n   - For \"Green Technologies\" (technologies developed as a response to climate change or aimed at phasing out fossil fuels. This includes wind energy, solar energy, electric vehicles, hydrogen power, battery storage, geothermal energy, and other renewable or low-carbon technologies excluding though nuclear energy), provide the assessment in the key \"i\":\n      - 0: No mention of the character.\n      - 1: Villain.\n      - 2: Hero.\n      - 3: Victim.\n      - 4: None of the roles applies.\n   \n   - For \"Nuclear Energy\" (all forms of nuclear energy, including both fusion and fission technologies. This encompasses nuclear power plants, nuclear reactors, nuclear fusion research, and related technologies used for energy production), provide the assessment in the key \"j\":\n      - 0: No mention of the character.\n      - 1: Villain.\n      - 2: Hero.\n      - 3: Victim.\n      - 4: None of the roles applies.\n\n3. Final Output: Respond with a JSON format containing the following keys:\n   - \"a\": (0-4 as defined in step 2).\n   - \"b\": (0-4 as defined in step 2).\n   - \"c\": (0-4 as defined in step 2).\n   - \"d\": (0-4 as defined in step 2).\n   - \"e\": (0-4 as defined in step 2).\n   - \"f\": (0-4 as defined in step 2).\n   - \"g\": (0-4 as defined in step 2).\n   - \"h\": (0-4 as defined in step 2).\n   - \"i\": (0-4 as defined in step 2).\n   - \"j\": (0-4 as defined in step 2).\n"
}
```
---

## 5) Obtaining the predictions - DT GPT Annotator 📊 
The final step of the pipeline consists of obtaining predictions for each text. This is the step where this package comes at use. After designing the prompts, now the last step before the analysis is that of obtaining the predictions from the LLM. This process uses OpenAI API to process every text in the dataset with GPT-40-mini, applying the same prompt consistently across all observations. Conceptually, the annotation process can be understood as the OpenAI API acting as a classification engine, taking in specific inputs and generating structured outputs.

#### INPUTS:



#### OUTPUTS:

---

## 7) Reproducibility
- **Version prompts** (store in `prompts/` with semantic version).  
- **Log models & params** (temperature, max tokens).  
- **Pin dependencies** (lockfile).  
- **Save intermediate responses** (cache).  

---

## Contributing
Discussions and issue templates will appear soon. For now, feel free to open an issue with ideas or edge cases.

## License
MIT (to be confirmed).

