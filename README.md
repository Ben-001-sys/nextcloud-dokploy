# Nextcloud Hub on Dokploy

A small Docker Compose deployment of Nextcloud for Dokploy, using PostgreSQL, Redis, Apache and a cron worker.

## Files

- `docker-compose.yml` — Nextcloud (`app`), PostgreSQL (`db`), Redis (`redis`) and background jobs (`cron`)
- `.env.example` — non-secret example variables
- `.gitignore` — protects `.env` and local backups from Git commits

## Deploy in Dokploy

1. Create a **private** GitHub repository and commit these files to its `main` branch.
2. In Dokploy, create a project and add a **Compose** service (Docker Compose, not Stack).
3. Select **GitHub** as source, choose the repository and `main` branch, and set **Compose Path** to `./docker-compose.yml`.
4. In **Environment**, define the following with your own values. Do not copy real secrets into GitHub:

   ```dotenv
   NEXTCLOUD_DOMAIN=cloud.example.com
   NEXTCLOUD_ADMIN_USER=admin
   NEXTCLOUD_ADMIN_PASSWORD=<unique-strong-password>
   POSTGRES_PASSWORD=<different-strong-password>
   ```

5. In **Utilities**, enable **Isolated Deployments**. This is important if deploying multiple Nextcloud stacks on the same server.
6. Configure a DNS **A** record for your Nextcloud domain pointing to the Dokploy server IP.
7. In **Domains**, add your domain using service **`app`** and container port **`80`**; enable HTTPS/Let's Encrypt.
8. Deploy (or redeploy after changing domains). Check logs and ensure `db`, `redis`, `app` and `cron` are running.
9. Open `https://<your-domain>` and log in with `NEXTCLOUD_ADMIN_USER` and `NEXTCLOUD_ADMIN_PASSWORD`. In Nextcloud's administration settings, select **Cron** for background jobs and review Overview security warnings.

### Two installations on the same server

Both people can deploy from the **same Git repository**, but must use **separate Dokploy Compose services**, **different domains**, **different passwords**, and **Isolated Deployments**. Named volumes are then scoped to each Compose project. Do **not** set a global `container_name` or `name` field in the Compose file, and do not publish shared host ports.

### Notes

- This repo supplies Nextcloud's core application and supporting infrastructure. Advanced Nextcloud Office editing and high-performance Talk calling require additional service/app setup.
- The `app` service only uses `expose: "80"`, so access is through Dokploy's reverse proxy. The Compose file does not configure HTTPS by itself.
- The named `postgres_data` and `nextcloud_data` Docker volumes hold persistent information. Back up both the database and user files; deleting volumes destroys persisted data.
- All Nextcloud version updates should be planned and backed up before changing the image tag. The same Nextcloud image tag must be used by `app` and `cron`.
- For configuration changes, update the repository and redeploy from Dokploy. Environment-specific passwords and the domain belong in Dokploy, not Git.

## Reference documentation

- [Dokploy Docker Compose](https://docs.dokploy.com/docs/core/docker-compose)
- [Dokploy Compose from GitHub](https://docs.dokploy.com/docs/core/docker-compose/example)
- [Dokploy Isolated Deployments](https://docs.dokploy.com/docs/core/docker-compose/utilities)
- [Nextcloud Docker](https://github.com/nextcloud/docker)
