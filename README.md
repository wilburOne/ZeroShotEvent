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
python zero_shot_final_test.py --test [trigger_structure_file] --test_result [trigger_prediction_file]
```
> To retrain the model, you can update the arguments in zero_shot_final.py file. The ACE training data can be generated from the data/sample/aceEventStructure.train.format.pos_neg.txt

> [trigger_structure_file]: input file containing all event structures generated from Step3

> [trigger_prediction_file]: output file containing all event structures and predicted types


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
python zero_shot_arg_final_test.py --test [arg_structure_file] --test_result [arg_prediction_file]
```

> [arg_structure_file]: argument structure file generated from Step5

> [arg_prediction_file]: output file containing all argument structures and argument role predictions


