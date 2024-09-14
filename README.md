# Azure OpenAI: How to process files, hosted in non-public Azure Storage accounts.

By default, blobs and containers on Azure Storage account are not accessible for anonymous calls. In this repo, I explain how to authenticate with Azure Storage account, download target blob (Microsoft logo), convert it into Base64 format and then feed to Azure OpenAI model for a further processing.

To build this demo, I used the latest version of OpenAI Python SDK - v1.x. To upgrade your _openai_ Python package, please use the following pip command:
```
pip install --upgrade openai
```

## Table of contents:
- [Step 1: Authenticating with Azure Storage account]()
- [Step 2: Downloading hosted image and convert it into Base64]()
- [Step 3: Processing image by Azure OpenAI model, e.g. GPT-4o]()

## Step 1: Authenticating with Azure Storage account

## Step 2: Downloading hosted image and convert it into Base64

## Step 3: Processing image by Azure OpenAI model, e.g. GPT-4o
1. To use API key authentication, assign the API endpoint name, version and key, along with the Azure OpenAI deployment name of DALL-E 3 model to **OPENAI_API_BASE**, **OPENAI_API_VERSION**, **OPENAI_API_KEY** and **OPENAI_API_DEPLOY_DALLE** environment variables respectively.
