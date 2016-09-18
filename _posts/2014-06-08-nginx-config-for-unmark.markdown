---
layout:     post
title:      "Nginx Config for Plainmade's Unmark"
date:       2014-06-08 12:00:00
author:     "Zach Yordy"
header-img: "img/iphone-6s-blue.jpg"
---

I've been looking for a good bookmark manager for a long time. A little while ago, I came across **[Unmark](http://unmark.it)**, a simple bookmark manager that treats your marks a little more like to-dos than anything else. After rereading [Lifehacker's guide](http://lifehacker.com/unmark-turns-your-overflowing-bookmarks-into-a-to-do-li-1545916184) on the service, I noticed an almost throwaway sentence right at the end of the article that this bookmark manager can also be self-hosted! Ready to plow on, I followed [Unmark's instructions](https://github.com/plainmade/unmark) on their [GitHub page](https://github.com/plainmade/unmark).

The instructions were pretty easy to follow, but as I got to the end of the configuration, I went to type in **/setup** into my new custom subdomain, and nothing happened. After lots of Googling around, I found out that Unmark had been created for Apache rather than Nginx.

Following the help of an [awesome guide on Nginx Rewrite Rules with CodeIgniter projects](http://www.farinspace.com/codeigniter-nginx-rewrite-rules/), I came up with my own Nginx config for Unmark:

	server {
		listen 80;

		server_name custom.domain.com;

		root /usr/share/nginx/unmark;
		index index.php index.html;

		# Removes access to "system" folder, also allows a "System.php" controller
		if ($request_uri ~* ^/system) {
			rewrite ^/(.*)$ /index.php?/$1 last;
			break;
		}

		# Removes access to "application" folder, since .htaccess specified so.
		if ($request_uri ~* ^/application) {
			rewrite ^/(.*)$ /index.php?/$1 last;
			break;
		}

		# Removes access to "application/cache" folder, since .htaccess specified so.
		if ($request_uri ~* ^/application/cache) {
			rewrite ^/(.*)$ /index.php?/$1 last;
			break;
		}

		# Unless the request is for a valid file (image, js, css, etc.), send to bootstrap
		if (!-e $request_filename) {
			rewrite ^/(.*)$ /index.php?/$1 last;
			break;
		}

		# Catch all
		error_page 404 /index.php;

		location ~ \.php$ {
			try_files $uri =404;
			include fastcgi_params;
			fastcgi_pass php5-fpm-sock;
			fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
			fastcgi_intercept_errors on;
		}

		# Deny access to apache .htaccess files
		location ~ /\.ht {
			deny all;
		}

		#error_page 404 /404.html;
	}

