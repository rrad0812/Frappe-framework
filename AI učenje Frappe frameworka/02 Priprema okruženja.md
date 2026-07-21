
# Učenje Frappe Frameworka

[Sadržaj][00]

## 02 Priprema okruženja

- **Preuzimnje ISO slike i instalacija**

  </br>

  - Sa adrese <https://releases.ubuntu.com/24.04.4/ubuntu-24.04.4-live-server-amd64.iso> preuzeti
    ISO za `Ubuntu server 24.04`.

  </br>
  
  - Izgradnja Ubuntu Server 24.04 VM  
    VM izgradnja na QUEMU/KVM root sesiji sa sledećim parametrima:

    - vCPU 2
    - RAM 8GB
    - SSD 60GB

</br>

- **Ažuriranje paketa distibucije**

  ```sh
  sudo apt update && sudo apt upgrade -y
  ```

</br>

- **Instalacija curl-a**

  ```sh
  sudo apt install curl
  ```

</br>

- **Aktiviranje firewall-a i dozvola ssh pristupa**

  </br>

  - Aktiviranje firewall-a
  
    ```sh
    sudo ufw enable
    ```
  
  </br>
  
  - Dozvola pristupa preko ssh
  
    ```sh
    sudo ufw allow OpenSSH
    ```
  
  </br>
  
  - Proba konekcije na VM ( samo sa localhost-a )
    Iz virtuelne mašine pokrenuti:

    ```sh
    ip a
    ```

    Vratiće, za KVM slučaj, nešto kao 192.168.122.X.
  
  </br>
  
  - Sa hosta ssh pristup na VM
  
    ```sh
    ssh username_na_VM@127.198.122.X
    ```
  
  - Reset VM
  
    ```sh
    sudo shutdown -r now
    ```

- **Promena vremenske zone i sync. vremena**

  ```sh
  sudo timedatectl set-timezone Europe/Belgrade
  sudo timedatectl set-ntp on
  ```

</br>

- **Promena locale**

  ```sh
  sudo dpkg-reconfigure locales
  ```

  </br>
  
  - Dodaj nove locale `sr_RS@latin UTF-8` i postavi ih za default.
  
  </br>
  
  - Reset VM
  
    ```sh
    sudo shutdown -r now
    ```

</br>

- **Instalacija osnovnih dev paketa**

  ```sh
  sudo apt python3-dev
  ```

</br>

- **Instalacija uv pajton paket i runtime managera**

  ```sh
  curl -LsSf https://astral.sh/uv/install.sh | sh
  ```
  
  </br>
  
  - Osveži sesiju:

    ```sh
    source "$HOME/.local/bin/env"
    ```

  </br>
  
  - Proveri instaliranost:

    ```sh
    uv --version
    ```

[Sadržaj][00]

[00]: 00%20Učenje%20Frape%20frameworka.md
