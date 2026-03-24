# .NET — Red Inn Court Dynamic Pricing

**Stack:** VB.NET migrated to .NET 8 (C#), Google Sheets API, Docker, AWS EC2 (Amazon Linux 2023)
**Repo:** (private)

---

## Deployment

- Hosted on AWS EC2, containerised with Docker
- Scheduled via `systemd` timer (not cron)
- Timezone: Asia/Kuala_Lumpur (UTC+8) — all pricing logic uses this explicitly

---

## Known Issues & Fixes

- **JWT auth errors:** Ensure Google service account JSON is mounted correctly into the Docker container; path mismatches silently fail
- **Timezone mismatches:** Always use `TimeZoneInfo.FindSystemTimeZoneById("Asia/Singapore")` (same UTC+8, more reliably available on Linux than "Asia/Kuala_Lumpur")
- **SMTP errors:** Verify port 587 is open in the EC2 security group for outbound mail
- **Deployment caching:** After pushing a new Docker image, always run `docker pull` explicitly before `docker run` to avoid running stale cached image

---

_Add new lessons here as discovered._
