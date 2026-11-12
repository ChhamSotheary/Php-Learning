# មេរៀនទី ៥០៖ Docker Compose សម្រាប់ PHP និង MySQL

## គោលបំណងមេរៀន

ក្រោយរៀនចប់ អ្នកនឹងអាច៖
- ពន្យល់ពី Container, Image, Service និង Volume។
- រៀបចំ PHP Apache Container ដែលមាន PDO MySQL Extension។
- ដំណើរការ PHP និង MySQL ជា Services ដោយ Docker Compose។
- រក្សាទិន្នន័យ Database នៅ Persistent Volume។
- ប្រើ Docker សម្រាប់ Local Development និងយល់ពីអ្វីដែលត្រូវកែសម្រាប់ Production។

> ឧទាហរណ៍នេះសម្រាប់ Local Development។ កុំប្រើ Password ឬ Configuration Local ដោយផ្ទាល់នៅ Production។

## ១. Docker Concept ខ្លីៗ

- **Image** ជាគំរូសម្រាប់បង្កើត Container។
- **Container** ជា Instance ដែលកំពុងដំណើរការពី Image។
- **Service** ក្នុង Compose កំណត់របៀបដំណើរការ Container មួយប្រភេទ។
- **Volume** រក្សាទិន្នន័យឱ្យនៅសល់ក្រោយ Container ត្រូវបានបង្កើតឡើងវិញ។
- **Network** អនុញ្ញាតឱ្យ Service ទាក់ទងគ្នាតាមឈ្មោះ Service។

## ២. រចនាសម្ព័ន្ធ Project

```text
project/
  app/
  public/
    index.php
  Dockerfile
  compose.yaml
  .env
  .env.example
  .dockerignore
```

Web Document Root គួរតែទៅ `public/` ដើម្បីកុំឱ្យបម្រើ Source/Configuration ដោយផ្ទាល់។

## ៣. Dockerfile សម្រាប់ PHP និង Apache

បង្កើត `Dockerfile`៖

```dockerfile
FROM php:8.3-apache
RUN docker-php-ext-install pdo_mysql
ENV APACHE_DOCUMENT_ROOT=/var/www/html/public
RUN sed -ri -e 's!/var/www/html!$${APACHE_DOCUMENT_ROOT}!g' \
    /etc/apache2/sites-available/*.conf \
    /etc/apache2/apache2.conf \
    /etc/apache2/conf-available/*.conf \
    && a2enmod rewrite
WORKDIR /var/www/html
COPY . /var/www/html
```

កែ PHP Image Version ឱ្យត្រូវនឹង Project។ សម្រាប់ Production ត្រូវ Build Artifact មានតែឯកសារចាំបាច់ និងកំណត់ User/Permission សមស្រប។

## ៤. Compose File សម្រាប់ PHP និង MySQL

បង្កើត `compose.yaml`៖

```yaml
services:
  app:
    build: .
    ports:
      - "8080:80"
    environment:
      DB_HOST: db
      DB_PORT: 3306
      DB_NAME: ${DB_NAME}
      DB_USER: ${DB_USER}
      DB_PASSWORD: ${DB_PASSWORD}
    volumes:
      - .:/var/www/html
    depends_on:
      db:
        condition: service_healthy

  db:
    image: mysql:8.4
    environment:
      MYSQL_DATABASE: ${DB_NAME}
      MYSQL_USER: ${DB_USER}
      MYSQL_PASSWORD: ${DB_PASSWORD}
      MYSQL_ROOT_PASSWORD: ${DB_ROOT_PASSWORD}
    volumes:
      - mysql_data:/var/lib/mysql
    healthcheck:
      test: ["CMD-SHELL", "mysqladmin ping -h localhost -u\"$$${MYSQL_USER}\" -p\"$$${MYSQL_PASSWORD}\""]
      interval: 5s
      timeout: 5s
      retries: 10

volumes:
  mysql_data:
```

Compose រង់ចាំ Health Check របស់ Database មុន Start App ក្នុង Compose ដែលគាំទ្រ `condition: service_healthy`។ App ក៏គួរមាន Retry/ការដោះស្រាយ Database មិនទាន់ Ready។

## ៥. Environment File និង Ignore

ឧទាហរណ៍ `.env.example` ដោយគ្មាន Secret ពិត៖

```dotenv
DB_NAME=learning_php
DB_USER=learning_user
DB_PASSWORD=change_for_local_only
DB_ROOT_PASSWORD=change_for_local_only
```

