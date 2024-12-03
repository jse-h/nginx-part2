# user-setup1

Run the user setup script.

must use sudo, Go over contents of script.

Edit the generate_index to have the new code

Change ownerships and permissions

# unit file setup

Run unit file script

go over contents of script, must use sudo

enabling/starting services and timers

# nginx-setup

install nginx

Using the sites-available and sites-enabled approach for web server

configure nginx.conf
copy paste to configure server block

enable server block with symlink
sudo ln -s /etc/nginx/sites-available/webgen.conf /etc/nginx/sited-enabled/

add /sites-enabled directory to nginx.conf

restart -> start nginx