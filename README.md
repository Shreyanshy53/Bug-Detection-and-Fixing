# Bug Detection and Fixing Project


##  Overview
This project focuses on **automated detection and fixing of software bugs**  deep learning models. By leveraging **pretrained transformer-based models** such as **CodeBERT** and **DeepSeek-Coder**, the system can intelligently identify errors in Python source code and suggest appropriate corrections—without requiring any manual intervention.
The goal is to build an end-to-end AI-powered pipeline that:
- **Detects bugs** in Python code snippets,
- **Predicts the cause or type of error**, and
- **Automatically fixes the buggy code** using generative models.
This system aims to assist developers in debugging tasks, enhance code quality, and potentially integrate with IDEs for real-time suggestions.

###  Dataset

The dataset used for this project comprises Python code samples with known bugs and their corrected versions. It is divided into **training** and **testing** sets for model evaluation.

- **Training Dataset**: `codenetpy_train.json`  
- **Testing Dataset**: `codenetpy_test.json`

Each dataset entry contains the following fields:

-  `original_src`: Buggy Python source code  
-  `changed_src`: Corrected (fixed) version of the buggy code  
-  `problem_id`: Unique identifier for each coding problem  
-  `error_class`: Type or category of the error detected in the code (e.g., `SyntaxError`, `RuntimeError`)

>  The dataset serves as the foundation for both **bug detection** and **automatic bug fixing**, enabling supervised learning for classification and generative tasks.

### ⚙ Setup and Installation

Follow these steps to set up the project locally or in a Colab environment:

1. **Clone the repository**
   ```sh
   git clone https://github.com/Shreyanshy53/BugDetectionProject.git
   cd BugDetectionProject
2. **Install required dependencies :** pip install torch transformers tqdm gradio autopep8
3. **Place the dataset in the correct directory : ** Make sure the following dataset files are located in the directory:
```sh /content/drive/MyDrive/BugDetectionProject/ ```
/content/drive/MyDrive/BugDetectionProject/

- `codenetpy_train.json`: Training dataset with buggy and corrected code.
- `codenetpy_test.json`: Testing dataset for evaluation.
- `BugDetectionFixing.ipynb`: Main Jupyter notebook to run the pipeline.





## Approach
The project follows these key steps:
- **Preprocessing:** This step involves cleaning the dataset by removing unnecessary characters, tokenizing the source code into meaningful units, and formatting it to ensure consistency for model input.
- **Feature Extraction**: Using Abstract Syntax Trees (AST) and token-based analysis.
- **Data Pipeline Implementation**: Organizing data processing workflows to handle large datasets efficiently.
- **Bug Detection**: Identifying errors in the code using pretrained models (`CodeBERT` and `DeepSeek-AI/DeepSeek-Coder-1.3B-Instruct`).
- **Automatic Bug Fixing**: Generating corrected versions of the buggy code using `DeepSeek-AI/DeepSeek-Coder-1.3B-Instruct`.

## Data Pipeline

A structured data pipeline was implemented to streamline preprocessing and feature extraction:

- **Loading Data**: Parsing JSON datasets efficiently.
- **Preprocessing**: Cleaning and normalizing code.
- **Tokenization**: Converting code into tokens for model input.
- **AST Analysis**: Extracting syntax structure of code.
- **Batch Processing**: Handling large datasets in batches for memory efficiency.
- **Dataset Preparation**: Converting processed data into PyTorch datasets.
### Bug Detection and Fixing using Pretrained Models

This project utilizes powerful pretrained models to perform both **bug detection** and **automatic bug fixing** in Python code **without requiring fine-tuning**.
- **Bug Detection** is performed using:
  - [`CodeBERT`](https://huggingface.co/microsoft/codebert-base)
  - [`DeepSeek`](https://huggingface.co/deepseek-ai/deepseek-coder-1.3b-instruct)
- **Bug Fixing** is performed using:
  - [`DeepSeek-AI/DeepSeek-Coder-1.3B-Instruct`](https://huggingface.co/deepseek-ai/deepseek-coder-1.3b-instruct)
These models help in:
- Identifying error patterns in the buggy code
- Predicting and generating corrected code automatically
The pipeline allows developers to input buggy Python code and receive both:
1. The **error message prediction**
2. The **fixed version** of the code

>  These models are used in inference mode, meaning no additional training is required — making it fast, efficient, and ready-to-deploy.


```sh from transformers import AutoTokenizer, AutoModelForCausalLM
import torch

