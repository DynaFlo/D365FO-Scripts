# Test AureStorage

## Test Azure Storage Class

``` X++
using Microsoft.Azure;
using Microsoft.WindowsAzure.Storage;
using Microsoft.WindowsAzure.Storage.Blob;
using Microsoft.WindowsAzure.Storage.File;class TestAzureStorage

{
    /// <summary>
    /// Runs the class with the specified arguments.
    /// </summary>
    /// <param name = "_args">The specified arguments.</param>
    public static void main(Args _args)
    {
        System.IO.MemoryStream memoryStream;

        var storageCredentials = new Microsoft.WindowsAzure.Storage.Auth.StorageCredentials('fjstorageaccount', '6UJdCqFzyZSH1N+nlF8/SFK1rB1yQ9MKIr5PmAECsOBMpYBtpim5Mf1gpZ/oH7Doz0lOYhq1XxuzY7IupmSuTw==');

        CloudStorageAccount storageAccount = new Microsoft.WindowsAzure.Storage.CloudStorageAccount(storageCredentials, true);

        CloudFileClient fileClient = storageAccount.CreateCloudFileClient();

        CloudFileShare share = fileClient.GetShareReference('sharetest');

        if (share.Exists(null, null))
        {
            CloudFileDirectory rootDir = share.GetRootDirectoryReference();

            CloudFileDirectory fileDir = rootDir.GetDirectoryReference('folder');

            if (fileDir.Exists(null, null))
            {
                CloudFile file = fileDir.GetFileReference('file.txt');

                if (file.Exists(null, null))
                {
                    memoryStream = new System.IO.MemoryStream();
                    file.DownloadToStream(memoryStream, null, null, null);
                    memoryStream.Position = 0;
                    var  sr  = new System.IO.StreamReader(memoryStream);
                    str  myStr = sr.ReadToEnd();

                    info(myStr);
                }
            }
        }

        CloudBlobClient     blobClient = storageAccount.CreateCloudBlobClient();

        CloudBlobContainer  blobContainer  = blobClient.GetContainerReference('divers');

        if(blobContainer.Exists(null, null))
        {
            CloudBlobDirectory rootDir = blobContainer.GetDirectoryReference('');

            CloudBlockBlob blockBlob = rootDir.GetBlockBlobReference('model.json');

            var myStr = blockBlob.DownloadText(null, null, null, null);


            info(myStr);

        }

    }

}
```

## Send file to azure Storage on attachment

Can be used for Sepa file export automatisation

``` x++
using Microsoft.Azure;
using Microsoft.WindowsAzure.Storage;
using Microsoft.WindowsAzure.Storage.Blob;
using Microsoft.WindowsAzure.Storage.File;

class ERDocuSubscriptionSample
{


    const str azureStorageAccount   = 'fjstorageaccount';
    const str azureStorageKey       = '6UJdCqFzyZSH1N+nlF8/SFK1rB1yQ9MKIr5PmAECsOBMpYBtpim5Mf1gpZ/oH7Doz0lOYhq1XxuzY7IupmSuTw==';
    const str azureFileShare        = 'sharetest';
    const str azureFileDirectory    = 'folder';


    [SubscribesTo(classStr(ERDocuManagementEvents),
    staticDelegateStr(ERDocuManagementEvents,
    attachingFile))]
    public static void ERDocuManagementEvents_attachingFile(ERDocuManagementAttachingFileEventArgs _args)
    {
        if (!_args.isHandled())
        {
            DocuType docuType = DocuType::find(_args.getDocuTypeId());
            if (strContains(docuType.Name, '(LOCAL)'))
            {
                _args.markAsHandled();
                var stream = _args.getStream();
                if (stream.CanSeek)
                {
                    stream.Seek(0, System.IO.SeekOrigin::Begin);
                }
                using (var localStream = System.IO.File::OpenWrite(@'c:\0\' + _args.getAttachmentName()))
                {
                    stream.CopyTo(localStream);
                }
            }

            if (strContains(docuType.Name, '(EXTAZURESTORAGE)'))
            {
                _args.markAsHandled();
                var stream = _args.getStream();
                if (stream.CanSeek)
                {
                    stream.Seek(0, System.IO.SeekOrigin::Begin);
                }
                System.IO.MemoryStream memoryStream;

                var storageCredentials = new Microsoft.WindowsAzure.Storage.Auth.StorageCredentials(azureStorageAccount, azureStorageKey);

                CloudStorageAccount storageAccount = new Microsoft.WindowsAzure.Storage.CloudStorageAccount(storageCredentials, true);

                CloudFileClient fileClient = storageAccount.CreateCloudFileClient();

                CloudFileShare share = fileClient.GetShareReference(azureFileShare);

                if (share.Exists(null, null))
                {
                    CloudFileDirectory rootDir = share.GetRootDirectoryReference();

                    CloudFileDirectory fileDir = rootDir.GetDirectoryReference(azureFileDirectory);

                    if (fileDir.Exists(null, null))
                    {
                        CloudFile file = fileDir.GetFileReference(strFmt('file%1.txt', newGuid()));
                        if (!file.Exists(null, null))
                        {
                            file.Create(0, null, null, null);
                            file.UploadFromStream(stream, null, null, null);
                        }
                    }
                }
            }
        }
    }

}
```
