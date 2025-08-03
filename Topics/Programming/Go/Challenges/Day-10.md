# **Day 10: Basic Interaction with cEOS (Manual SSH/eAPI prep)**

## **Introduction:** 
Understanding how to connect to cEOS. SSH for CLI access. Introduction to eAPI (EOS API) as the programmatic interface. Enabling eAPI on cEOS.

## **Code Example: Enabling eAPI on cEOS (via startup-config)**

Update your `initial_config.clab.yaml` (from Day 7) to include:

```yaml
# ... inside each ceos node config ...
    management api http-commands
        no shutdown
        protocol http # Or https, depending on your preference
        # To allow specific sources (optional, but good practice)
        # ip http-server source-interface Management0
        # ip http-server protocol https ssl profile server
# ...
```

* Deploy the lab with this updated YAML.
* Verify eAPI is enabled by trying to access it from your host (e.g., using `curl`). You'll need the container's IP address. You can get it with `docker inspect <container_name> | grep "IPAddress"`. Then `curl -k -u admin:admin https://<container_ip>/command-api`. (Note: replace `admin:admin` with your configured username/password and adjust for `http` or `https`).

## **Challenge 10:** 
Deploy the lab with eAPI enabled. From your host machine, use `curl` to send a `show version` command to one of your cEOS nodes via eAPI. Parse the JSON output manually to find the EOS version.



