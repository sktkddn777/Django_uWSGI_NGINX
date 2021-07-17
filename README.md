# Practice_Devops

set up Django on Nginx with uWSGI


# 모델 구조 이미지
![](https://pic3.zhimg.com/v2-9bc6cfcdb7b946a728ccfe26f4eb5c01_1200x500.jpg)

# 세팅하는 방법
1. AWS에 서버를 하나 만들어 둡니다. 
2. Ubuntu서버로 만들고, 인바운드 규칙에서 8000번과 80번을 추가해준다.
3. 서버를 업데이트 해준다.
```
sudo apt-get update
sudo apt-get upgrade 
```
4. 가상환경을 만들고 requirements.txt내용을 설치해 준다.
```
pip install -r requirements.txt
```

## uWSGI란
WSGI라는 규칙을 따라서 만들어진 소프트웨어이다.\
정적인 웹 서버(Apache / Nginx)와 python으로 작성된 Web Framework(Flask / Django) 사이의 통신을 도와주는 역할을 한다.\
 즉, 파이썬 앱이 웹 서버와 통신하기 위한 명세!!

## NGINX란
비동기 이벤트 기반구조의 웹서버 소프트웨어다.\
정적 파일을 처리하는 웹 서버의 역할을 수행하기도 하고\
클라이언트의 80포트 요청을 여러 application server로 보내주기도 한다.  
  
<br/>
<br/>
uwsgi가 돌아가는지 확인하는 법.  

```
uwsgi --http :8000 --wsgi-file test.py
```

만들어둔 test.py를 실행해보면 Hello World를 볼 수 있다.  
장고 프로젝트는 

     uwsgi --http :8000 --module mydevops.wsgi
이렇게 확인할 수 있다.


### NGINX 실행 하는 방법.
    sudo apt-get install nginx
nginx를 설치하였다면 /etc/nginx/sites-available 디렉토리가 있을 것이고  

/etc/nginx/sites-available에 mydevops.conf파일을 만들어준다.
```
# mydevops.conf
# the upstream component nginx needs to connect to
upstream django {
    #server 3.36.94.248:8000;
    server unix:///tmp/mydevops.sock;
}

# configuration of the server
server {
    listen      80;
    server_name 3.36.94.248;
    charset     utf-8;

    # max upload size
    client_max_body_size 75M;

    # Django media and static files
    location /media  {
        alias /tmp/media;
    }
    location /static {
        alias /tmp/static;
    }

    # Send all non-media requests to the Django server.
    location / {
        uwsgi_pass  django;
        include     /etc/nginx/uwsgi_params;
    }
}
```

uwsgi_params를 repo에 있는거처럼 만들어주고

    sudo ln -s /etc/nginx/sites-available/mydevops.conf /etc/nginx/sites-enabled/
symbolic link를 만들어준다.

장고의 settings.py에서는 

    STATIC_URL = '/static/'
    STATIC_ROOT = os.path.join(BASE_DIR, "static/")
위 코드를 추가해줘서 static file들을 static folder에 넣게 해준다.

<br/>

## uWSGI 파일 세팅.
mydevops_uwsgi.ini처럼 세팅을 해주면 된다.  
chmod-socket    = 666  

<br/>

# 실행
    uwsgi --ini /etc/uwsgi/sites/mydevops_uwsgi.ini


