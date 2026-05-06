# 🥨 Common Security Issue Devs overlooked

Authentication and Session Management Vulnerabilities<br>
---------------------------------------------------------

#### <i class="fa-house-laptop">:house-laptop:</i> Old Session Does Not Expire After Password Change

<details>

<summary>Step to Reproduce</summary>

1. Log in to the same account from two different browsers or devices.
2. In the first browser/device, go to account settings.
3. Change the account password successfully.
4. Go back to the second browser/device where the old session is still logged in.
5. Refresh the page.
6. Try to access any authenticated page or account setting.
7. Observe that the old session still remains active after the password change.

</details>

#### <i class="fa-house-laptop">:house-laptop:</i> Failure to Invalidate Session on Logout (Persistent Session)

<details>

<summary>Step to Reproduce</summary>

1. Log in to the account from a browser or device.
2. Copy or save the current authenticated page URL.
3. Click **Logout**.
4. After logout, paste the saved authenticated URL back into the browser.
5. Refresh the page or use the browser back button.
6. Try to access any authenticated page or account setting.
7. Observe that the session remains active or the protected page is still accessible after logout.

</details>

#### <i class="fa-house-laptop">:house-laptop:</i> Email Verification Bypass (Logic Flaw)

<details>

<summary>Step to Reproduce</summary>

1. Create an account and receive the initial email verification link. Do not click it yet.
2. Log in and change your email address to "Email B"
3. The system sends a new link to Email B. Verify that link.
4. Go back to settings and change your email back to the original "Email A".
5. If "Email A" is now marked as verified without you clicking its specific link, this is a bypass.

</details>

#### <i class="fa-house-laptop">:house-laptop:</i> Password Reset Token Persistence (Multiple Requests)

<details>

<summary>Step to Reproduce</summary>

1. Steps to Reproduce:
2. Create an account with a valid email.
3. Log out and request a "Forgot Password" link (Link 1).
4. Without using Link 1, request a second "Forgot Password" link (Link 2).
5. Go to your email and attempt to use Link 1 to change the password.
6. If Link 1 still works, report it.

</details>

#### <i class="fa-house-laptop">:house-laptop:</i> Password Reset Token Re-use

<details>

<summary>Step to Reproduce</summary>

1. Request a password reset link for a test account.
2. Open the password reset link from the email.
3. Set a new password successfully.
4. After the password is changed, open the same reset link again.
5. Try to set another new password using the same token.
6. Observe that the old password reset token can still be reused.

</details>

#### <i class="fa-house-laptop">:house-laptop:</i> Lack of Session Validation on Sensitive Endpoints

<details>

<summary>Step to Reproduce</summary>

1. Log in and navigate to the Profile section.
2. Edit a field (e.g., Name) but do not save yet.
3. Enable Burp Suite Proxy. Click "Save."
4. Capture the request and send it to Repeater. Drop the request in Proxy.
5. Log out of the website in your browser.
6. Go to Burp Repeater, modify the parameters, and send the request.
7. If the server responds with 200 OK and the data changes, the endpoint lacks validation.

</details>

#### <i class="fa-house-laptop">:house-laptop:</i> Session Fixation

<details>

<summary>Step to Reproduce</summary>

1. Go to the login page and note the PHPSESSID (or equivalent) value (e.g., XYZ123).
2. Log in with valid credentials.
3. Check the session cookie value again.
4. If the value remains XYZ123, the session was not rotated.

</details>

#### <i class="fa-house-laptop">:house-laptop:</i> Concurrent Session Limit Bypass

<details>

<summary>Step to Reproduce</summary>

1. Log in to the account on Browser A.
2. Log in to the same account on Browser B.
3. Check if Browser A is kicked out.
4. If not, use Burp Intruder to send 50 login requests simultaneously to see if there is any cap.

</details>

#### <i class="fa-house-laptop">:house-laptop:</i> Missing Session Rotation After Privilege Change

<details>

<summary>Step to Reproduce</summary>

1. Log in as a regular user
2. Note the current session ID.
3. Perform an action that increases privilege (for example, upgrading the account, joining an organization, enabling 2FA or accepting an admin invitation).
4. Check the session cookie again.
5. If the session ID is unchanged, the app failed to rotate

</details>

#### <i class="fa-house-laptop">:house-laptop:</i> Unrestricted Session Duration (Infinite Sessions)

<details>

<summary>Step to Reproduce</summary>

1. Log in.
2. Capture your session cookie.
3. Wait several hours or even days without activity.
4. Reuse the exact same cookie via Burp, or paste it back into your browser.
5. If it still works, the session lifetime is too long.
6. Impact: A stolen cookie becomes a long-term key to the account. Attackers can maintain access for weeks or months without detection.

</details>

