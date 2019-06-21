# Zero-Shot Transfer Learning for Event Extraction
## Setup Instruction
This repository is built on Theano (=0.8, verifed. Issues may be reported on recent versions), Python 2.7. The environment can be quickly set up by running 
```
pip install -r requirements.txt
```

## Run the Model

### Step 1
Preprocess Data: remove XML tags from ACE/ERE articles, sentence merging, sentence segmantation and tokenization.

```
java -jar removeXmlTag.jar [source_path] [source_without_tag_path]
```
```
python rsd2ltf.py [source_without_tag_path] [ltf_path] --extension .txt --seg_option nltk+linebreak
```
```
java -jar sentenceExtractor.jar [ltf_path] [sent_path] [sent_id_path]
```

 > [source_path]: the path for input files (all .sgm files from ACE source corpus)
 
 > [source_without_tag_path]: output path after merging sentences and removing xml tags from ACE source articles
 
 > [ltf_path]: output path after sentence segmentation and tokenization
 
 > [sent_path] [sent_id_path]: output paths for sentences and sentence ids  
 
 
### Step 2
Apply AMR parser (https://github.com/c-amr/camr, or https://github.com/jflanigan/jamr) to parse each single doc, and get aligned parsing output [amr_parsing_path].


### Step 3
Extract event mention structures from AMR parsing outputs
 
```
java -jar amrPostProcessing.jar [amr_parsing_path] [sent_id_path] [resource/amrRelationsAnnotated.txt] [resource/frameNetVN.txt] [trigger_structure_file]
```

 > [amr_parsing_path]: path of AMR parsing files generated from Step2
 
 > [sent_id_path]: path of sentence ids generated from Step1
 
 > [resource/amrRelationsAnnotated.txt]: path for resource/amrRelationsAnnotated.txt file
 
 > [resource/frameNetVN.txt]: path for resource/frameNetVN.txt file
 
 > [trigger_structure_file]: output file containing all event structures extracted from all the input files
 
 
### Step 4
Apply zero shot trigger extraction model
```
python zero_shot_final_test.py --test [trigger_structure_file] --test_result [trigger_prediction_file] --model_path [model_file] --embedding_path [embedding_file] --ontology_path [ontology_file] --norm_ontology_path [normalized_ontology_file] --seen_types [seen_type_file] --relation_path [file_of_amr_relations]
```
> To retrain the model, you can update the arguments in zero_shot_final.py file. The ACE training data can be generated from the data/sample/aceEventStructure.train.format.pos_neg.txt

> [trigger_structure_file]: input file containing all event structures generated from Step3

> [trigger_prediction_file]: output file containing all event structures and predicted types

> [embedding_file]: e.g., data/embedding/wsd.model.ace.filter.txt

> [ontology_file]: file with the whole target event ontology, e.g., if you use Framenet+ACE as the target ontology, it should be data/frame-ontology/event.ontology.new.txt

> [normalized_ontology_file]: e.g., data/frame-ontology/event.ontology.normalize.new.txt

> [seen_type_file]: file with all seen/training types, e.g., data/flags/train.10

> [file_of_amr_relations]: file with all meaningful AMR relations, e.g., data/resource/amrRelations.txt


### Step 5
Extract candidate arguments
```
java -jar prepareArgPrediction.jar [amr_parsing_path] [trigger_structure_file] [trigger_prediction_file] [resource/frame-ontology/event.ontology.new.txt] [arg_structure_file]
```

> [amr_parsing_path]: path of AMR parsing files generated from Step2

> [trigger_structure_file]: path of event structure file generated from Step3

> [trigger_prediction_file]: path of trigger prediction file generated from Step4

> [resource/frame-ontology/event.ontology.new.txt]: path to the file resource/frame-ontology/event.ontology.new.txt

> [arg_structure_file]: output file containing all argument structures


### Step 6
Apply zero shot argument extraction model
```
python zero_shot_arg_final_test.py --test [arg_structure_file] --test_result [arg_prediction_file] --model_path [model_file] --embedding_path [embedding_file] --ontology_path [ontology_file] --norm_ontology_path [normalized_ontology_file] --arg_path_file [argument_paths_file] --arg_path_file_universal [universal_argument_paths_file] --trigger_role_matrix [trigger_role_masks] --seen_argss [seen_argument_roles_file] --relation_path [file_of_amr_relations]
```

> [arg_structure_file]: argument structure file generated from Step5

> [arg_prediction_file]: output file containing all argument structures and argument role predictions

> [argument_paths_file]: list of all event types and argument roles, e.g., data/frame-ontology/event.ontology.args.merge.all.txt

> [universal_argument_paths_file]: list of all argument roles shared by all event types, e.g., data/frame-ontology/event.ontology.args.universal.txt

> [trigger_role_masks]: matrix to show each event types (index) with all predefined argument roles (index), e.g., data/frame-ontology/event.ontology.trigger.arg.matrix.new.txt

> [seen_argument_roles_file]: all seens/training argument roles, e.g., data/flags/train.arg.10


