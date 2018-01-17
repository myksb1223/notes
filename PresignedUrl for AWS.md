## Presigned url with Fresco for android and objective-c for HJImageCache

Recently, I have used aws `access_key` and `secret_key` when I connected to S3. But it's really bad and dangerous way. If I am hacked these keys, all datas will be deleted or changed by hacker. So I update this code using recent aws api.

But I got some problems because a way of making presigned url with `Fresco` was also changed.

### Android
In previous version, I have used `GetS3ObjectSample` for creating presigned url.

```java
Map<String, String> headers = GetS3ObjectSample.getS3Object(originalRequest.urlString(), ClassUpApplication.awsAccessKey, ClassUpApplication.awsSecretKey);

com.squareup.okhttp.Request requestWithToken = originalRequest.newBuilder()
    .addHeader("x-amz-content-sha256", headers.get("x-amz-content-sha256"))
    .addHeader("Authorization", headers.get("Authorization"))
    .addHeader("x-amz-date", headers.get("x-amz-date"))
    .addHeader("Host", headers.get("Host")).build();
```

Now, I use updated aws api.
**`GeneratePresignedUrlRequest`** can not be used in main thread.

```java
Date expiration = new java.util.Date();
long msec = expiration.getTime();
msec += 1000 * 60 * 60; // Add 1 hour.
expiration.setTime(msec);

GeneratePresignedUrlRequest generatePresignedUrlRequest = new GeneratePresignedUrlRequest(bucket, object_key);
generatePresignedUrlRequest.setMethod(HttpMethod.GET);
generatePresignedUrlRequest.setExpiration(expiration);
ps = Util.getS3Client(ClassUpApplication.this).generatePresignedUrl(generatePresignedUrlRequest);

com.squareup.okhttp.Request requestWithToken = originalRequest.newBuilder().url(ps).build();
```

### Objective-c

In previous version, I have used `AWS4SignerForQueryParameterAuth` to make headers and params of presigned url.

```objective-c
NSMutableDictionary *queryParams = [[[NSMutableDictionary alloc] init] autorelease];

NSInteger expiresIn = 7 * 24 * 60 * 60;
[queryParams setObject:[NSString stringWithFormat:@"%ld", (long)expiresIn] forKey:@"X-Amz-Expires"];

NSMutableDictionary *headers = [[[NSMutableDictionary alloc] init] autorelease];
AWS4SignerForQueryParameterAuth *signer = [[[AWS4SignerForQueryParameterAuth alloc] initWithUrl:endpointUrl httpMethod:@"GET" serviceName:@"s3" regionName:regionName] autorelease];
NSString *authorizationQueryParameters = [signer computeSignatureWithHeaders:headers queryParameters:queryParams bodyHash:UNSIGNED_PAYLOAD awsAccessKey:awsAccessKey awsSecretKey:awsSecretKey];

NSString *presignedUrl = [NSString stringWithFormat:@"%@?%@", [endpointUrl absoluteString], authorizationQueryParameters];
```

Now, I use updated aws api.

```objective-c
AWSS3GetPreSignedURLRequest *getPreSignedURLRequest = [AWSS3GetPreSignedURLRequest new];
getPreSignedURLRequest.bucket = [NSString stringWithFormat:@"classup/newNoteImages/%@", image_key];
getPreSignedURLRequest.key = [NSString stringWithFormat:@"%@_noteImage.jpeg", image_key];
getPreSignedURLRequest.HTTPMethod = AWSHTTPMethodGET;
getPreSignedURLRequest.expires = [NSDate dateWithTimeIntervalSinceNow:3600];

AWSTask<NSURL *> *task = [[[AWSS3PreSignedURLBuilder defaultS3PreSignedURLBuilder] getPreSignedURL:getPreSignedURLRequest] continueWithBlock:^id _Nullable(AWSTask<NSURL *> * _Nonnull task) {
    return task;
}];

NSURL *presignedURL = task.result;
```

Now, I can hide `access_key` and `secret_key` in `ClassUp`.
