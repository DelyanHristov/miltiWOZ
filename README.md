<h1 align="center">MultiWOZ Context-to-Response Evaluation</h1>

<h3 align="center">Standardized and easy to use Inform, Success, BLEU</h3>

<p>&nbsp;</p>

_______

Easy-to-use scripts for standardized evaluation of response generation on the [MultiWOZ benchmark](https://github.com/budzianowski/multiwoz). This repository contains an implementation of the MultiWOZ database with fuzzy matching, functions for normalization of slot names and values, and a careful implementation of the BLEU score and Inform & Succes rates. 

# :rocket: Usage

**Getting started:**

``` sh
git clone https://github.com/Tomiinek/MultiWOZ_Evaluation.git && cd MultiWOZ_Evaluation
pip install -r requirements.txt
```

**Running:**

``` sh
python evaluate.py [--bleu] [--success] [--richness] --input INPUT.json [--output OUTPUT.json]
```
Add `--bleu` to evaluate the BLEU score, `--success` to get the Success & Inform rate, and use `--richness` for getting lexical richness metrics such as the number of unique unigrams, trigrams, token entropy, bigram conditional entropy, corpus MSTTR-50, and average turn length. 


**Input format:**

``` python
{
    "xxx0000" : [
        {
            "response": "Your generated delexicalized response.",
            "state": {
                "restaurant" : {
                    "food" : "eatable"
                }, ...
            }, 
            "active_domains": ["restaurant"]
        }, ...
    ], ...
}
```
The input `.json` file should contain a dictionary with keys matching dialogue ids in the `xxx0000` format (e.g. `sng0073` instead of `SNG0073.json`), and values containing a list of turns. Each turn is a dictionary with keys:

- **`response`** – Your generated delexicalized response. You can use either the slot names with domain names, e.g. `restaurant_food`, or the domain adaptive delexicalization scheme, e.g. `food`.   
- **`state`** – **Optional**, the predicted dialog state. If not present (for example in the case of policy optimization models), the ground truth dialog state from MultiWOZ 2.2 is used during the Inform & Success computation. Slot names and values are normalized prior the usage.
- **`active_domains`** – **Optional**, list of active domains for the corresponding turn. If not present, the active domains are estimated from changes in the dialog state during the Inform & Success rate computation. If your model predicts the domain for each turn, place them here. If you use domains in slot names, run the following command to extract the active domains from slot names automatically: 

    ``` sh
    python add_slot_domains.py [-h] -i INPUT.json -o OUTPUT.json
    ```

See the [`predictions`](predictions) folder with examples.

**Alternative usage directly from your code.** First instantiate an evaluator and then call the `evaluate` method with dictionary of your predictions with the same format as describe above. Pseudo-code:

``` python
from multiwoz_evaluation.metrics import Evaluator
...

e = Evaluator(bleu=True, success=False, richness=False)
my_predictions = {}
for item in data:
    my_predictions[item.dialog_id] = model.predict(item)
    ...
    
results = e.evaluate(my_predictions)
print(f"Epoch {epoch} BLEU: {results}")
```


**Output format:**

``` python
{
    "bleu" : {'damd': … , 'uniconv': … , 'hdsa': … , 'lava': … , 'augpt': … , 'mwz22': … },
    "success" : {
        "inform"  : {'attraction': … , 'hotel': … , 'restaurant': … , 'taxi': … , 'total': … , 'train': … },
        "success" : {'attraction': … , 'hotel': … , 'restaurant': … , 'taxi': … , 'total': … , 'train': … },
    },
    "richness" : {
        'entropy': … , 'cond_entropy': … , 'avg_lengths': … , 'msttr': … , 
        'num_unigrams': … , 'num_bigrams': … , 'num_trigrams': … 
    }
}
```
The evaluation script outputs a dictionary with keys `bleu`, `success`, and `richness` corresponding to BLEU, Inform & Success rates, and lexical richness metrics, respectively. Their values can be `None` if not evaluated, otherwise: 

- **BLEU** results contain multiple scores corresponding to different delexicalization styles and refernces. Currently included references are DAMD, HDSA, AuGPT, LAVA, UniConv, and **MultiWOZ 2.2** whitch we consider to be the canonical one that should be reported in the future. 
- **Inform & Succes** rates are reported for each domain (i.e. attraction, restaurant, hotel, taxi, and train in case of the test set) separately and in total.
- **Lexical richness** contains the number of distinct uni-, bi-, and tri-grams, average number of tokens in generated responses, token entropy, conditional bigram entropy, and MSTTR-50 calculated on concatenated responses.

# 🏆 Results
Please see the [orginal MultiWOZ repository](https://github.com/budzianowski/multiwoz) for the benchmark results.


# :clap: Contributing

- **If you would like to add your results**, modify the particular table in the [original reposiotry](https://github.com/budzianowski/multiwoz) via a pull request, add the file with predictions into the `predictions` folder in this repository, and create another pull request here.
- **If you need to update the [slot name mapping](https://github.com/Tomiinek/MultiWOZ_Evaluation/blob/29512dec6df009e6b579a4aa8d26f8c1c6e85e35/normalization.py#L36-L55)** because of your different delexicalization style, feel free to make the changes, and create a pull request.
- **If you would like to improve [normalization of slot values](https://github.com/Tomiinek/MultiWOZ_Evaluation/blob/29512dec6df009e6b579a4aa8d26f8c1c6e85e35/normalization.py#L63-L254)**, add your new rules, and create a pull request.

# :thought_balloon: Citation
```
@inproceedings{nekvinda-dusek-2021-shades,
    title = "Shades of {BLEU}, Flavours of Success: The Case of {M}ulti{WOZ}",
    author = "Nekvinda, Tom{\'a}{\v{s}} and Du{\v{s}}ek, Ond{\v{r}}ej",
    booktitle = "Proceedings of the 1st Workshop on Natural Language Generation, Evaluation, and Metrics (GEM 2021)",
    month = aug,
    year = "2021",
    address = "Online",
    publisher = "Association for Computational Linguistics",
    url = "https://aclanthology.org/2021.gem-1.4",
    doi = "10.18653/v1/2021.gem-1.4",
    pages = "34--46"
}

```
