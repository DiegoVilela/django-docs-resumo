# Formulários a partir de models

```python
# forms.py
from django.forms import ModelForm, Textarea
from meuapp.models import Produto

class ProdutoForm(ModelForm):
    class Meta:
        model = Produto
        fields = ('nome', 'descricao', 'quantidade')
        widgets = {
            'descricao': Textarea(attrs={'cols': 80, 'rows': 20}),
        }
```

## modelform_factory

```python
from django.forms import modelform_factory

ProdutoForm = modelform_factory(Produto, fields=("nome", "descricao"))
```

## Inline formsets

```python
from django.forms import inlineformset_factory

ProdutoFormSet = inlineformset_factory(Produto, Patente, fields=('registro',))
produto = Produto.objects.get(pk=1)
formset = ProdutoFormSet(instance=produto)
```