Suricata + EveBox Docker Compose Setup
======================================

Quickly deploy [Suricata](https://suricata.io/) (Network Intrusion Detection System) and **EveBox** (web-based event viewer) using **Docker Compose**. **Suricata** monitors your network traffic, **EveBox** displays the alerts in a friendly UI.

This was developed to allow people to test out **Suricata**, not as a reliable method to spin up a production **Suricata** instance. For that, refer to the **Suricata** [documentation](https://suricata.io/).

**⚠️ Important Prerequisites**

*   Docker & Docker Compose installed on your host.
*   You must know the name of the network interface you want to monitor (run `ip a` or `ifconfig`).

* * *

🚀 Quick Start
--------------

1.  **Clone / download** the `docker-compose.yml` file.
2.  **Set your network interface** – create a `.env` file next to the compose file:
    
    `MONITOR\_IFACE=_your-interface-name_`
    
    Example: `MONITOR_IFACE=enp86s0`
3.  **Open firewall port** for EveBox (TCP 5636). Example with `ufw`:

    ```
    sudo ufw allow 5636/tcp comment 'EveBox'
    sudo ufw reload
    ```
    
4.  **Start the services:**
    
    `docker compose up -d`

* * *    

✅ Test IDS
----------

Trigger a harmless alert to verify everything is working:

`curl http://testmyids.com`

You can also check if alerts are being written to the log:

`docker compose logs suricata | grep '"event\_type":"alert"'`

* * *

🔐 EveBox Login
---------------

*   Default username: **admin**
*   Find the auto-generated password:
    
    docker compose logs evebox | grep -i password
    
*   Open your browser and go to:
    
    `https://<your-server-ip>:5636`
    
    **Note:** You must use `https://` and accept the self‑signed certificate warning.

* * *

🔄 Updating Suricata Rules
--------------------------

### Manual update (first time)

`docker exec suricata\_test suricata-update -f`

Then restart Suricata to load the new rules:

`docker compose restart suricata`

### Automated daily update (cron job)

Add the following to your host’s crontab (`sudo crontab -e`). This runs every day at 3:00 AM and only restarts Suricata, leaving EveBox untouched:

`0 3 \* \* \* (cd /path/to/your/suricata-directory && docker compose exec -T suricata suricata-update -q && docker compose restart suricata)`

**Remember:** replace `/path/to/your/suricata-directory` with the actual path.

* * *

🔔 Managing Noisy Alarms
------------------------

In the EveBox UI, go to **Alerts** or **Events**, find a noisy signature, and click the **“-”** (minus) button. That rule will be ignored from then on.

* * *

🧹 Stop the stack
-----------------

`docker compose down`

Volumes persist; to also remove all data add `-v`.
