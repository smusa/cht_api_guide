#CloudHealth Tagging API -- BETA

The CloudHealth Tagging API allows you to add tags (key-value pair) to accounts and assets in CloudHealth. These tags are completely independent of your cloud provider tags. **Please Note:** When you tag accounts or assets using the CloudHealth API the resources are only tagged on the CloudHealth platform and do not cascade down to your cloud provider (AWS). However, we will continue to pull your AWS tags via the AWS API. 
 
###<u>Use cases for adding custom tags in CloudHealth:</u> 
 
* **Theme based accounts management**: In case you have sets of multiple accounts with a similar purpose and you'd like to create groupings, say within a CloudHealth perspective, you could tag the accounts and leverage the tagging in the Perspective Builder.  
* **Categorization of Assets**: You could leverage the key-value pair tags to form groupings within a CloudHealth perspective based on the values of the keys. For example, you can mark all assets with an Environment tag and allocate values such as Development, Staging, and Production to the applicable assets. This will allow you to gain visibility into cost, usage, and performance breakdown of your different environments. You could also leverage these categorized assets to set thresholds and policies that will either notify you or trigger a workflow.


##Creating & Pushing Payloads

###Endpoint:

**POST: `https://chapi.cloudhealthtech.com/customer_tags?api_key=<YOUR API KEY>`**


where, you replace this `<YOUR API KEY>` with your API Key from `My Profile` section of [CloudHealth Technologies Application](https://apps.cloudhealthtech.com)

###Headers:
<pre><code>Accept: application/json
Content-Type: application/json
</code></pre> 

###Request Body Example for Tagging AWS Accounts:
<pre><code>{
  "assets" :
    [
      {
        "type" : "AwsAccount",
        "owner_id" : "123456789012",
        "tags" : {
           "environment" : "SA Labs",
           "owner"       : "Sidd"
        }
      }
    ]
}
</code></pre>

Where:
  
* `owner_id` is the 12 digit Amazon assigned Account ID for the AWS Account; you can find the `owner_id` by clicking on [My Account in the AWS Console](https://console.aws.amazon.com/billing/home#/account),
* `tags` is a hash (object) of key value pairs

###Request Body Example for Tagging AWS Assets:
<pre><code>{
  "assets":
    [
      {
        "aws_account_id": "206158430516",
        "instance_id": "i-9acdeffa",
        "tags": {
            "cost_center": ‘9323’,
            "environment": ‘Production’,
            "owner": "Vikram",
            ...
        }
      },
      ...
    ]
}</code></pre>  
  

Where:  

* `aws_account_id` is the CloudHealth Account ID for the AWS account that owns the instance to be tagged. You can find the `aws_account_id` by clicking on the bulls eye icon from the [AWS account page](https://apps.cloudhealthtech.com/aws_accounts/) and grabbing the number at the end of the generated URL 
* `instance_id` is the AWS instance ID to be tagged, and
* `tags` is a hash (object) of key value pairs

##Constraints:  

* No more than 100 instances per request
* No more than 100 tags per instance
* Tag keys must be between 1 and 127 characters long
* Tag values must be no longer than 255 characters long
* Tag values must be scalar.  Lists and objects are not allowed. Numbers will be converted to strings.
* Tag value will be stripped of leading and trailing whitespace
	
##Processing:  

* The entire payload will be processed.  Any failures will be noted, and processing will continue.
* If the tag 
  * does not exist, it will be created.
  * exists, and it came from AWS, a warning will be flagged and the tag skipped
  * exists, is customer set, and the new value is not null, it will be replaced
  * exists, is customer set, and the new value is null, it will be removed

##Response:
<pre><code>{
	"successful": "N tags altered",
	"failures":
	  [
	    "array of strings enumerating each failure and warning."
	  ]
 }
</code></pre>

* The response body is JSON.
* The count of successful writes will be given (shown as N).
* Each partial failure will be returned with an indicator as to which instance/tag failed and why.
* If there were no failures, an empty array will be returned.
* The HTTP response code will be 200 OK even if all writes failed.


###CURL example:
Here is an example of a post directly from the command line, using CURL:   
<pre><code>curl -H "Accept: application/json" -H "Content-Type: application/json" -XPOST "https://chapi.cloudhealthtech.com/customer_tags?api_key=99999999-9999-9999-9999-999999999999" -d '{ "assets": [{ "aws_account_id": 999, "instance_id": "i-c9999e99", "tags": {"foo": "bar", "baz": null, “qux”: “garply”}}]}</code></pre>
