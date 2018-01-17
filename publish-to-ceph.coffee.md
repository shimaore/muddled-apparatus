Ceph implements S3

    seem = require 'seem'
    AWS = require 'aws-sdk'

    ceph = JSON.parse process.env.CEPH_JSON

    s3 = new AWS.S3
      accessKeyId: ceph.key
      secretAccessKey: ceph.secret
      endpoint: ceph.endpoint
      useDualStack: true
      sslEnabled: true
      s3ForcePathStyle: true
      signatureVersion: 'v2'

    log = (x) ->
      console.log JSON.stringify x

    do seem ->

      bucket_params =
        Bucket: ceph.bucket
        CreateBucketConfiguration:
          LocationConstraint: ''

      log yield s3.createBucket(bucket_params).promise()
      # log yield s3.getBucketAcl(Bucket:ceph.bucket).promise()
      # log yield s3.listObjectsV2(Bucket:ceph.bucket).promise()

      content =
        Bucket: ceph.bucket
        Key: process.env.CEPH_LOCATION
        Body: process.stdin
        ContentType: process.env.CEPH_CONTENT_TYPE
        ACL: 'public-read'

      log yield s3.upload(content, {}).promise()
