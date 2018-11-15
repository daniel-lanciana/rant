## Ghost

```bash
# Run docker accessible via localhost:8888, which proxies to the Docker container proxy 2368
# -d detaches process to a background thread
docker run -d -p 8888:2368 ghost
# [2018-11-15 17:40:11] INFO Your blog is now available on http://localhost:2368/
```
