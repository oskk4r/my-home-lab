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
