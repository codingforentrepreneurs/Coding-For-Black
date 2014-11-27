Installation for nginx, gunicorn, and Django on a Beaglebone Black Debian System
=========
Installation guide that complments our Coding for Black tutorial series by Coding For Entrepreneurs. This guide assumes you have the Debian OS installed on a Beaglebone Black.



1. Ensure Network/Internet is connected through Ethernet

2. Power up Beaglebone Black (BBB)
3. SSH into your BBB with Terminal (Mac/Linux) or puTTY (Windows)

	```
	$ ssh 192.168.7.2 -l root
	```

3. Update and Upgrade (takes up to 1 hour)
	Respond to "yes" for extra disk space usage when prompted
	```
	# sudo apt-get update
	# sudo apt-get upgrade
	```

4. Install Dependancies

	```
	# sudo apt-get install build-essential python-dev python-pip python-smbus git nginx lynx supervisor
	# sudo pip install gunicorn
	```

5. Install Django & Create Project
	We're using the lastest version of [Django (v 1.7.1)](http://djangoproject.com/download); replace the X below as needed.

	```
	# sudo pip install django==1.7.X
	# cd ~/ 
	# django-admin.py startproject example_django_project
	```

6. Change into your Django project (the root of the project)

	```
	# cd example_django_project 
	# ls
	example_django_project manage.py
	```

7. Run Migrate (or Syncdb) and Runserver
	```
	# python manage.py migrate #or syncdb
	```

	then 

	```
	# python manage.py runserver
	November 27, 2014 - 05:41:36
	Django version 1.7.1, using settings 'cfb.settings'
	Starting development server at http://127.0.0.1:8000/
	Quit the server with CONTROL-C.
	```


8. Confirm Django is working with Lynx
	1. Open New Terminal (Mac/Linux) or puTTY (Windows) Window
	2. SSH Again (keep both sessions active):
	```
	$ ssh 192.168.7.2 -l root
	```
	3. Use Lynx to test Django is running:
	```
	# lynx http://127.0.0.1:8000/ 
	```
	http://127.0.0.1:8000/ comes from the `Django` runserver result

	3. Confirm you see the initial django welcome screen in text-only. Something like:

	```
	It worked!

	Congratulations on your first Django-powered page.

	Of course, you haven't actually done any work yet. Next, start your
	first app by running python manage.py startapp [app_label].

	You're seeing this message because you have DEBUG = True in your Django
	settings file and you haven't configured any URLs. Get to work!
	```
	
	4. Close this Window and go back to main SSH session


9. Setup Gunicorn
    1. Take note of which gunicorn and python you're using.
    ```
    # which python
    /usr/bin/python

    # which gunicorn
    /usr/local/bin/gunicorn
    ```

	2. Create new file at `etc/supervisor/conf.d/gunicorn.conf `
	Using Nano:
	```
	nano /etc/supervisor/conf.d/gunicorn.conf 

	```
	3. Add the following to the file
	Please note the location of your python, guincorn, and django directory
	```
	[program:gunicorn] 
	command = /usr/bin/python /usr/local/bin/gunicorn example_django_project.wsgi:application 
	directory = /root/example_django_project/
	user = root
	 autostart = true 
	autorestart = true 
	stdout_logfile = /var/log/supervisor/gunicorn.log 
	stderr_logfile = /var/log/supervisor/gunicorn_err.log 

	```
	4. Save the File and Exit.
	Using Nano, you do `Control + x` then `y` then `Return`

10. Update Supervisor
	Do this if you ever change your Guincorn settings in step #9
	```
	sudo supervisorctl reread
	sudo supervisorctl update
	sudo supervisorctl status
	```

11. Setup Nginx
	1. Create Nginx settings file:
	```
	nano /etc/nginx/sites-available/example_django_project
	```
	2. Paste the following in:
	```
	upstream gunicorn {
	    server 127.0.0.1:8000;
	}

	# define an nginx server at port 80
	server {
	    # listen on port 80
	    listen 80;

	    # for serving static files with django
	    root /root/static/;

	    # keep logs in these files
	    access_log /var/log/nginx/example_django_project_access.log;
	    error_log /var/log/nginx/example_django_project_error.log;

	    client_max_body_size 0;

	    # Attempt to serve files first, then pass the request up to Gunicorn
	    try_files $uri @gunicorn;

	    # define rules for gunicorn
	    location @gunicorn {
	        client_max_body_size 0;
	        proxy_pass http://gunicorn;
	        proxy_redirect off;
	        proxy_read_timeout 5m;

	        # make sure these HTTP headers are set properly
	        proxy_set_header Host            $host;
	        proxy_set_header X-Real-IP       $remote_addr;
	        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	    }
	}
	```
	3. Save and Close File.
	Using Nano, you do `Control + x` then `y` then `Return`

	4. Update Nginx Sites Enabled
	```
	# cd /etc/nginx/sites-enabled
	# ls
	default
	# rm default
	# sudo ln -s ../sites-available/example_django_project

	5. Restart Nginx
	```
	# sudo service nginx restart
	```

12. Deactivate Default Beaglebone Black Services:
	It's okay if some fail.
	```
	# systemctl disable cloud9.service
	# systemctl disable gateone.service
	# systemctl disable bonescript.service
	# systemctl disable bonescript.socket
	# systemctl disable bonescript-autorun.service
	# systemctl disable avahi-daemon.service
	# systemctl disable gdm.service
	# systemctl disable mpd.service
	```

13. Restart your BBB. Keep Ethernet plugged in.

14. SSH into you BBB
	```

	```

15. Read Status

	```
	# sudo supervisorctl status
	gunicorn         RUNNING    pid 1176, uptime 196 days, 16:45:54
	

	# sudo service nginx status
	nginx.service - A high performance web server and a reverse proxy server
	  Loaded: loaded (/lib/systemd/system/nginx.service; enabled)
	  Active: active (running) since Thu, 15 May 2014 02:19:00 +0000; 6 months and 14 days ago
	 Process: 838 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
	 Process: 593 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
	Main PID: 871 (nginx)
	  CGroup: name=systemd:/system/nginx.service
		  ├ 871 nginx: master process /usr/sbin/nginx -g daemon on...
		  ├ 872 nginx: worker process
		  ├ 875 nginx: worker process
		  ├ 876 nginx: worker process
		  └ 880 nginx: worker process


	```

16. Get your BBB's IP Address
	```
	# ifconfig eth0 | grep inet | awk '{ print $2 }'
	192.168.0.12

	```

17. Go to another computer, on your network, open a web browser (Safari, Chrome, Firefox, IE, etc):
	```
	http://192.168.0.12/
	```


18. Congratulations. Your Beaglebone is now ready to rock as a network server.



