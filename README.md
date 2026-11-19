# 🏠 My Home Lab

Dokumentacja mojej domowej infrastruktury serwerowej opartej na hypervisorze **Proxmox VE**, routerze **MikroTik hEX** oraz odizolowanej podsieci `10.10.10.0/24`, na której uruchomiona jest maszyna wirtualna **Debian 12** (`discord-bot-vm`) obsługująca bota Discord (`/opt/discord-bot/main.py`).

---

## 🚀 Stack Technologiczny

* **Hypervisor:** Proxmox VE (`serwer`)
* **Routing i Firewall:** MikroTik hEX (RouterOS)
* **System gościa:** Debian 12 (Bookworm) — `discord-bot-vm` (ID `100`)
* **Usługi i aplikacje:** Python 3.11 (`discord.py`), systemd

---

## 🖥️ Sprzęt i Topologia

* **Serwer (Proxmox VE):** Mini PC Lenovo (`serwer`), interfejs `nic0` (`enp0s31f6`) wpięty do mostka `vmbr0` ze statycznym adresem `10.10.10.200/24` i bramą `10.10.10.1`.
* **Router brzegowy (MikroTik hEX):**
  * `ether1` (WAN / połączenie z siecią domową `192.168.0.0/16`): adres `192.168.0.38/24`.
  * `ether2` wpięty w `bridgeLocal`: adres `10.10.10.1/24` (obsługa podsieci `10.10.10.0/24` i połączenie z serwerem Lenovo).
* **Maszyna wirtualna `discord-bot-vm` (ID `100`):** Debian 12, interfejs `ens18` ze statycznym adresem `10.10.10.201/24` i bramą `10.10.10.1`, uruchamiający bota w `/opt/discord-bot/main.py`.
* **Stacja robocza (PC):** adres `192.168.0.201` w sieci domowej.

---

## 🌐 Konfiguracja Sieciowa i Izolacja

* **Izolacja podsieci (`10.10.10.0/24` vs `192.168.0.0/16`):** W MikroTiku w `IP -> Firewall -> Filter Rules` (łańcuch `forward`) reguła `Isolate Proxmox from home LAN` (`action=drop`) blokuje ruch z `10.10.10.0/24` do `192.168.0.0/16`, podczas gdy reguła `accept` dla `connection-state=established,related` przepuszcza ruch powrotny.
* **Przekierowania portów (`DST-NAT`):** Ruch zarządzający z sieci domowej (`192.168.0.201`) kierowany na `192.168.0.38` (`ether1`) przekierowuje:
  * `TCP 8006` ➡️ `10.10.10.200:8006` (panel Proxmox Web UI),
  * `TCP 2222` ➡️ `10.10.10.201:22` (SSH do maszyny `discord-bot-vm`).
* **Konfiguracja `vmbr0` na hoście Proxmox (`/etc/network/interfaces`):**

```text
iface vmbr0 inet static
        address 10.10.10.200/24
        gateway 10.10.10.1
        bridge-ports nic0
        bridge-stp off
        bridge-fd 0
```

---

## 🤖 Maszyna `discord-bot-vm` (Debian 12)

* **Adresacja:** `10.10.10.201/24` na `ens18` (brama `10.10.10.1`).
* **Aplikacja:** `/opt/discord-bot/venv/bin/python3 -u /opt/discord-bot/main.py`.
* **Łączność i bezpieczeństwo:** VM ma wyjście do internetu przez bramę `10.10.10.1` i MikroTika, bot odpowiada na Discordzie, a próby połączeń inicjowane z VM do `192.168.0.201` są blokowane.

---

## 📌 Stan Obecny i Kolejne Kroki

* **Aktualny stan:** Proxmox (`10.10.10.200/24`) i `discord-bot-vm` (`10.10.10.201/24`) działają w podsieci `10.10.10.0/24` za MikroTikiem hEX, dostęp do Proxmoxa (`:8006`) i SSH na Debianie (`:2222`) z PC działa, izolacja do `192.168.0.201` jest aktywna, a bot działa.
* **W planach:** Dalsze dopracowanie reguł/limitów firewalla oraz weryfikacja tunelu Cloudflare WARP.
