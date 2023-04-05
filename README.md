
# Using IPv6 Tunnel Service to Bypass Port Forwarding Restrictions on CGNAT with Tunnelbroker.net

## Introduction: 
Carrier-Grade NAT (CGNAT) is a technique used by Internet Service Providers (ISPs) to share a single public IPv4 address among multiple customers. This allows ISPs to conserve IPv4 addresses but can also cause issues with certain applications that require port forwarding. One solution to bypass CGNAT restrictions is to use an IPv6 tunnel service, such as Tunnelbroker.net.

This tutorial will guide you through the process of setting up an IPv6 tunnel using Tunnelbroker.net. By following these steps, you will be able to bypass port forwarding restrictions on CGNAT and utilize IPv6 for your internet connection.

Note: This tutorial assumes you have zero experience with IPv6 tunnel services. Basic networking knowledge is helpful but not required.

#### Prerequisites:
-   A computer with internet access
-   A Tunnelbroker.net account (free)

#### Step 1: Creating a Tunnelbroker.net account

1.  Visit [https://tunnelbroker.net/](https://tunnelbroker.net/) and click on "Register."
2.  Fill out the registration form with your details, agree to the terms of service, and click on "Register."
3.  Once you receive the confirmation email, click on the confirmation link to activate your account.

#### Step 2: Creating a new tunnel

1.  Log in to your Tunnelbroker.net account.
2.  Click on "Create Regular Tunnel" under the "User Functions" section.
3.  In the "IPv4 Endpoint" field, enter your public IPv4 address. You can find this by visiting a site like [https://whatismyipaddress.com/](https://whatismyipaddress.com/).
4.  Select a nearby tunnel server from the "Available Tunnel Servers" dropdown menu. This will help ensure a lower latency connection.
5.  Click on "Create Tunnel." The system will then create an IPv6 tunnel for you.

Step 3: Configuring your computer to use the IPv6 tunnel To configure your computer to use the IPv6 tunnel, you will need to set up a new network interface. Instructions for Windows and Linux are provided below:

### For Windows users:

1.  Open the "Control Panel" and click on "Network and Internet."
2.  Click on "Network and Sharing Center."
3.  Click on "Change adapter settings."
4.  Press "Alt" on your keyboard, click on "File," and then "New Incoming Connection."
5.  Click "Next" and select the "Internet Protocol Version 6 (TCP/IPv6)" checkbox.
6.  Click "Properties" and enter the client IPv6 address provided by Tunnelbroker.net in the "IPv6 address" field.
7.  Enter the default IPv6 gateway provided by Tunnelbroker.net in the "Default gateway" field.
8.  Click "OK" and then "Close" to save the settings.
9.  Restart your computer for the changes to take effect.

### For Linux users:

1.  Open a terminal and enter the following command to install the "iproute2" package (if not already installed):

```
sudo apt-get update && sudo apt-get install iproute2
```

2.  Create a new network interface by entering the following command (replace "TUNNEL_INTERFACE_NAME" with your desired interface name):

```
sudo ip tunnel add TUNNEL_INTERFACE_NAME mode sit remote SERVER_IPV4_ADDRESS local CLIENT_IPV4_ADDRESS ttl 255
```

3.  Assign the client IPv6 address provided by Tunnelbroker.net to the new interface:

```
sudo ip -6 addr add CLIENT_IPV6_ADDRESS/64 dev TUNNEL_INTERFACE_NAME
```

4.  Configure the default IPv6 gateway using the server IPv6 address provided by Tunnelbroker.net:

```
sudo ip -6 route add default via SERVER_IPV6_ADDRESS dev TUNNEL_INTERFACE_NAME
```

5.  Enable the new interface with the following command:

```
sudo ip link set TUNNEL_INTERFACE_NAME up
```

6.  To make the configuration permanent, add the above commands to a startup script (e.g., /etc/rc.local) or use the appropriate configuration files for your Linux distribution.

#### Step 4: Testing the IPv6 connection 

To ensure that your computer is successfully using the IPv6 tunnel, visit an IPv6 test site such as [http://test-ipv6.com/](http://test-ipv6.com/). If the test shows that you have a valid IPv6 address, your tunnel is working correctly.

## Automating CLIENT_IPV4 _ADDRESS Retrieval (Linux Users)
To create a systemd service that automatically updates the CLIENT_IPV4_ADDRESS and configures the IPv6 tunnel, follow these steps:

1.  Create a new script file that contains the tunnel configuration commands:

```
sudo nano /usr/local/bin/configure_ipv6_tunnel.sh
```

2.  Add the following content to the script, replacing "TUNNEL_INTERFACE_NAME" with your desired interface name, "SERVER_IPV4_ADDRESS" with the Tunnelbroker.net server IPv4 address, "CLIENT_IPV6_ADDRESS" with the client IPv6 address provided by Tunnelbroker.net, and "SERVER_IPV6_ADDRESS" with the server IPv6 address provided by Tunnelbroker.net:

```
#!/bin/bash

# Retrieve the public IPv4 address
CLIENT_IPV4_ADDRESS=$(dig +short myip.opendns.com @resolver1.opendns.com)

# Save the current IP address to a file for comparison
IP_FILE="/tmp/current_ipv4_address.txt"
echo "$CLIENT_IPV4_ADDRESS" > "$IP_FILE"

# Remove the existing tunnel interface if it exists
sudo ip link set TUNNEL_INTERFACE_NAME down 2>/dev/null
sudo ip tunnel del TUNNEL_INTERFACE_NAME 2>/dev/null

# Create a new tunnel interface
sudo ip tunnel add TUNNEL_INTERFACE_NAME mode sit remote SERVER_IPV4_ADDRESS local $CLIENT_IPV4_ADDRESS ttl 255

# Assign the client IPv6 address to the new interface
sudo ip -6 addr add CLIENT_IPV6_ADDRESS/64 dev TUNNEL_INTERFACE_NAME

# Configure the default IPv6 gateway
sudo ip -6 route add default via SERVER_IPV6_ADDRESS dev TUNNEL_INTERFACE_NAME

# Enable the new interface
sudo ip link set TUNNEL_INTERFACE_NAME up
```

3.  Save the file and exit the editor (Ctrl + X, then press 'Y' and Enter).
    
4.  Make the script executable:
    
```
sudo chmod +x /usr/local/bin/configure_ipv6_tunnel.sh
``` 

5.  Create a new systemd service file:

```
sudo nano /etc/systemd/system/ipv6_tunnel.service
``` 

6.  Add the following content to the service file:

```
[Unit]
Description=IPv6 Tunnel Configuration
After=network.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/configure_ipv6_tunnel.sh
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

7.  Save the file and exit the editor (Ctrl + X, then press 'Y' and Enter).
    
8.  Enable the systemd service:

```
sudo systemctl enable ipv6_tunnel.service
``` 

9.  Start the systemd service:

```
sudo systemctl start ipv6_tunnel.service
``` 

Now, the IPv6 tunnel will be configured and activated whenever your device starts. To update the CLIENT_IPV4_ADDRESS and reconfigure the tunnel when your device's IP address changes, simply restart the ipv6_tunnel.service:

```
sudo systemctl restart ipv6_tunnel.service
``` 

To automate the restart process, you can create a script that periodically checks for IP address changes and restarts the service if needed. 

## Automating CLIENT_IPV4 _ADDRESS Updating (Linux Users)

To create a script that periodically checks for IP address changes and restarts the ipv6_tunnel.service if needed, follow these steps:

1.  Create a new script file:

```
sudo nano /usr/local/bin/update_ipv6_tunnel.sh
``` 

2.  Add the following content to the script:

```
#!/bin/bash

# Save the current IP address to a file for comparison
IP_FILE="/tmp/current_ipv4_address.txt"

# Define a function to retrieve the public IPv4 address using different services to balance requests
get_ipv4_address() {
    case $(( RANDOM % 4 )) in
        0)
            dig +short myip.opendns.com @resolver1.opendns.com
            ;;
        1)
            curl -s ifconfig.me
            ;;
        2)
            curl -s ipecho.net/plain
            ;;
        3)
            curl -s ipinfo.io/ip
            ;;
    esac
}

