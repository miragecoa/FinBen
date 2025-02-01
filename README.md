## create your project fork from https://github.com/The-FinAI/FinBen

## github repo clone
git clone https://github.com/Yan2266336/FinBen.git --recursive

## set environment
conda create -n finben python=3.12
conda activate finben

cd FinBen/finlm_eval/
pip install -e .
pip install -e .[vllm]

## login to your huggingface
export HF_TOKEN="your_hf_token"
## verify it
echo $HF_TOKEN

## model evaluation
cd FinBen/
### for GPT model
lm_eval --model openai-chat-completions\
        --model_args "model=gpt-4o" \
        --tasks GRQAGen \
        --output_path results \
        --use_cache ./cache \
        --hf_hub_log_args "hub_results_org=TheFinAI,details_repo_name=lm-eval-results,push_results_to_hub=True,push_samples_to_hub=True,public_repo=False" \
        --log_samples \
        --apply_chat_template \
        --include_path ./tasks

### for small model
lm_eval --model hf \
        --model_args "pretrained=TheFinAI/FinLLaMA" \
        --tasks regAbbreviation \
        --num_fewshot 0 \
        --device cuda:1 \
        --batch_size 8 \
        --output_path results \
        --hf_hub_log_args "hub_results_org=TheFinAI,details_repo_name=lm-eval-finllama-regulation-results,push_results_to_hub=True,push_samples_to_hub=True,public_repo=False" \
        --log_samples \
        --apply_chat_template \
        --include_path ./tasks

### for large model
export VLLM_WORKER_MULTIPROC_METHOD="spawn"

lm_eval --model vllm \
        --model_args "pretrained=google/gemma-2-27b-it,tensor_parallel_size=4,gpu_memory_utilization=0.8,max_model_len=1024" \
        --tasks GRQA \
        --batch_size auto \
        --output_path results \
        --hf_hub_log_args "hub_results_org=TheFinAI,details_repo_name=lm-eval-results,push_results_to_hub=True,push_samples_to_hub=True,public_repo=False" \
        --log_samples \
        --apply_chat_template \
        --include_path ./tasks
        
lm_eval --model vllm \
        --model_args "pretrained=meta-llama/Llama-3.2-1B-Instruct,tensor_parallel_size=4,gpu_memory_utilization=0.8,max_length=8192" \
        --tasks GRFNS2023 \
        --batch_size auto \
        --output_path results \
        --hf_hub_log_args "hub_results_org=TheFinAI,details_repo_name=lm-eval-results,push_results_to_hub=True,push_samples_to_hub=True,public_repo=False" \
        --log_samples \
        --apply_chat_template \
        --include_path ./tasks
***results will be saved to FinBen/results/, you could add it into .gitignore***

## add new task
cd FinBen/tasks/your_project_folder/ # create yaml file for new task
#### cd FinBen/tasks/fortune/ # create yaml file for new task
#### lm-evaluation-harness/docs/task_guide.md # Good Reference Tasks


## if push to leaderboard
vim FinBen/aggregate.py **change to your project huggingface repos in line 415 and 421**

## new model
vim FinBen/aggregate.py # add new model configuration to MODEL_DICT
**https://huggingface.co/spaces/open-llm-leaderboard/open_llm_leaderboard#/?search=qwen2.5-1.5b-instruct**
**https://mot.isitopen.ai/**
python aggregate.py

## new task
vim FinBen/aggregate.py **add new task to METRIC_DICT # for classification task, change 1.0 / 6.0 to your baseline**
python aggregate.py
### huggingface leaderboard part
#### backend/app/services/leaderboard.py
#### frontend/src/pages/LeaderboardPage/components/Leaderboard/utils/columnUtils.js
#### frontend/src/pages/LeaderboardPage/components/Leaderboard/constants/tooltips.js
#### frontend/src/pages/LeaderboardPage/components/Leaderboard/constants/defaults.js
#### frontend/src/pages/QuotePage/QuotePage.js