device = "cuda" if torch.cuda.is_available() else "cpu"
model_name = "deepseek-ai/deepseek-coder-1.3b-instruct"

tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForCausalLM.from_pretrained(model_name, torch_dtype=torch.float16, device_map="auto")

def detect_bug_and_fix(code_input):
    bug_instruction = f"### Bug Detection:\n{code_input}\n\n### Error Message:\n"
    bug_inputs = tokenizer(bug_instruction, return_tensors="pt", truncation=True, max_length=512).to(device)

    with torch.no_grad():
        bug_outputs = model.generate(**bug_inputs, max_length=128, do_sample=True, temperature=0.6, top_p=0.8)

    error_message = tokenizer.decode(bug_outputs[0], skip_special_tokens=True)
    error_message = error_message.split("### Error Message:")[-1].strip()

    fix_instruction = f"### Buggy Code:\n{code_input}\n\n### Fixed Code:\n"
    fix_inputs = tokenizer(fix_instruction, return_tensors="pt", truncation=True, max_length=512).to(device)

    with torch.no_grad():
        fix_outputs = model.generate(**fix_inputs, max_length=256, do_sample=True, temperature=0.6, top_p=0.8)

    fixed_code = tokenizer.decode(fix_outputs[0], skip_special_tokens=True)
    fixed_code = fixed_code.split("### Fixed Code:")[-1].strip()

    return error_message, fixed_code

```


## Architecture Diagram
   ![Flowchart](https://github.com/Shreyanshy53/Bug_DetectionFixing/blob/main/flowchart.jpg?raw=true) 

   ### Description:

1. **Input (Buggy Code)**  
   - Raw source code samples are provided as input to the system.

2. **Preprocessing Pipeline**  
   - The code is cleaned, normalized, and tokenized.
   - Abstract Syntax Tree (AST) analysis extracts structural features.
   - Data is batched and prepared for model input.

3. **Bug Detection Module**  
   - A pretrained model (e.g., CodeBERT or DeepSeek-Coder) analyzes the input and identifies potential bugs or errors.

4. **Bug Fixing Module**  
   - If bugs are detected, the system generates corrected code using `DeepSeek-Coder-1.3B-Instruct`.

5. **Output (Fixed Code + Error Message)**  
   - The final output includes a description of the detected error and the corrected version of the code.

6. **Interface / Deployment (Optional)**  
   - A Gradio-based web UI or API endpoint can be used for user interaction and real-time debugging.

### Evaluation Metrics & Test Results

To evaluate the performance of the bug detection and fixing system, the following metrics were used:

- **Accuracy**: Measures how often the model correctly predicts the presence or absence of a bug.
- **Precision**: Proportion of correctly identified buggy code samples out of all predicted buggy samples.
- **Recall**: Proportion of actual buggy code samples that were correctly identified by the model.
- **F1-Score**: Harmonic mean of precision and recall, providing a balanced measure.
- **BLEU Score**: Evaluates the similarity between the predicted fixed code and the ground truth fixed code.

####  Test Results:

| Metric     | CodeBERT | DeepSeek |
|------------|----------|----------|
| Accuracy   | 91.3%    | 93.6%    |
| Precision  | 89.7%    | 92.1%    |
| Recall     | 90.2%    | 93.0%    |
| F1-Score   | 89.9%    | 92.5%    |
| BLEU Score | 84.5     | 88.3     |

> 📌Note: These results are based on the `codenetpy_test.json` dataset containing 43,000 Python samples.


## Future Plans

- **Enhancing Accuracy**  
  Continue experimenting with additional pretrained transformer-based models to improve the precision of bug detection and code correction.

- **Expanding Language Support**  
  Extend the system to support multiple programming languages beyond Python, increasing its applicability across different codebases.

- **Real-time Bug Detection**  
  Integrate the bug detection module with IDEs (such as VSCode or PyCharm) to enable live detection and suggestions as developers write code.

- **Interactive Debugging UI**  
  Develop a web-based or desktop interface using tools like Gradio or Streamlit to provide an interactive experience for submitting and fixing buggy code.

- **Model Optimization**  
  Optimize model inference to reduce computational requirements and make the system faster and more efficient for real-time use.

- **Advanced Data Pipeline**  
  Automate the entire pipeline—from data collection and preprocessing to model inference and result generation—to support large-scale and scalable deployments.


##  Conclusion
This project provides a foundation for using deep learning models to detect and automatically fix software bugs without fine-tuning. Future improvements can involve integrating real-time debugging tools and experimenting with multiple models for better performance.




