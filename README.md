# Undocumented S3 APIs

Undocumented Amazon S3 APIs and third-party extensions.  This documents
features outside the official
[Amazon S3 REST APIs](https://docs.aws.amazon.com/AmazonS3/latest/API/Welcome.html).
Note that these are not consistently supported across implementations.

## Amazon APIs

### GET object by multipart number

Object metadata exposes the number of multipart upload parts via the
`x-amz-mp-parts-count` header.  Subsequent GET object requests can retrieve
individual parts via the `partNumber` parameter.  This allows retrieving
individual multipart upload parts, including the original ETag.

* [AWS Java SDK partNumber](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/s3/model/GetObjectMetadataRequest.html#withPartNumber-java.lang.Integer-)
* [AWS Java SDK x-amz-mp-parts-count](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/s3/model/ObjectMetadata.html#getPartCount--)

### HEAD bucket exposes region

HEAD bucket exposes the region via `x-amz-bucket-region` even if the request
lacks authentication.  This allows clients to interrogate a bucket so that they
can route subsequent requests to the correct endpoint.

### Multipart Upload ETag

Single part upload ETag are the MD5 hash of the object content.  Multipart
upload ETag concatenate the MD5 of each part, MD5 this data, and append a
hyphen and the number of parts.

* [Stack Overflow answer](https://stackoverflow.com/questions/12186993/what-is-the-algorithm-to-compute-the-amazon-s3-etag-for-a-file-larger-than-5gb)

## Third-party APIs

### GET bucket unordered

`allow-unordered=true` parameter overrides key sorting which allows higher
performance with large buckets.  Cannot be used with delimiter.

* [Ceph documentation](http://docs.ceph.com/docs/master/radosgw/s3/bucketops/#get-bucket)

### HEAD bucket extended

Returns bucket statistics via `x-rgw-bytes-used` and `x-rgw-object-count`
headers.

* [Ceph feature #2313](https://tracker.ceph.com/issues/2313)

### PUT object append

Append to an existing object at a given offset via the `append` and `position`
parameters.  Responses include `x-oss-next-append-position` and
`x-rgw-next-append-position` which indicate the next valid append offset.

* [Aliyun documentation](https://partners-intl.aliyun.com/help/doc-detail/31981.htm?spm=a2c63.p38356.b99.595.5783438dGk)
* [Ceph documentation](http://docs.ceph.com/docs/master/radosgw/s3/objectops/#append-object)

### PUT object conditional

`If-Match` and `If-None-Match` headers allow atomic replacement of objects.

* [Ceph feature #8562](https://tracker.ceph.com/issues/8562)

### Resumable uploads

Initiate a resumable upload with POST object with the `x-goog-resumable`
header.  Use the response `Location` to issue a PUT object with given
Content-Length and Content-Range headers.  Continuing issuing these requests to
build larger objects.

* [Google Cloud Storage documentation](https://cloud.google.com/storage/docs/xml-api/resumable-upload)

## References

* [Aliyun API documentation](https://partners-intl.aliyun.com/help/doc-detail/31947.htm?spm=a2c63.p38356.b99.563.3d3152e7qsZxhf)
* [Amazon API documentation](https://docs.aws.amazon.com/AmazonS3/latest/API/Welcome.html)
* [Ceph API documentation](http://docs.ceph.com/docs/master/radosgw/s3/)
* [Google API documentation](https://cloud.google.com/storage/docs/xml-api/overview)
