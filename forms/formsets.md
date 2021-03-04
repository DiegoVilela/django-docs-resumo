# Formsets

```python
# views.py
from django.forms import formset_factory
from django.shortcuts import render
from meuapp.forms import ProdutoForm

def produtos_admin(request):
    ProdutoFormSet = formset_factory(ProdutoForm, extra=10)
    if request.method == 'POST':
        formset = ProdutoFormSet(request.POST)
        if formset.is_valid():
            # processar dados em formset.cleaned_data
            pass
    else:
        formset = ProdutoFormSet()
    return render(request, 'produtos-admin.html', {'formset': formset})
```

```html
<form method="post">
    <table>
        {{ formset }}
    </table>
    <input type="submit" value="Save">
</form>
```

Nesse exemplo, 10 produtos poderão ser criados de uma só vez.