# Panduan Konfigurasi Windows Server 2022 & Windows 10

Panduan ini menyediakan arahan langkah demi langkah untuk menyediakan persekitaran makmal menggunakan Windows Server 2022 sebagai pelayan domain dan Windows 10 sebagai klien. Konfigurasi ini adalah berdasarkan satu set spesifikasi teknikal yang lazim digunakan dalam latihan dan pembelajaran.

**Penafian:** Konfigurasi ini, terutamanya dasar kata laluan yang dipermudahkan, direka **KHAS UNTUK PERSEKITARAN MAKMAL TERKAWAL SAHAJA**. Jangan gunakan tetapan ini dalam persekitaran produksi kerana ia tidak selamat.

**Nota Penting:** Dalam keseluruhan panduan ini, gantikan `SN` dengan nombor stesen anda.

## Senarai Kandungan
1.  [Pra-syarat](#pra-syarat)
2.  [Bahagian 1: Penyediaan Persekitaran Maya](#bahagian-1-penyediaan-persekitaran-maya)
3.  [Bahagian 2: Konfigurasi Windows Server 2022](#bahagian-2-konfigurasi-windows-server-2022)
    *   [2.1 Pemasangan & Konfigurasi Asas Pelayan](#21-pemasangan--konfigurasi-asas-pelayan)
    *   [2.2 Pemasangan Active Directory Domain Services (AD DS)](#22-pemasangan-active-directory-domain-services-ad-ds)
    *   [2.3 Mengubah Dasar Kata Laluan Domain (Langkah Kritikal)](#23-mengubah-dasar-kata-laluan-domain-langkah-kritikal)
    *   [2.4 Mencipta Pengguna & Kumpulan Active Directory](#24-mencipta-pengguna--kumpulan-active-directory)
    *   [2.5 Konfigurasi Pelayan DHCP](#25-konfigurasi-pelayan-dhcp)
    *   [2.6 Konfigurasi Pelayan DNS](#26-konfigurasi-pelayan-dns)
    *   [2.7 Konfigurasi Pelayan FTP](#27-konfigurasi-pelayan-ftp)
4.  [Bahagian 3: Konfigurasi Klien Windows 10](#bahagian-3-konfigurasi-klien-windows-10)
5.  [Bahagian 4: Pengujian & Pengesahan](#bahagian-4-pengujian--pengesahan)

---

### Pra-syarat

Sebelum bermula, pastikan anda mempunyai:
*   Sebuah PC Hos (cth: Windows 11) yang mampu menjalankan mesin maya.
*   Perisian virtualisasi seperti VMware Workstation, VirtualBox, atau Hyper-V.
*   Fail imej ISO untuk Windows Server 2022.
*   Fail imej ISO untuk Windows 10.

### Bahagian 1: Penyediaan Persekitaran Maya

1.  **Cipta Rangkaian Maya:**
    *   Dalam perisian virtualisasi anda, cipta satu suis maya (*virtual switch*) baru.
    *   Tetapkan jenis rangkaian kepada **Host-Only** atau **Private Network**. Ini akan mengasingkan komunikasi antara pelayan dan klien daripada rangkaian luar, mewujudkan persekitaran makmal yang selamat.
    *   Pastikan pelayan DHCP terbina dalam perisian virtualisasi untuk rangkaian ini **dimatikan**, kerana kita akan mengkonfigurasinya pada Windows Server.

### Bahagian 2: Konfigurasi Windows Server 2022

#### 2.1 Pemasangan & Konfigurasi Asas Pelayan

1.  **Cipta Mesin Maya Pelayan:**
    *   Cipta mesin maya baru menggunakan imej ISO Windows Server 2022.
    *   Sambungkan mesin maya ini ke rangkaian maya yang telah anda cipta tadi.
    *   Selesaikan proses pemasangan Windows Server.

2.  **Tetapan Awal Pelayan:**
    *   **Tukar Nama Komputer:**
        *   Buka PowerShell sebagai Administrator dan taip:
          ```powershell
          Rename-Computer -NewName "WINsrvSN" -Restart
          ```
    *   **Tetapkan Alamat IP Statik:**
        *   Pergi ke `Control Panel` > `Network and Sharing Center` > `Change adapter settings`.
        *   Klik kanan pada adapter rangkaian anda dan pilih `Properties`.
        *   Pilih `Internet Protocol Version 4 (TCP/IPv4)` dan klik `Properties`.
        *   Masukkan tetapan berikut:
            *   **IP address:** `192.168.SN.100`
            *   **Subnet mask:** `255.255.255.0`
            *   **Preferred DNS server:** `192.168.SN.100` (menunjuk kepada diri sendiri)

#### 2.2 Pemasangan Active Directory Domain Services (AD DS)

1.  **Pasang Peranan:**
    *   Buka **Server Manager**, klik `Manage` > `Add Roles and Features`.
    *   Ikuti wizard sehingga ke `Server Roles`, dan pilih `Active Directory Domain Services`.
    *   Terima ciri-ciri tambahan yang dicadangkan dan selesaikan pemasangan.

2.  **Promosikan sebagai Domain Controller:**
    *   Setelah pemasangan selesai, klik pada ikon notifikasi (bendera) di Server Manager dan pilih `Promote this server to a domain controller`.
    *   Pilih `Add a new forest`.
    *   **Root domain name:** `test1.com`
    *   Tetapkan kata laluan **Directory Services Restore Mode (DSRM)** kepada `lkmb123`.
    *   Teruskan dengan pilihan lalai sehingga pemasangan bermula. Pelayan akan dimulakan semula secara automatik.

#### 2.3 Mengubah Dasar Kata Laluan Domain (Langkah Kritikal)

Secara lalai, Windows Server menguatkuasakan kata laluan yang kompleks. Kita perlu melumpuhkan tetapan ini untuk menggunakan kata laluan mudah seperti `1234`.

1.  **Buka Group Policy Management:**
    *   Di **Server Manager**, pergi ke `Tools` > `Group Policy Management`.

2.  **Edit Default Domain Policy:**
    *   Kembangkan `Forest: test1.com` > `Domains` > `test1.com`.
    *   Klik kanan pada **Default Domain Policy** dan pilih `Edit...`.

3.  **Lumpuhkan Keperluan Kerumitan:**
    *   Navigasi ke: `Computer Configuration` > `Policies` > `Windows Settings` > `Security Settings` > `Account Policies` > `Password Policy`.
    *   Di panel kanan, klik dua kali pada `Password must meet complexity requirements`.
    *   Pilih **Disabled** dan klik `OK`.
    *   (Pilihan) Klik dua kali `Minimum password length` dan tetapkan kepada `0` jika perlu.

4.  **Kuatkuasakan Polisi:**
    *   Buka **Command Prompt** atau **PowerShell** sebagai Administrator dan laksanakan arahan berikut untuk mengemas kini polisi dengan serta-merta:
      ```cmd
      gpupdate /force
      ```

#### 2.4 Mencipta Pengguna & Kumpulan Active Directory

1.  Buka **Active Directory Users and Computers** dari `Tools` di Server Manager.
2.  Kembangkan domain `test1.com`.
3.  Cipta kumpulan (*group*) baru jika perlu (cth: `ftp`) dengan klik kanan pada `Users` > `New` > `Group`.
4.  Cipta akaun pengguna berikut dengan klik kanan pada `Users` > `New` > `User`. Pastikan untuk menambah mereka ke kumpulan yang betul.

| Akaun | Kata Laluan | Kumpulan |
| :--- | :--- | :--- |
| LocalAdmin | `skmtest1` | Administrators |
| Manager | `1234` | Administrators |
| Supervisor| `1234` | Power Users |
| User1 | `1234` | Domain Users |
| User2 | `1234` | Domain Users |
| User10 | `1234` | Domain Users |
| Webadmin | `123test1` | ftp |

#### 2.5 Konfigurasi Pelayan DHCP

1.  **Pasang Peranan DHCP:**
    *   Gunakan `Add Roles and Features` di Server Manager untuk memasang peranan `DHCP Server`.
2.  **Konfigurasi Skop:**
    *   Selesaikan konfigurasi pasca-pemasangan DHCP melalui notifikasi di Server Manager.
    *   Buka `DHCP` dari menu `Tools`.
    *   Klik kanan pada `IPv4` dan pilih `New Scope...`.
    *   **Scope Name:** `LAN`
    *   **IP Address Range:** `192.168.SN.110` hingga `192.168.SN.130`
    *   **Subnet Mask:** `255.255.255.0`
    *   **Router (Default Gateway):** Biarkan kosong.
    *   **DNS Servers:** Pastikan ia ditetapkan kepada `192.168.SN.100`.
    *   Aktifkan skop tersebut apabila wizard selesai.

#### 2.6 Konfigurasi Pelayan DNS

Peranan ini dipasang secara automatik bersama AD DS. Sahkan konfigurasinya:
1.  Buka `DNS` dari menu `Tools`.
2.  Kembangkan `Forward Lookup Zones` > `test1.com`.
3.  Pastikan terdapat rekod hos (A) untuk `WINsrvSN` yang menunjuk ke `192.168.SN.100`.

#### 2.7 Konfigurasi Pelayan FTP

1.  **Pasang Peranan IIS & FTP:**
    *   Gunakan `Add Roles and Features` untuk memasang peranan `Web Server (IIS)`.
    *   Dalam `Role Services`, pastikan anda memilih `FTP Server` dan `FTP Service`.

2.  **Sediakan Tapak FTP:**
    *   Cipta folder `C:\ftp_test1`.
    *   Buka `Internet Information Services (IIS) Manager` dari `Tools`.
    *   Klik kanan pada `Sites` > `Add FTP Site...`.
    *   **FTP site name:** `ftp test1`
    *   **Physical path:** `C:\ftp_test1`
    *   **Binding:** Tetapkan kepada IP Address `192.168.SN.100` pada Port `21`.
    *   **SSL:** Pilih `No SSL`.
    *   **Authentication:** `Basic`
    *   **Authorization:** `Specified users`, masukkan `webadmin`.
    *   **Permissions:** `Read` dan `Write`.
    *   Klik `Finish`.

### Bahagian 3: Konfigurasi Klien Windows 10

1.  **Cipta Mesin Maya Klien:**
    *   Cipta mesin maya baru menggunakan imej ISO Windows 10.
    *   Sambungkan mesin maya ini ke rangkaian maya yang sama dengan pelayan.
    *   Pastikan tetapan rangkaiannya ditetapkan untuk **mendapatkan alamat IP secara automatik (DHCP)**.

2.  **Tukar Nama Komputer & Sertai Domain:**
    *   Setelah Windows 10 dipasang, buka `Settings` > `System` > `About`.
    *   Klik `Rename this PC (advanced)`.
    *   Di bawah tab `Computer Name`, klik `Change...`.
    *   **Computer name:** `hostSN`
    *   **Member of:** Pilih `Domain` dan taip `test1.com`.
    *   Klik `OK`. Masukkan kredensial akaun domain yang mempunyai kebenaran (cth., `test1\Manager` dan kata laluan `1234`).
    *   Mulakan semula PC apabila diminta.

### Bahagian 4: Pengujian & Pengesahan

1.  **Ujian Log Masuk Domain:**
    *   Pada skrin log masuk Windows 10, log masuk menggunakan salah satu akaun pengguna domain. Contohnya:
        *   **Username:** `test1\User1`
        *   **Password:** `1234`

2.  **Ujian DHCP & DNS:**
    *   Buka **Command Prompt** pada Windows 10.
    *   Taip `ipconfig /all`. Sahkan klien mendapat alamat IP dalam julat `192.168.SN.110` - `192.168.SN.130` dan DNS Server ialah `192.168.SN.100`.
    *   Taip `ping WINsrvSN`. Anda sepatutnya menerima balasan dari `192.168.SN.100`.

3.  **Ujian Akses FTP:**
    *   Buka **File Explorer** pada Windows 10.
    *   Di bar alamat, taip `ftp://192.168.SN.100` dan tekan Enter.
    *   Apabila digesa, log masuk sebagai:
        *   **Username:** `webadmin`
        *   **Password:** `123test1`
    *   Sahkan anda boleh melihat kandungan folder dan cuba cipta fail baru untuk menguji kebenaran menulis (*write permission*).

Jika semua ujian di atas berjaya, persekitaran makmal anda telah berjaya dikonfigurasikan.
