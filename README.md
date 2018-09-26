# Aplicação de Gestão de Clientes

Antes de iniciar a configuração do projeto, é importante realizar a migração do módulo de clientes criado na primeira fase do curso. Basta apenas copiar e colar as pastas **clientes** e **templates**, colocando a pasta _templates_ dentro de _clientes_.

## Configurando o projeto

1. Definir a aplicação no **gestao_clietes/settings.py**

    ```py
    INSTALLED_APPS = [
        'django.contrib.admin',
        'django.contrib.auth',
        'django.contrib.contenttypes',
        'django.contrib.sessions',
        'django.contrib.messages',
        'django.contrib.staticfiles',
        'clientes',
    ]
    ```
2. Apontar as **gestao_clietes/urls.py** para a aplicação

    ```py
    from django.contrib import admin
    from django.urls import path, include
    from clientes import urls as clientes_urls

    urlpatterns = [
    path('clientes/', include(clientes_urls)),
    path('admin/', admin.site.urls),
    ]
    ```
3. Migrando a aplicação para o banco de dados

    ```sh
    (venv) eliabeleal@DESKTOP:~/projetoFinal/gestao_clientes$ python manage.py migrate
    ```
4. Fazendo a "gaba" para que os arquivos estáticos de mídia seja renderizado.

    Em **gestao_clientes/settings.py**

    ```py
    MEDIA_URL = '/media/'
    MEDIA_ROOT = 'media'
    ```

    Em **gestao_clientes/urls.py**

    ```py
    from django.conf  settings
    from django.conf.urls.static import static

    urlpatterns = [
        path('clientes/', include(clientes_urls)),
        path('admin/', admin.site.urls),
    ] + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
    ```

## Segurança (login)

Para que as aplicações não seja acessadas por qualquer pessoa, é preciso que haja uma proteção em todas. E para isso o django provê um sistema de login e logout.

1. Em **clientes/views.py**

    Protegendo os métodos com @login_required

    ```py
    from django.contrib.auth.decorators import login_required
    @login_required
    def persons_list(request):
        persons = Person.objects.all()
        return render(request, 'person.html', {'persons': persons})
    ```

2. Em **settings.py**

    Indicando as URLS de login e o redirecionamento depois de logado

    ```py
    LOGIN_URL = '/login/'

    LOGIN_REDIRECT_URL = 'person_list'
    ```

3. Em **urls.py**

    Criando a url de login e logout

    ```py
    from django.contrib.auth import views as auth_views 
    urlpatterns = [
        path('login/', auth_views.LoginView.as_view(), name='login'),
        path('logout/', auth_views.LogoutView.as_view(), name='logout'),
    ]
    ```
    No django 2.1 foram removidas (ou melhor movidas) algunas views, entre elas as de login. Que agora são usadas como classes.

4. Em **settings.py**

    Informando qual será a pasta usada para os templates do sistema (não são os da aplicação)

    ```py
    TEMPLATES = [
        {
            'DIRS': ['templates'],
        },
    ]
    ```

5. Em **templates/registration/login.html**

    Importanto o formulário para login já disponibilizado pelo django

    ```html
    <h2>Login</h2>
    <form method="post">
        {% csrf_token %}
        {{form.as_p}}
        <button type="submit">Login</button>
    </form>
    ```
6. Em **cliente/templates/person.html**

    Crindo um link para logout

    ```html
    <a href="{% url 'logout' %}">Sair</a>
    ```

## Home

1. Antes é preciso iniciar uma nova aplicação no projeto que tratará da página inicial

    ```sh
        (venv) eliabeleal@DESKTOP:~/projetoFinal/gestao_clientes$ python manage.py startapp home
    ```
2. Em **settings.py** cadastramos a aplicação

    ```py
    INSTALLED_APPS = [
        'clientes',
        'home',
    ]
    ```
3. Em **gestao_clientes/urls.py** criamos uma url vazia e importamos as urls da aplicação _home_. 

    ```py
    from home import urls as home_urls
    path('', include(home_urls)),
    ```
4. Em **home/urls.py**. Esse arquivo não existe, portanto é preciso criá-lo. Nele direcionamos duas views _home e my\_logout_.

    ```py
    from django.urls import path
    from .views import home, my_logout

    urlpatterns = [
        path('', home, name='home'),
        path('logout/', my_logout, name='logout'),
    ]
    ```
5. Em **home/views.py**. A primeira função está chamando o template **home**. Já a segunda chama a função logout e depois redireciona para o home

    ```py
    from django.shortcuts import render, redirect
    from django.contrib.auth import logout

    def home(request):
        return render(request, 'home.html')

    def my_logout(request):
        logout(request)
        return redirect('home')
    ```
6. Em **home/templates/home.html**. Aqui temos algumas variáveis e métodos já disponibilizados pelo django

    ```html
    <body>
        <h1>Seja bem vindo</h1>

        {% if user.is_authenticated %}
            <p>Ola {{user.username}}</p>
            <a href="{% url 'logout' %}">Sair</a>
        {% else %}
            <a href="{% url 'login' %}">Entrar</a>
        {% endif %}
    </body>
    ```

## Usando o postgres

1. Após o download do postgres, é necessário fazer a configuração do **settings.py**. Nele existe um link para a documentação do django que nos dá a configuração do django para o postegres.

    ```py
    # https://docs.djangoproject.com/en/2.1/ref/settings/#databases
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.postgresql',
            'NAME': 'gestao_clientes',
            'USER': 'postgres',
            'PASSWORD': 'labindy',
            'HOST': '127.0.0.1',
            'PORT': '5432',
        }
    }
    ```
2. Para fazer a migração das aplicações, é necessário instalar o **pip install psycopg2**

3. Logo depois pode-se migrar usando o **python manage.py migrate**

4. Como é um novo banco de dados usando o comando **python manage.py createsuperuser** criamos um super usuário. Se tudo der "Ok", temos um novo banco de dados em nossa aplicação.

## Deploy no Heroku

https://eli-gestao-clientes.herokuapp.com/ | https://git.heroku.com/eli-gestao-clientes.git