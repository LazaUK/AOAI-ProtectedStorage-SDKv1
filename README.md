# Azure OpenAI: How to process files, hosted in non-public Azure Storage accounts.

By default, blobs and containers on Azure Storage account are not accessible by anonymous accounts. In this repo, I explain how to authenticate with Azure Storage account, download target blob (**_Microsoft logo_** as an example), convert it into Base64 format and then feed to Azure OpenAI model for a further processing.

## Table of contents:
- [Pre-requisites](https://github.com/LazaUK/AOAI-ProtectedStorage-SDKv1#pre-requisites)
- [Step 1: Authenticating with Azure Storage account](https://github.com/LazaUK/AOAI-ProtectedStorage-SDKv1#step-1-authenticating-with-azure-storage-account)
- [Step 2: Downloading hosted image and convert it into Base64](https://github.com/LazaUK/AOAI-ProtectedStorage-SDKv1#step-2-downloading-hosted-image-and-convert-it-into-base64)
- [Step 3: Processing image by Azure OpenAI model, e.g. GPT-4o](https://github.com/LazaUK/AOAI-ProtectedStorage-SDKv1#step-3-processing-image-by-azure-openai-model-eg-gpt-4o)

## Pre-requisites
1. To build this demo, I used the latest version of OpenAI Python SDK - v1.x. To upgrade your _openai_ Python package, please use the following pip command:
```
pip install --upgrade openai
```
2. To use API key authentication, assign the API endpoint name, version and key, along with the Azure OpenAI deployment name of your Azure OpenAI model to **AZURE_OPENAI_API_BASE**, **AZURE_OPENAI_API_VERSION**, **AZURE_OPENAI_API_KEY** and **AZURE_OPENAI_API_DEPLOY** environment variables respectively.
>**Note**: If you want to use the Entra ID (former Azure Active Directory) authentication instead, you may find some implementation options [here](https://github.com/LazaUK/AOAI-EntraIDAuth-SDKv1).
3. Details of the target image can be defined through **STORAGE_URL**, **STORAGE_CONTAINER** and **STORAGE_BLOB** variables.

## Step 1: Authenticating with Azure Storage account
1. Authentication process for Azure Storage accounts in Python is described on the following [docuemntation page](https://learn.microsoft.com/en-us/azure/storage/blobs/storage-blob-python-get-started).
2. We'll re-use the above details, to create a Helper function initiate Blob service account with _Default Azure Credentials_, that can use available Azure credentials, e.g. from Az CLI environment or assigned Managed Identity (if running on Azure resource):
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

## Step 2: Downloading hosted image and convert it into Base64
1. Options to download target blob are described on the following Azure Storage [documentation page](https://learn.microsoft.com/en-us/azure/storage/blobs/storage-blob-download-python).
2. We'll borrow code sample to download blob to a string and adapt it to our image processing scenario. Particularly, we'll define a Helper function to convert downloaded string into Base64 format, that Azure OpenAI models like GPT-4o can process natively:
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
1. The last step in our exercise is to process the image by Azure OpenAI model. We'll create one more Helper function to initiate Azure OpenAI client and feed it with downloaded image in Base64 format:
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
2. Executing all 3 steps in a sequence should produce an outcome similar to this:
``` JSON
Step 1 - Authenticating with Azure Blob Storage: https://lazizaoaistorage.blob.core.windows.net.
Step 2 - Downloading blob: mslogo.png.
Step 3 - Analysing image using Azure OpenAI.
--------------------
Results: The image features a simple grid composed of four colored squares. The top left square is orange, the top right square is green, the bottom left square is blue, and the bottom right square is yellow. The squares are arranged in a 2x2 format.
```
