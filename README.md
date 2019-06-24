# acme-tiny-checkcrt

[acme-tiny](https://github.com/diafygi/acme-tiny) is a fantastic little python ACME client for LetsEncrypt certificate renewals. &nbsp;It is, as the name suggests, very small, easily auditable, and thus can be readily trusted.  What is completely lacks, though, is any way to automate the renewal process.  That is where the acmy-tiny checkcrt script comes along.

## checkcrt

`checkcrt` is a BASH script to safely automate the process of checking and renewing your certificate in a timely manner.

The method suggested in acme-tiny's documentation is to run `acme_tiny.py` in a cron tab every 30 days or so as a completely unprivileged system user.  This has a few weaknesses:
1. If `acme_tiny.py` fails, there is little recourse.  You either have to run it at less than half the expiry interval, and then just hope it never encounters an error more than once or twice, or accept that if it does then your server's certificate will expire.
2. In order to prevent acme-tiny from being run as root, the entire check must be run as an unprivileged user.  Which requires giving an unprivileged user the ability to restart services, which somewhat defeats the purpose of having and protecting these kind of privileges behind root.
3. Some software, for example `lighttpd`, likes having the certificate file tacked onto the server's private key.  Assembling this is something that is not appropriate for an unprivileged user - you don't want your server's private key sitting on your server with file permissions that allow it to be read by anything other than root.

`checkcrt` solves these problems by:
1. Checking the expiry date of your existing certificate.  If it isn't expiring with your preset time (defaulting to ten days), then it simply exits.  This way you can run `checkcrt` in a daily cron job.
2. Every time `checkcrt` attempts to renew your certificate, it emails your preset contact address with a copy of the log.  Rest assured when you get emails every 80 days saying your certificate is up to date.  On failure, you find out exactly what went wrong and know that it will try again every day until it succeeds.
3. When `checkcrt` does want to renew the certificate, it separates the job of renewal (which is run as an unprivileged user) from the job of assembling and publishing your certificate (which is done safely as root).
4. You can have `checkcrt` assemble your certificate in any way your various services need. Some software wants it as the certificate separate from the CA's intermediate.  Some software wants it together with the CA's certificate in one file.  Some software wants it as the certificate together with your server's private key.

## Documented, auditable, and safe
``checkcrt`` is about 255 lines.  This breaks down to:
* <100 lines of actual script
* ~60 lines of script comments
* ~100 lines of instruction manual at the end of the script

`checkcrt` ensures that nothing that touches the internet is run as root and that nothing that touches protected files is run as an unprivileged user.
