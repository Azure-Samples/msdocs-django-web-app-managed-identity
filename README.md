# Deploy a Python (Django) app to Azure with Managed Identity 

This Python app is a simple restaurant review application using the [Django](https://www.djangoproject.com/) framework. The app uses Azure App Service, Azure Database for PostgreSQL relational database service, and Azure Storage. When deployed, Azure managed identity allows the web app hosted in App Service to connect to the database and storage resources without the need to specify sensitive connection info in code or environment variables.

This sample app can be [run locally](#running-locally) and then deployed to Azure, hosted in a fully managed Azure App Service. For more information on how to deploy this to Azure, see the  [Overview: Deploy a Django web app to Azure with managed identity](https://docs.microsoft.com/azure/developer/python/tutorial-python-managed-identity-user-assigned-cli).

If you need an Azure account, you can [create on for free](https://azure.microsoft.com/free/).

A Flask sample application with similar functionality is at https://github.com/Azure-Samples/msdocs-flask-web-app-managed-identity.

## Requirements

The [requirements.txt](./requirements.txt) has the following packages:

| Package | Description |
| ------- | ----------- |
| [Django](https://pypi.org/project/Django/) | Web application framework. |
| [pyscopg2-binary](https://pypi.org/project/psycopg-binary/) | PostgreSQL database adapter for Python. |
| [python-dotenv](https://pypi.org/project/python-dotenv/) | Read key-value pairs from .env file and set them as environment variables. In this sample app, environment variables describe how to connect to the database and storage resources. Because managed identity is used no sensitive information is included in environment variables. <br><br> This package is used in the [manage.py](./manage.py) file to load environment variables. |
| [whitenoise](https://pypi.org/project/whitenoise/) | Static file serving for WSGI applications, used in the deployed app. <br><br> This package is used in the [azureproject/production.py](./azureproject/production.py) file, which configures production settings. |
| [azure-blob-storage](https://pypi.org/project/azure-storage/) | Microsoft Azure Storage SDK for Python |
| [azure-identity](https://pypi.org/project/azure-identity/) | Microsoft Azure Identity Library for Python |

## DefaultAzureCredential

The [DefaultAzureCredential](https://docs.microsoft.com/python/api/azure-identity/azure.identity.defaultazurecredential) is used in the [views.py](./restaurant_review/views.py) file. For example:

```python
from azure.identity import DefaultAzureCredential
from azure.storage.blob import BlobServiceClient

azure_credential = DefaultAzureCredential()
blob_service_client = BlobServiceClient(
    account_url=account_url,
    credential=azure_credential)
```

The DefaultAzureCredential is also used to get a token for PostgreSQL in the [get_token.py](./azureproject/get_token.py) file when running in Azure.

```python
azure_credential = DefaultAzureCredential()
token = azure_credential.get_token("https://ossrdbms-aad.database.windows.net")
conf.settings.DATABASES['default']['PASSWORD'] = token.token
```

You can customize the credential chain that an instance of `DefaultAzureCredential` evaluates for use in your intended environments. To learn more, see [DefaultAzureCredential overview](https://learn.microsoft.com/azure/developer/python/sdk/authentication/credential-chains?tabs=dac#defaultazurecredential-overview).

## start.sh

Deployed, the [start.sh](start.sh) file sets a SECRET_KEY and runs the Django migrate command. The SECRET_KEY is used to encrypt session data. The migrate command creates the database tables. Instead of a start.sh you could set the SECRET_KEY as an app setting and run the migrate command manually by ssh'ing into the app service.

## Running locally

To run the app locally, create a copy of *.env.example* and name it *.env*. Then, update the values in the *.env* file with your database and storage connection information. The *.env* file is used by the [manage.py](./manage.py) file to set environment variables.

Running locally can be challenging because if you want to be truly local you need to set up resources locally. For example, you need to install PostgreSQL and create a database. You also need to install Azure Storage Emulator and create a storage account.

As an alternative:

* You can point to resources in Azure and avoid creating them locally.
* Use GitHub codespace. For example, see [Create a GitHub Codespaces dev environment with FastAPI and Postgres](https://learn.microsoft.com/azure/developer/python/configure-python-web-app-codespaces)

Regardless of how you connect to resources, you need to run the Django migrate command in the *start.sh* before the app starts up.