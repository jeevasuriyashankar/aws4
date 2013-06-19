aws4
----

[![Build Status](https://secure.travis-ci.org/mhart/aws4.png?branch=master)](http://travis-ci.org/mhart/aws4)

A small utility to sign vanilla node.js http(s) request options using Amazon's
[AWS Signature Version 4](http://docs.amazonwebservices.com/general/latest/gr/signature-version-4.html).

This signature is supported by an increasing number of Amazon services, including
[SQS](http://docs.aws.amazon.com/AWSSimpleQueueService/latest/APIReference/),
[SNS](http://docs.aws.amazon.com/sns/latest/api/),
[IAM](http://docs.aws.amazon.com/IAM/latest/APIReference/),
[STS](http://docs.aws.amazon.com/STS/latest/APIReference/),
[DynamoDB](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/API.html),
[RDS](http://docs.aws.amazon.com/AmazonRDS/latest/APIReference/),
[CloudWatch](http://docs.aws.amazon.com/AmazonCloudWatch/latest/APIReference/),
[Glacier](http://docs.aws.amazon.com/amazonglacier/latest/dev/amazon-glacier-api.html),
[CloudSearch](http://docs.aws.amazon.com/cloudsearch/latest/developerguide/APIReq.html),
[Elastic Load Balancing](http://docs.aws.amazon.com/ElasticLoadBalancing/latest/APIReference/),
[CloudFormation](http://docs.aws.amazon.com/AWSCloudFormation/latest/APIReference/),
[Elastic Beanstalk](http://docs.aws.amazon.com/elasticbeanstalk/latest/api/),
[Storage Gateway](http://docs.aws.amazon.com/storagegateway/latest/userguide/AWSStorageGatewayAPI.html),
[Data Pipeline](http://docs.aws.amazon.com/datapipeline/latest/APIReference/),
[Direct Connect](http://docs.aws.amazon.com/directconnect/latest/APIReference/),
[Redshift](http://docs.aws.amazon.com/redshift/latest/APIReference/),
[OpsWorks](http://docs.aws.amazon.com/opsworks/latest/APIReference/),
[SES](http://docs.aws.amazon.com/ses/latest/APIReference/) and
[AutoScaling](http://docs.aws.amazon.com/AutoScaling/latest/APIReference/).

It also provides defaults for a number of core AWS headers and
request parameters, making it a very easy to query AWS services, or
build out a fully-featured AWS library.

Example
-------

```javascript
var http  = require('http')
  , https = require('https')
  , aws4  = require('aws4')

// given an options object you could pass to http.request
var opts = { host: 'sqs.us-east-1.amazonaws.com', path: '/?Action=ListQueues' }

aws4.sign(opts) // assumes AWS credentials are available in process.env

console.log(opts)
/*
{
  host: 'sqs.us-east-1.amazonaws.com',
  path: '/?Action=ListQueues',
  headers: {
    Host: 'sqs.us-east-1.amazonaws.com',
    'X-Amz-Date': '20121226T061030Z',
    Authorization: 'AWS4-HMAC-SHA256 Credential=ABCDEF/20121226/us-east-1/sqs/aws4_request, ...'
  }
}
*/

// we can now use this to query AWS using the standard node.js http API
http.request(opts, function(res) { res.pipe(process.stdout) }).end()
/*
<?xml version="1.0"?>
<ListQueuesResponse xmlns="http://queue.amazonaws.com/doc/2012-11-05/">
...
*/
```

More options
------------

```javascript
// you can pass AWS credentials in explicitly
aws4.sign(opts, { accessKeyId: '', secretAccessKey: '' })

// aws4 can infer the host from a service and region
opts = aws4.sign({ service: 'sqs', region: 'us-east-1', path: '/?Action=ListQueues' })

// create a utility function to pipe to stdout (with https this time)
function request(o) { https.request(o, function(res) { res.pipe(process.stdout) }).end(o.body || '') }

// aws4 can infer the HTTP method if a body is passed in
// method will be POST and Content-Type: 'application/x-www-form-urlencoded; charset=utf-8'
request(aws4.sign({ service: 'iam', body: 'Action=ListGroups&Version=2010-05-08' }))
/*
<ListGroupsResponse xmlns="https://iam.amazonaws.com/doc/2010-05-08/">
...
*/

// can specify any custom option or header as per usual
request(aws4.sign({
  service: 'dynamodb',
  region: 'ap-southeast-2',
  method: 'POST',
  path: '/',
  headers: {
    'Content-Type': 'application/x-amz-json-1.0',
    'X-Amz-Target': 'DynamoDB_20111205.ListTables'
  },
  body: '{}'
}))
/*
{"TableNames":[]}
...
*/

// works with all other services that support Signature Version 4

request(aws4.sign({ service: 'sns', path: '/?Action=ListTopics' }))
/*
<ListTopicsResponse xmlns="http://sns.amazonaws.com/doc/2010-03-31/">
...
*/

request(aws4.sign({ service: 'sts', path: '/?Action=GetSessionToken&Version=2011-06-15' }))
/*
<GetSessionTokenResponse xmlns="https://sts.amazonaws.com/doc/2011-06-15/">
...
*/

request(aws4.sign({ service: 'glacier', path: '/-/vaults', headers: { 'X-Amz-Glacier-Version': '2012-06-01' } }))
/*
{"Marker":null,"VaultList":[]}
...
*/

request(aws4.sign({ service: 'cloudsearch', path: '/?Action=DescribeDomains' }))
/*
<DescribeDomainsResponse xmlns="http://cloudsearch.amazonaws.com/doc/2011-02-01">
...
*/

request(aws4.sign({ service: 'ses', path: '/?Action=ListIdentities' }))
/*
<ListIdentitiesResponse xmlns="http://ses.amazonaws.com/doc/2010-12-01/">
...
*/

request(aws4.sign({ service: 'autoscaling', path: '/?Action=DescribeAutoScalingInstances&Version=2011-01-01' }))
/*
<DescribeAutoScalingInstancesResponse xmlns="http://autoscaling.amazonaws.com/doc/2011-01-01/">
...
*/

request(aws4.sign({ service: 'elasticloadbalancing', path: '/?Action=DescribeLoadBalancers&Version=2012-06-01' }))
/*
<DescribeLoadBalancersResponse xmlns="http://elasticloadbalancing.amazonaws.com/doc/2012-06-01/">
...
*/

request(aws4.sign({ service: 'cloudformation', path: '/?Action=ListStacks&Version=2010-05-15' }))
/*
<ListStacksResponse xmlns="http://cloudformation.amazonaws.com/doc/2010-05-15/">
...
*/

request(aws4.sign({ service: 'elasticbeanstalk', path: '/?Action=ListAvailableSolutionStacks&Version=2010-12-01' }))
/*
<ListAvailableSolutionStacksResponse xmlns="http://elasticbeanstalk.amazonaws.com/docs/2010-12-01/">
...
*/

request(aws4.sign({ service: 'rds', path: '/?Action=DescribeDBInstances&Version=2012-09-17' }))
/*
<DescribeDBInstancesResponse xmlns="http://rds.amazonaws.com/doc/2012-09-17/">
...
*/

request(aws4.sign({ service: 'monitoring', path: '/?Action=ListMetrics&Version=2010-08-01' }))
/*
<ListMetricsResponse xmlns="http://monitoring.amazonaws.com/doc/2010-08-01/">
...
*/

request(aws4.sign({ service: 'redshift', path: '/?Action=DescribeClusters&Version=2012-12-01' }))
/*
<DescribeClustersResponse xmlns="http://redshift.amazonaws.com/doc/2012-12-01/">
...
*/

request(aws4.sign({ service: 'storagegateway', body: '{}', headers: {
  'Content-Type': 'application/x-amz-json-1.1',
  'X-Amz-Target': 'StorageGateway_20120630.ListGateways'
}}))
/*
{"Gateways":[]}
...
*/

request(aws4.sign({ service: 'datapipeline', body: '{}', headers: {
  'Content-Type': 'application/x-amz-json-1.1',
  'X-Amz-Target': 'DataPipeline.ListPipelines'
}}))
/*
{"hasMoreResults":false,"pipelineIdList":[]}
...
*/

request(aws4.sign({ service: 'directconnect', body: '{}', headers: {
  'Content-Type': 'application/x-amz-json-1.1',
  'X-Amz-Target': 'OvertureService.DescribeConnections'
}}))
/*
{"connections":[]}
...
*/

request(aws4.sign({ service: 'opsworks', body: '{}', headers: {
  'Content-Type': 'application/x-amz-json-1.1',
  'X-Amz-Target': 'OpsWorks_20130218.DescribeInstances'
}}))
/*
{"Instances":[]}
...
*/
```

API
---

### aws4.sign(requestOptions, [credentials])

This calculates and populates the `Authorization` header of
`requestOptions`, and any other necessary AWS headers and/or request
options. Returns `requestOptions` as a convenience for chaining.

`requestOptions` is an object holding the same options that the node.js
[http.request](http://nodejs.org/docs/latest/api/http.html#http_http_request_options_callback)
function takes.

The following properties of `requestOptions` are used in the signing or
populated if they don't already exist:

- `hostname` or `host` (will be determined from `service` and `region` if not given)
- `method` (will use `'GET'` if not given or `'POST'` if there is a `body`)
- `path` (will use `'/'` if not given)
- `body` (will use `''` if not given)
- `service` (will be calculated from `hostname` or `host` if not given)
- `region` (will be calculated from `hostname` or `host` or use `'us-east-1'` if not given)
- `headers['Host']` (will use `hostname` or `host` or be calculated if not given)
- `headers['Content-Type']` (will use `'application/x-www-form-urlencoded; charset=utf-8'`
  if not given and there is a `body`)
- `headers['Date']` (used to calculate the signature date if given, otherwise `new Date` is used)

Your AWS credentials (which can be found in your
[AWS console](https://portal.aws.amazon.com/gp/aws/securityCredentials))
can be specified in one of two ways:

- As the second argument, like this:

```javascript
aws4.sign(requestOptions, {
  secretAccessKey: "<your-secret-access-key>",
  accessKeyId: "<your-access-key-id>"
})
```

- From `process.env`, such as this:

```
export AWS_SECRET_ACCESS_KEY="<your-secret-access-key>"
export AWS_ACCESS_KEY_ID="<your-access-key-id>"
```

(will also use `AWS_ACCESS_KEY` and `AWS_SECRET_KEY` if available)

Installation
------------

With [npm](http://npmjs.org/) do:

```
npm install aws4
```

Thanks
------

Thanks to [@jed](https://github.com/jed) for his
[dynamo-client](https://github.com/jed/dynamo-client) lib where I first
committed and subsequently extracted this code.

Also thanks to the
[official node.js AWS SDK](https://github.com/aws/aws-sdk-js) for giving
me a start on implementing the v4 signature.

