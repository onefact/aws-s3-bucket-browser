# AWS S3 Bucket Browser [![Sparkline](https://stars.medv.io/qoomon/aws-s3-bucket-browser.svg)](https://stars.medv.io/qoomon/aws-s3-bucket-browser)
![-](favicon.ico)

Single HTML file to browse AWS S3 buckets
## [Demo](https://qoomon.github.io/aws-s3-bucket-browser/index.html?bucket=https://s3.amazonaws.com/spacenet-dataset#spacenet/)

## Features
* List all files in a table view
* Treat and display `/` in keys as folders
* Render preview for Makrdown files
* Show `Install` button for `manifest.plist` on iOS devices

## Installation

#### Self-Hosted
* Just download [`index.html`](index.html) and upload it to your bucket.
  * Adjust [config](index.html#L8-L37) within `index.html` if needed, e.g.
    ```js
    const config = {
      title: 'Bucket Browser', // prefix value with `HTML> ` to render as html, see subtitle
      subtitle: 'HTML>made with ♥ by <b><a href="https://qoo.monster">qoomon</a></b>', // prefix value with `HTML> ` to render as html
      logo: 'https://qoomon.github.io/aws-s3-bucket-browser/logo.png',
      favicon: 'https://qoomon.github.io/aws-s3-bucket-browser/favicon.ico',
      primaryColor: '#167df0',
      
      bucketUrl: undefined,
      // If bucketUrl is undefined, this script tries to determine bucket Rest API URL from this file location itself.
      //   This will only work for locations like these
      //   * https://s3.BUCKET-REGION.amazonaws.com/BUCKET-NAME/index.html
      //   * http://BUCKET-NAME.s3-website-BUCKET-REGION.amazonaws.com/index.html
      //   * https://storage.googleapis.com/BUCKET-NAME/index.html
      //   * https://BUCKET-NAME.s3-web.BUCKET-REGION.cloud-object-storage.appdomain.cloud/
      // If bucketUrl is set manually, ensure this is the bucket Rest API URL, e.g.
      //   * https://s3.BUCKET-REGION.amazonaws.com/BUCKET-NAME
      //   * https://storage.googleapis.com/BUCKET-NAME
      //   The URL should return an XML document with <ListBucketResult> as root element.
      rootPrefix: undefined, // e.g. 'subfolder/'
      keyExcludePatterns: [/^index\.html$/], // matches againt object key relative to rootPrefix
      pageSize: 50,
      
      bucketMaskUrl: undefined, 
      // If bucketMaskUrl is set file urls will be changed from ${bucketUrl}/${file} to ${bucketMaskUrl}/${file}
      //   bucketMaskUrl: 'https://example.org'
      //     => https://example.org/foo/bar.txt 
      //   bucketMaskUrl: document.location.origin
      //     => ${document.location.origin}/foo/bar.txt 
      
      defaultOrder: 'name-asc' // (name|size|dateModified)-(asc|desc)
    }
    ```
* ##### ⚠️ Ensure Bucket Permissions
  * Go to `https://s3.console.aws.amazon.com/s3/buckets/<YOUR BUCKET NAME>/?tab=permissions`
  * Grant public read permission by `Access Control List` or `Bucket Policy`
    * Access Control List
      * Enable `List objects` for `Everyone`
    * Bucket Policy
      ```json
      {
          "Version": "2012-10-17",
          "Statement": [
              {
                  "Sid": "PublicRead",
                  "Principal": "*",
                  "Effect": "Allow",
                  "Action": [
                      "s3:ListBucket",
                      "s3:GetObject"
                  ],
                  "Resource": [
                      "arn:aws:s3:::<YOUR BUCKET NAME>",
                      "arn:aws:s3:::<YOUR BUCKET NAME>/*"
                  ]
              }
          ]
      }
      ```
* ##### ⚠️ Ensure Bucket CORS
  * Depending on your setup you may need need to ensure following `CORS Configuration`
  * Go to `https://s3.console.aws.amazon.com/s3/buckets/<YOUR BUCKET NAME>/?tab=permissions`
  * Grant Cross Origin Access by following `CORS Configuration`, replace `http://www.example.com` by your address of bucket browser `index.html` 
    * e.g `http://example.s3-website-eu-central-1.amazonaws.com/index.html`
    ```json
    [
      {
          "AllowedHeaders": [
              "*"
          ],
          "AllowedMethods": [
              "GET"
          ],
          "AllowedOrigins": [
              "http://www.example.com"
          ],
          "ExposeHeaders": [
              "x-amz-server-side-encryption",
              "x-amz-request-id",
              "x-amz-id-2"
          ],
          "MaxAgeSeconds": 3000
      }
    ]
    ```
* Open `<YOUR BUCKET URL>/index.html` in your browser.

#### Hosted
* ##### ⚠️ Ensure Bucket Permissions
  * see [Self-Hosted](#self-hosted)
* ##### ⚠️ Ensure Bucket CORS
  * see [Self-Hosted](#self-hosted)
* Open hosted `index.html` in your browser and provide bucket url as `bucket` request parameter
  * `${INDEX_FILE_LOCATION}?bucket=${S3_BUCKET_URL}` 
  * e.g. [`https://qoomon.github.io/aws-s3-bucket-browser/index.html?bucket=https://s3.eu-west-1.amazonaws.com/data.openspending.org`](https://qoomon.github.io/aws-s3-bucket-browser/index.html?bucket=https://s3.eu-west-1.amazonaws.com/data.openspending.org)


### CloudFront Setup
If you use CloudFront in upfront of your S3 bucket ensure following CloudFront settings.
- Allowed/Cached HTTP Methods: `GET`, `HEAD`, `OPTIONS`
- Cached Based on Selected Headers: `Whitelist`
  - `Access-Control-Request-Headers`
  - `Access-Control-Request-Method`
  - `Origin`
- Query String Forwarding and Caching: `Forward all`

<img width="910" alt="image" src="https://user-images.githubusercontent.com/5317244/226177533-206d915e-3609-440a-8f8f-7883e3d91261.png">

<img width="548" alt="image" src="https://user-images.githubusercontent.com/5317244/226177565-a8ff56fc-25d5-4708-934e-76f1ccd86004.png">


Alternate domain name:

<img width="662" alt="image" src="https://user-images.githubusercontent.com/5317244/226175872-95a53018-0087-47fb-be87-294c0b931126.png">


### IBM Cloud Object Storage Setup
IBM Cloud Object storage only supports virtual host-style addressing, i.e. `https://<bucket-name>s3-web.<region>.cloud-object-storage.appdomain.cloud/` for static website hosting. Otherwise follow the instructions
in this [tutorial](https://cloud.ibm.com/docs/cloud-object-storage?topic=cloud-object-storage-static-website-tutorial) to configure your bucket. In addition, you may need to [configure CORS](https://cloud.ibm.com/docs/cloud-object-storage?topic=cloud-object-storage-curl#curl-new-cors) for your bucket.

```xml
<CORSConfiguration>
  <CORSRule>
      <AllowedOrigin>*</AllowedOrigin>
      <AllowedMethod>GET</AllowedMethod>
      <AllowedHeader>*</AllowedHeader>
  </CORSRule>
</CORSConfiguration>
```

## Configuration specific to data.payless.health
1. Create SSL certificate in AWS Certificate Manager for `data.payless.health`
- Need to validate domain ownership using DNS with name and value from AWS Certificate Manager
2. Create Cloudfront distribution for `s3://payless.health` bucket.
3. Add `CNAME` record in DNS for `data.payless.health` to Cloudfront distribution domain name.

Default root object - optional: `index.html`. This is the default object that CloudFront returns when the viewer requests the root URL for your distribution. If you don't specify a default root object, CloudFront returns an HTTP 403 (Forbidden) error.

Invalidate the cache if Cloudfront caching causes errors: `aws cloudfront create-invalidation --distribution-id {$DISTRIBUTION_ID}
 --paths "/index.html"` or doing it on the web UI. 
 
Finally, ensure `Origin request policy` is set to `UserAgentRefererHeaders`.

 
