---
title: "finetuning Llama3.2 1B"
date: 2024-10-18T12:00:00Z
draft: false
---

# Status Update Log

## v0.0.1 - Initial Setup and Project Kickoff

- Created the project directory structure and initialized version control.
- Set up the basic environment using Python 3.12 and installed the required libraries for Transformer-based model training, including PyTorch and HuggingFace Transformers.

## v0.0.2 - Dataset Preparation

- Selected the SQuAD dataset for fine-tuning the model.
- Loaded the dataset and performed initial data cleaning and formatting.
- Created scripts for dataset exploration to understand question-answer distribution and context.

## v0.0.3 - Model Selection

- Chose the LLaMA-3.2-1B model as the base for fine-tuning due to its capabilities with language modeling tasks.
- Downloaded and set up the pre-trained model in the development environment.
- Verified model loading and initial inference to ensure correct setup.

## v0.0.4 - Experimentation with Quantization

- Tested quantization techniques to fit the LLaMA model in available GPU memory.
- Implemented 8-bit quantization using `BitsAndBytesConfig` to reduce memory footprint.
- Quantization helped in reducing memory usage and allowed the model to run smoothly on the g4dn.xlarge instance.

## v0.0.5 - Gradient Checkpointing and Mixed Precision

- Enabled gradient checkpointing to save memory during backpropagation by recalculating activations on-the-fly.
- Utilized mixed precision training (fp16) to further optimize memory usage.
- Both approaches combined allowed training without running into memory limitations.

## v0.0.6 - Parameter-Efficient Fine-Tuning (PEFT) Setup

- Implemented PEFT with LoRA to add trainable adapters to the LLaMA model.
- Used the `peft` library for seamless integration of LoRA adapters.
- Ensured that the adapters effectively reduced the number of trainable parameters while maintaining model performance.

## v0.0.7 - Custom GPU Memory Allocation Checks

- Encountered discrepancies between reported and actual available memory on the GPU instance.
- Wrote custom scripts to monitor GPU memory allocation and identify bottlenecks.
- Adjusted model configurations based on insights to ensure efficient use of GPU resources.

## v0.0.8 - Optimizer and Training Configuration

- Implemented `paged_adamw_8bit` optimizer to offload optimizer states to CPU, reducing GPU memory requirements.
- Introduced gradient accumulation to simulate larger batch sizes without increasing memory usage.
- Configured learning rate schedules and optimizer settings to suit the training requirements of LLaMA-3.2-1B.

## v0.0.9 - Initial Training Attempt and CUDA Out of Memory (OOM) Errors

- First training attempt resulted in CUDA OOM errors.
- Identified causes related to batch size and learning rate settings.
- Made adjustments to the batch size and added a configuration to handle memory fragmentation (`PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True`).

## v0.1.0 - Preventing System Hibernation

- Prevented the system from entering sleep, suspend, or hibernation states to avoid interruptions during training.
  ```bash
  sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
  ```
- Added symlinks for `/etc/systemd/system/sleep.target`, `/suspend.target`, `/hibernate.target`, and `/hybrid-sleep.target` to `/dev/null`.

## v0.1.1 - Iteration Speed Improvements

- Increased training iteration speed from 10s/iteration to 15s/iteration.
- Adjusted gradient checkpointing settings to reduce computation overhead.
- Achieved more stable training runs by reducing memory consumption through optimized batch sizes and accumulation steps.

## v0.1.2 - Learning Rate Finder and Checkpoint Issues

- Initiated the learning rate search process to find an optimal rate for model convergence.
- Received warnings about incompatible settings (`use_cache=True`) with gradient checkpointing; resolved by switching to `use_cache=False`.
- Iteration speed during learning rate search was approximately 5s/iteration, which needed adjustments to reduce overall time.
- Ended with a `KeyboardInterrupt` after the 27th iteration, requiring further refinements in checkpointing.

## v0.1.3 - Learning Rate Finder Success

- Successfully ran the learning rate finder without OOM errors.
- Estimated total time for the learning rate search was about 17 minutes, an acceptable duration.
- Achieved a significant improvement in stability, with consistent training progress without memory-related interruptions.
- Prepared for the main training loop using the identified learning rate.

## v0.1.4 - Taming the Llama: Training Begins

- Began fine-tuning LLaMA-3.2-1B on the SQuAD dataset.
- Set batch size to 12 and gradient accumulation steps to 3 for balanced training and memory efficiency.
- DataLoader workers were set to 4, contributing to efficient data handling during training.
- Iteration speed improved to 16.17s/iteration, allowing a full training cycle to complete in around 4 hours.
- Addressed frequent tokenizer parallelism warnings by setting the following environment variable:
  ```python
  import os
  os.environ["TOKENIZERS_PARALLELISM"] = "false"
  ```

## v0.1.5 - Training Results and Warnings Addressed

- Fine-tuned the LLaMA model over 834 iterations, successfully completing training.
- Observed loss reduction from **9.73** initially to **7.95** at the end of epoch 3.
- Learning rate started from **0.0099** and adjusted dynamically during training for optimal learning.
- Validation losses reduced consistently, reaching **8.11** during regular evaluations.
- Encountered warnings related to quantization and checkpointing, resolved by modifying training parameters.