ចម្លងទៅ `.env` ហើយដាក់ `.env` ក្នុង `.gitignore`៖

```gitignore
.env
/vendor/
```

បង្កើត `.dockerignore` ដើម្បីកុំ Copy File មិនចាំបាច់ ឬ Secret ចូល Image៖

```dockerignore
.git
.env
vendor
var
tests
```

កែ Ignore តាមតម្រូវការ Project។ Compose `.env` គឺសម្រាប់ Interpolation ក្នុង Local; វាមិនមែនជា Secret Manager សម្រាប់ Production ទេ។

## ៦. ដំណើរការ និងពិនិត្យ

```bash
docker compose config
docker compose up --build
```

បើក `http://localhost:8080`។ ពាក្យបញ្ជាដែលមានប្រយោជន៍៖

```bash
docker compose ps
docker compose logs -f app
docker compose exec app php -v
docker compose down
```

`docker compose down` បញ្ឈប់ និងលុប Container/Network ប៉ុន្តែ Named Volume នៅសល់។ `docker compose down -v` លុប Volume ផងដែរ ហើយអាចលុបទិន្នន័យ Database ទាំងអស់; ប្រើតែពេលចង់សម្អាត Data ពិតប្រាកដ។

## ៧. ភ្ជាប់ PHP ទៅ Database

ក្នុង PHP Container ប្រើ Hostname `db` មិនមែន `localhost`៖

```php
<?php
$dsn = sprintf(
    'mysql:host=%s;port=%s;dbname=%s;charset=utf8mb4',
    getenv('DB_HOST'),
    getenv('DB_PORT') ?: '3306',
    getenv('DB_NAME')
);

$pdo = new PDO(
    $dsn,
    getenv('DB_USER'),
    getenv('DB_PASSWORD'),
    [PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION]
);
```

ក្នុង Container `localhost` សំដៅទៅ PHP Container ខ្លួនឯង មិនមែន MySQL Service ទេ។ ប្រើ Prepared Statements សម្រាប់ Query ទាំងអស់។

## ៨. កំហុសដែលជួបញឹកញាប់

- ប្រើ `localhost` ជា Database Host ពី App Container។
- ភ្លេចដំឡើង `pdo_mysql` Extension ក្នុង PHP Image។
- គិតថា App ចាប់ផ្ដើមក្រោយ Database Container Start មានន័យថា DB Ready។
- Commit `.env` ឬ Hardcode Production Secret ក្នុង Dockerfile/Compose។
- ភ្លេច Persistent Volume ហើយបាត់ Database ពេលបង្កើត Container ថ្មី។
- ប្រើ Local Development Configuration ដូច Production ដោយមិនពិនិត្យ Permission/Security។
- លុប Volume ដោយមិនដឹងថាវាមានទិន្នន័យអ្វី។

## ៩. លំហាត់

1. បង្កើត Dockerfile ដែលបើក PDO MySQL Extension។
2. បង្កើត Compose Services `app` និង `db` ជាមួយ Named Volume។
3. ភ្ជាប់ PDO ទៅ Hostname `db` ហើយបង្កើត Table សាកល្បង។
4. ដាក់ `.env` ក្នុង `.gitignore` និង Ignore Secret/Dependency តាមតម្រូវការ។
5. ពិនិត្យ Logs និង Health Status ពេល Database មិនទាន់ Ready។
6. ពន្យល់ភាពខុសគ្នារវាង `docker compose down` និង `docker compose down -v`។

## ១០. ចម្លើយណែនាំខ្លី

- ប្រើ PDO MySQL Extension និង Hostname តាម Compose Service។
- ប្រើ Named Volume ដើម្បីរក្សា Database Data។
- `.env` Local មិនត្រូវ Commit និងមិនមែនជា Production Secret Manager។
- Health Check ជួយឱ្យ App រង់ចាំ DB ត្រៀម; App នៅតែគួរដោះស្រាយ Retry។
- `down -v` លុប Volume; ត្រូវយល់ពីផលប៉ះពាល់មុនប្រើ។

## ឯកសារយោង

- [Docker Compose Quickstart](https://docs.docker.com/compose/gettingstarted/)
- [Official PHP Docker Image Documentation](https://github.com/docker-library/docs/blob/master/php/README.md)
- [មេរៀនទី ៣៣៖ Configuration និង Secrets ក្នុង PHP](lesson-33-php-configuration-secrets.md)
- [មេរៀនទី ៤៦៖ Deploy PHP Application ទៅ Production](lesson-46-php-production-deployment.md)
