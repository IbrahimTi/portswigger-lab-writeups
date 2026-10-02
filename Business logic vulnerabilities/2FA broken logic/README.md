# 2FA Broken Logic

## Lab Overview

* **Platform:** PortSwigger Web Security Academy
* **Difficulty:** Practitioner
* **Category:** Authentication
* **Goal:** Access Carlos's account without login

## Vulnerability

The application uses the client-controlled `verify` parameter to decide which user's 2FA process is being verified.

The parameter can be changed from `wiener` to `carlos`, allowing the 2FA flow to be manipulated.

## Steps

### 1. Login

Logged in with the provided credentials:

```text
wiener:peter
```

<img width="955" height="747" alt="image" src="https://github.com/user-attachments/assets/3f42e688-7b6d-4433-a6fa-a594b29756ac" />


### 2. Find the 2FA Request

After logging in, I checked the requests in Burp Suite and found:

```http
GET /login2?verify=wiener
```

The `verify` parameter looked interesting because it was being used to identify the user.

<img width="595" height="428" alt="image" src="https://github.com/user-attachments/assets/b4b9cb4a-fd10-4c87-a522-bdf37b0e5eda" />


### 3. Change the User

Sent the request to Burp Repeater and changed:

```text
verify=wiener
```

to:

```text
verify=carlos
```

Request:

```http
GET /login2?verify=carlos
```

This triggered the 2FA process for Carlos.

<img width="956" height="797" alt="image" src="https://github.com/user-attachments/assets/bfab90d8-e3ff-4fd8-aa68-23e9f4ac1e7e" />


### 4. Prepare the MFA Request

I went back to the login page and submitted an invalid 2FA code to capture the verification request.

The request looked like:

```http
POST /login2

verify=wiener&mfa-code=0000
```

I sent this request to Burp Intruder.

<img width="937" height="837" alt="image" src="https://github.com/user-attachments/assets/c8b77b5b-c77f-4536-9dd7-6850643abf3b" />


### 5. Brute-force the MFA Code

Changed:

```text
verify=wiener
```

to:

```text
verify=carlos
```

Then placed the payload position on the MFA code:

```text
verify=carlos&mfa-code=§0000§
```

The code was 4 digits, so I used:

```text
0000 - 9999
```


### 6. Find the Correct Code

After running the attack, I looked for the response that was different from the failed attempts.

The successful request returned a `302` redirect to:

```text
/my-account
```

📸 **Screenshot:** Successful Intruder result showing `302`

### 7. Access Carlos's Account

I loaded the successful response in the browser and opened **My account**.

This gave access to Carlos's account and completed the lab.

📸 **Screenshot:** Carlos's account / Lab solved

## Impact

The flaw allows an attacker to manipulate the 2FA process and potentially access another user's account without knowing their legitimate 2FA code.

## Root Cause

The application trusts the client-controlled `verify` parameter instead of securely binding the 2FA process to the authenticated user's session.

## Fix

The server should determine the user from the authenticated session and bind the MFA challenge and code to that user.

## Key Takeaway

The important part of this lab was noticing that the `verify` parameter controlled the account being verified. Authentication workflows should always be tested for client-controlled parameters that affect user identity.
