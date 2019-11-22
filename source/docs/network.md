#  Network useful commands

## Detect services and ports

* Linux

    ```
    nmap -sn <ip_address>  # ip alive? 
    nmap -sn -Pn <ip_address> # Check for all host online if ip alive
    netstat -tulpn # check your network services udp tcp
    ss -nlptau # check your network services udp tcp
    route -n # gateway, ip 
    ip a # network interfaces
    ip route show or route
    sudo ufw enable # disable or status
    sudo ufw deny from <ip_address> to any port 80
    sudo ufw deny from <ip_address> to any
    ```

* Windows 

    ```
    netstat -an # check services and port
    netsh firewall show state # check firewall state
    netsh firewall show config # check firewall configuration
    ```

## References
* [nmap](https://subscription.packtpub.com/book/networking_and_servers/9781784392918/2/ch02lvl1sec18/scanning-and-identifying-services-with-nmap)
* [route](https://www.cyberciti.biz/faq/how-to-find-out-default-gateway-in-ubuntu/)
* [network configuration netplan](https://help.ubuntu.com/lts/serverguide/network-configuration.html)
* [windows firewall and scanning](https://9to5it.com/windows-firewall-is-my-port-being-blocked/)
* [ufw firewall](https://www.cyberciti.biz/faq/how-to-block-an-ip-address-with-ufw-on-ubuntu-linux-server/)