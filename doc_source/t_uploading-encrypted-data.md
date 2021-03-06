# Uploading encrypted data to Amazon S3<a name="t_uploading-encrypted-data"></a>

Amazon S3 supports both server\-side encryption and client\-side encryption\. This topic discusses the differences between the server\-side and client\-side encryption and describes the steps to use client\-side encryption with Amazon Redshift\. Server\-side encryption is transparent to Amazon Redshift\. 

## Server\-side encryption<a name="server-side-encryption"></a>

Server\-side encryption is data encryption at rest—that is, Amazon S3 encrypts your data as it uploads it and decrypts it for you when you access it\. When you load tables using a COPY command, there is no difference in the way you load from server\-side encrypted or unencrypted objects on Amazon S3\. For more information about server\-side encryption, see [Using Server\-Side Encryption](https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingServerSideEncryption.html) in the *Amazon Simple Storage Service User Guide*\.

## Client\-side encryption<a name="client-side-encryption"></a>

In client\-side encryption, your client application manages encryption of your data, the encryption keys, and related tools\. You can upload data to an Amazon S3 bucket using client\-side encryption, and then load the data using the COPY command with the ENCRYPTED option and a private encryption key to provide greater security\.

You encrypt your data using envelope encryption\. With *envelope encryption,* your application handles all encryption exclusively\. Your private encryption keys and your unencrypted data are never sent to AWS, so it's very important that you safely manage your encryption keys\. If you lose your encryption keys, you won't be able to unencrypt your data, and you can't recover your encryption keys from AWS\. Envelope encryption combines the performance of fast symmetric encryption while maintaining the greater security that key management with asymmetric keys provides\. A one\-time\-use symmetric key \(the envelope symmetric key\) is generated by your Amazon S3 encryption client to encrypt your data, then that key is encrypted by your root key and stored alongside your data in Amazon S3\. When Amazon Redshift accesses your data during a load, the encrypted symmetric key is retrieved and decrypted with your real key, then the data is decrypted\.

To work with Amazon S3 client\-side encrypted data in Amazon Redshift, follow the steps outlined in [Protecting Data Using Client\-Side Encryption](https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingClientSideEncryption.html) in the *Amazon Simple Storage Service User Guide*, with the additional requirements that you use:
+ **Symmetric encryption –** The AWS SDK for Java `AmazonS3EncryptionClient` class uses envelope encryption, described preceding, which is based on symmetric key encryption\. Use this class to create an Amazon S3 client to upload client\-side encrypted data\.
+ **A 256\-bit AES root symmetric key –** A root key encrypts the envelope key\. You pass the root key to your instance of the `AmazonS3EncryptionClient` class\. Save this key, because you will need it to copy data into Amazon Redshift\.
+ **Object metadata to store encrypted envelope key –** By default, Amazon S3 stores the envelope key as object metadata for the `AmazonS3EncryptionClient` class\. The encrypted envelope key that is stored as object metadata is used during the decryption process\. 

**Note**  
If you get a cipher encryption error message when you use the encryption API for the first time, your version of the JDK may have a Java Cryptography Extension \(JCE\) jurisdiction policy file that limits the maximum key length for encryption and decryption transformations to 128 bits\. For information about addressing this issue, go to [Specifying Client\-Side Encryption Using the AWS SDK for Java](https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingClientSideEncryptionUpload.html) in the *Amazon Simple Storage Service User Guide*\. 

For information about loading client\-side encrypted files into your Amazon Redshift tables using the COPY command, see [Loading encrypted data files from Amazon S3](c_loading-encrypted-files.md)\.

## Example: Uploading client\-side encrypted data<a name="client-side-encryption-example"></a>

For an example of how to use the AWS SDK for Java to upload client\-side encrypted data, go to [Protecting data using client\-side encryption](https://docs.aws.amazon.com/AmazonS3/latest/dev/encrypt-client-side-symmetric-master-key.html) in the *Amazon Simple Storage Service User Guide*\. 

The second option shows the choices you must make during client\-side encryption so that the data can be loaded in Amazon Redshift\. Specifically, the example shows using object metadata to store the encrypted envelope key and the use of a 256\-bit AES root symmetric key\. 

This example provides example code using the AWS SDK for Java to create a 256\-bit AES symmetric root key and save it to a file\. Then the example upload an object to Amazon S3 using an S3 encryption client that first encrypts sample data on the client\-side\. The example also downloads the object and verifies that the data is the same\.