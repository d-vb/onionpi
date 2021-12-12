# onionpi

Ansible role to build an Onion Pi Tor proxy with a Raspberry Pi 4 Model 4 B.

You can plug the Ethernet cable into any Internet provider in your home, work, hotel or conference/event, power up the Pi with the USB cable to your laptop or to the wall adapter. The Pi will boot up and create a new secure wireless access point. Connecting to that access point will automatically route any traffic from your computer through the anonymizing Tor network.

## Prepare your Pi

* Download the latest image of [Raspberry Pi OS (32-bit) Lite](https://downloads.raspberrypi.org/raspios_lite_armhf_latest) and follow the [installation manual](https://www.raspberrypi.org/documentation/installation/installing-images/).

  ```sh
  unzip YYYY-MM-DD-raspios-bullseye-armhf-lite.zip
  sudo dd if=YYYY-MM-DD-raspios-bullseye-armhf-lite.img of=/dev/sdX bs=4M conv=fsync status=progress
  sudo dd if=/dev/sdX of=from-sd-card.img bs=4M count=xxx
  sudo truncate --reference YYYY-MM-DD-raspios-bullseye-armhf-lite.img from-sd-card.img
  diff -s from-sd-card.img YYYY-MM-DD-raspios-bullseye-armhf-lite.img
  sync
  ```

* Login as user `pi` with the password `raspberry` (don't worry about the default password, we will delete this user while installtion). Start the [Raspberry Pi configuration tool](https://www.raspberrypi.org/documentation/configuration/raspi-config.md), set a hostname, [enable SSH](https://www.raspberrypi.org/documentation/remote-access/ssh/README.md) and log out.

  ```sh
  sudo raspi-config
  1 System Options -> S4 Hostname -> onionpi
  3 Interfacing Options -> P2 SSH -> Yes
  ```

* Configure [SSH access](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-2) for root user.

  ```sh
  cat ~/.ssh/id_rsa.pub | ssh pi@onionpi "sudo sh -c 'mkdir -p /root/.ssh && chmod 700 /root/.ssh && cat >> /root/.ssh/authorized_keys'"
  ```

## Deploy Ansible role

* Change to a suitable local directory and clone this repository to `roles/onionpi`.

  ```sh
  git clone https://github.com/d-vb/onionpi.git roles/onionpi
  ```

* Create an [Ansible inventory](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html) `inventories/onionpi.yml` and specify the Wifi passphrase you want to use to connect to your Onion Pi.

  ```yaml
  all:
    children:
      onionpi:
        hosts:
          onionpi:
            ansible_user: "root"
            wpa_passphrase: "ENTER_YOUR_PASSPHRASE_HERE"
  ```

* Create an [Ansible playbook](https://docs.ansible.com/ansible/latest/user_guide/playbooks_intro.html) `onionpi.yml` that will run your Ansible role.

  ```yaml
  - hosts: onionpi
    become: true
    roles:
      - onionpi
  ```

* [Install Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) and run [ansible-playbook](https://docs.ansible.com/ansible/latest/cli/ansible-playbook.html).

  ```sh
  ansible-playbook onionpi.yml -i inventories/onionpi.yml
  ```

## And finally

* Connect to your Onion Pi Tor Wifi (default SSID is onionpi) with the passphrase you specified earlier.

* Visit https://check.torproject.org/ to verify you are using Tor.

## Customizing

If you want to adjust some default settings like SSID and IP-Addresses, just edit `defaults/main.yml`.
