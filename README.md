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

### DELETE bucket force

`x-minio-force-delete: true` header deletes a bucket and its contents, which
Minio implements by renaming the bucket rather than deleting each object.
Requires the separate `ForceDeleteBucketAction` permission and rejects buckets
with object lock or active replication rules.

* [Minio discussion #20367](https://github.com/minio/minio/discussions/20367)

### GET bucket unordered

`allow-unordered=true` parameter overrides key sorting which allows higher
performance with large buckets.  Cannot be used with delimiter.

* [Ceph documentation](http://docs.ceph.com/docs/master/radosgw/s3/bucketops/#get-bucket)

### GET bucket with metadata

`metadata=true` parameter includes each object's user metadata in the listing
via `UserMetadata` elements, avoiding a HEAD object request per key.  Minio
strips its internal reserved keys and omits the metadata for objects which the
caller lacks permission to read.

* [Minio SDK documentation](https://pkg.go.dev/github.com/minio/minio-go/v7#ListObjectsOptions)

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

`If-Match` and `If-None-Match` (Ceph) and `x-goog-if-generation-match` and
`x-goog-if-metageneration-match` (Google) headers allow atomic replacement of
objects.  `x-oss-forbid-overwrite` (Aliyun) and `x-cos-forbid-overwrite`
(Tencent) forbid overwriting an existing object.  Amazon standardized part of
this in August 2024 for PUT object and complete multipart upload and in
September 2025 for DELETE object.  Amazon `If-None-Match` accepts only `*`
while Ceph also accepts an ETag, and Google conditions on the object generation
and metageneration instead of the ETag.

* [Aliyun documentation](https://www.alibabacloud.com/help/en/oss/developer-reference/prevent-objects-from-being-overwritten-by-objects-that-have-the-same-names-4)
* [Amazon conditional writes](https://aws.amazon.com/about-aws/whats-new/2024/08/amazon-s3-conditional-writes)
* [Amazon conditional deletes](https://aws.amazon.com/about-aws/whats-new/2025/09/amazon-s3-conditional-deletes-s3-general-purpose-buckets)
* [Ceph feature #8562](https://tracker.ceph.com/issues/8562)
* [Google Cloud Storage documentation](https://cloud.google.com/storage/docs/generations-preconditions#_XMLAPI)
* [Tencent documentation](https://www.tencentcloud.com/document/product/436/7749)

### PUT object extract

`X-Amz-Meta-Snowball-Auto-Extract: true` header unpacks an uploaded tar archive
into individual objects rather than storing the archive itself.  Minio also
accepts tar compressed with gzip, bzip2, zstd, lz4, or S2.
`X-Amz-Meta-Minio-Snowball-Prefix` prefixes the extracted keys,
`X-Amz-Meta-Minio-Snowball-Ignore-Dirs` skips empty directory objects, and
`X-Amz-Meta-Minio-Snowball-Ignore-Errors` continues past individual failures.

* [Minio blog](https://min.io/blog/minio-optimizes-small-objects)

### Resumable uploads

Initiate a resumable upload with POST object with the `x-goog-resumable`
header.  Use the response `Location` to issue a PUT object with given
Content-Length and Content-Range headers.  Continuing issuing these requests to
build larger objects.

* [Google Cloud Storage documentation](https://cloud.google.com/storage/docs/xml-api/resumable-upload)

### Small file archives

Allows interaction with ZIP file contents as if they were objects.  HeadObject,
GetObject, and ListObjectsV2 support these operations when the header
`x-minio-extract` has a `true` value.

* [Minio documentation](https://blog.min.io/small-file-archives/)

## References

* [Aliyun API documentation](https://partners-intl.aliyun.com/help/doc-detail/31947.htm?spm=a2c63.p38356.b99.563.3d3152e7qsZxhf)
* [Amazon API documentation](https://docs.aws.amazon.com/AmazonS3/latest/API/Welcome.html)
* [Ceph API documentation](http://docs.ceph.com/docs/master/radosgw/s3/)
* [Google API documentation](https://cloud.google.com/storage/docs/xml-api/overview)
