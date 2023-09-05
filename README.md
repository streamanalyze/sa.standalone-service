# sa.standalone-service

This package allows you to host your own standalone SA Studio along with a nameserver. It is not intended for production use with high availability or for more than a few edges.

The easiest way to get started is by launching a container with the environment variable `SA_FED_SERVICES_EXTERNAL_ADDRESS` set to the address of your server accessible to users and edges. Make sure to map the public port from your address to port 3001 in the container:

```bash
docker run --env SA_FED_SERVICES_EXTERNAL_ADDRESS=https://172.26.53.154:443 -p 443:3001
```

Running the container like this will generate self-signed certificates and create an admin user with a password that will be displayed in the log. A successful startup will produce output similar to this:

```plaintext
########### SA Standalone-studio ###########

WARNING: Using 172.26.53.154 as the hostname of the Nameserver. You can override this by setting SA_HOSTNAME when creating the container.
WARNING: Using 172.26.53.154 as the Common Name of the federation. You can override this by setting SA_FED_SERVICES_COMMON_NAME when creating the container.
If you are providing your own certificates, the common name (CN) must match the certificate authority that signed the certificates.


Adding a password for the user admin
No htpasswd file detected.
Adding an admin user with the password 4e1085481d716d2c1b9a313360dafa


JWT token is: dd8519e20599f8c620212ea47811b3

Standalone Studio starting at https://172.26.53.154:443

2023-09-05 12:30:48,825 INFO Set UID to user 0 succeeded
2023-09-05 12:30:48,827 INFO supervisord started with PID 35
2023-09-05 12:30:49,830 INFO spawned: 'sa.engine' with PID 37
2023-09-05 12:30:49,832 INFO spawned: 'sa.studio' with PID 38
2023-09-05 12:30:51,532 INFO success: sa.engine entered RUNNING state, process has been up for more than 1 second (startsecs)
2023-09-05 12:30:51,532 INFO success: sa.studio entered RUNNING state, process has been up for more than 1 second (startsecs)
```

## Manual Configuration of Your Standalone Studio

You can provide your own certificates and `.htpasswd` file by mounting a folder from your host to `/home/root/SA` and adding the following files to that folder:

* `tls/ca.crt` - The certificate authority used to sign `cert.crt`. The common name (CN) of the CA must match either the host of `SA_FED_SERVICES_EXTERNAL_ADDRESS` or the name provided by `SA_FED_SERVICES_COMMON_NAME`.
* `tls/cert.crt` - The certificate that the nginx server will use to handle https/wss traffic.
* `tls/private.key` - The private key for the server's certificate.
* `tls/.htpasswd` - An htpasswd file containing usernames and passwords. Refer to [https://httpd.apache.org/docs/2.4/programs/htpasswd.html](https://httpd.apache.org/docs/2.4/programs/htpasswd.htm).

### Pro Tip: Let the Standalone Studio Create Your Files

If you mount an empty directory as `/home/root/SA`, the container will generate the required files in it. You can reuse the same configuration for subsequent container startups:

```bash
mkdir SA
docker run --env SA_FED_SERVICES_EXTERNAL_ADDRESS=https://172.26.53.154:443 -p 443:3001 -v $(pwd)/SA:/home/root/SA sa.standalone-studio:0.0.1
########### SA Standalone-studio ###########
...
Adding a password for the user admin
No htpasswd file detected.
Adding an admin user with the password 4e1085481d716d2c1b9a313360dafa
...
```

Now you can examine the SA folder:

```bash
ls -laR SA
```

Next time, starting the container in the same way will reuse the certificates and `.htpasswd` file:

```bash
docker run --env SA_FED_SERVICES_EXTERNAL_ADDRESS=https://172.26.53.154:443 -p 443:3001 -v $(pwd)/SA:/home/root/SA sa.standalone-studio:0.0.1
```

### Pro Tip No. 2: Update the htpasswd File During Operations

When using a mounted folder, you can add users to the `.htpasswd` file using the `htpasswd` tool from apache2-utils:

```bash
htpasswd -c SA/tls/.htpasswd USERNAME
```

