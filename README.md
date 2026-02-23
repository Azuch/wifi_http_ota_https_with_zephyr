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


** Update - 23/02/2026 **
- The TlS damn chain; it combine of leaf, intermediate and root. When we connect to the server; it send to us it server cert; and we, normally just need to have the root CA cert in our store, and then it works. BUT NO, sometime, we must need to set the intermediate cert too, for this shit to work. And other board like nRF has it default CA bundle and it not need us to add normally; but the ESP32, yes, you must set it or it won't work.

- The command to get all the certs is: 
```bash
openssl s_client -showcerts \                               
-connect release-assets.githubusercontent.com:443 \
-servername release-assets.githubusercontent.com </dev/null \
| awk '/BEGIN CERTIFICATE/,/END CERTIFICATE/{print}' \
| awk 'BEGIN{c=0}/BEGIN CERTIFICATE/{c++}{print > "cert" c ".pem"}' 
```
to get 3 cert, cert1 is leaf, cert2 is intermediate and cert3 is root and 
```bash
openssl x509 -outform der -in cert3.pem -out src/root_ca.der
```
is to get the root CA cert.

- Then we need to modify the CMakeLists.txt 
```bash
set(gen_dir ${ZEPHYR_BINARY_DIR}/include/generated/)

generate_inc_file_for_target(
    app
    src/root_ca.der
    ${gen_dir}/root_ca.der.inc
)
```
- Next is the src/ca_certificate.h: static const unsigned char ca_certificate[] = {
#include "root_ca.der.inc"
};

- And in the src/main.c, we must set the buffer big enough to handle the response: TLS decrypted record + HTTP header + data; so it must be 8192 bytes at least, or it will yell at you.

- Oh, and the write_buffer, let set it 8192, too, for syncho, i know it may not relevant, but who know?

- Then the callback, we must use the rsp->buf_frag_len and other; dont use data_len; data_len include: HTTP headers + previous body fragments still in buffer + current fragment; body_frag is only the new body fragment.

- And last, for the https.conf, add this: CONFIG_MBEDTLS_SSL_MAX_CONTENT_LEN=16384.

Thanks again.
