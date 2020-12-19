## AWS EFS With Lambda Function

### EFS (Elastic File System)

Amazon Elastic File System [Amazon EFS](https://aws.amazon.com/efs/) provides a simple, scalable, fully managed elastic NFS file system for use with AWS Cloud services and on-premises resources. It is built to scale on demand to petabytes without disrupting applications, growing and shrinking automatically as you add and remove files, eliminating the need to provision and manage capacity to accommodate growth.

Amazon EFS is designed to provide massively parallel shared access to thousands of Amazon EC2 instances, enabling your applications to achieve high levels of aggregate throughput and IOPS with consistent low latencies.

## Why We Use EFS With Lambda

Serverless applications are event-driven, using ephemeral compute functions to integrate services and transform data. While AWS Lambda includes a 512-MB temporary file system for your code, this is an ephemeral scratch resource not intended for durable storage.

Amazon EFS is a fully managed, elastic, shared file system designed to be consumed by other AWS services, such as Lambda. With the release of Amazon EFS for Lambda, you can now easily share data across function invocations. You can also read large reference data files, and write function output to a persistent and shared store.

## How To Connect EFS With Lambda

### Step 1 (Create An EFS File System)

`   const myVpc = new ec2.Vpc(this, "Vpc", {
      maxAzs: 2,
    });
    const fileSystem = new efs.FileSystem(this, "lambdaEfsFileSystem", {
      vpc: myVpc
    });
`

Amazon Virtual Private Cloud (Amazon VPC) is a service that lets you launch AWS resources in a logically isolated virtual network that you define. You have complete control over your virtual networking environment, including selection of your own IP address range, creation of subnets, and configuration of route tables and network gateways.

A Virtual Private Cloud (VPC) is required to create an Amazon EFS file system.

### Step 2 (Creating An Access Ponit)
`
    const accessPoint = fileSystem.addAccessPoint("AccessPoint", {
      createAcl:{
        ownerGid: "1001",
        ownerUid: "1001",
        permissions: "750",
      },
      path:"/export/lambda",
      posixUser:{
        gid: "1001",
        uid: "1001",
      },
    });
`

Amazon EFS access points are application-specific entry points into an EFS file system that make it easier to manage application access to shared datasets. Access points can enforce a user identity, including the user's POSIX groups, for all file system requests that are made through the access point. Access points can also enforce a different root directory for the file system so that clients can only access data in the specified directory or its subdirectories.

### Step 3 (Creating a Lambda Function)
`
    const efsLambda = new lambda.Function(this, "efsLambdaFunction", {
      runtime: lambda.Runtime.NODEJS_12_X,
      code: lambda.Code.fromAsset("lambda"),
      handler: "msg.handler",
      vpc: myVpc,
      filesystem: lambda.FileSystem.fromEfsAccessPoint(accessPoint,"/mnt/msg"),
    });
`

You can configure a function to mount an Amazon Elastic File System (Amazon EFS) to a directory in your runtime environment with the filesystem property. To access Amazon EFS from lambda function, the Amazon EFS access point will be required.

This sample allows the lambda function to mount the Amazon EFS access point to /mnt/msg in the runtime environment and access the filesystem with the POSIX identity defined in posixUser.

### Step 4 (Creating an API)
`
    const api = new apigw.HttpApi(this, "Endpoint", {
      defaultIntegration: new integrations.LambdaProxyIntegration({
        handler: efsLambda,
      }),
    });
`

Lambda integrations enable integrating an HTTP API route with a Lambda function. When a client invokes the route, the API Gateway service forwards the request to the Lambda function and returns the function's response to the client.

<!-- To connect an EFS file system with a Lambda function, you use an EFS access point, an application-specific entry point into an EFS file system that includes the operating system user and group to use when accessing the file system, file system permissions, and can limit access to a specific path in the file system. This helps keeping file system configuration decoupled from the application code.

You can access the same EFS file system from multiple functions, using the same or different access points. For example, using different EFS access points, each Lambda function can access different paths in a file system, or use different file system permissions. -->
