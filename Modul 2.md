# Modul 2

##### Grahito Ardani B (1202199006) 

##### M Iqbal Maulana (120190023)

------

Mengganti lxc_landing dan lxc_php7 dari ubuntu 18 ke ubuntu 20 yaitu ubuuntu focal

![2](https://user-images.githubusercontent.com/93030868/144252358-a3791a35-d040-4570-a34d-f9469fa9d050.PNG)

```markdown
sudo lxc-destroy ubuntu_landing

sudo lxc-create -n ubuntu_landing -t download -- --dist ubuntu --release focal --arch amd64 --force-cache --no-validate --server images.linuxcontainers.org
```

mengatur IP ubuntu_landing 10.0.3.103

![4](https://user-images.githubusercontent.com/93030868/144252396-1d404913-171b-49ba-8a41-e8467cdb9f58.PNG)

```markdown
sudo lxc-start -n ubuntu_landing
sudo lxc-attach  -n ubuntu_landing
nano /etc/netplan/10-lxc.yaml
netplan apply
```

install dan setting open ssh

![7](https://user-images.githubusercontent.com/93030868/144252471-18aa58c5-95e0-4903-ae8c-0c2687c9f751.PNG)

```markdown
apt install openssh-server -y
```

![8](https://user-images.githubusercontent.com/93030868/144252492-11655c2c-a61b-4632-b12a-9927c840c121.PNG)

```markdown
PermitRootLogin yes
RSAAuthentication yes
service sshd restart
```

cek ssh apakah sudah berjalan

![10](https://user-images.githubusercontent.com/93030868/144252502-2dcd9660-40a7-45dc-a93c-0d76315f427a.PNG)

```markdown
ssh root@10.0.3.103
```

ubuntu7

![2](https://user-images.githubusercontent.com/93030868/144252730-76e8c159-33a1-455f-b48b-d79b437f3e14.PNG)

```markdown
sudo lxc-destroy ubuntu_landing

sudo lxc-create -n ubuntu_php7.4 -t download -- --dist ubuntu --release focal --arch amd64 --force-cache --no-validate --server images.linuxcontainers.org
```

lakukan seperti lxc_landing

#### vm.local/

install laravel menggunakan ansible

kita harus masuk ke ansible dan membuat folder laravel

![2](https://user-images.githubusercontent.com/93030868/144252957-4018bb46-c078-4c65-8692-5e077b0fee17.PNG)

```markdown
cd ~/ansible/
mkdir laravel
cd laravel/
nano host
```

membuat host untuk lxc yang akan di otomasi

![3](https://user-images.githubusercontent.com/93030868/144252962-6075e5f4-5838-4558-9ad6-8b8c55a78446.PNG)

```markdown
ubuntu_landing ansible_host=lxc_landing.dev ansible_ssh_user=root ansible_become_pass=123zse456
```

setelah itu mambuat drektori dan apasaja yang akan di jalan kan pada folder php dan masuk ke nginxphp.yml dan lakukan instalasi

![4](https://user-images.githubusercontent.com/93030868/144252935-e5649fc7-7125-4820-958a-4f408f6e5bd4.PNG)

```markdown
---
- hosts: all
  become : yes
  tasks:
    - name: install nginx nginx extras
      apt:
       pkg:
         - nginx
         - nginx-extras
       state: latest
    - name: start nginx
      service:
       name: nginx
       state: started
    - name: menginstall tools
      apt:
       pkg:
         - curl
         - software-properties-common
         - unzip
       state: latest
    - name: "Repo PHP 7.4"
      apt_repository:
        repo="ppa:ondrej/php"
    - name: "Updating the repo"
      apt: update_cache=yes
    - name: Installation PHP 7.4
      apt: name=php7.4 state=present
    - name: install php untuk laravel
      apt:
       pkg:
          - php7.4-fpm
          - php7.4-mysql
          - php7.4-mbstring
          - php7.4-xml
          - php7.4-bcmath
          - php7.4-json
          - php7.4-zip
          - php7.4-common
       state: present

```

lakukan instalasi

![5](https://user-images.githubusercontent.com/93030868/144252942-72256779-b6d0-4b24-a8e7-16402bf69a03.PNG)

```markdown
ansible-playbook -i hosts nginxphp.yml -k
```

buat folder installcomposer.yml

![6](https://user-images.githubusercontent.com/93030868/144252949-42018196-4f19-4f2c-814d-262bed54db38.PNG)

```markdown
nano installcomposer.yml

---
- hosts: all
  become: yes
  tasks:
   - name: Download and install Composer
     shell: curl -sS https://getcomposer.org/installer | php
     args:
      chdir: /usr/src/
      creates: /usr/local/bin/composer
      warn: false
   - name: Add Composer to global path
     copy:
      dest: /usr/local/bin/composer
      group: root
      mode: '0755'
      owner: root
      src: /usr/src/composer.phar
      remote_src: yes
   - name: Composer create project
     become_user: root
     composer:
      command: create-project
      arguments: laravel/laravel landing 
      working_dir: /var/www/html
      prefer_dist: yes
     environment:
        COMPOSER_NO_INTERACTION: "1"
   - name: mengkopi file .env.example jadi .env
     copy:
      dest: /var/www/html/landing/.env.example
      src: /var/www/html/landing/.env
      remote_src: yes
   - name: mengganti konfigurasi .env
     lineinfile:
      path: /var/www/html/landing/.env
      regexp: "{{ item.regexp }}"
      line: "{{ item.line }}"
      backrefs: yes
     loop:
      - { regexp: '^(.*)DB_HOST(.*)$', line: 'DB_HOST=10.0.3.200' }
      - { regexp: '^(.*)DB_DATABASE(.*)$', line: 'DB_DATABASE=landing' }
      - { regexp: '^(.*)DB_USERNAME(.*)$', line: 'DB_USERNAME=admin' }
      - { regexp: '^(.*)DB_PASSWORD(.*)$', line: 'DB_PASSWORD=123zse456' }
      - { regexp: '^(.*)APP_URL(.*)$', line: 'APP_URL=http://vm.local' }
      - { regexp: '^(.*)APP_NAME=(.*)$', line: 'APP_NAME=landing' }
   - name: Composer install ke landing
     composer:
       command: install
       working_dir: /var/www/html/landing
     environment:
       COMPOSER_NO_INTERACTION: "1"
   - name: generate php artisan
     args:
      chdir: /var/www/html/landing
     shell: php artisan key:generate
   - name: mengganti permission storage
     file:
      path: /var/www/html/landing/storage
      mode: 0777
      recurse: yes


```

lakukan instalasi 

![7](https://user-images.githubusercontent.com/93030868/144252952-9dff3846-c8e8-4af7-9ef8-d05882a84f54.PNG)

```markdown
ansible-playbook -i hosts installcomposer.yml -k
```

membuat folder config.yml dan settings

![8](https://user-images.githubusercontent.com/93030868/144253176-a994d300-740b-4d52-8c4f-70ac4e4d4e4d.PNG)

```markdown
nano config.yml

---
- hosts: all
  become : yes
  vars:
    domain: 'lxc_landing.dev'
  tasks:
   - name: stop apache2
     service:
      name: apache2
      state: stopped
      enabled: no
   - name: Write {{ domain }} to /etc/hosts
     lineinfile:
      dest: /etc/hosts
      regexp: '.*{{ domain }}$'
      line: "127.0.0.1 {{ domain }}"
      state: present
   - name: ensure nginx is at the latest version
     apt: name=nginx state=latest
   - name: start nginx
     service:
      name: nginx
      state: started
   - name: copy the nginx config file 
     copy:
      src: ~/ansible/laravel/lxc_landing.dev
      dest: /etc/nginx/sites-available/lxc_landing.dev
   - name: Symlink lxc_landing.dev
     command: ln -sfn /etc/nginx/sites-available/lxc_landing.dev /etc/nginx/sites-enabled/lxc_landing.dev
     args:
      warn: false
   - name: restart nginx
     service:
      name: nginx
      state: restarted
   - name: restart php7
     service:
      name: php7.4-fpm
      state: restarted
   - name: curl web
     command: curl -i http://lxc_landing.dev
     args:
      warn: false

```

instalasi

![9](https://user-images.githubusercontent.com/93030868/144253243-5ecdcbea-4154-4222-8d8a-c86c44d770d7.PNG)

```markdown
ansible-playbook -i hosts config.yml -k
```

mengatur lxc_landing.dev

![8 1](https://user-images.githubusercontent.com/93030868/144253189-549c1caf-6152-4a29-bd39-64c8d7e5850c.PNG)

```markdown
server {
        listen 80;

        root /var/www/html/landing/public;
        index index.php index.html index.htm;
        server_name lxc_landing.dev;

        error_log /var/log/nginx/landing_error.log;
        access_log /var/log/nginx/landing_access.log;

        client_max_body_size 100M;
        location / {
                try_files $uri $uri/ /index.php$args;
        }
        location ~\.php$ {
                include snippets/fastcgi-php.conf;
                fastcgi_pass unix:run/php/php7.4-fpm.sock;
                fastcgi_param SCRIPTFILENAME $document_root$fastcgi_script_name;
        }
}

```

instalasi sukses dan cek dengan buka vm.local

![10](https://user-images.githubusercontent.com/93030868/144253343-68de9c66-e3af-40bf-adf5-30fdfc6a31dd.PNG)

dan laravel sukses dijalan kan

#### vm.local/blog

masuk pada folder ansible/modul2-ansible dan membuat file dengan nama install-wp.yml

![1](https://user-images.githubusercontent.com/93030868/144253612-04dc22a5-a52f-49ed-b6ae-3a8f1a9863db.png)

```markdown
nano install-wp.yml

- hosts: ubuntu_php7
  vars:
    username: 'admin'
    password: 'iqbal'
  roles:
    - wordpress
```

![2](https://user-images.githubusercontent.com/93030868/144253624-75774fdf-6465-46f2-8b09-e6f27dce9087.png)

lalu kita membuat direktori untuk tasks,templates dan handlers in folder wordpress dan masuk ke folder tasks untuk install paket

```markdown
cd roles/wordpress/tasks
nano main.yml
```

![3](https://user-images.githubusercontent.com/93030868/144253477-03a6ff8a-74fb-49ad-b42f-a80748c824f6.png)

```markdown
---
- name: delete apt chache
  become: yes
  become_user: root
  become_method: su
  command: rm -vf /var/lib/apt/lists/*

- name: install requirement
  become: yes
  become_user: root
  become_method: su
  apt: name={{ item }} state=latest update_cache=true
  with_items:
    - nginx
    - nginx-extras
    - curl
    - wget
    - php7.4
    - php7.4-fpm
    - php7.4-curl
    - php7.4-xml
    - php7.4-gd
    - php7.4-opcache
    - php7.4-mbstring
    - php7.4-zip
    - php7.4-json
    - php7.4-cli
    - php7.4-mysqlnd
    - php7.4-xmlrpc
    - php7.4-curl

- name: wget wordpress
  shell: wget -c http://wordpress.org/latest.tar.gz

- name: tar latest.tar.gz
  shell: tar -xvzf latest.tar.gz

- name: copy folder wordpress
  shell: cp -R wordpress /var/www/html/blog

- name: chmod
  become: yes
  become_user: root
  become_method: su
  command: chmod 775 -R /var/www/html/blog/

- name: copy .wp-config.conf
  template:
    src=templates/wp.conf
    dest=/var/www/html/blog/wp-config.php

- name: copy wordpress.conf
  template:
    src=templates/wordpress.conf
    dest=/etc/nginx/sites-available/{{ domain }}
  vars:
    servername: '{{ domain }}'

- name: Symlink wordpress.conf
  command: ln -sfn /etc/nginx/sites-available/{{ domain }} /etc/nginx/sites-enabled/{{ domain }}
  notify:
    - restart nginx

- name: Write {{ domain }} to /etc/hosts
  lineinfile:
    dest: /etc/hosts
    regexp: '.*{{ domain }}$'
    line: "127.0.0.1 {{ domain }}"
    state: present

- name: enable module php mbstring
  command: phpenmod mbstring
  notify:
    - restart php
```

lalu masuk templates wp.conf yang merupakan configuration pada wordpress

![4](https://user-images.githubusercontent.com/93030868/144253501-cd222639-4f8c-4f81-bf76-158bc710992a.png)

```markdown
cd roles/wp/templates
nano wordpress.conf

<?php
/**
 * The base configuration for WordPress
 *
 * The wp-config.php creation script uses this file during the installation.
 * You don't have to use the web site, you can copy this file to "wp-config.php"
 * and fill in the values.
 *
 * This file contains the following configurations:
 *
 * * MySQL settings
 * * Secret keys
 * * Database table prefix
 * * ABSPATH
 *
 * @link https://wordpress.org/support/article/editing-wp-config-php/
 *
 * @package WordPress
 */

define( 'WP_HOME', 'http://vm.local/blog' );
define( 'WP_SITEURL', 'http://vm.local/blog' );

// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'blog' );

/** MySQL database username */
define( 'DB_USER', 'admin' );

/** MySQL database password */
define( 'DB_PASSWORD', 'admin' );

/** MySQL hostname */
define( 'DB_HOST', '10.0.3.200:3306' );

/** Database charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8' );

/** The database collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );

/**#@+
 * Authentication unique keys and salts.
 *
 * Change these to different unique phrases! You can generate these using
 * the {@link https://api.wordpress.org/secret-key/1.1/salt/ WordPress.org secret-key service}.
 *
 * You can change these at any point in time to invalidate all existing cookies.
 * This will force all users to have to log in again.
 *
 * @since 2.6.0
 */
define( 'AUTH_KEY',         'put your unique phrase here' );
define( 'SECURE_AUTH_KEY',  'put your unique phrase here' );
define( 'LOGGED_IN_KEY',    'put your unique phrase here' );
define( 'NONCE_KEY',        'put your unique phrase here' );
define( 'AUTH_SALT',        'put your unique phrase here' );
define( 'SECURE_AUTH_SALT', 'put your unique phrase here' );
define( 'LOGGED_IN_SALT',   'put your unique phrase here' );
define( 'NONCE_SALT',       'put your unique phrase here' );

/**#@-*/

/**
 * WordPress database table prefix.
 *
 * You can have multiple installations in one database if you give each
 * a unique prefix. Only numbers, letters, and underscores please!
 */
$table_prefix = 'wp_';

/**
 * For developers: WordPress debugging mode.
 *
 * Change this to true to enable the display of notices during development.
 * It is strongly recommended that plugin and theme developers use WP_DEBUG
 * in their development environments.
 *
 * For information on other constants that can be used for debugging,
 * visit the documentation.
 *
 * @link https://wordpress.org/support/article/debugging-in-wordpress/
 */
define( 'WP_DEBUG', false );

/* Add any custom values between this line and the "stop editing" line. */



/* That's all, stop editing! Happy publishing. */

/** Absolute path to the WordPress directory. */
if ( ! defined( 'ABSPATH' ) ) {
        define( 'ABSPATH', __DIR__ . '/' );
}

/** Sets up WordPress vars and included files. */
require_once ABSPATH . 'wp-settings.php';
```

setelah itu masuk ke templates wordpress.conf

![5](https://user-images.githubusercontent.com/93030868/144253531-4becd5ee-dbba-48c3-ac86-6697c7ef4bb0.png)

nano wordpress.conf

lalu kita perlu masuk ke foldeer handlers main.yml

![6](https://user-images.githubusercontent.com/93030868/144253567-8756b485-ad57-49a4-a52c-fd4a24c027f6.png)

```markdown
cd roles/wp/handlers
nano main.yml
```

lalu jalan kan ansible untuk menginstall

![7](https://user-images.githubusercontent.com/93030868/144253571-14a5cb9c-c9f3-499b-88a9-3c9536bddb52.png)

```markdown
sudo ansible-playbook -i hosts install-wp.yml -k
```

lalu cek dengan membuka vm.local/blog/

![8](https://user-images.githubusercontent.com/93030868/144253576-5c612d76-8774-4b43-aa87-0934d0d0de2d.png)

![9](https://user-images.githubusercontent.com/93030868/144253580-105dcce3-058a-4ee5-b6e0-7d2953260cfe.png)

![10](https://user-images.githubusercontent.com/93030868/144253581-1390e281-be0b-4b46-9354-5ba99cd4a524.png)

![11](https://user-images.githubusercontent.com/93030868/144253583-bf597af9-c26f-4cc9-89b0-38f74c56de52.png)


wordpress sudah dapat dijalankan
