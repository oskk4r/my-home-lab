🏠 My Home Lab
/n Witaj w moim osobistym projekcie Home Lab! Jest to miejsce, w którym dokumentuję budowę, konfigurację oraz rozwój mojej infrastruktury serwerowej. Projekt ma na celu naukę wirtualizacji, administracji systemami Linux oraz automatyzacji usług.

🖥 Specyfikacja Sprzętowa
Host (Hypervisor): Lenovo Tiny m720q6 x Intel(R) Core(TM) i5-8400T CPU

RAM: 16GB DDR4
Dysk: 256GB NVMe SSD
System hosta: Proxmox VE


📖 Dziennik Projektu
Krok 1: Przygotowanie środowiska i dostęp SSH
Cel: Wdrożenie bazowej maszyny wirtualnej (VM) i zapewnienie zdalnego zarządzania.

Szczegóły:

Utworzono VM o nazwie "Reverse-Portier" z systemem Debian 12 (Bookworm).
Skonfigurowano dostęp przez SSH (PuTTY) w celu wyeliminowania ograniczeń konsoli webowej NoVNC.
Ustalono podstawowe zasady bezpieczeństwa dostępu do serwera.

Krok 2: Instalacja Nginx Reverse Proxy
