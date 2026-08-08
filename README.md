# Rasp5-Router
Making slight adjustments to PiDIYLAB router tutorial for Raspberry Pi 4, making it compatible with Raspberry Pi 5 and setting up VLANs



## Challenges and How they were resolved
1 - Inability to push to git rep:
    - When running the git push command in terminal, I was prompted for github username and password. After proviing credentials, an error message appeared saying:
    "Invalid username or token. Password authentication is not supported for git operations.."
    - I did not know this at the item so i had to research the solution. I found two options
        OPTION A - Personal Access Token
            - This is a quick fix and replaces your password with a token. However, tokens expire.

        OPTION B - SSH Key
            - This utilizes a SSH key that is generated on your device and copied to github. 

            - I chose this option because it does not expire, and it required me to switch my repo remote from HTTPS to SSH using the following "git remote set-url origin git@github.com:profilename/nameofrepo.git"