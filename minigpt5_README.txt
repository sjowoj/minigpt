#config change
1.base llm weight change:/MiniGPT-5/minigpt4/configs/models/minigpt4.yaml
2.finetuned minigpt ckpt change:/MiniGPT-5/config/minigpt4.yaml
WEIGHT_FOLDER:/MiniGPT-5/ckpt

#code change(if function 'from_pretrained' can't download the checkpoints from huggingface and copy those to a specific dir i.e. /ckpt)
1.  /MiniGPT-5/minigpt4/models/blip2.py 'init_tokenizer' and 'init_Qformer' the args of 'from_pretrained' should be replaced by above dir
2.  /MiniGPT-5/model.py 'MiniGPT4Args'  if use demo 'cfg_path' should begin with "../", else should begin with "./"
3.  /MiniGPT-5/model.py 'MiniGPT5_Model' __init_ if can't download the stable_diffusion model,then the 'sd_model_name' should be changed to the dir to the your own dir 


#use demo to test minigpt5
cd examples
export IS_STAGE2=True
#in this docker environment, the following command could be replaced by 'demo'
#just typewritting 'demo' and then enter
python3 playground.py --stage1_weight WEIGHT_FOLDER/stage1_cc3m.ckpt 
                        --test_weight WEIGHT_FOLDER/stage2_vist.ckpt or stage2_mmdialog.ckpt


#stage1 evaluation
export IS_STAGE2=False
export WEIGHTFOLDER=WEIGHT_FOLDER
export DATAFOLDER=datasets/CC3M
export OUTPUT_FOLDER=outputs

CUDA_VISIBLE_DEVICES=N(if needed)  python3 train_eval.py --test_data_path cc3m_val.tsv 
                        --test_weight stage1_cc3m.ckpt
                        --gpus 0

export CC3M_FOLDER=datasets/CC3M
python3 metric.py --test_weight stage1_cc3m.ckpt


#stage2 evaluation
export IS_STAGE2=True
export WEIGHTFOLDER=WEIGHT_FOLDER
export DATAFOLDER=datasets/VIST
export OUTPUT_FOLDER=outputs
python3 train_eval.py --test_data_path val_cleaned.json 
                        --test_weight stage2_vist.ckpt
                        --stage1_weight stage1_cc3m.ckpt
                        --gpus 0

python3 metric.py --test_weight stage2_vist.ckpt
(in such stage,'vist' could be replaced by 'mmdialog')

#stage1 training
export IS_STAGE2=False
export WEIGHTFOLDER=WEIGHT_FOLDER
export DATAFOLDER=datasets/CC3M
python3 train_eval.py --is_training True
                        --train_data_path cc3m_val.tsv
                        --val_data_path cc3m_val.tsv
                        --model_save_name stage1_cc3m_{epoch}-{step}
                        --gpus 0

#stage2 training
export IS_STAGE2=True
export WEIGHTFOLDER=WEIGHT_FOLDER
export DATAFOLDER=datasets/VIST
python3 train_eval.py --is_training True
                        --train_data_path val_cleaned.json
                        --val_data_path val_cleaned.json
                        --stage1_weight stage1_cc3m.ckpt
                        --model_save_name stage2_vist_{epoch}-{step}
                        --gpus 0
(the same as stage2 evaluation, 'vist' could be replaced by 'mmdialog')

