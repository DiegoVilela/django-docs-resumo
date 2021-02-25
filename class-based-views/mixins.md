# Mixins

1. [Usando SingleObjectMixin com View](#usando-singleobjectmixin-com-view)
2. [Usando SingleObjectMixin com ListView](#usando-singleobjectmixin-com-listview)

As views baseadas em classes embutidas no Django contêm várias funcionalidades
que podem ser utilizadas separadamente.

`TemplateResponseMixin` fornece o método `render_to_response()`, que é chamado
automaticamente por `DetailView` e `ListView`, por exemplo. Da mesma forma,
`ContextMixin` fornece `get_context_data()` e o atributo
[`extra_context`](https://docs.djangoproject.com/en/3.1/ref/class-based-views/mixins-simple/#django.views.generic.base.ContextMixin.extra_context).

Por exemplo, `DetailView` depende de `SingleObjectMixin` para obter `get_object()`, `get_query()`
e sobrescreve `get_context_data()` (implementado em `ContextMixin`).
`ListView` depende de `MultipleObjectMixin` para obter `get_queryset()`, `paginate_queryset()`
e sobrescreve `get_context_data()` para incluir as variáveis de contexto apropriadas
para paginação.


## Usando SingleObjectMixin com View

Para escrever uma class-based view que responda apenas requisições **POST**,
podemos estender a classe `View` e escrever um método `post()`. O método `get()`
é implementado em um nível abaixo, em `BaseDetailView` ou `BaseListView`, por exemplo).

`SingleObjectMixin` adiciona a capacidade de processar um objeto em particular, identificado
pela URL.

```python
# views.py
from django.http import HttpResponseForbidden, HttpResponseRedirect
from django.urls import reverse
from django.views import View
from django.views.generic.detail import SingleObjectMixin
from books.models import Author

class SalvarInteresse(SingleObjectMixin, View):
    """Salva o interesse do user logado em um produto"""
    model = Produto

    def post(self, request, *args, **kwargs):
        if not request.user.is_authenticated:
            return HttpResponseForbidden()

        # Obtém o produto alvo do interesse
        self.object = self.get_object()
        # supondo no model Produto
        # interessados = models.ManyToMany(User)
        self.object.interessados.add(self.request.user)

        return HttpResponseRedirect(reverse('produto-detail', kwargs={'pk': self.object.pk}))
```

```python
# urls.py
from django.urls import path
from books.views import SalvarInteresse

urlpatterns = [
    #...
    path('produto/<int:pk>/interesse/', SalvarInteresse.as_view(), name='produto-interesse'),
]
```


## Usando SingleObjectMixin com ListView

Para paginar uma lista de objetos que estão lincados via chave estrangeira a outro objeto,
uma lista paginada de produtos por determinado fabricante, por exemplo, podemos combinar
`SingleObjectMixin` com `ListView`:

```python
from django.views.generic import ListView
from django.views.generic.detail import SingleObjectMixin
from meuapp.models import Fabricante

class FabricanteDetail(SingleObjectMixin, ListView):
    paginate_by = 2
    template_name = "meuapp/fabricante_detail.html"

    def get(self, request, *args, **kwargs):
        self.object = self.get_object(queryset=Fabricante.objects.all())
        return super().get(request, *args, **kwargs)

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['fabricante'] = self.object
        return context

    def get_queryset(self):
        return self.object.produto_set.all()
```

Também é possível implementar o [mesmo comportamento apenas com `ListView`](./generic_display.md#method-resolution-order-de-listview).
