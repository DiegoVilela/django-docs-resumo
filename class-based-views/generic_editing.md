# Processamento de formulários

1. [FormView](#formview)
    * [Method Resolution Order de FormView](#method-resolution-order-de-formview)
2. [Usando models](#usando-models)


## FormView

O processamento de formulários tem geralmente 3 passos:
* GET inicial com o form vazio ou populado
* POST com dados inválidos (retorna o form com os erros)
* POST com dados válidos (processa os dados e redireciona)


### Method Resolution Order de FormView

    django.views.generic.base.TemplateResponseMixin
    django.views.generic.edit.BaseFormView
    django.views.generic.edit.FormMixin
    django.views.generic.edit.ProcessFormView
    django.views.generic.base.View

Exemplo básico:

```python
# forms.py
from django import forms

class ContatoForm(forms.Form):
    nome = forms.CharField()
    mensagem = forms.CharField(widget=forms.Textarea)

    def enviar_email(self):
        # enviar e-mail usando o dicionário self.cleaned_data
        pass
```

```python
# views.py
from meuapp.forms import ContatoForm
from django.views.generic.edit import FormView

class ContatoView(FormView):
    template_name = 'contato.html'
    form_class = ContatoForm
    success_url = '/obrigado/'

    def form_valid(self, form):
        # Método chamado quando dados válidos são POSTados.
        # A implementação padrão redireciona para sucess_url
        form.enviar_email()
        return super().form_valid(form)
```

## Usando models

```python
# models.py
from django.db import models
from django.urls import reverse

class Produto(models.Model):
    nome = models.CharField(max_length=200)
    criado_por = models.ForeignKey(User, on_delete=models.CASCADE)

    def get_absolute_url(self):
        return reverse('produto-detail', kwargs={'pk': self.pk})
```

```python
# views.py
from django.urls import reverse_lazy
from django.views.generic.edit import CreateView, DeleteView, UpdateView
from meuapp.models import Produto

class ProdutoCreate(CreateView):
    model = Produto
    fields = ['nome']

    def form_valid(self, form):
        form.instance.criado_por = self.request.user
        return super().form_valid(form)

class ProdutoUpdate(UpdateView):
    model = Produto
    fields = ['nome']

class ProdutoDelete(DeleteView):
    model = Produto
    # Como as urls não são carregados quando o arquivo é importado,
    # temos que usar reverse_lazy() ao invés de reverse().
    success_url = reverse_lazy('produto-list')
```

Essas views genéricas criam um [`ModelForm`](https://docs.djangoproject.com/en/3.1/topics/forms/modelforms/#modelform) automaticamente:
* Se o atributo `model` for definido, este é utilizado.
* A classe do objeto retornado por `get_object()` será utilizada se aquele método for sobrescrito.
* Se o atributo `queryset` for definido, seu model será utilizado. 

```python
# urls.py
from django.urls import path
from meuapp.views import ProdutoCreate, ProdutoDelete, ProdutoUpdate

urlpatterns = [
    # ...
    path('produto/criar/', ProdutoCreate.as_view(), name='produto-criar'),
    path('produto/<int:pk>/', ProdutoUpdate.as_view(), name='produto-editar'),
    path('produto/<int:pk>/apagar/', ProdutoDelete.as_view(), name='produto-apagar'),
]
```
Essas views herdam `SingleObjectTemplateResponseMixin`, que usa `template_name_suffix` para construir o `template_name`
baseado no model.

Nesse exemplo:
   * `CreateView` e `UpdateView` usam `meuapp/produto_form.html`
   * `DeleteView` usa `meuapp/produto_confirm_delete.html`

Para separar os templates de `CreateView` e `UpdateView`, podemos sobrescrever o
atributo `template_name` ou `template_name_suffix` na classe na view.
