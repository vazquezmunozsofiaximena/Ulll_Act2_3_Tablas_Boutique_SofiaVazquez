
# Guía paso a paso (Primera parte) — Proyecto **Boutique** (Django, Python, VS Code)

---

# 1 — Crear la carpeta del proyecto

Abre tu terminal (PowerShell / CMD / Terminal) y ejecuta:

```bash
mkdir UIII_Boutique_0390
cd UIII_Boutique_0390
```

---

# 2 — Abrir VS Code sobre la carpeta

Si tienes `code` en PATH:

```bash
code .
```

(esto abre VS Code en `UIII_Boutique_0390`)

---

# 3 — Abrir terminal en VS Code

En VS Code: menú **Terminal → New Terminal** (o `Ctrl + ñ`).

---

# 4 — Crear carpeta/entorno virtual `.venv` desde la terminal de VS Code

```bash
python -m venv .venv
```

---

# 5 — Activar el entorno virtual

**PowerShell (Windows):**

```powershell
.\.venv\Scripts\Activate.ps1
```

**CMD (Windows):**

```cmd
.\.venv\Scripts\activate
```

**macOS / Linux:**

```bash
source .venv/bin/activate
```

Cuando esté activo verás `(.venv)` al inicio de la línea.

---

# 6 — Activar intérprete de Python en VS Code

1. Presiona `Ctrl + Shift + P`
2. Escribe **Python: Select Interpreter**
3. Selecciona el que está dentro de `.venv`

---

# 7 — Instalar Django

```bash
pip install --upgrade pip
pip install django
```

Verifica versión:

```bash
python -m django --version
```

---

# 8 — Crear proyecto **backend_Boutique** sin duplicar carpeta

```bash
django-admin startproject backend_Boutique .
```

---

# 9 — Ejecutar servidor en el puerto **8390**

```bash
python manage.py runserver 8390
```

Abre en el navegador:
`http://127.0.0.1:8390/`

---

# 10 — Copiar y pegar el link en el navegador

Abre tu navegador y pega:
`http://127.0.0.1:8390/`

---

# 11 — Crear aplicación `app_Boutique`

```bash
python manage.py startapp app_Boutique
```

---

# 12 — Código del archivo `models.py`

Abre `app_Boutique/models.py` y pega:

```python
from django.db import models

# ==========================================
# MODELO: CATEGORIA
# ==========================================
class Categoria(models.Model):
    idcategoria = models.AutoField(primary_key=True, unique=True)
    nombre = models.CharField(max_length=45)
    descripcion = models.TextField()
    fechacreacion = models.DateField()
    activo = models.BooleanField(default=True)
    tipo = models.CharField(max_length=20)
    slug = models.SlugField(max_length=100, unique=True)

    def __str__(self):
        return self.nombre


# ==========================================
# MODELO: PRODUCTO
# ==========================================
class Producto(models.Model):
    idproducto = models.AutoField(primary_key=True, unique=True)
    nombre = models.CharField(max_length=100)
    descripcion = models.TextField()
    precio = models.DecimalField(max_digits=10, decimal_places=2)
    stock = models.IntegerField()
    fechaagregado = models.DateTimeField(auto_now_add=True)
    imagen = models.CharField(max_length=255, blank=True, null=True)
    idcategoria2 = models.ForeignKey(
        Categoria,
        on_delete=models.CASCADE,
        db_column='idcategoria2',
        related_name='productos'
    )

    def __str__(self):
        return self.nombre


# ==========================================
# MODELO: PROVEEDOR
# ==========================================
class Proveedor(models.Model):
    idproveedor = models.AutoField(primary_key=True, unique=True)
    nombre = models.CharField(max_length=100)
    email = models.EmailField(max_length=100)
    telefono = models.CharField(max_length=20)
    direccion = models.CharField(max_length=150)
    ciudad = models.CharField(max_length=100)
    pais = models.CharField(max_length=100)
    id_producto2 = models.ForeignKey(
        Producto,
        on_delete=models.CASCADE,
        db_column='id_producto2',
        related_name='proveedores'
    )

    def __str__(self):
        return self.nombre
```

---

# 12.5 — Realizar migraciones

```bash
python manage.py makemigrations app_Boutique
python manage.py migrate
```

---

# 13 — Primero trabajamos con el modelo: **CATEGORIA**

El CRUD será primero para **Categoría**.
Los modelos de **Producto** y **Proveedor** quedan listos para más adelante.

---

# 14 — `views.py` de `app_Boutique`

Abre `app_Boutique/views.py` y pega:

