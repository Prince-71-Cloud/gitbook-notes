# Homoglyph Attack

#### 1. Change victim’s password using IDN Homograph Attack

<details>

<summary>Account takeover</summary>

**Steps to reproduce**

1. Open the burp collaborator client > Generate Collaborator payload .
2. Go to the sign up page of target.com and create a new account with email- abc@gmail.com.burpcollaboratorpayloadhere
3. Now if the target.com has email confirmation > you will receive the email confirmation link in burp collaborator client > verify the email.
4. Go to password reset page of target.com > enter email as abc@gmáil.com.burpcollaboratorpayloadhere
5. If the target.com is vulnerable then it will send password reset link to the mail- abc@xn — gmil-6na.com.burpcollaboratorpayloadhere and you will receive password reset link in burp collaborator client. Make sure to check in burp collaborator client -received email details: To- abc@xn — gmil-6na.com.burpcollaboratorpayloadhere.
6. Now you can change the password and access the victim’s account.

</details>

#### 2. Username or Restricted keyword bypass through Homoglyph payload
