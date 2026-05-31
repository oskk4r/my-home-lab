# 🏠 My Home Lab

Witaj w moim osobistym projekcie Home Lab! Jest to miejsce, w którym dokumentuję budowę, konfigurację oraz rozwój mojej domowej infrastruktury serwerowej. 

### 🎯 Cele projektu:
* Nauka wirtualizacji (Proxmox VE)
* Zaawansowana administracja systemami Linux (Debian/Ubuntu)
* Automatyzacja usług i konteneryzacja (Docker/Ansible)
* Bezpieczeństwo sieciowe i reverse proxy

---

## 🖥️ Specyfikacja Sprzętowa

* **Host (Hypervisor):** Lenovo ThinkCentre m720q Tiny
* **CPU:** Intel(R) Core(TM) i5-8400T (6 rdzeni, 6 wątków)
* **RAM:** 16GB DDR4
* **Dysk:** 256GB NVMe SSD
* **System operacyjny:** Proxmox VE 8.x

---

## 🌐 Konfiguracja Sieciowa (Proxmox Host)

Plik `/etc/network/interfaces` na hoście Proxmox zawiera konfigurację dwóch mostków sieciowych:
* `vmbr0` – Mostek główny, spięty z fizyczną kartą sieciową (`nic0`), odpowiada za dostęp do sieci domowej i przekierowania portów (SSH, HTTP, HTTPS) do kontenera Portiera.
* `vmbr1` – Odizolowana, wewnętrzna sieć labowa (`10.0.0.0/24`) z włączonym NAT-em (Masquerade), dzięki czemu maszyny wirtualne mają dostęp do internetu, ale sieć domowa ich nie widzi.

<details>
<summary>🛠️ Zobacz pełny plik /etc/network/interfaces</summary>

```text
auto vmbr0
iface vmbr0 inet static
        address 192.168.0.200/24
        gateway 192.168.0.1
        bridge-ports nic0
        bridge-stp off
        bridge-fd 0
        # Przekierowania do maszyny "Reverse-Portier" (10.0.0.10)
        post-up iptables -t nat -A PREROUTING -p tcp --dport 2222 -j DNAT --to-destination 10.0.0.10:22
        post-down iptables -t nat -D PREROUTING -p tcp --dport 2222 -j DNAT --to-destination 10.0.0.10:22
        
        post-up iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 10.0.0.10:80
        post-down iptables -t nat -D PREROUTING -p tcp --dport 80 -j DNAT --to-destination 10.0.0.10:80

        post-up iptables -t nat -A PREROUTING -p tcp --dport 443 -j DNAT --to-destination 10.0.0.10:443
        post-down iptables -t nat -D PREROUTING -p tcp --dport 443 -j DNAT --to-destination 10.0.0.10:443

        post-up iptables -t nat -A PREROUTING -p tcp --dport 81 -j DNAT --to-destination 10.0.0.10:81
        post-down iptables -t nat -D PREROUTING -p tcp --dport 81 -j DNAT --to-destination 10.0.0.10:81
```
auto vmbr1
iface vmbr1 inet static
        address 10.0.0.1/24
        bridge-ports none
        bridge-stp off
        bridge-fd 0
        # Lab-Izolowany z dostępem do świata (NAT)
        post-up echo 1 > /proc/sys/net/ipv4/ip_forward
        post-up iptables -t nat -A POSTROUTING -s '10.0.0.0/24' -o vmbr0 -j MASQUERADE
        post-down iptables -t nat -D POSTROUTING -s '10.0.0.0/24' -o vmbr0 -j MASQUERADE

## 📖 Dziennik Projektu & Architektura

### 🟢 Krok 1: Przygotowanie środowiska i dostęp SSH
* **Cel:** Wdrożenie bazowej maszyny wirtualnej (VM) i zapewnienie bezpiecznego, zdalnego zarządzania.
* **Szczegóły:**
  * Utworzono VM o nazwie `Reverse-Portier` z systemem **Debian 12 (Bookworm)**.
  * Skonfigurowano dostęp przez SSH w celu wyeliminowania ograniczeń konsoli webowej NoVNC.
  * Wdrożono podstawowe zasady bezpieczeństwa (logowanie kluczem SSH, wyłączenie logowania na `root`).

### 🟡 Krok 2: Instalacja i konfiguracja Nginx Reverse Proxy
* **Cel:** Zapewnienie bezpiecznego punktu wejścia do sieci domowej i przekierowywanie ruchu na podstawie domen.
* **Status:** W trakcie realizacji...
