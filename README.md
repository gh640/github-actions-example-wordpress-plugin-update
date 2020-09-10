# `github-actions-example-wordpress-plugin-update`

This is a GitHub Actions example to update WordPress plugins.

```yaml
name: Update WordPress plugins

on:
  schedule:
    # Runs tasks at 00:00 everyday.
    - cron: '0 0 * * *'

jobs:
  update:
    runs-on: ubuntu-latest
    env:
      MYSQL_DATABASE: wp
      MYSQL_USER: wp
      MYSQL_PASSWORD: wp
    steps:
      - name: Checkout
        uses: actions/checkout@v2
      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: '7.4'
          extensions: gd, mbstring, mysqli, zip
      - name: Install WP-CLI
        run: |
          curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar
          chmod +x wp-cli.phar
          mv wp-cli.phar /usr/local/bin/wp
      - name: Start MySQL
        run: sudo systemctl start mysql.service
      - name: Create MySQL database
        run: >
          mysql --user=root --password=root -e "
          CREATE DATABASE \`$MYSQL_DATABASE\` ;
          CREATE USER '$MYSQL_USER'@'%' IDENTIFIED BY '$MYSQL_PASSWORD' ;
          GRANT ALL ON \`$MYSQL_DATABASE\`.* TO '$MYSQL_USER'@'%' ;
          FLUSH PRIVILEGES ;
          "
      - name: Configure WordPress
        run: >
          wp config create
          --allow-root
          --dbname=$MYSQL_DATABASE
          --dbuser=$MYSQL_USER
          --dbpass=$MYSQL_PASSWORD
          --force
        working-directory: wp_root
      - name: Install WordPress
        run: >
          wp core install
          --allow-root
          --url=localhost
          --title=WP
          --admin_user=admin
          --admin_email=example@example.com
          --admin_password=password
          --skip-email
        working-directory: wp_root
      - name: Enable Plugins
        run: >
          wp plugin activate
          --allow-root
          --all
        working-directory: wp_root
      - name: Update Plugins
        run: >
          wp plugin update
          --allow-root
          --all
        working-directory: wp_root
      - name: Create Pull Request
        uses: peter-evans/create-pull-request@v3
```

## Reference

A post written in Japanase:

- [GitHub Actions で WordPress のプラグインを自動更新する方法](https://gotohayato.com/content/521/)
