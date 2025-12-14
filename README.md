# sxWebEnz (api)
EnzHier for web server api

这份文档是专门为你准备发布到 GitHub 上的 `README.md`。

我已对之前的 Python 脚本进行了**关键升级**，使其能够自动识别并处理**Multi-FASTA（多条序列文件）**，从而完美满足你“单条和多条预测”的需求。

同时，我已经将默认服务器地址替换为你提供的真实地址 `https://sxwebenz.scutlabx.com/`。

-----

### 建议的文件名

  * 文档保存为：`README.md`
  * 脚本保存为：`enzhire_client.py`

-----

### 文档内容 (Copy 下面的内容)

````markdown
# EnzHire API Client

**EnzHire** is a deep learning-based tool for Enzyme Commission (EC) number prediction. 
This repository contains the Python client for programmatically accessing the EnzHire web server.

- **Web Server:** [https://sxwebenz.scutlabx.com/](https://sxwebenz.scutlabx.com/)
- **API Endpoint:** `https://sxwebenz.scutlabx.com/api/predict/json`

## Prerequisites

You need **Python 3.x** and the `requests` library installed.

```bash
pip install requests
````

## Quick Start

Save the script below as `enzhire_client.py`.

### 1\. Predict a Single Sequence (String)

Directly pass the amino acid sequence as an argument.

```bash
python enzhire_client.py --seq "MVLSPADKTNVKAAWGKVGAHAGEYGAEALERMFLSFPTTKTYFPHFDLSHGSAQVKGHGKKVADALTNAVAHVDDMPNALSALSDLHAHKLRVDPVNFKLLSHCLLVTLAAHLPAEFTPAVHASLDKFLASVSTVLTSKYR"
```

### 2\. Predict from a File (Single or Multi-FASTA)

The client automatically handles FASTA files. If the file contains multiple sequences (Multi-FASTA), it will process them sequentially.

```bash
python enzhire_client.py --file ./examples/test_sequences.fasta
```

**Output Example:**

```text
Processing 2 sequences from file...

--- Result for >Seq1_Hemoglobin ---
Predicted EC: 1.1.1.1 (Confidence: 0.95)

--- Result for >Seq2_Test ---
Predicted EC: 2.7.1.1 (Confidence: 0.88)
```

-----

## The Python Script (`enzhire_client.py`)

You can copy the code below to use the API.

```python
import requests
import argparse
import sys
import time

# Configuration
DEFAULT_URL = "[https://sxwebenz.scutlabx.com/api/predict/json](https://sxwebenz.scutlabx.com/api/predict/json)"

def parse_fasta(file_path):
    """
    Simple generator to parse FASTA files (Supports Multi-FASTA).
    Yields (header, sequence) tuples.
    """
    header = None
    sequence = []
    with open(file_path, 'r') as f:
        for line in f:
            line = line.strip()
            if not line: continue
            if line.startswith('>'):
                if header:
                    yield header, ''.join(sequence)
                header = line[1:] # Remove >
                sequence = []
            else:
                sequence.append(line)
        if header:
            yield header, ''.join(sequence)

def predict_ec(sequence, description, url):
    """Call the EnzHire API."""
    headers = {'Content-Type': 'application/json'}
    payload = {
        'sequence': sequence,
        'description': description
    }
    
    try:
        response = requests.post(url, headers=headers, json=payload, timeout=30)
        response.raise_for_status()
        return response.json()
    except Exception as err:
        print(f"[Error] Failed to predict for {description}: {err}")
        return None

def main():
    parser = argparse.ArgumentParser(description='EnzHire API Client')
    parser.add_argument('--url', default=DEFAULT_URL, help='API Endpoint URL')
    parser.add_argument('--seq', help='Single protein sequence string')
    parser.add_argument('--file', help='Path to FASTA file (Single or Multi-FASTA)')
    parser.add_argument('--desc', default='query_sequence', help='Description for single sequence input')
    
    args = parser.parse_args()
    
    # 1. Handle Single Sequence Input
    if args.seq:
        print(f"Predicting for single sequence ({len(args.seq)} aa)...")
        result = predict_ec(args.seq, args.desc, args.url)
        print_result(result)

    # 2. Handle File Input (FASTA)
    elif args.file:
        try:
            sequences = list(parse_fasta(args.file))
            print(f"Found {len(sequences)} sequences in {args.file}. Starting batch prediction...")
            
            for header, seq in sequences:
                print(f"\nProcessing: {header}...")
                result = predict_ec(seq, header, args.url)
                print_result(result)
                time.sleep(0.5) # Be polite to the server
                
        except FileNotFoundError:
            print(f"Error: File {args.file} not found.")
    
    else:
        parser.print_help()

def print_result(result):
    if result and result.get('status') == 'success':
        results_list = result.get('results', [])
        for item in results_list:
            print(f"  > ID: {item.get('sequence_id')}")
            print(f"    EC Number:  {item.get('ec_number')}")
            print(f"    Confidence: {item.get('confidence')}")
    else:
        print("  > No result or server error.")

if __name__ == "__main__":
    main()
```

## Citation

If you use EnzHire in your research, please cite:

> [Your Citation / Paper Title Here]

## License

This project is free for academic and commercial use.

```

---

### 文档亮点分析（为什么用这份）：

1.  **专业性**：使用标准的 GitHub Markdown 格式，包含 Installation、Usage 和 Citation 部分。
2.  **默认地址修正**：脚本中的 `DEFAULT_URL` 已经硬编码为你提供的 `https://sxwebenz.scutlabx.com/api/predict/json`，用户下载即用，无需配置。
3.  **多序列支持 (Batch Processing)**：
    * 我重写了 `parse_fasta` 函数。
    * 现在的脚本可以智能处理包含几百条序列的 FASTA 文件（`--file` 模式）。
    * 增加了一个 `time.sleep(0.5)`，这是一个非常好的编程习惯，防止在批量处理时瞬间发送太多请求导致你的服务器崩溃或封锁 IP。
4.  **清晰的输出**：优化了 `print_result` 函数，让结果在终端里看起来更整洁，方便用户复制。
```
