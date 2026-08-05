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

### Multipart upload ETag

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

### GET bucket metadata search

`search` (Scality) and `query` (Dell) parameters filter a listing with a
SQL-like predicate over object metadata instead of only by key prefix.  Dell
also accepts `attributes`, `sorted`, and `include-older-versions` and requires
declaring the searchable keys on the bucket.

* [Dell documentation](https://portal.ecstestdrive.com/Document/ECS/3.8/apidocs/S3MetadataSearchOperation_getObjectsList_f8907356ba2c22b312b538e2a72efeb5_ba672412ac371bb6cf4e69291344510e_detail.html)
* [Scality documentation](https://zenko.readthedocs.io/en/latest/operation/Metadata_Search/Metadata_Search.html)

### GET bucket unordered

Ceph `allow-unordered=true` parameter overrides key sorting which allows
higher performance with large buckets.  Cannot be used with delimiter.

* [Ceph documentation](https://docs.ceph.com/en/latest/radosgw/s3/bucketops/#get-bucket)

### GET bucket with metadata

Minio `metadata=true` parameter includes each object's user metadata in the
listing via `UserMetadata` elements, avoiding a HEAD object request per key.
It strips internal reserved keys and omits the metadata for objects which the
caller lacks permission to read.

* [Minio SDK documentation](https://pkg.go.dev/github.com/minio/minio-go/v7#ListObjectsOptions)

### GET object generation

Google `generation` parameter selects an object version while
`x-goog-generation` and `x-goog-metageneration` responses identify it.
Generations are large integers rather than opaque strings and metageneration
counts metadata changes separately, resetting when the content is replaced.
`generation-marker` paginates version listings.

* [Google Cloud Storage documentation](https://docs.cloud.google.com/storage/docs/object-versioning)

### GET object media processing

`x-oss-process` (Aliyun), `ci-process` (Tencent), and `x-image-process`
(Huawei) parameters transform an object while serving it, e.g.,
`?x-oss-process=image/resize,w_100`.  Aliyun supports only the `image/` and
`style/` operations via the S3 protocol.

* [Aliyun documentation](https://www.alibabacloud.com/help/en/oss/developer-reference/compatibility-with-amazon-s3)
* [Huawei documentation](https://support.huaweicloud.com/intl/en-us/fg-obs/obs_01_0670.html)
* [Tencent documentation](https://www.tencentcloud.com/document/product/436/35280)

### GET object transcoded

Google decompresses objects stored with `Content-Encoding: gzip`, omitting
`Content-Length` and `Content-Encoding` from the response and invalidating
checksums.  `x-goog-stored-content-length` and `x-goog-stored-content-encoding`
report what is actually stored.  `Accept-Encoding: gzip` or a `no-transform`
`Cache-Control` serves the stored bytes instead.

* [Google Cloud Storage documentation](https://docs.cloud.google.com/storage/docs/transcoding)

### GET service pagination

Cloudflare `cf-prefix`, `cf-start-after`, `cf-continuation-token`, and
`cf-max-keys` headers paginate the bucket listing and take precedence over the
equivalent parameters.  Responses include `cf-is-truncated` and
`cf-next-continuation-token`.  Amazon later added `prefix`,
`continuation-token`, `max-buckets`, and `bucket-region` parameters but has no
equivalent of `cf-start-after`.

* [Cloudflare documentation](https://developers.cloudflare.com/r2/api/s3/extensions/)

### GET service project

Google `x-goog-project-id` header scopes listing and creating buckets to a
project unless the account has a default project for interoperable access.
Amazon scopes these to the credentials instead.

* [Google Cloud Storage documentation](https://docs.cloud.google.com/storage/docs/xml-api/get-service)

### HEAD bucket extended

Ceph returns bucket statistics via `x-rgw-bytes-used` and `x-rgw-object-count`
headers.  Newer versions return these only when the request includes the
`read-stats` parameter, since computing them reads every bucket index shard.

* [Ceph feature #2313](https://tracker.ceph.com/issues/2313)
* [Ceph pull request #56756](https://github.com/ceph/ceph/pull/56756)

### Object checksums

`x-goog-hash` header carries `crc32c=` and `md5=` base64 values which Google
validates on upload and returns on download.  Only CRC32C is available for
multipart uploads.

* [Google Cloud Storage documentation](https://docs.cloud.google.com/storage/docs/data-validation)

### PUT object append

Aliyun and Ceph append to an existing object at a given offset via the
`append` and `position` parameters.  Responses include
`x-oss-next-append-position` and `x-rgw-next-append-position` which indicate
the next valid append offset.
Amazon added this in November 2024 via the `x-amz-write-offset-bytes` header,
returning the resulting size via `x-amz-object-size`, although only for the S3
Express One Zone storage class in directory buckets.  Dell appends via a
`Range` header instead.

* [Aliyun documentation](https://www.alibabacloud.com/help/en/oss/developer-reference/appendobject)
* [Amazon append](https://aws.amazon.com/about-aws/whats-new/2024/11/amazon-s3-express-one-zone-append-data-object/)
* [Amazon PutObject](https://docs.aws.amazon.com/AmazonS3/latest/API/API_PutObject.html)
* [Ceph documentation](https://docs.ceph.com/en/latest/radosgw/s3/objectops/#append-object)

### PUT object byte range

Dell `Range` header on PUT modifies part of an object instead of replacing it:
`bytes=10-14` updates that range, `bytes=10-` overwrites from that offset, and
`bytes=-1-` atomically appends without knowing the current size.  Responses
include `x-emc-previous-object-size` and `x-emc-content-md5`.  This differs
from the `?append&position=` model which Aliyun and Ceph use.

* [Dell documentation](https://portal.ecstestdrive.com/Document/ECS/3.8/apidocs/S3ObjectOperations_createOrUpdateObject_7916bd6f789d0ae0ff39961c0e660d00_ba672412ac371bb6cf4e69291344510e_detail.html)

### PUT object callback

`x-oss-callback` and `x-oss-callback-var` headers carry base64-encoded JSON
which makes OSS POST to an application server after a successful upload and
return that server's response to the client instead of its own.  PUT object,
POST object, and complete multipart upload accept these.

* [Aliyun documentation](https://www.alibabacloud.com/help/en/oss/developer-reference/callback)

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
* [Amazon conditional writes](https://aws.amazon.com/about-aws/whats-new/2024/08/amazon-s3-conditional-writes/)
* [Amazon conditional deletes](https://aws.amazon.com/about-aws/whats-new/2025/09/amazon-s3-conditional-deletes-s3-general-purpose-buckets/)
* [Ceph feature #8562](https://tracker.ceph.com/issues/8562)
* [Google Cloud Storage documentation](https://docs.cloud.google.com/storage/docs/request-preconditions)
* [Tencent documentation](https://www.tencentcloud.com/document/product/436/7749)

### PUT object copy conditional

Cloudflare `cf-copy-destination-if-match`, `cf-copy-destination-if-none-match`,
`cf-copy-destination-if-modified-since`, and
`cf-copy-destination-if-unmodified-since` headers condition a copy on the
destination object rather than the source, failing with 412 when unmet.

* [Cloudflare documentation](https://developers.cloudflare.com/r2/api/s3/extensions/)

### PUT object copy metadata merge

Cloudflare `x-amz-metadata-directive: MERGE` copies the source metadata and
replaces only the keys present in the request, unlike the all-or-nothing
`COPY` and `REPLACE`.

* [Cloudflare documentation](https://developers.cloudflare.com/r2/api/s3/extensions/)

### PUT object copy source

Google `x-goog-copy-source` header names the source object while
`x-goog-copy-source-generation` selects its version.
`x-goog-copy-source-if-generation-match` and
`x-goog-copy-source-if-metageneration-match` condition the copy on the source
version rather than on its ETag.

* [Google Cloud Storage documentation](https://docs.cloud.google.com/storage/docs/xml-api/reference-headers)

### PUT object create bucket

Cloudflare `cf-create-bucket-if-missing: true` header creates the bucket when
it does not already exist, avoiding a separate PUT bucket request.  Also
applies to initiate multipart upload.

* [Cloudflare documentation](https://developers.cloudflare.com/r2/api/s3/extensions/)

### PUT object extract

Minio `X-Amz-Meta-Snowball-Auto-Extract: true` header unpacks an uploaded tar
archive into individual objects rather than storing the archive itself.  It
also accepts tar compressed with gzip, bzip2, zstd, lz4, or S2.
`X-Amz-Meta-Minio-Snowball-Prefix` prefixes the extracted keys,
`X-Amz-Meta-Minio-Snowball-Ignore-Dirs` skips empty directory objects, and
`X-Amz-Meta-Minio-Snowball-Ignore-Errors` continues past individual failures.

* [Minio blog](https://www.min.io/blog/minio-optimizes-small-objects)

### PUT object symlink

Aliyun `symlink` subresource creates an object which redirects to another via
the `x-oss-symlink-target` header, which GET object follows and GET symlink
returns.  OSS does not validate that the target exists and symlinks cannot
point to other symlinks.

* [Aliyun documentation](https://www.alibabacloud.com/help/en/oss/developer-reference/putsymlink)

### PUT object unicode metadata

Cloudflare encodes non-ASCII `x-amz-meta-` keys and values with RFC 2047,
which Amazon leaves unspecified.

* [Cloudflare documentation](https://developers.cloudflare.com/r2/api/s3/extensions/)

### Resumable uploads

Initiate a Google resumable upload with POST object with the `x-goog-resumable`
header.  Use the response `Location` to issue a PUT object with given
Content-Length and Content-Range headers.  Continuing issuing these requests to
build larger objects.

* [Google Cloud Storage documentation](https://docs.cloud.google.com/storage/docs/resumable-uploads)

### Small file archives

Minio allows interaction with ZIP file contents as if they were objects.
HEAD object, GET object, and GET bucket support these operations when the
header `x-minio-extract` has a `true` value.

* [Minio documentation](https://www.min.io/blog/small-file-archives)

## References

* [Aliyun API documentation](https://partners-intl.aliyun.com/help/en/doc-detail/31947.htm)
* [Amazon API documentation](https://docs.aws.amazon.com/AmazonS3/latest/API/Welcome.html)
* [Ceph API documentation](https://docs.ceph.com/en/latest/radosgw/s3/)
* [Google API documentation](https://docs.cloud.google.com/storage/docs/xml-api/overview)
