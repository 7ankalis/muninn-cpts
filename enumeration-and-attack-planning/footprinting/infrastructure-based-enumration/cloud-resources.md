---
description: >-
  AWS s3 backets, GCP Cloud storage, Azure blobs. After getting the
  company-hosted servers.
---

# Cloud Resources

{% hint style="info" %}
<mark style="color:blue;">Often cloud storage is added to the DNS list when used for administrative purposes by other employees.</mark>\
&#x20;         example: s3-website-us-west-2.amazonaws.com
{% endhint %}

## 1. Google search+ Google Dorks:

{% embed url="https://www.exploit-db.com/google-hacking-database" %}
Google Dorks examples.
{% endembed %}

```bash
intext:<SNIP> inurl:amazongaws.com
intext:<SNIP> inurl:blob.core.windows.net
```

## 2. We can also check a page's source code:

<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

## 3. 3rd Party Solutions:

{% hint style="info" %}
Third-party providers such as [domain.glass](https://domain.glass/) can also tell us a lot about the company's infrastructure. As a positive side effect, we can also see that Cloudflare's security assessment status has been classified as "Safe". This means we have already found a security measure that can be noted for the second layer (gateway).
{% endhint %}

### 3.1 Domain.glass:

{% embed url="https://domain.glass/" %}

<figure><img src="../../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

### 3.2. GrayHatWafare:

{% embed url="https://buckets.grayhatwarfare.com/" %}

We can do many different searches, discover AWS, Azure, and GCP cloud storage, and even sort and filter by file format. Therefore, once we have found them through Google, we can also search for them on GrayHatWarefare and passively discover what files are stored on the given cloud storage.