#### <i class="fa-house-laptop">:house-laptop:</i> Weak "Remember Me" Token Implementation

<details>

<summary>Step to Reproduce</summary>

1. Check if the website offers a "Stay logged in" feature.
2. Log in with it enabled and inspect cookies for remember-me values.
3. Log out.
4. Paste back the cookie and refresh.
5. If it logs you in again, the token is static.

</details>

#### <i class="fa-house-laptop">:house-laptop:</i> JWT Misconfigurations (Stateless Session Issues)

<details>

<summary>Step to Reproduce</summary>

1. Log in and capture your JWT.
2. Perform logout.
3. Reuse the same JWT in Postman or Burp.
4. If it still works, the server is not revoking tokens.

</details>

## User Registration/Sign Up Features

#### <i class="fa-house-laptop">:house-laptop:</i> Denial of Service (DoS) at Input Fields

<details>

<summary>Step to Reproduce</summary>

1. Go to the Sign-up form.
2. Fill in normal data for most fields.
3. In the Password field (or sometimes the Username field), enter an extremely long string of characters (e.g., 10,000+ 'A's). You can generate this easily in a text editor or using Python: python -c "print('A'\*20000)". and Submit the form.
4. Observation: Monitor the response time. If the request hangs for a long time and eventually returns a 500 Internal Server Error, it indicates the server struggled to process the input, suggesting a vulnerability.

</details>

#### <i class="fa-house-laptop">:house-laptop:</i> Lack of Rate Limiting (Mass Assignment)

<details>

<summary>Step to Reproduce</summary>

