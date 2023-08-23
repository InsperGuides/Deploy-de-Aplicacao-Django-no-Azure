| **Lab**         |     **Data**   |
|  :----:     |  :----:    |
| Laboratório Ágil     | 2023       |

# Deploy-de-Aplicacao-Django-no-Azure
Guia de como ajustar sua aplicação para o deploy dentro do Microsoft Azure
===
## :warning: É RECOMENDADO SEGUIR ESTE GUIA EM UMA VERSÃO NOVA DA SUA APLICAÇÃO NO GITHUB PARA VOCÊ NÃO QUEBRAR SUA VERSÃO DE PRODUÇÃO(EM CASO DE DÚVIDAS, PESQUISE COMO CRIAR UM FORK DO SEU REPOSITÓRIO). <br /><br />Passo 1: Criando seu Web App no Azure:

### 1.a: Login:
Para começarmos, iremos fazer os ajustes necessários dentro do Microsoft Azure com sua conta. Para isso, acesse o link abaixo e **logue com sua conta do Insper no portal**. <br> https://portal.azure.com/#home

### 1.b: Criando o Web App:
Após acessar o portal da Azure, procure pela opção **Create a Resource**, e dentro desta opção, pesquise pela opção **Web App + Database**:

![image](https://github.com/InsperGuides/Deploy-de-Aplicacao-Django-no-Azure/assets/18387737/a04840f5-774f-46ef-b0ea-d5331ac8180b)

Em seguida iremos configurar o Web App com as informações abaixo:

![image](https://github.com/InsperGuides/Deploy-de-Aplicacao-Django-no-Azure/assets/18387737/08b440c3-0c24-4131-86a2-47f5f634f0a0)

**1 - Subscription: Selecione sua assinatura **Azure for Students**;**

**2 - Resource Group: Um novo grupo será criado automaticamente com o nome do seu Web App por padrão;**

**3 - Region: Mantenha a região mais próxima a você, que provavelmente já estará por padrão na configuração;**

**4 - Name: Nome da sua aplicação(Este nome deve ser único e irá definir a URL do seu App,pense com carinho;**

**5 - Runtime Stack: O Ambiente da sua Aplicação. Selecione o Python 3.11 ou o mais atual;**

**6 - Engine: Selecione o POSTGRESQL - Flexible Server;**

**7 -Server Name: pode manter o padrão criado pelo Azure;**

**8 - Database Name: pode manter o padrão criado pelo Azure;**

**9 - Hosting: Selecione o plano Basic.**

Após ajustar todas as configurações, clique no botão **Review + Create**, e após confirmar que todos os dados estão corretos, clique em **Create** para finalizar a configuração. Em seguida, o Azure irá iniciar o deploy da sua aplicação, e este processo pode levar vários minutos para ser finalizado.

Quando o processo de Deploy concluir, você tera um novo Resource Group no seu Azure similar como o abaixo. Para os próximos passos, iremos trabalhar dentro do **App Service** dentro deste grupo.

![image](https://github.com/InsperGuides/Deploy-de-Aplicacao-Django-no-Azure/assets/18387737/0244be5f-f630-489e-b129-33d82c789ea9)


### 1.c: Configurando o App Service

Agora que temos nosso Web App criado dentro do Azure, precisamos configurar algumas partes dele para que o nosso projeto Django funcione corretamente. Primeiro, vamos configurar as **Application Settings** para podermos conectar o banco de dados do PostgreSQL ao projeto, assim como fizemos com os containers anteriormente.

Para acessar, procura no painel esquerdo, na área de **Settings**, pela opção **Configuration**. Você deverá encontrar uma tela similar a abaixo:

![image](https://github.com/InsperGuides/Deploy-de-Aplicacao-Django-no-Azure/assets/18387737/a9d0e81e-9d67-4218-b2e3-d108a60c63f0)

Provavelmente você terá apenas uma Application Setting criada no seu painel, como nome de **AZURE_POSTGRESQL_CONNECTIONSTRING**. Esta configuração possui uma string contendo todas as informações necessárias para a aplicação conectar à base de dados do Azure, que será enviada para a aplicação como uma **Variável de Ambiente**. Porém, tratar esta string dentro do Django pode ser um tanto chato, então iremos por um caminho diferente, criando diversas Variáveis Ambiente para cada dado que precisamos utilizar na nossa configuração.

Agora, o que devemos fazer, é selecionar a opção **Advanced Edit**, que nos irá apresentar um arquivo JSON para modificar. Copie o código abaixo no seu JSON após a parte da Connection String existente:

⚠️ **Não apague sua Connection String ainda, iremos utilizar informações dela primeiro**

```
,
{
    "name": "ALLOWED_HOSTS",
    "value": "127.0.0.1 [::1] <link do App>",
    "slotSetting": true
  },
  {
    "name": "CSRF_TRUSTED_ORIGINS",
    "value": "https://<link do App>",
    "slotSetting": true
  },
  {
    "name": "DBHOST",
    "value": "<host>",
    "slotSetting": true
  },
  {
    "name": "DBNAME",
    "value": "<dbname>",
    "slotSetting": true
  },
  {
    "name": "DBPASS",
    "value": "<password>",
    "slotSetting": true
  },
  {
    "name": "DBUSER",
    "value": "<user>",
    "slotSetting": true
  },
  {
    "name": "DEGUB",
    "value": "1",
    "slotSetting": true
  },
  {
    "name": "SECRET_KEY",
    "value": "w7a8a@lj8nax7tem0caa2f2rjm2ahsascyf83sa5alyv68vea",
    "slotSetting": true
  },
  {
    "name": "SECURE_SSL_REDIRECT",
    "value": "0",
    "slotSetting": true
  }
```
Após copiar o código acima, você irá substituir os campos temporários `<exemplo>` pelas informações dentro da Connection String já criada pelo Azure. Depois deste passo, clique em **Ok** e, em seguida, clique em **Save** no topo da tela de Application Settings para finalizar a edição. Você deverá ter todas as variáveis de ambiente agora como a imagem acima estava mostrando.

**Obs: o link do seu App se encontra na parte de *Overview* da página**.

## Passo 2: Ajustando a Aplicação

Agora que temos nossas variáveis ambiente criadas, temos que ajustar o `settings.py` da aplicação para poder se conectar à base de dados online do Azure. Como dito anteriormente, ***é recomendado você ajustar esta parte em um repositório isolado do github da sua aplicação, para não impactar seu ambiente de produção.***

Abra seu `settings.py`, e substitua o topo do código, abaixo do `import dj_database_url` pelo abaixo:
```
import os
from pathlib import Path

from dotenv import load_dotenv

# Build paths inside the project like this: BASE_DIR / 'subdir'.
BASE_DIR = Path(__file__).resolve().parent.parent

load_dotenv(BASE_DIR / '.env')
```
Em seguida, abaixo da região adicionada, iremos definir as variáveis de ambiente que ajustamos no Azure para nossa aplicação reconhecer. Utilize o código abaixo:
```
# core/settings.py

# SECURITY WARNING: keep the secret key used in production secret!
SECRET_KEY = os.getenv('SECRET_KEY')

# SECURITY WARNING: don't run with debug turned on in production!
DEBUG = os.getenv('DEBUG', '0').lower() in ['true', 't', '1']

ALLOWED_HOSTS = os.getenv('ALLOWED_HOSTS').split(' ')
CSRF_TRUSTED_ORIGINS = os.getenv('CSRF_TRUSTED_ORIGINS').split(' ')

SECURE_SSL_REDIRECT = \
    os.getenv('SECURE_SSL_REDIRECT', '0').lower() in ['true', 't', '1']
if SECURE_SSL_REDIRECT:
    SECURE_PROXY_SSL_HEADER = ('HTTP_X_FORWARDED_PROTO', 'https')
```

Por fim, iremos agora ajustar nossa conexão à base de dados com nossas variáveis criadas. Encontre a parte de **DATABASES**, e troque pelo código abaixo:
```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.environ.get('DBNAME'),
        'HOST': os.environ.get('DBHOST'),
        'USER': os.environ.get('DBUSER'),
        'PASSWORD': os.environ.get('DBPASS'),
        'OPTIONS': {'sslmode': 'require'},
    }
}
```

Muito Bem, agora nosso settings está ajustado. Porém, como podem ter notado, estamos utilizando uma nova biblioteca chamada `dotenv`. Esta biblioteca é responsável em carregar as variáveis ambiente na nossa aplicação, então devemos adicionar a mesma no nosso arquivo `requirements.txt`:

```
python-dotenv==1.0.0
```

Além disso, confirme se as bilbiotecas `gunicorn` e `psycopg-2-binary` também ja estão adicionadas:

```
gunicorn==20.1.0
psycopg2-binary==2.9.5
```

Pronto, agora vamos ao último passo: Realizar o Deploy da nossa aplicação para o Web App!

## Passo 3: Deploy da Aplicação
Voltando para o Azure, iremos agora para a seção **Deployment Center** do App Service. Iremos configurar o deployment para pegar os arquivos da aplicação no GitHub. Temos aqui duas opções possíveis. Caso seu código esteja em um repositório que pertence a sua conta, Selecione no campo Source a opção ***GitHub***, e acesse com sua conta conforme for guiado:

![image](https://github.com/InsperGuides/Deploy-de-Aplicacao-Django-no-Azure/assets/18387737/7547af3a-2c6d-42ff-9193-6c30b658aec2)

Em seguida, preencha os campos abaixo com o caminho onde seu repositório está salvo, e clique em **Save** no topo da tela para confirmar sua conexão.

![image](https://github.com/InsperGuides/Deploy-de-Aplicacao-Django-no-Azure/assets/18387737/deecdf43-d954-4155-a1c1-87e9a491f1b5)

Agora, caso você esteja em um repositório externo, utilize a opção ***External Git*** e, seguida, preencha os campos abaixo com as informações do repositório que irá utilizar. 
⚠️ **Este repositório deve estar como público no GitHub para poder ser utilizado sem Autenticação do dono**
Por fim, salve suas informações com o botão no topo da tela.

![image](https://github.com/InsperGuides/Deploy-de-Aplicacao-Django-no-Azure/assets/18387737/c871a6bd-664c-482f-ba43-23dd63a5ca0c)

Agora que sua conexão está configurada, basta esperar o deployment concluir, o que pode levar alguns minutos, e então pode testar sua conexão através do link do seu Web App em ***Overview***.

Sua aplicação irá abrir, porém irá acusar que não encontrou a base de dados. Isto é normal, pois embora a conexão está criada, nós não realizamos o migrate da  `models.py` da aplicação no Web App. Para tal, acesse a opção de ***SSH*** no menu esquerdo do Web App, e clique em **Go**, para abrir o prompt de comando do nosso servidor.

Dentro do prompt, basta utilizar os comandos abaixo para migrar a base de dados e adicionar o superusuário, assim como foi feito durante o desenvolvimento da aplicação:

```
python manage.py migrate
python manage.py createsuperuser
```
Após criar o superuser, tente novamente acessar sua aplicação,e tudo deverá funcionar corretamente.
