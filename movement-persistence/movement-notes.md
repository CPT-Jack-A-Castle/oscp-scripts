# Movement 
## Port Redirection & Tunneling
Port redirection and tunneling is useful in restricted network environments. Consider a scenario where there is a machine on an internal corporate network behind a firewall. This firewall only permits traffic to other machines within the corporate LAN. If you have already compromised an external-facing web server within the corporate LAN, you could tunnel traffic from the web server to the "protected" machine.
### Port Forwarding
- `ssh -R <local-port>:127.0.0.1:<target-port> <username>@<local-machine>`