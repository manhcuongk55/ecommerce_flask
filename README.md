#### Get Started

This based on Ubuntu/Debian，please skip if you had set up Python 3 environment.

```bash
# set up python3 environment
sudo apt-get update
sudo apt-get install python3-pip python3-dev
sudo apt-get install build-essential libssl-dev libffi-dev python-dev

# set up virtualenv
sudo pip3 install virtualenv virtualenvwrapper
echo "export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3" >> ~/.bashrc
echo "export WORKON_HOME=~/Env" >> ~/.bashrc
echo "source /usr/local/bin/virtualenvwrapper.sh" >> ~/.bashrc
source ~/.bashrc

# Now virtual env for python3 will be installed in ~/Env

mkvirtualenv bibi # rename bibi
workon bibi # activate bibi env

# set up mongodb # 2.6 version
# set up redis
# set up rabbitMQ

mongod &              # start mongodb
redis-server &        # start redis
rabbitmq-server &     # start RabbitMQ
```

Install dependencies
```bash
pip3 install -r requirements.txt
```

Initial database
```python
python3 manage.py shell
# into Python3 shell
>>> from application.models import User
>>> user = User.create(email="xxxx@xxx.com", password="xxx", name="xxxx")
# Rename the email, password, name
>>> user.roles.append("ADMIN")
>>> user.save()
```

Run server
```
# start celery
celery -A application.cel worker -l info &

python3 manage.py runserver
```
Now open http://127.0.0.1:5000/admin/ on local.



#### Deploy
```bash
# set up supervisor
sudo apt-get install supervisor
# set up gunicorn
pip3 install gunicorn
```
Create supervisor config

`sudo vim /etc/supervisor/conf.d/bibi.conf`
```
[program:bibi]
command=/root/Env/bibi/bin/gunicorn
    -w 3
    -b 0.0.0.0:8080
    --log-level debug
    "application.app:create_app()"

directory=/opt/py-maybi/                                       ; Project dir
autostart=false
autorestart=false
stdout_logfile=/opt/logs/gunicorn.log                          ; log dir
redirect_stderr=true
```
PS: -w  the workers number，formula：（CPUs*2 + 1)

Create nginx config

`sudo vim /etc/nginx/sites-enabled/bibi.conf`

```nginx
server {
    listen 80;
    server_name bigbang.maybi.cn;

    location / {
        proxy_pass http://127.0.0.1:8080; # Pointing to the gunicorn host
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

  }
```

Start supervisor, nginx
```bash
sudo supervisorctl reload
sudo supervisorctl start bibi

sudo service nginx restart
```

Bravo! It's done.
