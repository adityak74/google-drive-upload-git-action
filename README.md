[![build](https://github.com/adityak74/google-drive-upload-git-action/actions/workflows/ci.yaml/badge.svg?branch=main)](https://github.com/adityak74/google-drive-upload-git-action/actions)
[![Go Report Card](https://goreportcard.com/badge/github.com/adityak74/google-drive-upload-git-action)](https://goreportcard.com/report/github.com/adityak74/google-drive-upload-git-action)

# google-drive-upload-git-action
Github action that uploads files to Google Drive.

![star-history-2025323 (1)](https://github.com/user-attachments/assets/5f54c80b-96cd-4314-a8b5-3f1ac5504fd4)

**This only works with a Google Service Account!**

Thanks to [Team Tumbleweed](https://github.com/team-tumbleweed) for developing the initial version of this actions package.

To make a GSA go to the [Credentials Dashboard](https://console.cloud.google.com/apis/credentials). You will need to download the **.json key** and base64 encode it. You will use this string as the `credentials` input. To convert the *json* file to base64 without having to use an online tool (which is insecure), use this command:

`base64 credentials.json -w0`

On mac this the base64 by default opts for -w as 0, you can skip and just use base64 without any params.

You will also need to **share the drive with the servie account.** To do this, just share the folder like you would normally with a friend, except you share it with the service account email address. Additionally you will need to give the service account acccess to the google drive API. 
Go to `https://console.developers.google.com/apis/api/drive.googleapis.com/overview?project={PROJECT_ID}`. Where `{PROJECT_ID}` is the id of your GCP project. Find more info about that [here.](https://support.google.com/googleapi/answer/7014113?hl=en)

# Inputs

## ``filename``
Required: **YES**.  

The name of the file you want to upload. Wildcards can be used to upload more than one file.

## ``name``
Required: **NO**

The name you want the file to have in Google Drive. If this input is not provided, it will use only the filename of the source path. It will be ignored if there are more than one file to be uploaded.

## ``overwrite``
Required: **NO**

If you want to overwrite the filename with existing file, it will use the target filename.
## ``mimeType``
Required: **NO**

file MimeType. If absent, Google Drive will attempt to automatically detect an appropriate value.

## ``useCompleteSourceFilenameAsName``
Required: **NO**

If true, the target file name will be the complete source filename and `name` parameter will be ignored.

## ``mirrorDirectoryStructure``
Required: **NO**

## ``workingDirectory``
Required: **NO**

If set, the directory to change to before the action is run. This is likely most useful for mirroring a directory structure (See the example below)


If true, the directory structure of the source file will be recreated relative to ``folderId``.

## ``namePrefix``
Required: **NO**

Prefix to be added to target filename.

## ``folderId``
Required: **YES**. 

The [ID of the folder](https://ploi.io/documentation/database/where-do-i-get-google-drive-folder-id) you want to upload to.

## ``credentials``
Required: **YES**.

A base64 encoded string with the [GSA credentials](https://stackoverflow.com/questions/46287267/how-can-i-get-the-file-service-account-json-for-google-translate-api/46290808).


# Usage Example

## Simple Workflow
In this example we stored the folderId and credentials as action secrets. This is highly recommended as leaking your credentials key will allow anyone to use your service account.
```yaml
# .github/workflows/main.yml
name: Main
on: [push]

jobs:
  my_job:
    runs-on: ubuntu-latest

    steps:

      - name: Checkout code
        uses: actions/checkout@v2

      - name: Archive files
        run: |
          sudo apt-get update
          sudo apt-get install zip
          zip -r archive.zip *

      - name: Upload to gdrive
        uses: adityak74/google-drive-upload-git-action@main
        with:
          credentials: ${{ secrets.credentials }}
          filename: "archive.zip"
          folderId: ${{ secrets.folderId }}
          name: "documentation.zip" # optional string
          overwrite: "true" # optional boolean
      - name: Make Directory Structure
        run: |
          mkdir -p w/x/y
          date +%s > w/x/y/z
      - name: Mirror Directory Structure
        uses: adityak74/google-drive-upload-git-action@main
        with:
          credentials: ${{ secrets.DRIVE_CREDENTIALS }}
          filename: output/w/x/y/z
          folderId: ${{ secrets.folderId }}
          overwrite: "true"
          mirrorDirectoryStructure: "true"
          workingDirectory: output/
          
```
