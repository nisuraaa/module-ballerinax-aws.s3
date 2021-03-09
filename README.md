[![Build Status](https://travis-ci.org/ballerina-platform/module-ballerinax-aws.s3.svg?branch=master)](https://travis-ci.org/ballerina-platform/module-ballerinax-aws.s3)
[![GitHub Last Commit](https://img.shields.io/github/last-commit/ballerina-platform/module-ballerinax-aws.s3.svg)](https://github.com/ballerina-platform/module-ballerinax-aws.s3./commits/master)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

# Ballerina Amazon S3 Client

The Amazon S3 client allows you to access the Amazon S3 REST API through Ballerina. The following sections provide the details on client operations.

## Compatibility
| Ballerina Language Version | Amazon S3 API version  |
| -------------------------- | ---------------------- |
|     Swan Lake Alpha2       |       2006-03-01       |


## Pull and Install

### Pull the Module
You can pull the Amazon S3 client from Ballerina Central:
```shell
$ bal pull ballerinax/aws.s3
```

### Install from Source
Alternatively, you can install AmazonS3 client from the source using the following instructions.

**Building the source**
1. Clone this repository using the following command:
    ```shell
    $ git clone https://github.com/ballerina-platform/module-amazons3.git
    ```

2. Run this command from the `module-amazons3` root directory:
    ```shell
    $ bal build
    ```

### Obtaining Access Keys

You need to get credentials such as **Access Key** and **Secret Access Key (API Secret)** from Amazon S3.

1. Create an Amazon account by visiting <https://aws.amazon.com/s3/>
2. Create a new access key, which includes a new secret access key.
- To create a new secret access key for your root account, use the [security credentials](https://console.aws.amazon.com/iam/home?#security_credential) page. Expand the Access Keys section, and then click **Create New Root Key**.
- To create a new secret access key for an IAM user, open the [IAM console](https://console.aws.amazon.com/iam/home?region=us-east-1#home). Click **Users** in the **Details** pane, click the appropriate IAM user, and then click **Create Access Key** on the **Security Credentials** tab.
3. Download the newly created credentials, when prompted to do so in the key creation wizard.

## Running Tests

1. Create `Config.toml` file in `module-ballerinax-aws.s3` with the following configurations and provide appropriate value.

    ```
    accessKeyId = "testAccessKeyValue"
    secretAccessKey = "testSecretAccessKeyValue"
    region = "testRegion"
    testBucketName = "testBucketName"
    ```

2. Navigate to the `module-ballerinax-aws.s3` directory.

3. Run tests :

    ```ballerina
    bal test
    ```

## Samples

#### Create a new Bucket

```ballerina
import ballerina/io;
import ballerinax/aws.s3;

configurable string accessKeyId = ?;
configurable string secretAccessKey = ?;
configurable string region = ?;
configurable string bucketName = ?;

s3:ClientConfiguration amazonS3Config = {
    accessKeyId: accessKeyId,
    secretAccessKey: secretAccessKey,
    region: region
};

s3:Client amazonS3Client = checkpanic new (amazonS3Config);

public function main() {
    s3:CannedACL cannedACL = s3:ACL_PRIVATE;
    s3:ConnectorError? createBucketResponse = amazonS3Client->createBucket(bucketName, cannedACL);
    if (createBucketResponse is s3:ConnectorError) {
        io:println("Error: ", createBucketResponse.message());
    } else {
        io:println("Bucket Creation Status: Success");
    }
}
```
#### List Buckets

```ballerina
import ballerina/io;
import ballerinax/aws.s3;

configurable string accessKeyId = ?;
configurable string secretAccessKey = ?;
configurable string region = ?;

s3:ClientConfiguration amazonS3Config = {
    accessKeyId: accessKeyId,
    secretAccessKey: secretAccessKey,
    region: region
};

s3:Client amazonS3Client = checkpanic new (amazonS3Config);

public function main() {
    var listBucketResponse = amazonS3Client->listBuckets();
    if (listBucketResponse is s3:Bucket[]) {
        io:println("Listing all buckets: ");
        foreach var bucket in listBucketResponse {
            io:println("Bucket Name: ", bucket.name);
        }
    } else {
        io:println("Error: ", listBucketResponse);
    }
}
```
#### Create a new Object

```ballerina
import ballerina/io;
import ballerinax/aws.s3;

configurable string accessKeyId = ?;
configurable string secretAccessKey = ?;
configurable string region = ?;
configurable string bucketName = ?;

s3:ClientConfiguration amazonS3Config = {
    accessKeyId: accessKeyId,
    secretAccessKey: secretAccessKey,
    region: region
};

s3:Client amazonS3Client = checkpanic new (amazonS3Config);

public function main() {
    s3:ConnectorError? createObjectResponse = amazonS3Client->createObject(bucketName, "test.txt", "Sample content");
    if (createObjectResponse is s3:ConnectorError) {
        io:println("Error: ", createObjectResponse.message());
    } else {
        io:println("Object created successfully");
    }
}
```

#### List Objects

```ballerina
import ballerina/io;
import ballerinax/aws.s3;

configurable string accessKeyId = ?;
configurable string secretAccessKey = ?;
configurable string region = ?;
configurable string bucketName = ?;

s3:ClientConfiguration amazonS3Config = {
    accessKeyId: accessKeyId,
    secretAccessKey: secretAccessKey,
    region: region
};

s3:Client amazonS3Client = checkpanic new (amazonS3Config);

public function main() returns error? {
    var listObjectsResponse = amazonS3Client->listObjects(bucketName);
    if (listObjectsResponse is s3:S3Object[]) {
        io:println("Listing all object: ");
        foreach var s3Object in listObjectsResponse {
            io:println("---------------------------------");
            io:println("Object Name: ", s3Object["objectName"]);
            io:println("Object Size: ", s3Object["objectSize"]);
        }
    } else {
        io:println("Error: ", listObjectsResponse);
    }
}
```
#### Get an Object

```ballerina
import ballerina/io;
import ballerina/lang.'string as strings;
import ballerinax/aws.s3;

configurable string accessKeyId = ?;
configurable string secretAccessKey = ?;
configurable string region = ?;
configurable string bucketName = ?;

s3:ClientConfiguration amazonS3Config = {
    accessKeyId: accessKeyId,
    secretAccessKey: secretAccessKey,
    region: region
};

s3:Client amazonS3Client = checkpanic new (amazonS3Config);

public function main() returns error? {
    var getObjectResponse = amazonS3Client->getObject(bucketName, "test.txt");
    if (getObjectResponse is s3:S3Object) {
        io:println(getObjectResponse);
        byte[]? byteArray = getObjectResponse["content"];
        if (byteArray is byte[]) {
            string content = check strings:fromBytes(byteArray);
            io:println("Object content: ", content);
        }
    } else {
        io:println("Error: ", getObjectResponse);
    }
}
```
#### Delete an Object

```ballerina
import ballerina/io;
import ballerinax/aws.s3;

configurable string accessKeyId = ?;
configurable string secretAccessKey = ?;
configurable string region = ?;
configurable string bucketName = ?;

s3:ClientConfiguration amazonS3Config = {
    accessKeyId: accessKeyId,
    secretAccessKey: secretAccessKey,
    region: region
};

s3:Client amazonS3Client = checkpanic new(amazonS3Config);

public function main() {
    s3:ConnectorError? deleteObjectResponse = amazonS3Client->deleteObject(bucketName, "test.txt");
    if (deleteObjectResponse is s3:ConnectorError) {
        io:println("Error: ", deleteObjectResponse.message());
    } else {
        io:println("Successfully deleted object");
    }
}
```

#### Delete a Bucket

```ballerina
import ballerina/io;
import ballerinax/aws.s3;

configurable string accessKeyId = ?;
configurable string secretAccessKey = ?;
configurable string region = ?;
configurable string bucketName = ?;

s3:ClientConfiguration amazonS3Config = {
    accessKeyId: accessKeyId,
    secretAccessKey: secretAccessKey,
    region: region
};

s3:Client amazonS3Client = checkpanic new (amazonS3Config);

public function main() {
    s3:ConnectorError? deleteBucketResponse = amazonS3Client->deleteBucket(bucketName);
    if (deleteBucketResponse is s3:ConnectorError) {
        io:println("Error: ", deleteBucketResponse.message());
    } else {
        io:println("Successfully deleted bucket");
    }
}
```

## How you can contribute

As an open source project, we welcome contributions from the community. Check the [issue tracker](https://github.com/ballerina-platform/module-amazons3/issues) for open issues that interest you. We look forward to receiving your contributions.
