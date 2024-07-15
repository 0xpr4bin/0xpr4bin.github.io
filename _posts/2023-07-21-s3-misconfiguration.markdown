---
title: "S3 bucket misconfiguration"
date:  2023-06-08 15:04:23
categories: [bugbounty]
tags: [bugbounty]
---

### S3 bucket misconfigured worth $$$ bounty


Hello everyone,
I hope you are hunting on wild.This is my first bug of 2023.
I am not much into writing but this is something i found beyond classics XSS,CSRF,Idors and so on which everyone is finding nowdays.

This is about s3 bucket misconfigurations which i found recently on hackerone and it was medium severity, if you dont know about s3 bucket,then you can refer here.

So lets start the story, i was testing on one of the private program from hackerone (say example.com).I wasnot finding anything for nearly 2 hours. So my hobby is that if i dont find anything for 2 or more hours,i will take break or i quit for that day.Likewise i again start testing that site evening and say what i still didnot find any thing to report. That's kinda frustating right.So i quit for that day and watched movie and slept.

Next day when i opened the burp,i noticed some issues poping around the corner,though it was low severity saying The following bucket was confirmed to be valid:

![Burp Suite](https://prabinsigdel.com.np/images/burp.jpg)

It was in the format `https://bucket-name.s3.amazonaws.com/`
Then i suddenly jumped to the terminal and checked what permisssioins are allowed on that specific bucket starting with the command `aws s3 ls s3://bucket-name/`

It gave access denied error at first,after trying every possible checks i found out that it was allowing the `s3:ListMultipartUploadParts` permission and can be exploited with 
`aws s3api list-multipart-uploads --bucket my-bucket`.

Ok this list every possible upload parts inside the bucket alongside upload-id,which can be useful for other permissions like

- Create Multipart Upload
- Upload Part
- Upload Part (Copy)
- Complete Multipart Upload
- Abort Multipart Upload
- List Parts
- List Multipart Uploads

Then i reported to the program,after 2 days they triaged and awarded me with bounty.
So,if you dont find anything,just quit or take break,there is frustrations,relief and satisfactions.

![Bounty](https://prabinsigdel.com.np/images/reward.jpg)

**Just remember if there is ROSE there would be thorn and if there is money there would be hard-work.