# Retrieve the current public IPv4 address
CURRENT_IPV4_ADDRESS=$(get_ipv4_address)

if [ ! -f "$IP_FILE" ]; then
    # Save the current IP address if the file does not exist
    echo "$CURRENT_IPV4_ADDRESS" > "$IP_FILE"
fi

# Read the saved IP address from the file
SAVED_IPV4_ADDRESS=$(cat "$IP_FILE")

# Compare the current IP address with the saved one
if [ "$CURRENT_IPV4_ADDRESS" != "$SAVED_IPV4_ADDRESS" ]; then
    # If the IP addresses differ, restart the ipv6_tunnel service
    sudo systemctl restart ipv6_tunnel.service
fi
```

3.  Save the file and exit the editor (Ctrl + X, then press 'Y' and Enter).
    
4.  Make the script executable:
    
```
sudo chmod +x /usr/local/bin/update_ipv6_tunnel.sh
``` 

5.  To run this script periodically, add it to your crontab. Open your crontab file using the following command:

```
sudo crontab -e
``` 

6.  Add the following line at the end of the file to run the script every 1 minute (you can adjust the frequency as needed):

```
*/1 * * * * /usr/local/bin/update_ipv6_tunnel.sh
``` 

7.  Save the file and exit the editor.

Now, the `update_ipv6_tunnel.sh` script will run every 1 minute and check for IP address changes. If it detects a change, it will restart the ipv6_tunnel.service, which in turn updates the CLIENT_IPV4_ADDRESS and reconfigures the IPv6 tunnel.
