# Bootstrapping NER annotations with a Large Language Model and a Knowledge Base

This folder contains a recipe which uses both an LLM and a KB to pre-annotate text with NER labels.
Whenever LLM annotations are not present in the KB or the labels do not correspond a warning is shown in the UI.
If the LLM annotations are successfully validated by the KB, the annotations can be automatically accepted reducing the number of examples that an annotator must curate.

This recipe is a customised (and simplified) version of the Prodigy [`ner.llm.correct`](https://prodi.gy/docs/recipes#ner-llm.correct) recipe.
It uses DBpedia Spotlight entity linker as a KB via [`spacy-dbpedia-spotlight`](https://github.com/MartinoMensio/spacy-dbpedia-spotlight) library. 

![](demo.png)

## Try it out 

To try this recipe make sure you have at least Prodigy 1.13 installed as well as `spacy-dbpedia-spotlight` (listed in `requirements.txt`).

Have some input data available, for example `news_headlines.jsonl` (found in the `example-datasets` folder).

Adapt the `spacy-llm` config file: `spacy.cfg` to use the LLM of your choice (make sure the name of the component is `llm`). Here you can find our [guide](https://prodi.gy/docs/large-language-models#getting-started-spacy-llm) on how to work with [`spacy-llm`](https://github.com/explosion/spacy-llm)

To start the server run (assuming the environment variables for the LLM API are in `.env` file):
```
dotenv run -- python -m prodigy ner.llm.kb.correct ner-annotated spacy.cfg news_headlines.jsonl -F ner_llm_kb_correct.py
```

Please note that the NER labels to be used will be sourced from the spaCy config provided (here `spact.cfg`)

When setting the `-V` flag, the examples that are successfully validated by the KB will be automatically accepted.

You can now curate the examples and flag the ones that require some further postprocessing. For example, you could use the [flag feature](https://prodi.gy/docs/api-web-app#flagging) to mark the examples that should be used for augmenting your custom KB.

DBpedia spotlight is a general domain knowledge base, but you can use your domain specific knowledge base in a similar way to even better constraint the LLM annotations and find new entities to feed back to your KB.

If you have an non-English LLM backend, you can also set DBpedia Spotlight accordingly by setting `-l` flag on the CLI. You can check the list of supported languages [here](https://github.com/MartinoMensio/spacy-dbpedia-spotlight#using-dbpedia-in-a-specific-language).

If you're interested in building a custom KB as a spaCy pipeline we recommend considering spaCy [`KnowledgeBase`](https://spacy.io/api/kb/#:~:text=The%20KnowledgeBase%20object%20is%20an%20abstract%20class%20providing,as%20its%20frequency%20in%20text%20and%20possible%20aliases.) API. If your KB is behing a REST API (just like [DBPedia Spotlight](https://www.dbpedia-spotlight.org/) here) that also can be built into a spaCy component. You can find some examples in the spaCy [docs](https://spacy.io/usage/processing-pipelines/#component-example3).

## Things to watch out for

The main objective of this recipe is to show how LLMs and KBs can be used in tandem to facilitate the annotation process, not to provide a generic, all-purpose solution. You will probably need to adjust it quite a lot to your use case and for that there are some caveates that should be highlighted upfront:

* Please note the usefulness of this workflow depends a lot on how good your entity linker is.
* Another assumption is that there is a correspondence between the LLM labels and the KB labels. This is the reason we use `Person` and `Organisation` and not `PER` and `ORG`. You need to get familiar with DBPedia types to extend/adjust it to other labels.
