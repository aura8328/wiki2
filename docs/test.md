<h3 id="safs-onm에-websocket-설치">SAFS ONM에 Websocket 설치</h3>
<ul>
<li>rpm 패키징 시에 websoket 관련 패키지를 내장하여 패키징 할 수 있도록 구성</li>
<li>rpm 설치 후 /calamari-web/setting.py 파일에 websocket 관련 설정 추가</li>
<li>websocket이 재대로 동작 하는지 확인</li>
<li>현재 사용하고 있는 apache+wsgi 환경에 uwsgi 를 연동 하여 웹서버 동작 확인</li>
<li>osd 관련 정보를 websocket 으로 전달하는 테스트 프로세스 작성하여 클라이언트와 연동 테스트</li>
</ul>
<hr>
<p>vi /opt/calamari/venv/lib/python2.7/site-packages/calamari_web-0.1-py2.7.egg/calamari_web/settings.py</p>
<pre class=" language-python"><code class="prism  language-python"><span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span>
<span class="token comment" spellcheck="true"># WSGI_APPLICATION = 'calamari_web.wsgi.application'
</span>WSGI_APPLICATION <span class="token operator">=</span> <span class="token string">'ws4redis.django_runserver.application'</span>
WEBSOCKET_URL <span class="token operator">=</span> <span class="token string">'/ws/'</span>
<span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span>
INSTALLED_APPS <span class="token operator">=</span> <span class="token punctuation">(</span>
    <span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span>
    <span class="token string">'ws4redis'</span>
<span class="token punctuation">)</span>
</code></pre>
<p>vi /opt/calamari/venv/lib/python2.7/site-packages/calamari_web-0.1-py2.7.egg/calamari_web/wsgi.py</p>
<pre class=" language-python"><code class="prism  language-python"><span class="token string">"""
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

"""</span>
<span class="token keyword">import</span> os

<span class="token comment" spellcheck="true"># We defer to a DJANGO_SETTINGS_MODULE already in the environment. This breaks
</span><span class="token comment" spellcheck="true"># if running multiple sites in the same mod_wsgi process. To fix this, use
</span><span class="token comment" spellcheck="true"># mod_wsgi daemon mode with each site in its own daemon process, or use
</span><span class="token comment" spellcheck="true"># os.environ["DJANGO_SETTINGS_MODULE"] = "calamari.settings"
</span>os<span class="token punctuation">.</span>environ<span class="token punctuation">.</span>setdefault<span class="token punctuation">(</span><span class="token string">"DJANGO_SETTINGS_MODULE"</span><span class="token punctuation">,</span> <span class="token string">"calamari_web.settings"</span><span class="token punctuation">)</span>

<span class="token keyword">from</span> django<span class="token punctuation">.</span>conf <span class="token keyword">import</span> settings
<span class="token keyword">from</span> django<span class="token punctuation">.</span>core<span class="token punctuation">.</span>wsgi <span class="token keyword">import</span> get_wsgi_application
<span class="token keyword">from</span> ws4redis<span class="token punctuation">.</span>uwsgi_runserver <span class="token keyword">import</span> uWSGIWebsocketServer

_django_app <span class="token operator">=</span> get_wsgi_application<span class="token punctuation">(</span><span class="token punctuation">)</span>
_websocket_app <span class="token operator">=</span> uWSGIWebsocketServer<span class="token punctuation">(</span><span class="token punctuation">)</span>

<span class="token keyword">def</span> application<span class="token punctuation">(</span>environ<span class="token punctuation">,</span> start_response<span class="token punctuation">)</span><span class="token punctuation">:</span>
    <span class="token keyword">if</span> environ<span class="token punctuation">.</span>get<span class="token punctuation">(</span><span class="token string">'PATH_INFO'</span><span class="token punctuation">)</span><span class="token punctuation">.</span>startswith<span class="token punctuation">(</span>settings<span class="token punctuation">.</span>WEBSOCKET_URL<span class="token punctuation">)</span><span class="token punctuation">:</span>
        <span class="token keyword">return</span> _websocket_app<span class="token punctuation">(</span>environ<span class="token punctuation">,</span> start_response<span class="token punctuation">)</span>
    <span class="token keyword">return</span> _django_app<span class="token punctuation">(</span>environ<span class="token punctuation">,</span> start_response<span class="token punctuation">)</span>

<span class="token comment" spellcheck="true"># This application object is used by any WSGI server configured to use this
</span><span class="token comment" spellcheck="true"># file. This includes Django's development server, if the WSGI_APPLICATION
</span><span class="token comment" spellcheck="true"># setting points here.
</span>
<span class="token comment" spellcheck="true"># Apply WSGI middleware here.
</span><span class="token comment" spellcheck="true"># from helloworld.wsgi import HelloWorldApplication
</span><span class="token comment" spellcheck="true"># application = HelloWorldApplication(application)
</span></code></pre>
<p>/opt/calamari/venv/bin/uwsgi --virtualenv /opt/calamari/venv --http :8080 --gevent 100 --http-websockets --module wsgi</p>
