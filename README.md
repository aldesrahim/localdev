# localdev

A Docker-based local development environment for PHP/Laravel projects with multiple database options. Inspired by [Laradock](https://github.com/laradock/) but simplified for personal use.

## Pre-requisite
- Docker CE (engine)
- Docker Compose (already installed within latest docker)

## Available services
- nginx
- php-fpm
- postgres
- mysql
- mariadb
- pgadmin
- adminer
- redis-webui
- mailpit
- workspace

## Usage

1. Clone this repo
```bash
git clone https://github.com/aldesrahim/localdev
cd localdev
```

2. Configure your environment
```bash
cp .env.example .env
```

3. Start the services
```bash
docker compose up -d workspace nginx redis postgres adminer redis-webui mailpit
```

And that's it, You're all set!

## Tips
1. Attach into service shell. e.g. workspace
```bash
docker compose exec -it workspace bash
```

2. Run `php artisan` command
```bash
docker compose exec workspace php $project_dir/artisan
# replace $project_dir with your project directory name
```

3. Apply `.env` changes. e.g. workspace
```bash
docker compose down workspace
docker compose up -d workspace --build --force-recreate
```

4. Apply configuration changes. e.g. add nginx sites
```bash
docker compose restart nginx
```

5. Run `npm run dev` command
```bash
# attach into workspace shell
docker compose exec -it workspace bash

# from container
cd $project_dir
# replace $project_dir with your project directory name

npm run dev -- --host

# Open `http://localhost:5173` in browser to test it out.
```

## Troubleshoot

1. Laravel storage and cache directories permission denied
```bash
# from host machine
chmod 777 -R $project_dir/bootstrap $project_dir/storage
# replace $project_dir with your project path
```
