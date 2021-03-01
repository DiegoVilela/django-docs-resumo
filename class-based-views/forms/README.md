# Formulários

1. [Formsets](forms/formsets.md)
2. [ModelForm](forms/modelform.md)

## Construindo um formulário

```python
# forms.py
from django import forms

class NomeForm(forms.Form):
    seu_nome = forms.CharField(label='Seu nome', max_length=100)

```

```python
# view.py
from django.http import HttpResponseRedirect
from django.shortcuts import render

from meuapp.forms import NomeForm

def get_name(request):
    if request.method == 'POST':
        # cria um formulário com os dados postados
        form = NomeForm(request.POST)
        if form.is_valid():
            # processa os dados em form.cleaned_data conforme necessário
            # ...
            # redireciona
            return HttpResponseRedirect('/obrigado/')

    # se um GET (ou qualquer outro método) cria um formulário vazio
    else:
        form = NomeForm()

    return render(request, 'nome.html', {'form': form})
```

```html
<!-- nome.html -->
<form action="/seu-nome/" method="post">
    {% csrf_token %}
    {{ form }}
    <input type="submit" value="Submit">
</form>
```

## Renderizando manualmente

```html
{{ form.non_field_errors }}
<div class="fieldWrapper">
    {{ form.seu_nome.errors }}
    {{ form.seu_nome.label_tag }}
    {{ form.seu_nome }}
</div>
```

## Iterando sobre os campos do formulário
```html
{% for campo in form %}
    <div class="fieldWrapper">
        {{ campo.errors }}
        {{ campo.label_tag }} {{ campo }}
        {% if campo.help_text %}
        <p class="help">{{ campo.help_text|safe }}</p>
        {% endif %}
    </div>
{% endfor %}
```

## Templates reutilizáveis

```html
# No template:
{% include "form_snippet.html" %}

# No form_snippet.html:
{% for campo in form %}
    <div class="fieldWrapper">
        {{ campo.errors }}
        {{ campo.label_tag }} {{ campo }}
    </div>
{% endfor %}
```

Se o object passado ao template tem um nome diferente:

```html
{% include "form_snippet.html" with form=comentario_form %}
```

Fonte:

[`class django.forms.Form`](https://docs.djangoproject.com/en/3.1/ref/forms/api/#django.forms.Form)