1. Fill out the signup form with generic details and submit it, intercepting the request in a proxy tool like Burp Suite.
2. Send the captured POST request to Burp Intruder.
3. In the Intruder "Positions" tab, clear existing payload markers. Add markers (§§) around the email parameter value. Example: email=testuser§1§@example.com
4. In the "Payloads" tab, choose "Numbers" (e.g., from 1 to 1000) or provide a list of different email addresses and Start the attack.
5. Observation: Analyze the results. If you receive a 200 OK (or whatever the success response code is for hundreds of consecutive requests without being blocked or presented with a CAPTCHA, the endpoint lacks rate limiting.

</details>

#### <i class="fa-house-laptop">:house-laptop:</i> Cross-Site Scripting (XSS) in Registration Fields

<details>

<summary>Step to Reproduce</summary>

1. Go to the registration/sign-up page.
2. Enter a normal email and password.
3. In a registration field such as name, username, or full name, enter a test XSS payload:

{% code overflow="wrap" %}
```
<script>alert(1)</script    
```
{% endcode %}

4. Submit the registration form.
5. Log in or visit the profile/account page where the submitted field is displayed.
6. Observe whether the script executes or an alert popup appears.

</details>

#### <i class="fa-house-laptop">:house-laptop:</i> Insufficient Email Verification

<details>

<summary>Step to Reproduce</summary>

A. **Response Manipulation**

1. Intercept the response from the server after submitting the signup or clicking a verification link.
2. Look for parameters in the JSON response body like "is\_verified": false, "status": "pending", or "success": false.
3. Change the values to true/success (e.g., "is\_verified": true).
4. Forward the manipulated response to the browser and see if it grants access.<br>

B. **Status Code Manipulation**

1. If accessing a protected page redirects you with a 403 Forbidden or 302 Redirect, intercept the response.
2. Change the status code from 403 to 200 OK.
3. Sometimes you need to remove redirect headers (like Location: /login) as well.<br>

C. **Direct/Forced Browsing**

1. Register an account but do not verify the email.
2. Try to guess or force-browse directly to pages that should only be accessible after verification.

Examples:

&#x20;/user/dashboard, /account/settings, /onboarding/step2.<br>

D. **Email Verification Swap**

1. Sign up using an attacker-controlled email like attacker@mail.com
2. Wait for the confirmation email but don't open the link yet.
3. If the app lets you access profile or account settings before verifying, go there.
4. Try changing the account email to victim@mail.com
5. The app will now send a fresh verification link to victim@mail.com Ignore it.
6. Instead, go back to attacker@mail.com and open the original verification link.
7. If this verifies victim@mail.com using the older token, the app is vulnerable to an Email Verification Bypass via stale token reuse.

</details>

#### <i class="fa-house-laptop">:house-laptop:</i> Weak Registration Implementation

<details>

<summary>Step to Reproduce</summary>

1. Allows Disposable Email Addresses
2. Registration Form on non-HTTPS

</details>

#### &#x20;<i class="fa-house-laptop">:house-laptop:</i> Weak Password Policy

<details>

<summary>Step to Reproduce</summary>

* **Easily Guessable Passwords:** Does the form accept passwords like 123456, password, qwerty, or admin?
* **Username as Password:** Does the form allow the password to be identical to the username? This is a very common pattern for lazy users.
* **Email as Password:** Does it allow the password to be the same as the email address?
* **Improper Password Recovery:** While technically part of the login flow, the foundation is laid at registration. Ensure the mechanism for future password resets (e.g., security questions set during signup) is secure and not easily guessable.

</details>

#### <i class="fa-house-laptop">:house-laptop:</i>  Path Overwrite (Route Collision)

<details>

<summary>Step to Reproduce</summary>

1. Identify URL Pattern: Confirm that user profiles are accessible directly via target.tld/username.
2. Register Reserved Names: Attempt to signup with usernames corresponding to critical pages or files.
3. Modern Apps: login, admin, signup, api, dashboard.
4. Legacy Apps: index.php, login.php, signup.php, admin.aspx.
5. 3\. Verify Overwrite: Navigate to the target URL (e.g., target.tld/login.php).

</details>

#### <i class="fa-house-laptop">:house-laptop:</i> Server-Side Validation Bypass (Client-Side Only Checks)

<details>

<summary>Step to Reproduce</summary>

1. Open the signup page and fill in the fields with any data.
2. Intercept the outgoing request using Burp Suite, OWASP ZAP, or a browser extension like Tamper.
3. Modify parameters that would normally be blocked by frontend rules.
4. Forward the request to the server.

Examples:

1. Empty username or email
2. Password shorter than the minimum requirement
3. Invalid email format (e.g., test@test, a@b, abc)
4. Special characters in fields that normally block them

</details>

#### <i class="fa-house-laptop">:house-laptop:</i>  Hidden / Unlinked Registration Endpoints

<details>

<summary>Step to Reproduce</summary>

1. Spider or crawl the application.
2. Look for endpoints like:

{% code overflow="wrap" %}
```
/api/v1/register
/auth/create
/user/create
/legacy/signup
/mobile/register
```
{% endcode %}

3\. Compare the validation rules between each endpoint.

</details>

#### <i class="fa-house-laptop">:house-laptop:</i>  Parameter Pollution in Signup Requests

<details>

<summary>Step to Reproduce</summary>

1. Intercept the signup request.
2. Add multiple values to the same parameter. Example:

<p align="center">email=victim@gmail.com&#x26;email=attacker@gmail.com</p>

3. Forward the request.

</details>

#### <i class="fa-house-laptop">:house-laptop:</i>  Weak or Predictable Verification Link

<details>

<summary>Step to Reproduce</summary>

1. Register a legitimate account and inspect the verification link format.
2. Look for patterns like:

* Base64 email
* Short token values
* Incrementing IDs

3\. Attempt token manipulation.

</details>

#### <i class="fa-house-laptop">:house-laptop:</i>  Punycode and IDN Homograph Bypass

<details>

<summary>Step to Reproduce</summary>

1. Go to the domain/email verification or URL submission feature.
2. Enter a domain that visually looks similar to a trusted domain using Unicode characters.

_Example:_

раypal.com

Note: The first characters are Cyrillic, not normal Latin letters.

3. Submit the form.
4. Check whether the application accepts the lookalike domain as if it were the real trusted domain.
5. Repeat the test using the punycode version of the same domain.
6. Observe whether the application fails to normalize and validate the domain correctly.

</details>

#### <i class="fa-house-laptop">:house-laptop:</i>  OTP Verification Brute-Force During Signup

<details>

<summary>Step to Reproduce</summary>

1. Start the signup process using a test email or phone number.&#x20;
2. Trigger the OTP verification step.&#x20;
3. Enter an incorrect OTP code.
4. Repeat the incorrect OTP submission multiple times.&#x20;
5. Observe whether the application allows unlimited or excessive OTP attempts.&#x20;
6. Check whether there is no account lockout, rate limit, cooldown, or CAPTCHA after repeated failed attempts.

</details>

#### <i class="fa-house-laptop">:house-laptop:</i> Null Byte Injection (%00) in Signup Inputs

<details>

<summary>Step to Reproduce</summary>

1. Register using values like: attacker@mail.com%00victim@mail.com or username%00.jpg
2. Observe how the backend stores or displays the data.
3. Attempt login or verification afterward.

</details>

<i class="fa-house-laptop">:house-laptop:</i>  Missing Email Confirmation Enforcement

<details>

<summary>Step to Reproduce</summary>

1. Register using a random email.
2. Skip the confirmation link.
3. Try logging in directly.
4. Attempt actions like profile update or password reset.

</details>

<i class="fa-house-laptop">:house-laptop:</i> Cache Control Issues in Signup & Verification Flows

<details>

<summary>Step to Reproduce</summary>

1. Complete signup or verification.
2. Use the back button or offline mode.
3. Inspect cached pages.
4. Test private browsing and shared device scenarios.

</details>
