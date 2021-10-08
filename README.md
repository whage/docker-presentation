# My docker basics presentation

Based on https://github.com/hakimel/reveal.js

```
docker run \
  --rm \
  --name nginx \
  -v /home/vagrant/work/tmp/docker-presentation/nginx/nginx.conf:/etc/nginx/nginx.conf:ro \
  -v /home/vagrant/work/tmp/docker-presentation:/var/www \
  -p 9999:80 \
  -d \
  nginx
```