```python
from django.shortcuts import render, redirect, get_object_or_404
from .models import Categoria

def inicio_boutique(request):
    total_categorias = Categoria.objects.count()
    categorias = Categoria.objects.all().order_by('-fechacreacion')[:5]
    return render(request, 'inicio.html', {'total_categorias': total_categorias, 'categorias': categorias})

def agregar_categoria(request):
    if request.method == 'POST':
        nombre = request.POST.get('nombre')
        descripcion = request.POST.get('descripcion')
        fechacreacion = request.POST.get('fechacreacion')
        activo = True if request.POST.get('activo') == 'on' else False
        tipo = request.POST.get('tipo')
        slug = request.POST.get('slug')

        Categoria.objects.create(
            nombre=nombre,
            descripcion=descripcion,
            fechacreacion=fechacreacion,
            activo=activo,
            tipo=tipo,
            slug=slug
        )
        return redirect('ver_categorias')
    return render(request, 'categoria/agregar_categoria.html')

def ver_categorias(request):
    categorias = Categoria.objects.all().order_by('nombre')
    return render(request, 'categoria/ver_categorias.html', {'categorias': categorias})

def actualizar_categoria(request, idcategoria):
    categoria = get_object_or_404(Categoria, idcategoria=idcategoria)
    return render(request, 'categoria/actualizar_categoria.html', {'categoria': categoria})

def realizar_actualizacion_categoria(request, idcategoria):
    categoria = get_object_or_404(Categoria, idcategoria=idcategoria)
    if request.method == 'POST':
        categoria.nombre = request.POST.get('nombre')
        categoria.descripcion = request.POST.get('descripcion')
        categoria.fechacreacion = request.POST.get('fechacreacion')
        categoria.activo = True if request.POST.get('activo') == 'on' else False
        categoria.tipo = request.POST.get('tipo')
        categoria.slug = request.POST.get('slug')
        categoria.save()
        return redirect('ver_categorias')
    return redirect('ver_categorias')

def borrar_categoria(request, idcategoria):
    categoria = get_object_or_404(Categoria, idcategoria=idcategoria)
    if request.method == 'POST':
        categoria.delete()
        return redirect('ver_categorias')
    return render(request, 'categoria/borrar_categoria.html', {'categoria': categoria})
```

---

# 24 — `urls.py` en `app_Boutique`

Crea el archivo `app_Boutique/urls.py`:

```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.inicio_boutique, name='inicio'),
    path('categoria/agregar/', views.agregar_categoria, name='agregar_categoria'),
    path('categoria/ver/', views.ver_categorias, name='ver_categorias'),
    path('categoria/actualizar/<int:idcategoria>/', views.actualizar_categoria, name='actualizar_categoria'),
    path('categoria/actualizar/realizar/<int:idcategoria>/', views.realizar_actualizacion_categoria, name='realizar_actualizacion_categoria'),
    path('categoria/borrar/<int:idcategoria>/', views.borrar_categoria, name='borrar_categoria'),
]
```

---

# 25 — Agregar `app_Boutique` en `settings.py`

Edita `backend_Boutique/settings.py` y agrega en `INSTALLED_APPS`:

```python
'app_Boutique',
```

---

# 26 — Configurar `urls.py` del proyecto

Edita `backend_Boutique/urls.py`:

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('app_Boutique.urls')),
]
```

---

# 27 — Registrar los modelos en `admin.py`

Abre `app_Boutique/admin.py` y pega:

```python
from django.contrib import admin
from .models import Categoria, Producto, Proveedor

admin.site.register(Categoria)
admin.site.register(Producto)
admin.site.register(Proveedor)
```

Luego ejecuta:

```bash
python manage.py makemigrations
python manage.py migrate
python manage.py createsuperuser
```

---

# 27 (segunda) — Solo trabajar con Categoría

Deja **Producto** y **Proveedor** pendientes hasta terminar el CRUD de **Categoría**.

---

# 28 — Crear carpeta `templates` dentro de `app_Boutique`

Ruta: `app_Boutique/templates/`

Archivos:

```
base.html
navbar.html
footer.html
inicio.html
categoria/agregar_categoria.html
categoria/ver_categorias.html
categoria/actualizar_categoria.html
categoria/borrar_categoria.html
```

---

# 29 — Estructura de carpetas

```
UIII_Boutique_0390/
├─ manage.py
├─ backend_Boutique/
│  ├─ settings.py
│  ├─ urls.py
│  └─ ...
├─ app_Boutique/
│  ├─ models.py
│  ├─ views.py
│  ├─ urls.py
│  ├─ admin.py
│  ├─ templates/
│  │  ├─ base.html
│  │  ├─ navbar.html
│  │  ├─ footer.html
│  │  ├─ inicio.html
│  │  └─ categoria/
│  │     ├─ agregar_categoria.html
│  │     ├─ ver_categorias.html
│  │     ├─ actualizar_categoria.html
│  │     └─ borrar_categoria.html
```

---

# 30 — Ejecutar el servidor

```bash
python manage.py runserver 8390
```

Abre en navegador:
`http://127.0.0.1:8390/`

---

# 31 — Proyecto funcional

Ya tienes el CRUD de **Categoría** completamente funcional.
Los modelos de **Producto** y **Proveedor** están listos para usarse en siguientes etapas.

---


