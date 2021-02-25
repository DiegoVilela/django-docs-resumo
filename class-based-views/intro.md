# Introdução a class-based views

1. [Configuração de atributos da classe](#configuração-de-atributos-da-classe)
2. [Mixins](#Mixins)
3. [Processamento de formulários](#processamento-de-formulários)
4. [Decoradores](#decoradores)
    * [Decorando no URLconf](#decorando-no-urlconf)
    * [Decorando a Classe](#decorando-a-classe)


Algumas vantagens em relação a function-based views:
* A organização do código pode ser feita por métodos HTTP específicos aos
invés de condicionais.
* Técnicas de orientação a objetos tais como mixins (herança múltipla) podem
ser usadas para fator o código em componentes reutilizáveis.


## Configuração de atributos da classe
1. Maneira padrão do Python criando subclasses e substituindo atributos e
   métodos na subclasse:
```python
from django.http import HttpResponse
from django.views import View

class MinhaView(View):
    def get(self, request):
        saudacao = "Olá, como vai?"

        def get(self, request):
            return HttpResponse(self.saudacao)
```

2. Passar argumentos nomedos para o método [`as_view()`](https://docs.djangoproject.com/en/3.1/ref/class-based-views/base/#django.views.generic.base.View.as_view)
```python
urlpatterns = [
    path('', MinhaView.as_view(saudacao="Olá")),
]
```

O URL resolver do Django espera enviar a requisição e os argumentos associados
para uma função, não para uma classe, por isso as views baseadas em classes têm um
método de classe [`as_view()`](https://docs.djangoproject.com/en/3.1/ref/class-based-views/base/#django.views.generic.base.View.as_view)
que retorna uma função que pode ser chamada quando uma requisição
é feita para uma URL que corresponda a um padrão associado. A função cria uma instância
da classe, chama `setup()` para inicializar seus atributos e então chama seu método
`dispatch()`, que por sua vez verifica qual o método HTTP utilizado na requisição e a
transmite para o método correspondente, se algum estiver definido, ou lança a excecão
`HttpResponseNotAllowed`.

## Mixins
Uma forma de herança múltipla onde métodos e atributos de múltiplas classes são combinados
em subclasses.

Exemplo:

[`class django.views.generic.base.TemplateResponseMixin`](https://docs.djangoproject.com/en/3.1/ref/class-based-views/mixins-simple/#templateresponsemixin)

Utilizado em todas as class-based views, retorna um objeto `TemplateResponse` com base no atributo `template_name`.


## Processamento de formulários
```python
from django.http import HttpResponseRedirect
from django.shortcuts import render
from django.views import View

from .forms import MeuForm

class MeuFormView(View):
    form_class = MeuForm
    initial = {'key': 'value'}
    template_name = 'form_template.html'

    def get(self, request, *args, **kwargs):
        form = self.form_class(initial=self.initial)
        return render(request, self.template_name, {'form': form})

    def post(self, request, *args, **kwargs):
        form = self.form_class(request.POST)
        if form.is_valid():
            # <process form cleaned data>
            return HttpResponseRedirect('/success/')

        return render(request, self.template_name, {'form': form})
```

___


## Decoradores

### Decorando no URLconf

```python
from django.contrib.auth.decorators import login_required, permission_required
from django.views.generic import TemplateView

from meuapp.views import ProdutoCreate

urlpatterns = [
    path('colaboradores/', login_required(TemplateView.as_view(template_name="colaboradores.html"))),
    path('produto/novo/', permission_required('meuapp.add_produto')(ProdutoCreate.as_view())),
]
```

Essa abordagem aplica o decorador por instância.  

### Decorando a Classe

Para que cada instância seja decorada, o decorador deve ser aplicado ao método `dispatch()` da classe, que é herdado de
`class django.views.generic.base.View`.


`distatch()` aceita requisições mais seus argumentos e retorna uma resposta HTTP.
A implementação padrão inspecionará o método HTTP e tentará delegá-lo ao método correspondente na classe;
**GET** será delegado a `get()`, **POST** para `post()` e assim por diante. Por padrão,
uma requisição **HEAD** será delegada a `get()`, mas se for necessário um processamento diferente, `head()`pode
ser sobrescrito.

```python
from django.contrib.auth.decorators import never_cache, login_required
from django.utils.decorators import method_decorator
from django.views.generic import TemplateView

decoradores = [never_cache, login_required]

@method_decorator(decoradores, name='dispatch')
class ConfidencialView(TemplateView):
    template_name = 'confidencial.html'
```
