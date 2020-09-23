# Hebrew Contextual Relevance Model For UMLS Entity Linking Using BERT
Linking Hebrew terms to Unified Medical Language System (UMLs) entities to benefit text-analytics methods,  make information retrieval and analysis 
easier and more accurate.


## Our approach
We utlize **BERT** (Bidirectional Encoder Representations from Transformers), a technique for natural language processing (NLP) pre-training developed by Google.
Its key technical innovation is applying the bidirectional training of Transformer, a popular attention model, to language modelling.
This is in contrast to previous efforts which looked at a text sequence either from left to right or combined left-to-right and right-to-left training.

## Hypothesis
Observation of the [MDTEL](https://github.com/yonatanbitton/mdtel)'s High Recall Matcher (HRM) output shows that in many cases it chooses candidate terms which are not actual UMLS entities, for example:

given the post:

`
ברוכים הבאים לפורום דיכאון וחרדה  כאן תוכלו לשאול  להתייעץ וגם לייעץ בנושאי חרדה ודיכאון  עצב ותחושות נפשיות  מטרת הפורום בין היתר היא לסייע לחולים כרוניים באתר כמוני הנמצאים במצבי דיכאון וחרדה כתוצאה מהמצב הרפואי בו הם נתונים  ובעיקר שיהיה לבריאות מערכת כמוני
`

the HRM output contains the following match:
```
{"cand_match": "מוני", "umls_match": "מוני", "sim": 1.0, "cui": "C0590695", "match_eng": "Monit", "hebrew_key": "segmented_text", "match_tui": "T109", "semantic_type": "Chemical or drug", "all_match_occ": [162, 256], "curr_occurence_offset": 162}
```

the candidate match 'מוני' is mapped to UMLS 'Monit' which is a drug for chest pains. It was extracted from the word 'כ**מוני**' which is the name of the online forum.

Intuitively, we want to use the context of the word in the post to get better indication of whether or not this is an actual medical term or not. In the above case, 'מערכת כמוני' can imply that 'כמוני' is a type/name of a system and not a drug.

Using N-grams for context analysis, i.e.- combinations of one or more words that represent entities, phrases, concepts, and themes that appear in the text. The exact process we chose id described in the _Training data construction_ section.

## Training data construction
We use the output from HRM and the manual annotations, which can be found [here](https://drive.google.com/file/d/17JTxutH15P3R-Wd4x3d5ulY22KW0vVUC/view?usp=sharing) and attempt to use a contextual relevance language model to improve
the results of the tagged UMLS entities.

For each entry in the data (post), we go over the matches found and do the following:

1) <u>n-grams</u><br>
Using the offset of the term we find the original word from the text and create a window around it (depending on the chosen n-gram value) _to get the word context_ of the term (n words to the left of the term and n words to the right). 

2) <u>UMLS from HRM</u><br>
We collect the UMLS from the HRM output (under 'umls_match')

3) <u>UMLS from manual annotations</u><br>
We collect the term tagged in the mannual annotations that corresponds to the HRM match (using the offset) and look it up in the UMLS database to validate it's an actual UMLS (since in some cases it's not), so we take either the UMLS match or `Nan`.
<br><br>
_One-time runtime step: fix annotations data issues:_<br>
If there is no UMLS for the annotations' match, then we want to either insert the HRM UMLS instead or leave it as `Nan`. This is done during run time using a manual user-input method, where the user decides based on the context-window if the HRM's UMLS matches (`1` or `0` input). This process was done once and then we saved the fixed file separately (can be found [here](training_data/output_data/corrected_annotations.json)) and used it instead of the original manual annotations file. The process' [code](training_data/main.py) is commented out but can be repeated if necessary.

## Results
xxxxxx

## Acknowledgements
+ Yonatan Bitton's [MDTEL](https://github.com/yonatanbitton/mdtel) work.
+ [Google's Multilingual BERT](https://github.com/google-research/bert/blob/master/multilingual.md).
