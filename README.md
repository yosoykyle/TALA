# OpenEduCat Community Edition 🎓

[![License: LGPL v3](https://img.shields.io/badge/License-LGPL%20v3-blue.svg)](LICENSE)
[![GitHub stars](https://img.shields.io/github/stars/openeducat/openeducat_erp.svg)](https://github.com/openeducat/openeducat_erp/stargazers)

## TALA Project Notice

This repository is used as the local codebase for **TALA**, an academic information system implementation based on the OpenEduCat Community Edition for Odoo 18.

TALA is a customized/forked implementation built on top of OpenEduCat. Original OpenEduCat source attribution, copyright notices, module metadata, and LGPL-3.0 licensing are preserved. Any TALA-specific setup notes in this README describe how this local development environment is expected to run the system; they do not replace the upstream OpenEduCat documentation.

## Introduction 🚀

OpenEduCat is a powerful, feature-rich **Open Source Educational ERP** designed to streamline academic and administrative processes in educational institutions. Whether you’re managing admissions, academics, finance, or human resources, OpenEduCat provides an integrated platform that empowers your institution with flexibility and innovation. Join our community to transform education management and embrace the future of learning! 🌟

---

## Table of Contents
- [TALA Project Notice](#tala-project-notice)
- [Features 🚀📚](#features-)
- [Demo & Live Links 🌐](#demo--live-links-)
- [Installation](#installation)
- [TALA Local Docker Setup](#tala-local-docker-setup)
- [Documentation 📖](#documentation-)
- [Community & Support 🤝](#community--support-)
- [Roadmap 🗺️](#roadmap-)
- [License 📄](#license-)
- [Contact 📞](#contact-)

---

## Features 🚀📚

OpenEduCat offers a comprehensive suite of features tailored for modern educational institutions:

- **Admissions & Registration** 🎟️: Simplify enrollment and registration processes.
- **Student Information Management** 👨‍🎓👩‍🎓: Manage student profiles, academic history, and personal details.
- **Course & Batch Management** 📚: Organize courses, batches, and scheduling with ease.
- **Examination Management** 📝: Streamline exam scheduling, evaluation, and result processing.
- **Fee & Finance Management** 💰: Automate fee collection, invoicing, and financial reporting.
- **Attendance & Timetable** ⏰: Keep track of attendance and manage class schedules efficiently.
- **Library Management** 📖: Handle book lending, cataloging, and member management.
- **Transport & Hostel Management** 🚍🏠: Oversee transportation logistics and hostel accommodations.
- **Communication Tools** 📢: Enhance collaboration with integrated messaging and notifications.
- **Reporting & Analytics** 📊: Generate insightful reports for data-driven decision-making.
- **HR & Payroll Management** 👥: Manage staff records, payroll, and performance reviews.
- **Customizable & Modular** 🔧: Adapt or extend modules to meet your institution’s unique needs.
- **Secure & Scalable** 🔒: Robust security features ensure your data is protected while scaling with your growth.

For a full list of features, please visit our [Features Page](https://openeducat.org/features) 😊

---

## Demo & Live Links 🌐

Experience OpenEduCat firsthand:
- **Online Demo**: [Try our live demo](https://openeducat.org/demo) 🎥
- **Official Website**: [Visit OpenEduCat.org](https://openeducat.org) 🌟
- **Community Meetings & Webinars**:
  - [Join our next community meeting](https://openeducat.org/meeting) 🤝
  - [Register for upcoming webinars](https://webinars.openeducat.org/events) 🎤

---

## Installation 🛠️

The upstream OpenEduCat installation guide is available at https://doc.openeducat.org/administration/install.html.

For this local TALA setup, use Docker for Odoo, PostgreSQL, and pgAdmin. This keeps Odoo's runtime dependencies separate from the Windows host and allows the OpenEduCat/TALA add-ons in this repository to be mounted into the Odoo container.

---

## TALA Local Docker Setup

### Recommended Local Architecture

Use Docker Compose with three services:

- `odoo`: runs Odoo 18.
- `db`: runs PostgreSQL for Odoo.
- `pgadmin`: provides a browser-based UI for inspecting the PostgreSQL database.

PostgreSQL does not need to be installed locally on Windows. pgAdmin is a separate database UI/client and can connect to PostgreSQL even when PostgreSQL runs inside Docker.

### Folder Layout

This repository should remain as the Odoo add-ons source folder:

```text
D:\D SCHOOL\SYSTEMS\TALA
```

The Docker runtime files should live in a separate folder:

```text
D:\D SCHOOL\SYSTEMS\tala-odoo-docker
```

Expected Docker folder structure:

```text
D:\D SCHOOL\SYSTEMS\tala-odoo-docker
|-- compose.yaml
`-- config
    `-- odoo.conf
```

The Docker folder is intentionally outside this repository because it contains machine-specific runtime configuration, local ports, database volumes, and secrets. The source repository remains focused on Odoo/OpenEduCat modules.

### Compose File

Create `D:\D SCHOOL\SYSTEMS\tala-odoo-docker\compose.yaml`:

```yaml
services:
  db:
    image: postgres:15
    container_name: tala-postgres
    environment:
      POSTGRES_USER: attalasys
      POSTGRES_PASSWORD: "CHANGE_ME"
      POSTGRES_DB: postgres
    volumes:
      - tala-db-data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  odoo:
    image: odoo:18.0
    container_name: tala-odoo
    depends_on:
      - db
    ports:
      - "8069:8069"
    volumes:
      - tala-odoo-data:/var/lib/odoo
      - ./config:/etc/odoo
      - "D:/D SCHOOL/SYSTEMS/TALA:/mnt/extra-addons"
    environment:
      HOST: db
      PORT: 5432
      USER: attalasys
      PASSWORD: "CHANGE_ME"

  pgadmin:
    image: dpage/pgadmin4:latest
    container_name: tala-pgadmin
    depends_on:
      - db
    ports:
      - "5050:80"
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@example.com
      PGADMIN_DEFAULT_PASSWORD: "CHANGE_ME"
    volumes:
      - tala-pgadmin-data:/var/lib/pgadmin

volumes:
  tala-db-data:
  tala-odoo-data:
  tala-pgadmin-data:
```

Replace every `CHANGE_ME` value with a local development password. Keep the password values quoted in `compose.yaml`, especially if the password starts with special characters such as `@`, `#`, `!`, `{`, `}`, `[`, `]`, `*`, `&`, or `:`. Do not commit real production or personal passwords to Git.

### Odoo Configuration

Create `D:\D SCHOOL\SYSTEMS\tala-odoo-docker\config\odoo.conf`:

```ini
[options]
admin_passwd = CHANGE_ME
db_host = db
db_port = 5432
db_user = attalasys
db_password = CHANGE_ME
addons_path = /usr/lib/python3/dist-packages/odoo/addons,/mnt/extra-addons
```

The `/mnt/extra-addons` path points to this repository inside the Odoo container. That is how Odoo discovers the OpenEduCat/TALA modules.

### Start the System

From PowerShell:

```powershell
cd "D:\D SCHOOL\SYSTEMS\tala-odoo-docker"
docker compose up -d
```

Check that the containers are running:

```powershell
docker ps
```

Expected containers:

```text
tala-postgres
tala-odoo
tala-pgadmin
```

### Open Odoo

Open:

```text
http://localhost:8069
```

Use the master password from `admin_passwd` in `odoo.conf` when creating a database.

Suggested local development database:

```text
Database name: tala
Email: admin@example.com
Password: admin
Language: English
Country: Philippines
```

After database creation:

1. Open Odoo.
2. Go to Apps.
3. Remove the default Apps filter if needed.
4. Search for `OpenEduCat`.
5. Install `OpenEduCat ERP` or install the individual OpenEduCat modules needed for development.

### Open pgAdmin

Open:

```text
http://localhost:5050
```

Login with the values from the `pgadmin` service in `compose.yaml`.

Add a server in pgAdmin:

```text
Name: TALA Docker Postgres
Host name/address: db
Port: 5432
Maintenance database: postgres
Username: attalasys
Password: same value as POSTGRES_PASSWORD
```

When pgAdmin runs inside the same Compose stack, use `db` as the host. If using a locally installed pgAdmin outside Docker, use `localhost` as the host because the PostgreSQL container exposes port `5432` to Windows.

### Daily Commands

Start the stack:

```powershell
cd "D:\D SCHOOL\SYSTEMS\tala-odoo-docker"
docker compose up -d
```

Stop the stack while keeping data:

```powershell
docker compose down
```

Restart Odoo after code or XML changes:

```powershell
docker restart tala-odoo
```

View Odoo logs:

```powershell
docker logs -f tala-odoo
```

Reset all local Docker data, including the database:

```powershell
docker compose down -v
```

Only use `docker compose down -v` when intentionally deleting the local database and Odoo filestore.

### Validation and Troubleshooting

Use these checks when starting the local stack or diagnosing errors.

#### Validate the Compose File Before Starting

Run this from the Docker runtime folder:

```powershell
cd "D:\D SCHOOL\SYSTEMS\tala-odoo-docker"
docker compose config
```

Expected result:

- Compose prints the normalized configuration.
- There is no `yaml:` error.
- The `odoo` service shows `/mnt/extra-addons`.
- The bind mount source points to `D:\D SCHOOL\SYSTEMS\TALA`.

If `docker compose config` fails, fix the YAML before running `docker compose up -d`.

#### Check Whether Containers Are Running

Use either command:

```powershell
docker compose ps
```

or:

```powershell
docker ps
```

Expected running services:

```text
tala-postgres
tala-odoo
tala-pgadmin
```

Expected ports:

```text
5432 -> PostgreSQL
8069 -> Odoo
5050 -> pgAdmin
```

If `docker ps` shows no containers after `docker compose up -d`, check whether the image pull failed or timed out:

```powershell
docker compose up -d
docker compose ps
docker ps -a
```

#### First Start Takes a Long Time

The first run may take several minutes because Docker needs to pull:

```text
postgres:15
odoo:18.0
dpage/pgadmin4:latest
```

If startup appears stuck, pull the images explicitly:

```powershell
docker pull postgres:15
docker pull odoo:18.0
docker pull dpage/pgadmin4:latest
docker compose up -d
```

#### YAML Error With Passwords Starting With `@`

Error example:

```text
yaml: while scanning for the next token
found character that cannot start any token
```

Common cause:

```yaml
POSTGRES_PASSWORD: @@examplePassword
PASSWORD: @@examplePassword
```

Fix:

```yaml
POSTGRES_PASSWORD: "@@examplePassword"
PASSWORD: "@@examplePassword"
```

YAML treats some unquoted special characters as syntax. Quote password values in `compose.yaml` to avoid this.

The `odoo.conf` file is not YAML, so this is valid there:

```ini
admin_passwd = @@examplePassword
db_password = @@examplePassword
```

#### pgAdmin Exits Immediately

Check pgAdmin logs:

```powershell
docker logs --tail 80 tala-pgadmin
```

If the log says the email is invalid, avoid reserved local domains such as:

```text
admin@tala.local
```

Use a normal-looking email instead:

```yaml
PGADMIN_DEFAULT_EMAIL: admin@example.com
```

Then recreate pgAdmin:

```powershell
docker compose up -d --force-recreate pgadmin
```

#### Odoo Is Running But Browser Cannot Open It

Check status:

```powershell
docker compose ps
```

Check Odoo logs:

```powershell
docker logs --tail 100 tala-odoo
```

Open:

```text
http://localhost:8069
```

If port `8069` is already used by another app, change the Odoo port mapping in `compose.yaml`:

```yaml
ports:
  - "8070:8069"
```

Then restart:

```powershell
docker compose up -d
```

Open:

```text
http://localhost:8070
```

#### pgAdmin Cannot Connect to PostgreSQL

If pgAdmin runs inside this Compose stack, use:

```text
Host name/address: db
Port: 5432
Maintenance database: postgres
Username: attalasys
Password: same value as POSTGRES_PASSWORD
```

If pgAdmin is installed locally on Windows instead of running in Docker, use:

```text
Host name/address: localhost
Port: 5432
Maintenance database: postgres
Username: attalasys
Password: same value as POSTGRES_PASSWORD
```

Reason:

- `db` works between containers on the same Docker Compose network.
- `localhost` works from Windows because PostgreSQL is exposed through `5432:5432`.

#### Odoo Cannot Connect to PostgreSQL

Check Odoo logs:

```powershell
docker logs --tail 100 tala-odoo
```

Verify that these values match between `compose.yaml` and `config\odoo.conf`:

```text
POSTGRES_USER     -> db_user
POSTGRES_PASSWORD -> db_password
HOST/db_host      -> db
PORT/db_port      -> 5432
```

The Odoo service should use:

```yaml
environment:
  HOST: db
  PORT: 5432
  USER: attalasys
  PASSWORD: "same value as POSTGRES_PASSWORD"
```

The Odoo config should use:

```ini
db_host = db
db_port = 5432
db_user = attalasys
db_password = same value as POSTGRES_PASSWORD
```

#### OpenEduCat Modules Do Not Appear in Odoo Apps

Confirm that Odoo sees this repository as an add-ons path:

```powershell
docker logs --tail 80 tala-odoo
```

Look for:

```text
addons paths: ... '/mnt/extra-addons'
```

Then in Odoo:

1. Enable developer mode if needed.
2. Go to Apps.
3. Update the app list.
4. Remove the default Apps filter if needed.
5. Search for `OpenEduCat`.

If `/mnt/extra-addons` is missing, check this mount in `compose.yaml`:

```yaml
volumes:
  - "D:/D SCHOOL/SYSTEMS/TALA:/mnt/extra-addons"
```

#### Code Changes Do Not Show Up

For Python, XML, manifest, or security rule changes, restart Odoo:

```powershell
docker restart tala-odoo
```

For installed module changes, update the module from Odoo Apps or use the Odoo upgrade flow. Restarting reloads the server process, but installed module data may still need a module update.

#### Factory Reset the Local System

Use this when the local Odoo system should be reset as if it was newly installed.

Stop containers but keep all data:

```powershell
docker compose down
```

This only stops the stack. It does not delete the database, Odoo filestore, installed modules, uploaded files, website edits, students, faculty, courses, or pgAdmin saved server configuration.

Full factory reset:

```powershell
cd "D:\D SCHOOL\SYSTEMS\tala-odoo-docker"
docker compose down -v
docker compose up -d
```

The `-v` flag deletes the Docker volumes for this stack. This removes:

```text
PostgreSQL database data
Odoo filestore data
pgAdmin saved server configuration
```

In this setup, the volume names are:

```text
tala-odoo-docker_tala-db-data
tala-odoo-docker_tala-odoo-data
tala-odoo-docker_tala-pgadmin-data
```

After the reset, open Odoo again:

```text
http://localhost:8069
```

Odoo should show the database creation page again.

To delete only the Odoo database but keep the Docker volumes and containers:

```powershell
docker exec tala-postgres dropdb -U attalasys tala
```

Use this only when you want to remove the `tala` database but keep the Odoo filestore volume and pgAdmin configuration.

The reset commands do not delete the source code repository:

```text
D:\D SCHOOL\SYSTEMS\TALA
```

They also do not delete the Docker runtime files:

```text
D:\D SCHOOL\SYSTEMS\tala-odoo-docker
```

Only use `docker compose down -v` when intentionally deleting local system data.

---

## Documentation 📖

Learn more about OpenEduCat:
- **Documentation Portal**: [OpenEduCat Documentation](https://doc.openeducat.org/)
- **User Guides**: Comprehensive guides to help you master the system quickly.

---

## Community & Support 🤝

Join our active and vibrant community:
- **Discussion Forum**: [OpenEduCat Forum](https://openeducat.org/forum)
- **Issue Tracker**: Report bugs and request features on [GitHub Issues](https://github.com/openeducat/openeducat_erp/issues)
- **Community Chat**: Connect with peers on our [Community Chat](https://community.openeducat.org)
- **Social Media**:
  - LinkedIN: [@OpenEduCat Company Page](https://www.linkedin.com/company/openeducat-inc/)
  - Instagram: [@OpenEduCat Profile](https://www.instagram.com/openeducat)
  - Twitter: [@OpenEduCat](https://twitter.com/openeducat)
  - Facebook: [OpenEduCat Facebook Page](https://facebook.com/openeducat)

---

## Roadmap 🗺️

We’re continuously evolving! Here’s a glimpse of what’s coming:
- **Enhanced Mobile Experience** 📱: Optimizing for a seamless mobile interface.
- **New Modules** 🆕: Introducing additional modules based on community feedback.
- **Performance Optimization** ⚡: Continuous improvements for faster and smoother operations.
- **Extended Integrations** 🔗: More integrations with popular third-party services.
- **User Experience Enhancements** 🎨: Regular UI/UX updates to make navigation even easier.

Stay tuned for future updates and contribute to shaping our roadmap!

---

## License 📄

OpenEduCat is distributed under the **LGPL-3.0 License**. See the [LICENSE](LICENSE) file for more details.

---

## Contact 📞

Have questions or need support? Get in touch:
- **Email**: [support@openeducat.org](mailto:support@openeducat.org)
- **Forum**: [OpenEduCat Forum](https://openeducat.org/forum)
- **Twitter**: [@OpenEduCat](https://twitter.com/openeducat)

---

Thank you for choosing **OpenEduCat** – empowering educational institutions with open source technology. We appreciate your support and look forward to your contributions! 🙌

*Happy Learning & Coding! 💻🎉*
