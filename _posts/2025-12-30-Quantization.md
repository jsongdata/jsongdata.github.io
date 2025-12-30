---
title: "Intro to Quantization"
date: 2025-12-30
categories: [LLM]
tags: [llm, llmops, quantizaiton]     # TAG names should always be lowercase
---

# Complete Guide to LLM Quantization & Compression: Summary

## 1. Overview & Key Statistics
* **Objective**: Enable large-scale models (70B+) to run on standard gaming laptops or small GPU environments.
* **Impact**: Reduce memory usage by up to **87.5%** while maintaining performance loss within **1~2%**.
* **Philosophy**: "A leaned-down giant is smarter than a perfect dwarf" (Compressed 70B model > Native 8B model).

## 2. Principles of Quantization
* **Definition**: A technique to reduce the precision of numbers used in model calculations (similar to compressing high-res photos into JPGs).
* **Benefits by Precision**: 
    * **8-bit**: Saves 75% memory with negligible performance loss.
    * **4-bit**: Saves 87.5% memory; the most recommended level for practical use.

## 3. Hardware & Purpose Selection Guide (Cheat Sheet)

| Category | Recommended Tech | Key Characteristics |
| :--- | :--- | :--- |
| **NVIDIA GPU** | **GPTQ** | Extremely fast and easy to apply (Best for rapid testing). |
| **NVIDIA GPU** | **AWQ** | Precise compression that prioritizes model quality (Best for production services). |
| **CPU / Apple Silicon** | **GGUF** | High universality; runs almost anywhere. |

## 4. Additional Compression Techniques
* **Pruning**: Removing low-importance parameters from the model to increase efficiency.
* **Knowledge Distillation**: A large "Teacher" model (e.g., GPT-4) trains a smaller "Student" model to transfer its capabilities.

## 5. Conclusion & Recommended Strategy
* **Performance Threshold**: Performance remains stable up to 4-bit, but accuracy drops significantly at 2-bit or lower.
* **Action Plan**: Use **AWQ** for commercial services, **GGUF** for local personal chatbots, and **GPTQ** for quick technical validation.
