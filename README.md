# Retrieval-Augmented Generation (RAG) with Azure OpenAI and Azure AI Search
By default, blobs and containers on Azure Storage account are not accessible by anonymous accounts.

This repo explains how to authenticate with an Azure Storage account, download a target blob (using the **_Microsoft logo_** as an example), convert it into Base64 format and then feed it to an Azure OpenAI model for further processing.

## Table of contents:
- [Pre-requisites](https://github.com/LazaUK/AOAI-ProtectedStorage-SDKv1#pre-requisites)
- [Step 1: Authenticating with Azure Storage account](https://github.com/LazaUK/AOAI-ProtectedStorage-SDKv1#step-1-authenticating-with-azure-storage-account)
- [Step 2: Downloading source image and converting it into Base64](https://github.com/LazaUK/AOAI-ProtectedStorage-SDKv1#step-2-downloading-source-image-and-converting-it-into-base64)
- [Step 3: Processing image by Azure OpenAI model, e.g. GPT-4o](https://github.com/LazaUK/AOAI-ProtectedStorage-SDKv1#step-3-processing-image-by-azure-openai-model-eg-gpt-4o)

## Pre-requisites
1. To build this demo, you'll need the latest version of the OpenAI Python SDK (v1.x). Use the following pip command to upgrade your _openai_ Python package:
```
pip install --upgrade openai
```
2. For Azure OpenAI's _API key authentication_, set the following environment variables:
- ```AZURE_OPENAI_API_BASE```: Azure OpenAI API endpoint base URL,
- ```AZURE_OPENAI_API_VERSION```: Azure OpenAI API version,
- ```AZURE_OPENAI_API_KEY```: Your Azure OpenAI API key,
- ```AZURE_OPENAI_API_DEPLOY```: Deployment name of your Azure OpenAI model.
>**Note**: If you prefer _Entra ID (formerly Azure Active Directory) authentication_, you can find implementation options [here](https://github.com/LazaUK/AOAI-EntraIDAuth-SDKv1).
3. Details of the source image can be defined through the following environment variables:
- ```STORAGE_URL```: URL of your Azure Storage account,
- ```STORAGE_CONTAINER```: Name of the container containing the image,
- ```STORAGE_BLOB```: Name of the image blob.

## Step 1: Authenticating with Azure Storage account
This step demonstrates how to authenticate with your Azure Storage account in Python. For specifics of the process, please refer to the following Azure Storage [documentation page](https://learn.microsoft.com/en-us/azure/storage/blobs/storage-blob-python-get-started).

We'll use the _DefaultAzureCredential_ class which can leverage available credentials, e.g. from the Azure CLI environment or assigned Managed Identity (if running on an Azure resource):
``` Python
def get_blob_service_client(storage_url):
    print(f"Step 1 - Authenticating with Azure Blob Storage: {storage_url}.")
    credential = DefaultAzureCredential()
    blob_service_client = BlobServiceClient(
        storage_url,
        credential=credential
    )

    return blob_service_client
```

## Step 2: Downloading source image and converting it into Base64
Here, we'll download the target blob and convert it to Base64 format, which Azure OpenAI models like GPT-4o can process natively.:
``` Python
def get_blob_to_base64(blob_service_client: BlobServiceClient, container_name, blob_name):
    print(f"Step 2 - Downloading blob: {blob_name}.")
    blob_client = blob_service_client.get_blob_client(
        container=container_name,
        blob=blob_name
    )
    downloader = blob_client.download_blob(
        max_concurrency=1
    )
    blob_content = downloader.readall()
    blob_base64 = base64.b64encode(blob_content).decode('utf-8')

    return blob_base64
```

## Step 3: Processing image by Azure OpenAI model, e.g. GPT-4o
Finally, we'll use the downloaded image in Base64 format to analyse it with an Azure OpenAI model:
``` Python
def analyse_blob(base64_image):
    print(f"Step 3 - Analysing image using Azure OpenAI.")

    client = AzureOpenAI(
        azure_endpoint = AOAI_API_BASE,
        api_key = AOAI_API_KEY,
        api_version = AOAI_API_VERSION
    )

    response = client.chat.completions.create(
        model = AOAI_DEPLOYMENT,
        messages = [
            {"role": "system", "content": "You are a useful image analyser."},
            {"role": "user", "content": [  
                { 
                    "type": "text", 
                    "text": "Please, describe the image." 
                },
                { 
                    "type": "image_url",
                    "image_url": {
                        "url": f"data:image/jpeg;base64,{base64_image}"
                    }
                }
            ]} 
        ]
    )
    print("--------------------")
    print(f"Results: {response.choices[0].message.content}")
    return response
```

Executing all 3 steps in a sequence should produce an output similar to this:
``` JSON
Step 1 - Authenticating with Azure Blob Storage: https://lazizaoaistorage.blob.core.windows.net.
Step 2 - Downloading blob: mslogo.png.
Step 3 - Analysing image using Azure OpenAI.
--------------------
Results: The image features a simple grid composed of four colored squares. The top left square is orange, the top right square is green, the bottom left square is blue, and the bottom right square is yellow. The squares are arranged in a 2x2 format.
```
