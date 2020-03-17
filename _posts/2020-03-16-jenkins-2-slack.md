---
layout: article
title: Upload artifact from jenkins server to slack using curl
tags: jenkins, slack, curl, pipeline
mode: immersive
header:
  theme: dark
article_header:
  type: overlay
  theme: dark
#   background_color: '#203028'
  background_image:
    gradient: 'linear-gradient(135deg, rgba(43, 80, 99, .5), rgba(229, 32, 39, .5))'
    src: https://images.unsplash.com/photo-1543674892-7d64d45df18b?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=2130&q=80
---


Everytime when jenkins completes building an artifact I want it to upload an artifact to the slack channel. There are slack plugins available for jenkins pipeline that uploads file but its not always the case for everyone. What I have covered below is the alternative approach using `curl`.
<!--more-->

**What you'll need?**
1. A slack channel. 

2. A Slack app. [Go here to create one](https://api.slack.com/apps).

3. Slack app's OAuth Token.
   
   ![Image](/res/2020-03-16-Slack-API-Applications.png){:.rounded width="500px"}
4. On the same page you'll find Scope section. In order to upload file, app will need `files:write` [scope](https://api.slack.com/scopes/files:write).
   
   ![Image](/res/2020-03-16-Slack-API-perm.png){:.rounded width="500px"}
5. Now, open your slack channel and **ADD** the slack app you just created. 


6. Open your `Jenkinsfile` and add the following function.
    ```
    def uploadFileToSlack() {
        def filePattern = "**/Release*.zip" // file that starts with Release and ends with .zip
        def file = findFiles(glob: "${filePattern}")[0]
        def botAuthToken = "xoxb-BOT_USER_AUTH_ACCESS_TOKEN" // see step #3
        def channelName = "#YOUR-CHANNEL_NAME" // replace with the appropriate channel name.
        try {
            sh(script: """
                     curl --request POST \
                     --url https://slack.com/api/files.upload \
                     --header 'content-type: multipart/form-data' \
                     --form token=$botAuthToken \
                     --form 'channels=$channelName' \
                     --form 'title=${file.name}' \
                     --form file="@${file}"
                      """
                      )
        } catch (Exception e) {
        
        }
    }
    ```
    
    Replace the value of `filePattern` with your desired [glob pattern](https://en.wikipedia.org/wiki/Glob_(programming)). 

7. Lastly, there must be a *stage* in your `Jenkinsfile` where it creates an artifact. Call the above function `uploadFileToSlack()` from there.


Thats all. I hope you find this useful. Thanks!

---

- <a style="background-color:black;color:white;text-decoration:none;padding:4px 6px;font-family:-apple-system, BlinkMacSystemFont, &quot;San Francisco&quot;, &quot;Helvetica Neue&quot;, Helvetica, Ubuntu, Roboto, Noto, &quot;Segoe UI&quot;, Arial, sans-serif;font-size:12px;font-weight:bold;line-height:1.2;display:inline-block;border-radius:3px" href="https://unsplash.com/@realaxer?utm_medium=referral&amp;utm_campaign=photographer-credit&amp;utm_content=creditBadge" target="_blank" rel="noopener noreferrer" title="Download free do whatever you want high-resolution photos from tian kuan"><span style="display:inline-block;padding:2px 3px"><svg xmlns="http://www.w3.org/2000/svg" style="height:12px;width:auto;position:relative;vertical-align:middle;top:-2px;fill:white" viewBox="0 0 32 32"><title>unsplash-logo</title><path d="M10 9V0h12v9H10zm12 5h10v18H0V14h10v9h12v-9z"></path></svg></span><span style="display:inline-block;padding:2px 3px">tian kuan</span></a>

---