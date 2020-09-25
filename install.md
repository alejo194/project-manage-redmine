docker-compose.yml
```bash
version: '2'
services:
  redmine:
    image: redmine:passenger
    container_name: redmine
    hostname: redmine
    restart: always
    ports:
      - 30000:3000
    environment:
      REDMINE_DB_MYSQL: db
      #REDMINE_DB_USERNAME: redmine
      REDMINE_DB_PASSWORD: m*m*
      #RAILS_RELATIVE_URL_ROOT: "./"
    network_mode: maxwin_default
    volumes:
      #- ./environment.rb:/usr/src/redmine/config/environment.rb
      #- ./config.ru:/usr/src/redmine/config.ru
      - ./passenger-nginx-config-template.erb:/passenger-nginx-config-template.erb
      - ./plugins:/usr/src/redmine/plugins
      - redmine_data:/usr/src/redmine/files
    links:
      - db:db
    command: ["passenger", "start", "--nginx-config-template", "/passenger-nginx-config-template.erb"]
    logging:
      driver: "json-file"
      options:
        max-size: "20m"
        max-file: "10"
  db:
    image: mysql:5.7
    container_name: db
    hostname: db
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: m*m*
      MYSQL_DATABASE: redmine
      #MYSQL_RANDOM_ROOT_PASSWORD: 111
      LANG: C.UTF-8
      MYSQL_CHARSET: utf-8
    network_mode: maxwin_default
    volumes:
      #- ./mysql:/etc/mysql/conf.d
      - mysql_data:/var/lib/mysql
    command: mysqld --character-set-server=utf8 --collation-server=utf8_unicode_ci
    logging:
      driver: "json-file"
      options:
        max-size: "20m"
        max-file: "10"
volumes:
  mysql_data:
    driver: local
  redmine_data:
    driver: local
```
passenger-nginx-config-template.erb
```bash
<%= include_passenger_internal_template('global.erb') %>

worker_processes 1;
events {
    worker_connections 4096;
}

http {
    <%= include_passenger_internal_template('http.erb', 4) %>

    default_type application/octet-stream;
    types_hash_max_size 2048;
    server_names_hash_bucket_size 64;
    client_max_body_size 1024m;
    access_log off;
    keepalive_timeout 60;
    underscores_in_headers on;
    gzip on;
    gzip_comp_level 3;
    gzip_min_length 150;
    gzip_proxied any;
    gzip_types text/plain text/css text/json text/javascript
        application/javascript application/x-javascript application/json
        application/rss+xml application/vnd.ms-fontobject application/x-font-ttf
        application/xml font/opentype image/svg+xml text/xml;

    server {
        server_name _;
        listen 0.0.0.0:3000;
        root '/usr/src/redmine/public';
        passenger_app_env 'production';
        passenger_spawn_method 'smart';
        passenger_load_shell_envvars off;

        location ~ ^/redmine(/.*|$) {
            alias /usr/src/redmine/public$1;
            passenger_base_uri /redmine;
            passenger_app_root /usr/src/redmine;
            passenger_document_root /usr/src/redmine/public;
            passenger_enabled on;
        }
    }

    passenger_pre_start http://0.0.0.0:3000/;
}
```


#### 新版mysql8 redmine:4-passenger
```bash
vi docker-compose.yml
redmine:
    image: redmine:4-passenger
    container_name: redmine
    hostname: redmine
    privileged: true
    restart: always
    ports:
      - 30000:3000
    environment:
      REDMINE_DB_MYSQL: db
      #REDMINE_DB_USERNAME: redmine
      REDMINE_DB_PASSWORD: maxwin.maxwin
      #RAILS_RELATIVE_URL_ROOT: "./"
    network_mode: maxwin_default
    volumes:
      #- ./passenger-nginx-config-template.erb:/passenger-nginx-config-template.erb
      #- ./redmine/plugins:/usr/src/redmine/plugins
      - ./redmine/data:/usr/src/redmine/files
    links:
      - db:db
    #command: ["passenger", "start", "--nginx-config-template", "/passenger-nginx-config-template.erb"]
    logging:
      driver: "json-file"
      options:
        max-size: "20m"
        max-file: "10"
  db:
    image: mysql:8.0
    container_name: db
    hostname: db
    restart: always
    ports:
      - 3306:3306
    environment:
      MYSQL_ROOT_PASSWORD: maxwin.maxwin
      MYSQL_DATABASE: redmine
      LANG: C.UTF-8
      MYSQL_CHARSET: utf-8
    network_mode: maxwin_default
    volumes:
      - ./mysql/mysqld.cnf:/etc/mysql/conf.d/mysqld.cnf
      - ./mysql/data:/var/lib/mysql
    command: mysqld --character-set-server=utf8 --collation-server=utf8_unicode_ci
    logging:
      driver: "json-file"
      options:
        max-size: "20m"
        max-file: "10"
```
