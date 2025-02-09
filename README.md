# **MikroTik RouterOS v6.x.x Game Port Configuration**

Port game ini di-*torch* menggunakan MikroTik secara manual untuk setiap game yang dimainkan dengan target IP tunggal serta satu perangkat tanpa batas latar belakang saat memainkan game.

## **Konfigurasi Firewall MikroTik**
Anda dapat menyesuaikan nama interface dengan nama interface yang Anda gunakan.

**Harap perhatikan catatan untuk setiap pengaturan.**

### **Metode yang Digunakan:**
1. Address List
2. RAW
3. Mangle
4. Route
5. Queue Simple

---
## **Pengaturan Address List**
```mikrotik
/ip firewall address-list
add address=10.0.0.0/8 list=CLIENT
```

**Catatan:** Sesuaikan address dengan IP WAN milik pelanggan/client Anda.

---
## **Pengaturan RAW**
```mikrotik
/ip firewall raw
add action=add-dst-to-address-list address-list=IP-GAME address-list-timeout=\
    30m chain=prerouting comment=TCP-PORT dst-address-list=!CLIENT dst-port=\
    xxxx,xxxxx,xxxx protocol=\
    tcp src-address-list=CLIENT
add action=add-dst-to-address-list address-list=IP-GAME address-list-timeout=\
    30m chain=prerouting  comment=UDP-PORT dst-address-list=!CLIENT dst-port=\
    xxxx,xxxxx,xxxx protocol=udp src-address-list=\
    CLIENT
```

**Catatan:** Ganti **xxxx,xxxxx,xxxx** di **dst-port** pada RAW TCP dan RAW UDP sesuai game yang anda inginkan.
- Silahkan buka halaman port game yang anda inginkan untuk mengcopy paste portnya
- **RAW baru yang anda buat harus berada di bawah raw game lain, agar tidak mengganggu raw anda**
  - RAW Port TCP Mobile Legend
  - RAW Port UDP Mobile Legend
  - RAW Port TCP Free Fire
  - Raw Port UDP Free Fire
  - dan port game lainnya
  - RAW anda ada disini
  - .......

---
## **Pengaturan Mangle**
```mikrotik
/ip firewall mangle
add action=mark-connection chain=prerouting comment=KONEKSI-GAME \
    dst-address-list=IP-GAME new-connection-mark=Koneksi-Game passthrough=yes \
    src-address-list=CLIENT
add action=mark-routing chain=prerouting comment="ROUTING-GAME" \
    connection-mark=Koneksi-Game new-routing-mark=Route-Game passthrough=no \
    src-address-list=CLIENT
add action=mark-packet chain=forward comment="MARK-PACKET-GAME" \
    connection-mark=Koneksi-Game new-packet-mark=Paket-Game passthrough=no
```

**Catatan:**
- Jika menggunakan Load Balance, pindahkan tiga pengaturan Mangle ini tepat di atas pengaturan Load Balance milik Anda.
- **Jika menggunakan Load Balance, urutan Mangle harus:**
  - Mangle Accept Koneksi Lokal, Client, dll.
  - KONEKSI-GAME
  - ROUTING-GAME
  - MARK-PACKET-GAME
  - Pengaturan Load Balance
  - .......

---
## **Pengaturan Route Tanpa Load Balance**
```mikrotik
/ip route
add check-gateway=ping comment="ROUTE GAME" distance=1 gateway=\
    192.168.110.1 routing-mark=Route-Game
```

**Catatan:** Sesuaikan Gateway dengan Gateway ISP milik Anda.

---
## **Pengaturan Route Dengan Load Balance**
```mikrotik
/ip route
add check-gateway=ping comment="ROUTE GAME A" distance=1 gateway=\
    192.168.110.1 routing-mark=Route-Game
add check-gateway=ping comment="ROUTE GAME B" distance=2 gateway=\
    192.168.120.1 routing-mark=Route-Game
add check-gateway=ping comment="ROUTE GAME C" distance=3 gateway=\
    192.168.130.1 routing-mark=Route-Game
```

**Catatan:**
- **Distance 2 dan 3 hanya sebagai backup** jika ISP utama mengalami gangguan.
- Sesuaikan Gateway dengan Gateway ISP milik Anda.

---
## **Pengaturan Queue Simple**
```mikrotik
/queue simple
add name="ALL TRAFIK" target=10.0.0.0/8
add limit-at=5M/5M max-limit=5M/5M name=TRAFIK GAME packet-marks=Paket-Game parent=\
    "ALL TRAFIK" priority=1/1 target=10.0.0.0/8
```

**Catatan:**
- Sesuaikan target dengan IP pelanggan/client.
- **Urutan Queue harus:**
  - ALL TRAFIK
  - TRAFIK GAME
  - Queue Simple Bandwith
  - Queue Simple lainnya
  - ......
