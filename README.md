# 🏠 My Home Lab

Witaj w moim osobistym projekcie Home Lab! Jest to miejsce, w którym dokumentuję budowę, konfigurację oraz rozwój mojej domowej infrastruktury serwerowej skoncentrowanej wokół bezpiecznej i odizolowanej sieci.

---

### 🚀 Stack Technologiczny
![Proxmox](https://img.shields.io/badge/Proxmox_VE-E74C3C?style=for-the-badge&logo=proxmox&logoColor=white)
![Debian](https://img.shields.io/badge/Debian_12-A81D33?style=for-the-badge&logo=debian&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Nginx](https://img.shields.io/badge/Nginx_Proxy_Manager-009639?style=for-the-badge&logo=nginx&logoColor=white)

### 🎯 Cele projektu
* **Wirtualizacja:** Praktyczne zarządzanie zasobami hypervisora Proxmox VE.
* **Administracja Linux:** Zaawansowana konfiguracja systemów Debian/Ubuntu w środowisku bezgui (Headless).
* **Konteneryzacja:** Wdrażanie i orkiestracja usług za pomocą Dockera.
* **Bezpieczeństwo sieciowe:** Implementacja architektury DMZ/Lab, trasowanie ruchu przez Reverse Proxy oraz zarządzanie regułami firewall z poziomu `iptables`.

---

## 🖥️ Specyfikacja Sprzętowa

| Podzespół | Specyfikacja | Szczegóły |
| :--- | :--- | :--- |
| **Host (Hypervisor)** | Lenovo ThinkCentre m720q Tiny | Kompaktowy węzeł typu Mini PC |
| **CPU** | Intel(R) Core(TM) i5-8400T | 6 rdzeni, 6 wątków (baza 1.70 GHz, turbo 3.30 GHz) |
| **RAM** | 16GB DDR4 | Pamięć operacyjna hosta i maszyn wirtualnych |
| **Dysk** | 256GB NVMe SSD | Szybka pamięć masowa na system i kontenery |
| **System operacyjny**| Proxmox VE 8.x | Hypervisor typu 1 (Bare-metal) |

---

## 🌐 Konfiguracja Sieciowa (Proxmox Host)

Architektura sieci opiera się na **rygorystycznej izolacji (Network Segmentation)** za pomocą dwóch wirtualnych mostków (Linux Bridges):

* **`vmbr0` (WAN / Sieć Domowa):** Mostek spięty z fizyczną kartą sieciową. Odpowiada za komunikację zewnętrzną hosta Proxmox oraz zbieranie ruchu z sieci domowej (`192.168.0.0/24`). Za pomocą reguł `DNAT` przekazuje pakiety na wybrane porty do kontenera z Reverse Proxy.
* **`vmbr1` (LAN / Izolowany Lab):** Wewnętrzna, odizolowana sieć (`10.0.0.0/24`) dedykowana dla maszyn wirtualnych i kontenerów. Posiada włączoną maskaradę (NAT/SNAT), zapewniając maszynom dostęp do internetu bez ujawniania ich tożsamości w sieci domowej.

<details>
<summary>🛠️ Zobacz produkcyjny plik /etc/network/interfaces</summary>

```text
auto vmbr0
iface vmbr0 inet static
        address 192.168.0.200/24
        gateway 192.168.0.1
        bridge-ports nic0
        bridge-stp off
        bridge-fd 0
        
        # Przekierowania ruchu (DNAT) do maszyny "Reverse-Portier" (10.0.0.10)
        post-up iptables -t nat -A PREROUTING -p tcp --dport 2222 -j DNAT --to-destination 10.0.0.10:22
        post-down iptables -t nat -D PREROUTING -p tcp --dport 2222 -j DNAT --to-destination 10.0.0.10:22
        
        post-up iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 10.0.0.10:80
        post-down iptables -t nat -D PREROUTING -p tcp --dport 80 -j DNAT --to-destination 10.0.0.10:80

        post-up iptables -t nat -A PREROUTING -p tcp --dport 443 -j DNAT --to-destination 10.0.0.10:443
        post-down iptables -t nat -D PREROUTING -p tcp --dport 443 -j DNAT --to-destination 10.0.0.10:443

        post-up iptables -t nat -A PREROUTING -p tcp --dport 81 -j DNAT --to-destination 10.0.0.10:81
        post-down iptables -t nat -D PREROUTING -p tcp --dport 81 -j DNAT --to-destination 10.0.0.10:81

auto vmbr1
iface vmbr1 inet static
        address 10.0.0.1/24
        bridge-ports none
        bridge-stp off
        bridge-fd 0
        # Włączenie forwardowania IP oraz translacji adresów sieciowych (NAT)
        post-up echo 1 > /proc/sys/net/ipv4/ip_forward
        post-up iptables -t nat -A POSTROUTING -s '10.0.0.0/24' -o vmbr0 -j MASQUERADE
        post-down iptables -t nat -D POSTROUTING -s '10.0.0.0/24' -o vmbr0 -j MASQUERADE
```
</details>

## 📖 Dziennik Projektu & Kamienie Milowe

### 🟢 Krok 1: Przygotowanie Środowiska & Hardening SSH
STATUS: ZAKOŃCZONY | SYSTEM: Debian 12 Bookworm | PORT SSH: 2222

🏗️ Powołanie maszyny: Wdrożenie bazowej maszyny wirtualnej (VM) o nazwie Reverse-Portier pełniącej rolę bramy sieciowej.
🔒 Hardening dostępu: Pełne zabezpieczenie demona SSH poprzez całkowite zablokowanie logowania na konto root oraz wyłączenie autoryzacji tradycyjnym hasłem. Ruch dopuszczany jest wyłącznie za pomocą kluczy asymetrycznych.
🛠️ Routing: Pomyślne przekierowanie dedykowanego portu z firewalla hosta Proxmox bezpośrednio do kontenera w celu bezpiecznego zarządzania zdalnego z pominięciem konsoli NoVNC.

### 🟢 Krok 2: Wdrożenie Nginx Reverse Proxy
STATUS: ZAKOŃCZONY | ŚRODOWISKO: Docker & Docker Compose | STREFA DOMENOWA: .local

🐳 Konteneryzacja: Instalacja silnika Docker oraz narzędzia Docker Compose na maszynie brzegowej jako fundamentu pod mikrotransakcje sieciowe.
🛡️ Strażnik Ruchu: Uruchomienie i konfiguracja panelu Nginx Proxy Manager (NPM) jako centralnego punktu terminacji ruchu.
🔑 Szyfrowanie SSL: Ręczne wygenerowanie autorskich certyfikatów samopodpisanych (Self-Signed) za pomocą OpenSSL dla lokalnej strefy sieciowej.
🕸️ Obsługa Websockets: Skonfigurowanie stabilnego proxy dla domeny proxmox.local z wymuszonym protokołem HTTPS oraz pełną obsługą Websocketów, co ostatecznie wyeliminowało błędy wygasania sesji i tokenów (401: No ticket).
🔗 Mapowanie panelu NPM: Przypisanie dedykowanej nazwy domeny nginx.local kierującej bezpośrednio na port administracyjny 81, usuwając konieczność ręcznego pamiętania portów.

### 🟡 Krok 3: Wdrożenie Wirtualnego Firewalla (OPNsense)
STATUS: W TRAKCIE | SYSTEM: OPNsense | ARCHITEKTURA: Zero-Trust Management

🌐 Migracja warstwy sieciowej: Przeniesienie routingu oraz translacji adresów (NAT) z iptables do dedykowanego systemu OPNsense.
🛡️ Zasada "Głuchego WAN-u": Wdrożenie rygorystycznej blokady sieci prywatnych na interfejsie WAN. Całkowita rezygnacja z wystawiania panelu administracyjnego na świat.
🔑 Architektura Bastion Host: Zaprojektowanie bezpiecznego punktu dostępu (Admin VM) wewnątrz sieci LAN. Zarządzanie firewallem odbywa się wyłącznie poprzez zaufaną strefę, co eliminuje ryzyko ataków na interfejs zarządzający.
📊 Outbound NAT: Konfiguracja reguł Hybrid Outbound, zapewniająca bezpieczną komunikację labu z Internetem.

---
