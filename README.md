# sxWebEnz (api)
EnzHier for web server api


# EnzHire API Client

**EnzHire** is a deep learning-based tool for Enzyme Commission (EC) number prediction. 
This repository contains the Python client for programmatically accessing the EnzHire web server.

- **Web Server:** [https://sxwebenz.scutlabx.com/](https://sxwebenz.scutlabx.com/)
- **API Endpoint:** `https://sxwebenz.scutlabx.com/api/predict/json`

## Prerequisites

You need **Python 3.x** and the `requests` library installed.

```bash
pip install requests


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

Duan, Hongyu, et al. "EnzHier: Accurate Enzyme Function Prediction Through Multi-scale Feature Integration and Hierarchical Contrastive Learning." International Symposium on Bioinformatics Research and Applications. Singapore: Springer Nature Singapore, 2025.

## License

This project is free for academic and commercial use.
