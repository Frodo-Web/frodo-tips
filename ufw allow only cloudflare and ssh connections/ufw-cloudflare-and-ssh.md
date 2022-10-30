## Allow only SSH and Cloudflare connections on Linux server with UFW
````
$ sudo apt install ufw
$ sudo ufw status verbose // Should be Status: inactive
$ sudo ufw disable // If not, disable it
$ sudo ufw reset // Clear out any existing rules
$ sudo ufw default deny incoming // Deny incoming connections
$ sudo ufw default allow outgoing // Allow outgoing connections
$ sudo ufw allow from 192.168.1.0/24
$ sudo ufw allow from 192.168.0.0/24 // Allow connections from local network
$ sudo ufw allow 2222 // Type your port on which SSH is configured. Allow connections on that port
$ sudo ufw enable // Enable firewall
$ sudo ufw status verbose // To see the rules applied
$ sudo ./cloudflare-ufw.sh // Run the script to add cloudflare ips
$ sudo crontab -e // Open crontab
$ 0 0 * * 1 /full/path/cloudflare-ufw.sh > /dev/null 2>&1 // Run the script every Monday at 00:00 to check for and update cloudflare ips
````
### Delete a single UFW rule
````
$ sudo ufw status numbered // Get numbered list of all rules
# sudo ufw delete 37 // Delete the rule by number
````

