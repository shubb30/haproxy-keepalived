# haproxy-keepalived
HAProxy and Keepalived sidekick pair for Rancher load balancing with floating VIPs.

## Purpose
The built-in load balancers that come with Rancher are only able to provide services on the IP address of the host.  If you want to have a load balancer listen on multiple IP addresses (and serve different services on each IP) in high-availability, then you need to run Keepalived between two containers.  Keepalived will run a VRRP instance between the containers.  The IPs will only exist on one host at a time, and if the host fails, the other host will take over the IPs and continue serving.

## Compose
### docker-compose.yml
* Replace the `<<VRRP auth passphrase>>` from the Keepalived contiainer to your own passphrase for VRRP.
* Change the `<<IP_ADDRESS_X>>` to the IP address for your service.  Add as many `VIP_X` variables for as many virtual IP addresses that you need.

If you want to limit the load balancers to specific hosts, then modify the `io.rancher.scheduler.affinity:host_label: role=loadbalancer` tag to whatever tag you are using for the load balancer hosts.

If you want to limit the containers that can run on the LB hosts so that only load balancer containers run, then modify the `job: loadbalancer` label.  You also need to configure the `Require Container Label` option on the LB hosts to match the LB labels.

Both the HAProxy and Keepalived containers need to run with host networking.  The Keepalived container needs the `NET_ADMIN` and `NET_BROADCAST` privileges.  The label `io.rancher.container.dns: 'true'` configures the containers to use the internal Rancher DNS settings instead of the settings from the host so it can reach the services inside Rancher.

## Files
### haproxy.cfg
Place the haproxy.cfg file on the volume that serves `/usr/local/etc/haproxy`.
Replace `<<IP_ADDRESS_X>>` with the IP address of the each VIP used in the Keepalived container.
Replace `<<SERVICE_X>>` with the name of the service, and replace `<<STACK_X>>` with the name of the Rancher stack.

### 00-alpine.conf
Place the 00-alpine.conf file on the volume that serves `/etc/sysctl.d`.
The only thing this does is add the `net.ipv4.ip_nonlocal_bind = 1` flag to allow HAproxy to bind to the VIP on the standby container.  Alternatively you can also build your own HAproxy container and add this file and remove the volume.   

### SSL certificates
Add the SSL certificates for each VIP to the `/usr/local/etc/haproxy/ssl/private` volume.  The file is a concatenation of the certificate and private key in PEM format.