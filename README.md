- Well, i put the mcuboot.pem inside this dir as a template. When you use it as production; DON'T push it to github or anywhere else.

- The problem when you get the red scary error for dealing with https is that: the mixing of sizeof and strlen; please look it carefully. And look what i deal with the host_and_port without using .port when crafting the request;

- Next is the https.conf; do not set the timeout for net_init too short; or just dont set it at all (i remove it). It can abandon the HDNS register which make all later step fail.

- You can use the command: openssl s_client -showcerts -connect google.com:443 < /dev/null | openssl x509 -outform PEM -out r1.pem to get the certificate of any cert; in this example is google.com

- The bootloader i choose is mcuboot; which make it easy to deal with; but it requires the instructions.md's steps.

- For testing, i hardcoded a lot (A LOT), but as a good practice, you should make a secrets.conf to contains those.

- Zephyr is good, but it lacks of a complete project tutorials.

- The improvements i can think after adding HTTPS and OTA: improve RAM and flash (could be make with -p always -S flash-16M -S psram-8M), then -t menucofig; let tweak some config; i remember that there are some configs that let you transfer a lot from ram to flash.

- The code is ugly, i know. In the future, i will reform it as subsys, then implement Finite state machine + event drivent + threading.

- The security must be take care too: Right now, it is using CRC for checking error; and using AES (the mcuboot.pem) to sign the firmware. Maybe, the latest version will be push to to github release(we can set it to private? for HTTPS advantage) and make use of encryption.

Thanks for reading.
