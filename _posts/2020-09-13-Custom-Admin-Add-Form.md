---
layout: post
title: Custom add/change from in django admin
---


In `admin.ModelAdmin` class, there is `form` field that we can customize Admin change and creation form.

For example:
```py
# forms.py
from django import forms
class CustomFooForm(forms.ModelForm):
    class Meta:
        model = Foo
        fields = [
            'a',
            'b',
        ]
        widgets = {
            'a': forms.TextInput(),
        }

# admin.py
from django.contrib import admin
from app.forms import CustomFooForm

@admin.register(Foo)
class FooAdmin(admin.ModelAdmin):
    form = CustomFooForm
```

But, how to use custom form for add Foo and another custom form for change foo? The solution is overriding `get_form` method!

Here is it:
```py

# admin.py
from django.contrib import admin
from app.forms import CustomFooForm

@admin.register(Foo)
class FooAdmin(admin.ModelAdmin):
    form = CustomFooForm
    add_form = CustomAddFooForm # It is not a native django field. I created this field and use it in get_form method.

    def get_form(self, request, obj=None, **kwargs):
        """
        Use special form during foo creation
        """
        defaults = {}
        if obj is None:
            defaults['form'] = self.add_form
        defaults.update(kwargs)
        return super().get_form(request, obj, **defaults)

```

**Note:** This is a best practice. Django also uses this idea for custom user add form: [See django's source code](https://github.com/django/django/blob/0004daa/django/contrib/auth/admin.py#L72-L80)