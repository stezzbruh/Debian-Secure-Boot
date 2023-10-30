# **Debian Secure Boot**
this is a shortened guide to getting secure boot working with Nvidia drivers and MOK manager
before we begin make sure you are root user just type `su` into terminal

1. Create the MOK folder (this is where MOK stores the keys for secure boot to read):

   ```mkdir -p /var/lib/shim-signed/mok/```

3. Open the Folder:

   ```cd /var/lib/shim-signed/mok/```

5. Create the key (replace UR NAME with your name):

   1.```1. openssl req -new -x509 -newkey rsa:2048 -keyout MOK.priv -outform DER -out MOK.der -days 36500 -subj "/CN=UR NAME/"```

   2.```openssl x509 -inform der -in MOK.der -out MOK.pem```

7. Import the key you made to MOK manager (will ask for a 1 time password. be sure to remember it):

   ```mokutil --import /var/lib/shim-signed/mok/MOK.der```

9. Test the key (it will show as enrolled):

   ```mokutil --test-key /var/lib/shim-signed/mok/MOK.der```

11. Editing the framework configuration:

    1.```sudo nano /etc/dkms/framework.conf```

add these next few lines in at the bottom
   
   2.```mok_signing_key="/var/lib/shim-signed/mok/MOK.priv"```
   
   3.```mok_certificate="/var/lib/shim-signed/mok/MOK.der"```
   
   
   4.```sign_tool="/etc/dkms/sign_helper.sh"```
   
   5. press `ctrl`+`s` then `ctrl` + `x` to save

11. Create the DKMS signer (this will go and automatically sign Dynamic Kernel Module Sources a.k.a DKMS):

    1.```sudo nano /etc/dkms/sign_helper.sh```
   add these into the file

    2.```/lib/modules/"$1"/build/scripts/sign-file sha512 /root/.mok/client.priv /root/.mok/client.der "$2"```

    3.```$ VERSION="$(uname -r)"
        $ SHORT_VERSION="$(uname -r | cut -d . -f 1-2)"
        $ MODULES_DIR=/lib/modules/$VERSION
        $ KBUILD_DIR=/usr/lib/linux-kbuild-$SHORT_VERSION```
   
   4. press `ctrl`+`s` then `ctrl`+`x` to save

11. Import the DKMS key for secure boot to read:

   ```sudo mokutil --import /var/lib/dkms/mok.pub```

13. Reboot and MOK manager will appear go to the `enroll`tab and use that passward from step 4 to enroll the key

# **Congratulations**
   you now have a working secure boot with nvidia-driver
