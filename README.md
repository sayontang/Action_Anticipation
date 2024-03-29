# Action_Anticipation

- ## Teacher training
  1. Copy the following directory /home/sayontan/Action_anticipation/  .The directory I am referring here is in the context (NLP-grp)/AI-DGX server.
      ```
        data  
        └───50_salad_action_seq/
        └───egtea_action_seq/ 
        └───epic55_action_seq/
      ```
  2. To train EGTEA+ 
    ```
      python ./code/egtea_finetuning.py \
      -model_type bert \ # bert/roberta/distillbert/alberta/deberta/electra
      -batch_size 16 \   # batch-size
      -num_epochs 5 \    # no. of epochs
      -max_len 512 \ # Max # of tokens in the input, tokens beyond this number will be truncated
      -checkpoint_path ./out/bert_pretrained/checkpoint-200000 \ # path to the model checkpoint (for initialization) that was trained on 1M-Recipe through MLM
      -weigh_classes True/ #for imbalanced data, if True, then the loss will be weighted Cross-Entropy; will not work with EPIC-55 as few clases have 0 data instances
      -hist_len 15 \ # context length, i.e. how many actions in the past conditioned on which you want to predict the action after the anticipation time
      -gappy_hist True \ # EGTEA data (action sequence) have gaps as action segments in a video are partiioned into train/test set
      -multi_task True \ # Instead of just predicting the action, also predict the verb and noun
      -sort_seg True \ # Action segnment in the training batch should be sorted by their temporal order ?
    ```
  2. To train EPIC-55 
    ```
      python ./code/epic55_finetuning.py \
      -model_type distillbert \
      -batch_size  16 \
      -num_epochs 10 \
      -max_len 512 \
      -hist_len 5 \
      -checkpoint_path ./out/distilbert_pretrained/checkpoint-200000 \
      -weigh_classes False \
      -multi_task True
    ```
    
    3. Directory of pre-trained LM, which are used for initlaizing the models for teacher training: 
    ```
        ./out/<model>_pretrained/checkpoint-200000
        model = bert/albert/distilbert/electra/roberta
    ```
      
    4. Basic steps: 
      For each action (query) segment in the video, identified by a uique UID, there is a sequence of sequence of action segments (history of the query UID). 
      The sequence of text associated with the history-UID sequence is the input for the LM, and the label of the query UID is the target. The task for the teacher (LM       based classifier) is the learn this mapping (ip  -> distribution over action classes for the query UID). This distribution generated by the teacher is then used      by the student as a lable prior for learning the task of action anticipation from Video.
      
    5. I've left out the code for LM pre-training/action extraction from 1M-Recipe as they might be irrelevant for the time being.
    6. **TODO:** Add many-shot class mean recall computation metric in the *compute_metric()* function in the code. 
