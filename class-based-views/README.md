# Class-based views

1. [Introdução](intro.md)
2. [Views genéricas embutidas](generic_display.md)
3. [Processamento de formulários](generic_editing.md)
4. [Mixins](mixins.md)

## Exemplos básicos 

### Direto de URLconf

```python
# url.py
from django.urls import path
from django.views.generic.base import TemplateView, RedirectView

urlpatterns = [
    path(
        'sobre-nos/',
        TemplateView.as_view(template_name='sobre.html'),
        name='sobre',
    ),
    path(
        'ir-para-fonte/',
        RedirectView.as_view(url='https://djangoproject.com'),
        name='fonte'
    ),
]
```

Fonte:

[`class django.views.generic.base.TemplateView`](https://docs.djangoproject.com/en/3.1/ref/class-based-views/base/#templateview)

[`class django.views.generic.base.RedirectView`](https://docs.djangoproject.com/en/3.1/ref/class-based-views/base/#redirectview)

___

### Respondendo a outros métodos HTTP

Como todas as _class-based views_ estendem a classe [`django.views.generic.base.View`](https://docs.djangoproject.com/en/3.1/ref/class-based-views/base/#view),
os métodos HTTP definidos em [`http_method_names`](https://docs.djangoproject.com/en/3.1/ref/class-based-views/base/#django.views.generic.base.View.http_method_names) podem ser implementados nas subclasses.

```python
http_method_names = ['get', 'post', 'put', 'patch', 'delete', 'head', 'options', 'trace']
```

Exemplo:

```python
# view.py
from django.http import HttpResponse
from django.views.generic import ListView
from meuapp.models import Produto

class ProdutoListView(ListView):
    model = Produto

    def head(self, *args, **kwargs):
        ultimo_produto_cadastrado = self.get_queryset().latest('criado_em')
        response = HttpResponse()
        # RFC 1123 date format
        response['Last-Modified'] = ultimo_produto_cadastrado.criado_em.strftime('%a, %d %b %Y %H:%M:%S GMT')
        return response
```

```python
# url.py
from django.urls import path
from meuapp.views import ProdutoListView

urlpatterns = [
    path('produtos/', ProdutoListView.as_view()),
]
```

Se a view for acessada via **GET** request, uma lista de objetos é retornada em resposta (com o template `produto_list.html`).
Mas se o cliente faz uma **HEAD** request, a resposta terá um body vazio e o header `Last-Modified` indicará quando o produto
mas recente foi cadastrado. Baseado nessa informação, o cliente pode ou não requisitar a lista de produtos completa.

Fonte:

[Supporting other HTTP methods](https://docs.djangoproject.com/en/3.1/topics/class-based-views/#supporting-other-http-methods)


