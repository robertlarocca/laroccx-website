# robertlarocca.github.io

The public website and source code for [www.laroccx.com](https://www.laroccx.com) generated with Hugo.

## Test website with Hugo server

```shell
hugo server -D -E -F
```

## Build production website with Hugo

```shell
hugo --cleanDestinationDir --forceSyncStatic --gc
```

## Sync changes to web server

If hosting this website using GitHub Pages, just use `git commit -a` and `git push` the changes.

```shell
rsync -av doc/ root@nginx.example.com:/var/www/html
```

## Change website ownership and permissions

### Ubuntu

```shell
sudo find /var/www/ -type d -print0 | xargs -0 chmod 0755
sudo find /var/www/ -type f -print0 | xargs -0 chmod 0644

sudo chown -R www-data:www-data /var/www/
sudo chown root:root /var/www

sudo chmod 0755 /var/www
```

### Red Hat Enterprise Linux

```shell
sudo find /var/www/ -type d -print0 | xargs -0 chmod 0755
sudo find /var/www/ -type f -print0 | xargs -0 chmod 0644

sudo chown -R apache:apache /var/www/
sudo chown root:root /var/www

sudo chmod 0755 /var/www
```