## v0.1.6 - Evaluation Planning for LLaMA Model

- Developed an evaluation plan to assess the performance of the fine-tuned LLaMA-3.2-1B model on the SQuAD dataset.
- Key steps included:
  1. Loading the fine-tuned model and tokenizer using the Transformers library.
  2. Defining evaluation metrics, focusing on Exact Match (EM) and F1 Score.
  3. Preparing the validation set from the SQuAD dataset, loaded through DataLoader.
  4. Generating answers using the fine-tuned model to predict responses for questions in the validation set.
  5. Calculating EM and F1 scores for each prediction to evaluate performance.
  6. Looping through the entire validation set to derive average scores, providing insights into overall model efficacy.
  7. Saving evaluation results for future reference.
- Next steps involve running the evaluation process, interpreting results, and identifying areas for further improvement or fine-tuning.

## v0.1.7 - Prevented System Hibernation (Note Added)

- Added a note regarding the persistence of power-saving feature changes across reboots:
  ```bash
  sudo systemctl unmask sleep.target suspend.target hibernate.target hybrid-sleep.target
  ```

## v0.1.8 - Increased Training Iteration Speed, Adjusting for Duration

- Increased iteration speed from 10s/iteration to 15s/iteration.
- However, the number of iterations also increased from 5000 to 8000, resulting in an estimated training duration of 32 hours.
- Searching for new ways to decrease overall training time.
- Attached are screenshots of progress for two different versions (try1 and try2) for comparison.

## v0.1.9 - Learning Rate Search and Issues with Checkpointing

- Began the process of finding the optimal learning rate for the LLaMA model fine-tuning.
- Successfully loaded the SQuAD dataset and prepared it for training.
- GPU Memory updates:
  - After loading the tokenizer: Allocated: 0.00 GB, Reserved: 0.00 GB.
  - After model loading with quantization and adding LoRA configuration: Allocated: 2.03 GB, Reserved: 2.64 GB.
- Set up gradient checkpointing and started the learning rate search.
- Result: 5s/iteration it is taking 2 hours to find the learning rate
- Issues encountered:
  - `use_cache=True` found incompatible with gradient checkpointing, switched to `use_cache=False`.
  - Received user warnings regarding deprecated parameters (`use_reentrant` parameter) and quantization specifics.
  - The learning rate search indicated increasing learning rates with corresponding loss values.
  - Ended with a KeyboardInterrupt after the 27th iteration, requiring adjustments to proceed further.
  - Planning to run a subset of the data to reduce the time to 5 odd minutes

## v0.1.10 - Learning Rate Finder Running Without OOM Error

- The learning rate finder is now running without encountering Out of Memory (OOM) errors, which is a significant improvement.
- Current iteration speed is approximately 2.49 seconds per iteration, with an estimated total time of 17 minutes to complete.
- Observations and potential next steps:
  - Total time of 17 minutes is longer than the initial 10-minute target but still reasonable.
  - Potential adjustments to reduce time further:
    - Reduce `num_iterations` from 50 to 40 or 30.
    - Increase `batch_size` if GPU memory allows.
    - Decrease `accumulation_steps` if needed.
  - Once the learning rate finder completes, the optimal learning rate will be used for the main training loop.
  - Monitoring the following after learning rate completion:
    - Optimal learning rate found.
    - Initial training loss.
    - Training speed in the main loop.

## v0.1.11 - Increased Batch Size and Improved Training Time

- Improved training iteration speed (7sec -> 10sec -> 15.5sec -> 16.17 sec/it) by adjusting several hyperparameters:
  - Batch size increased to 12.
  - Gradient accumulation steps set to 3.
  - DataLoader workers set to 4.
- Achieved better GPU utilization without running into Out of Memory (OOM) errors.

## v0.1.12 - Evaluation Plan Execution for LLaMA Model

- Initiated the evaluation process of the fine-tuned LLaMA-3.2-1B model on the SQuAD validation set.
- Key stages completed:
  1. Loaded the fine-tuned model and tokenizer from the saved directory.
  2. Defined Exact Match (EM) and F1 Score as evaluation metrics.
  3. Prepared the validation set using DataLoader, ensuring efficient data batching.
  4. Generated predictions for the validation set questions using a question-context input format.
  5. Calculated EM and F1 scores for each prediction.
  6. Evaluated the entire validation set to derive average EM and F1 scores.
  7. Saved evaluation results to a JSON file for future analysis.
- Next steps involve analyzing the results, interpreting the model's performance against benchmarks, and making decisions on further training or adjustments.

## v0.1.13 - Final Training Summary and Future Plans

- Completed training and evaluation of LLaMA-3.2-1B on the SQuAD dataset.
- Achieved a final loss of **7.95** and a validation loss of **8.11**.
- Recorded average F1 Score and Exact Match (EM) metrics from the evaluation process.
- Evaluation results indicated areas of strong performance and potential for further fine-tuning.
- Planned next steps:
  - Analyze specific instances of lower performance to understand failure cases.
  - Adjust hyperparameters based on evaluation insights.
  - Consider additional rounds of fine-tuning or model architecture adjustments if needed.
  - Start preparing for model deployment if evaluation results meet project benchmarks.


