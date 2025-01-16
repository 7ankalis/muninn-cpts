---
description: >-
  AWS s3 backets, GCP Cloud storage, Azure blobs. After getting the
  company-hosted servers.
---

# Cloud Resources

{% hint style="info" %}
<mark style="color:blue;">Often cloud storage is added to the DNS list when used for administrative purposes by other employees.</mark>\
&#x20;         example: s3-website-us-west-2.amazonaws.com

<mark style="color:blue;">So it's in our scope to look for cloud resources in DNS queries.</mark>
{% endhint %}

## Google search + Google Dorks:

{% embed url="https://www.exploit-db.com/google-hacking-database" %}
Google Dorks examples.
{% endembed %}

```bash
intext:<SNIP> inurl:amazongaws.com
intext:<SNIP> inurl:blob.core.windows.net
```

## Page's source code:

<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

Even it's a low hanging fruit, it can expose some endpoints or some cloud resources being referred from the page we're consulting. Even we don't find anything related to  the cloud, we can find API endpoints, other pages or hidden directories.

## 3rd Party Solutions:

{% hint style="info" %}
Third-party providers such as [domain.glass](https://domain.glass/) can also tell us a lot about the company's infrastructure. As a positive side effect, we can also see that Cloudflare's security assessment status has been classified as "Safe". This means we have already found a security measure that can be noted for the second layer (gateway).
{% endhint %}

### Domain.glass:

{% embed url="https://domain.glass/" %}

<figure><img src="../../../.gitbook/assets/image (2) (1).png" alt=""><figcaption><p>Example output from domain.glass</p></figcaption></figure>

### GrayHatWafare:

{% embed url="https://buckets.grayhatwarfare.com/" %}
Another source to check for cloud storage online.
{% endembed %}

We can do many different searches, discover AWS, Azure, and GCP cloud storage, and even sort and filter by file format. Therefore, once we have found them through Google, we can also search for them on GrayHatWarefare and passively discover what files are stored on the given cloud storage.
