---
title: Upload artifact from jenkins server to slack using curl
tags: jenkins, slack, curl, pipeline
---

**Background**:
Everytime when jenkins completes building an artifact I want it to upload an artifact to the slack channel. There are slack plugin available for jenkins pipeline that uploads file but its not always the case everyone. What I have covered below is the alternative approach using `curl`.
<!--more-->

**What you need?**
1. A slack channel. Of course.

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

---