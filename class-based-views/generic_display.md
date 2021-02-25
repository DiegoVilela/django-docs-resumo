# Views genéricas embutidas

1. [ListView](#listview)
   * [Method Resolution Order de ListView](#method-resolution-order-de-listview)
2. [DetailView](#DetailView)
   * [Method Resolution Order de DetailView](#method-resolution-order-de-detailview)


## ListView
```python
# views.py
from django.views.generic import ListView
from meuapp.models import Produto

class ProdutoList(ListView):
    model = Produto
```

```python
# urls.py
from django.urls import path
from meuapp.views import ProdutoList

urlpatterns = [
    path('produtos/', ProdutoList.as_view(), name='produto-list'),
]
```

```html
<!-- /meuapp/templates/meuapp/produto_list.html -->
{% extends "base.html" %}

{% block content %}
    <h2>Produtos</h2>
    <ul>
        {% for produto in object_list %}
            <li>{{ produto.descricao }}</li>
        {% endfor %}
    </ul>
{% endblock %}
```

### Method Resolution Order de ListView

    django.views.generic.list.MultipleObjectTemplateResponseMixin
    django.views.generic.base.TemplateResponseMixin
    django.views.generic.list.BaseListView
    django.views.generic.list.MultipleObjectMixin
    django.views.generic.base.View

Sobrescrevendo atributos e métodos:
```python
from django.shortcuts import get_object_or_404
from django.views.generic.list import ListView

from meuapp.models import Produto, Fabricante

class FabricanteProdutoList(ListView):
    """
    Lista os produtos de um determinado fabricante.

    model = Produto
    É uma forma abreviada de:

    queryset = Produto.objects.all()
    Aqui temos a opção de personalizar a busca.
    """

    paginate_by = 50
    context_object_name = 'lista_de_produtos_por_fabricante'
    template_name = 'app/produtos_por_fabricante.html'

    def get_queryset(self):
        # Objeto único
        self.fabricante = get_object_or_404(Fabricante, nome=self.kwargs['fabricante'])

        # Model utilizado em ListView
        return Produto.objects.filter(fabricante=self.fabricante)

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['fabricante'] = self.fabricante
        context['conteudo_extra'] = 'conteúdo extra para o template'
        return context
```
```python
# urls.py
from django.urls import path
from meuapp.views import FabricanteProdutoList

urlpatterns = [
    path('produtos/<fabricante>/', PublisherBookList.as_view()),
]
```

## DetailView

Por padrão, o objeto estará disponível no template `templates/meuapp/produto_detail.html` na variável `object` assim como
em uma variável definida pelo nome do model em minúsculo.

### Method Resolution Order de DetailView

    django.views.generic.detail.SingleObjectTemplateResponseMixin
    django.views.generic.base.TemplateResponseMixin
    django.views.generic.detail.BaseDetailView
    django.views.generic.detail.SingleObjectMixin
    django.views.generic.base.View


Sobrescrevendo atributos e métodos:

```python
from django.utils import timezone
from django.views.generic import DetailView
from meuapp.models import Produto

class ProdutoDetailView(DetailView):

    queryset = Produto.objects.all()

    def get_object(self):
        obj = super().get_object()
        # Grava a última vez em este objeto foi acessado
        obj.acessado_em = timezone.now()
        obj.save()
        return obj
```
