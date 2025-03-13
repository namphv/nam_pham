### Troubleshooting Approach

Base on the setup, a VM is running Ubuntu 24.04, and has 1 service only which is NGINX load balancer. We should be focus to how NGINX service setup, other issues from server setup is not in scope for this task.

Possible issues cause by NGINX services:

#### Log rotation issue
Logging from NGINX service is not rotate or log rotation cron job is not working

1. Verify
    ```
    df -h /  # Check root partition usage
    du -sh /var/log/nginx/*  # Inspect NGINX log directory size
    ls -lh /var/log/nginx/
    ```
2. Cleanup log
    ```
    sudo truncate -s 0 /var/log/nginx/access.log
    sudo rm /var/log/nginx/*.gz
    ```
3. Check log rotation
    ```
    cat /etc/logrotate.d/nginx
    grep logrotate /var/log/cron* 
    sudo systemctl status logrotate.timer
    ```
    Run log rotate manully
    `sudo logrotate -vf /etc/logrotate.d/nginx`

#### Service caching issue
NGINX service persistent disk cache cause high storage usage

1. Verify
    ```
    du -sh /var/cache/nginx  # NGINX cache directory on config or default 
    ```
2. Get NGINX caching config
    ```
    sudo nginx -T | grep proxy_cache_path  # Locate cache configuration
    cat /etc/nginx/nginx.conf | grep -A10 "proxy_cache_path"  # Inspect cache settings
    ```
    Look for proxy_cache_path parameters:
        - max_size: If unset or too large, cache grows indefinitely.
        - inactive: If misconfigured, old cached files aren't purged.
    Get cache purge setup with `grep proxy_cache_purge /etc/nginx/nginx.conf`
3. Cache Cleanup
    ```
    sudo find /var/cache/nginx -type f -mtime +7 -delete
    ```
4. Update cache config
    ```
    proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=my_cache:10m max_size=5g inactive=7d use_temp_path=off;
    ```
    Also add API purge cache and add cron job cache purge
5. Restart NGINX service
    This action will cause minimal downtime but since NGINX is very fast on start up, the down time might be negligible

#### Large files stored on VM
    Unintended files are on VM, looks for the folder with heaviest size and work from there

    ```
    du -h --max-depth=1
    ```
    
    Remove tempfile
    ```
    rm -rf /tmp/*
    ```
