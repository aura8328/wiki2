
### SAFS ONM에 Websocket 설치 
- rpm 패키징 시에 websoket 관련 패키지를 내장하여 패키징 할 수 있도록 구성
- rpm 설치 후 /calamari-web/setting.py 파일에 websocket 관련 설정 추가
- websocket이 재대로 동작 하는지 확인
- 현재 사용하고 있는 apache+wsgi 환경에 uwsgi 를 연동 하여 웹서버 동작 확인
- osd 관련 정보를 websocket 으로 전달하는 테스트 프로세스 작성하여 클라이언트와 연동 테스트 

_ _ _

vi /opt/calamari/venv/lib/python2.7/site-packages/calamari_web-0.1-py2.7.egg/calamari_web/settings.py
```python
...
# WSGI_APPLICATION = 'calamari_web.wsgi.application'
WSGI_APPLICATION = 'ws4redis.django_runserver.application'
WEBSOCKET_URL = '/ws/'
...
INSTALLED_APPS = (
    ...
    'ws4redis'
)
```

vi /opt/calamari/venv/lib/python2.7/site-packages/calamari_web-0.1-py2.7.egg/calamari_web/wsgi.py
```python
"""
WSGI config for calamari project.

This module contains the WSGI application used by Django's development server
and any production WSGI deployments. It should expose a module-level variable
named ``application``. Django's ``runserver`` and ``runfcgi`` commands discover
this application via the ``WSGI_APPLICATION`` setting.

Usually you will have the standard Django WSGI application here, but it also
might make sense to replace the whole Django WSGI application with a custom one
that later delegates to the Django one. For example, you could introduce WSGI
middleware here, or combine a Django application with an application of another
framework.

"""
import os

# We defer to a DJANGO_SETTINGS_MODULE already in the environment. This breaks
# if running multiple sites in the same mod_wsgi process. To fix this, use
# mod_wsgi daemon mode with each site in its own daemon process, or use
# os.environ["DJANGO_SETTINGS_MODULE"] = "calamari.settings"
os.environ.setdefault("DJANGO_SETTINGS_MODULE", "calamari_web.settings")

from django.conf import settings
from django.core.wsgi import get_wsgi_application
from ws4redis.uwsgi_runserver import uWSGIWebsocketServer

_django_app = get_wsgi_application()
_websocket_app = uWSGIWebsocketServer()

def application(environ, start_response):
    if environ.get('PATH_INFO').startswith(settings.WEBSOCKET_URL):
        return _websocket_app(environ, start_response)
    return _django_app(environ, start_response)

# This application object is used by any WSGI server configured to use this
# file. This includes Django's development server, if the WSGI_APPLICATION
# setting points here.

# Apply WSGI middleware here.
# from helloworld.wsgi import HelloWorldApplication
# application = HelloWorldApplication(application)
```

/opt/calamari/venv/bin/uwsgi --virtualenv /opt/calamari/venv --http :8080 --gevent 100 --http-websockets --module wsgi


