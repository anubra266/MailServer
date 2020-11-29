```
<VirtualHost *:80>
       
        #ServerName www.example.com
        ServerName anubra.tech
        ServerAlias *.anubra.tech



	<Directory /var/www/html/anubra.tech>
	    Options Indexes FollowSymLinks MultiViews
	    AllowOverride All
	    Order allow,deny
	    allow from all
	</Directory>        

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>

```
