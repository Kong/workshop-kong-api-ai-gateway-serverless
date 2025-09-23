---
title : "PAT - Personal Access Token"
weight : 112
---

[decK](https://developer.konghq.com/deck/) requires a [Konnect Personal Access Token (PAT)](https://docs.konghq.com/konnect/org-management/access-tokens/) to manage your Control Plane. To generate your PAT,  click on your initials in the upper right corner of the Konnect home page, then select **Personal Access Tokens**. Click on **+ Generate Token**, name your PAT, set its expiration time, and be sure to copy and save it in an evironment variable, as Konnect won’t display it again.

![pat](/static/images/pat.png)


> [!NOTE]
> Be sure to copy and save your PAT, as Konnect won’t display it again.


#### Konnect PAT secret

* Save PAT in an environment variables

{{<highlight>}}
export PAT=<PASTE_THE_CONTENTS_OF_COPIED_PAT>
{{</highlight>}}



### Test your PAT

```
deck gateway ping --konnect-control-plane-name serverless-default --konnect-token $PAT
```

You should get a response like this

```
Successfully Konnected to the AcquaOrg organization!
```


