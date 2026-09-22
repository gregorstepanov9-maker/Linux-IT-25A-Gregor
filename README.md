# Linux-IT-25A-Gregor

disk-report.service

[Unit]
Description=Append disk usage to log
Documentation=man:df(1)
After=local-fs.target

[Service]
Type=oneshot
User=reports
ExecStart=/usr/local/bin/disk-report.sh
StandardOutput=append:/var/log/disk-report.log
StandardError=append:/var/log/disk-report.log

disk-report.timer 
[Unit]
Description=Run disk-report every five minutes
Documentation=systemd.time(7)

[Timer]
OnCalendar=daily
Persistent=true

[Install]
WantedBy=timers.target

2.
vboxuser@vboxuser:~$ journalctl -u disk-report.service -f
Sep 22 05:45:17 vboxuser systemd[1]: Failed to start disk-report.service - Append disk usage to log.
Sep 22 06:13:37 vboxuser systemd[1]: Starting disk-report.service - Append disk usage to log...
Sep 22 06:13:37 vboxuser systemd[1]: disk-report.service: Deactivated successfully.
Sep 22 06:13:37 vboxuser systemd[1]: Finished disk-report.service - Append disk usage to log.
Sep 22 06:13:46 vboxuser systemd[1]: Starting disk-report.service - Append disk usage to log...
Sep 22 06:13:46 vboxuser systemd[1]: disk-report.service: Deactivated successfully.
Sep 22 06:13:46 vboxuser systemd[1]: Finished disk-report.service - Append disk usage to log.
Sep 22 06:15:25 vboxuser systemd[1]: Starting disk-report.service - Append disk usage to log...
Sep 22 06:15:25 vboxuser systemd[1]: disk-report.service: Deactivated successfully.
Sep 22 06:15:25 vboxuser systemd[1]: Finished disk-report.service - Append disk usage to log.

3.
Option B is better because it follows the **principle of least privilege**: the `reports` user does not need direct write permission to `/var/log`, because `systemd` handles opening and appending to the log file as part of the service configuration. With Option A, the `reports` user must own the log file, so if the file is deleted or recreated, the permissions can break again. Option B also keeps the logging configuration in the service unit rather than putting file-management logic in the script, making the setup cleaner, safer, and easier to maintain.

4.
vboxuser@vboxuser:~$ systemctl list-timers disk-report.timer
NEXT                        LEFT LAST                           PASSED UNIT              ACTIVATES   >
Wed 2026-09-23 00:00:00 UTC  17h Tue 2026-09-22 05:45:17 UTC 39min ago disk-report.timer disk-report.>

5.
vboxuser@vboxuser:~$ journalctl -u disk-report.service --no-pager | grep Finished
Sep 22 06:13:37 vboxuser systemd[1]: Finished disk-report.service - Append disk usage to log.
Sep 22 06:13:46 vboxuser systemd[1]: Finished disk-report.service - Append disk usage to log.
Sep 22 06:15:25 vboxuser systemd[1]: Finished disk-report.service - Append disk usage to log.

6.
The `reports` user was created with `--no-create-home --shell /usr/sbin/nologin` to make it a restricted service account with no home directory or interactive login access. If those flags were omitted, an attacker who compromised the account could potentially log in interactively and use its home directory to store files or maintain persistence.
