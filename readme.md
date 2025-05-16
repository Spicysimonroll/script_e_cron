# 🖥️ Linux Server – Script e Cron

## 📋 Obiettivo

Creare un sistema automatico che esegua il **backup giornaliero della directory `/home`**, salvandolo in `/opt/backup`.

Il sistema deve:

- Eseguire il backup una volta al giorno (in formato `.tar.gz`).
- Conservare solo gli ultimi 7 backup.
- Eliminare automaticamente quelli più vecchi.

---

## ⚙️ Configurazione

### 1. ✅ Creazione della cartella `backup`

Assicurarsi che la cartella di destinazione esista:

```bash
sudo mkdir -p /opt/backup
sudo chown root:root /opt/backup
sudo chmod 700 /opt/backup
```

> ℹ️ I seguenti comandi creano la cartella nel caso non esistesse e configurano il proprietario ed i relativi permessi

---

### 2. 🕒 Pianificazione con Cron

Modificare il `crontab` dell’utente `root`:

```bash
sudo crontab -e
```

#### ➕ Aggiungere le seguenti righe:

**Backup giornaliero alle 2:00:**

```cron
0 2 * * * /bin/bash -c 'tar -czf /opt/backup/backup_home_$(date +\%Y\%m\%d).tar.gz /home'
```

> 📌 I nomi dei file includono la data (es. `20250516`) per evitare sovrascritture tra i backup giornalieri.


**Eliminazione dei backup più vecchi di 7 giorni:**

```cron
5 2 * * * /bin/bash -c 'find /opt/backup -name "backup_home_*.tar.gz" -mtime +6 -delete'
```

> ℹ️ L’eliminazione è separata di 5 minuti per evitare conflitti durante la creazione del file.

---

### 3. ✅ Verifica del funzionamento

Controllare che il servizio `cron` sia attivo:

```bash
sudo systemctl status cron
```

> ⚠️ Se il servizio `cron` non è attivo, i backup non verranno eseguiti. **Assicurarsi di avviarlo e abilitarlo**.

Per avviare e abilitare `cron`:

```bash
sudo systemctl start cron
sudo systemctl enable cron
```

Eseguire manualmente i comandi per testare il funzionamento:

```bash
sudo tar -czf /opt/backup/backup_home_$(date +%Y%m%d).tar.gz /home
sudo find /opt/backup -name "backup_home_*.tar.gz" -mtime +6 -delete
ls -l /opt/backup
```

## 📂 Esempio di output

```bash
/opt/backup/
├── backup_home_20250510.tar.gz
├── backup_home_20250511.tar.gz
...
└── backup_home_20250516.tar.gz
```

Dopo 7 giorni, i backup più vecchi vengono eliminati automaticamente.

## 📌 Note

- L’orario del backup può essere modificato cambiando l’espressione nel `crontab`.
- Il backup è eseguito come `root` per garantire l’accesso a tutti i file.
- Verificare che il sistema sia acceso all’orario previsto per l’esecuzione.
