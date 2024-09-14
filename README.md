# Azure OpenAI: How to process files, hosted in non-public Azure Storage accounts.

By default, blobs and containers on Azure Storage account are not accessible by anonymous accounts. In this repo, I explain how to authenticate with Azure Storage account, download target blob (**_Microsoft logo_** as an example), convert it into Base64 format and then feed to Azure OpenAI model for a further processing.

## Table of contents:
- [Pre-requisites]()
- [Step 1: Authenticating with Azure Storage account](https://github.com/LazaUK/AOAI-ProtectedStorage-SDKv1/tree/main#step-1-authenticating-with-azure-storage-account)
- [Step 2: Downloading hosted image and convert it into Base64]()
- [Step 3: Processing image by Azure OpenAI model, e.g. GPT-4o]()

## Pre-requisites
1. To build this demo, I used the latest version of OpenAI Python SDK - v1.x. To upgrade your _openai_ Python package, please use the following pip command:
```
pip install --upgrade openai
```
2. To use API key authentication, assign the API endpoint name, version and key, along with the Azure OpenAI deployment name of your Azure OpenAI model to **AZURE_OPENAI_API_BASE**, **AZURE_OPENAI_API_VERSION**, **AZURE_OPENAI_API_KEY** and **AZURE_OPENAI_API_DEPLOY** environment variables respectively.
>**Note**: If you want to use the Entra ID (former Azure Active Directory) authentication instead, you may find some implementation options [here](https://github.com/LazaUK/AOAI-EntraIDAuth-SDKv1).

## Step 1: Authenticating with Azure Storage account



## Step 2: Downloading hosted image and convert it into Base64

## Step 3: Processing image by Azure OpenAI model, e.g. GPT-4o

