This note was taken after i spend few hours to read about
[GnuPG](https://www.gnupg.org)

- The first thing, you need understand how to encrypt and decrypt a data by
using key pair: Public key and Private key


- Next, you need to create key pair (including Public key and Private key),
public key to share to others, and private key for yourself. It will be saved
in $HOME/.gnupg/

    - To create: `$ gpg --gen-key`
    - Export to file with ASCII format: `$ gpg --armor --export "your_id" >
    publickey.txt`
    - You can distribute your public key by upload to `keyserver` or your
    index. To upload to keyserver, you can type: `$ gpg --send-keys 'your_id'
    --keyserver hkp://subkeys.pgp.net`. **Note**: Keyserver usally listen on
    port 11371, so make sure that your firewall is allow this port.

- To encrypt the data (For yourseft): `$ gpg -e -r "your_id" data.txt`.
Wait for minute, a file named data.txt.gpg will be created. This is
your data was encrypted.
- To encrypt the data then send to others, you need import their public key by
two way:
    - Using their public key: `$ gpg --import theirpublickey.txt`
    - Search on key server: `$ gpg --keyserver hkp://subkeys.pgp.net 
    --search-keys 'recipient_id'`. You can see the list then select it to
    import.

After import recipient's public key successfull, you can encrypt the data to
send their by command:
`$ gpg -e -r "recipient_id" data.txt`
Finally, you only send the data encrypted (here is data.txt.gpg) to recipient.

Decrypt data you have received by: `$ gpg -d data.txt.gpg > data.txt`. You
must enter your private key to decrypt data.

- To sign data by your key (including to your data): `$ gpg --sign data.txt`.
- To sign data into seperate file: `$ gpg --detach-sign data.txt`.You will see
an file named data.txt.asc.
To verify them by: `$ gpg --verify data.txt.asc data.txt`.

- Combination: You can encrypt data and sign it (encrypted data). Then send
to recipient.
```
$ gpg -u Sender -r Recipient --encrypt data.txt
$ gpg --detach-sign data.txt.gpg
```
You will see two file: data.txt.gpg data.txt.gpg.sign
=> Send there files to recipient. The receiver must be verify it by:
`$ gpg --verify data.txt.sign data.txt.gpg`. Will be correct if nothing wrong
show in output.

- To manage keys:
    - List keys: `$ gpg --list-keys`
    - Delete key: `$ gpg --delete-key user_id`

- For big data, The encryption with GPG will take a lot of resource (CPU) and
times. That is reason for asymmetric encryption. GPG also supported symmetric
encryption. To do that, you can use --symmetric option. This is way to encrypt
big data with GnuPG:
    - Generate a strong password and save it into a file named `passwd.txt`
    - Encrypt big file with symmetric encription: `$ gpg --symmetric big_file`
    then enter the password that saved in `passwd.txt`
    - Encrypt `passwd.txt` by asymmetric encryption.
    - You have two file `passwd.txt.gpg` and `big_file.gpg`. The receiver can
    get password by decrypt password from `passwd.txt.gpg` then use it to
    decrypt `big_file.gpg`.

To be ontinue...