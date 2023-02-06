my-first-blog/blog/templates/blog/base.html:
```
{% load static %}
<html>

<head>
  
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.9.1/font/bootstrap-icons.css">
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Lobster&display=swap" rel="stylesheet">

  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.2.2/dist/css/bootstrap.min.css" rel="stylesheet"
    integrity="sha384-Zenh87qX5JnK2Jl0vWa8Ck2rdkQ2Bzep5IDxbcnCeuOxjzrPF/et3URy9Bv1WTRi" crossorigin="anonymous">
  <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.2.2/dist/js/bootstrap.bundle.min.js"
    integrity="sha384-OERcA2EqjJCMA+/3y+gxIOqMEjwtxJY7qPCqsdltbNJuaOe923+mo//f6V8Qbsw3"
    crossorigin="anonymous"></script>
  <link rel="stylesheet" href="{% static 'css/blog.css' %}">
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.9.1/font/bootstrap-icons.css">
</head>

<body>
    <nav class="navbar navbar-expand-lg bg-light">

      <div class="container">
        <a class="navbar-brand" href="/">Мой блог</a>
        <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNavAltMarkup" aria-controls="navbarNavAltMarkup" aria-expanded="false" aria-label="Toggle navigation">
          <span class="navbar-toggler-icon"></span>
        </button>
        <div class="collapse navbar-collapse" id="navbarNavAltMarkup">
          <ul class="navbar-nav" id="navbarNavAltMarkup">
            {%if user.get_username %}
            <li class="nav-item">
              <a class="nav-link text-nowrap" href="{%url 'post_new'%}">
                <i class="bi-plus"></i>Добавить пост</a>
            </li>
          </ul>
          <ul class="nav justify-content-end w-100" id="navbarNavAltMarkup">
            <li class="nav-item">
              <span class="navbar-text">{{user.get_username}}</span>
            </li> 
          </ul>
          <ul class="nav justify-content-end">
            <div class="navbar-nav">
              <li class="nav-item">                              
                <a class="nav-link" href="{%url 'logout'%}">Выход</a>
              </li>
            </div>
            {%else%}
            <div class="navbar-nav">
              <li class="nav-item">
                <a class="nav-link" href="{%url 'login'%}">Вход</a>
              </li>
            </div>
            {%endif%}
          </ul>
        </div>
      </div>
  </nav>
  <div class="container">
        {%block content%}
        {%endblock%}
      </div>
      <div class="container">
        {% block footer %}{% endblock %}
      </div>
    </div>

  </div>
</body>

</html>
```

my-first-blog/blog/templates/blog/post.html:
```
<br>
<div class="card">
    <div class="card-body">
        <a href="{%url 'post_detail' pk=post.pk %}" class="nav-link1">{{ post.title }}</a>
        <p class="card-text">{{ post.text|linebreaksbr }}</p>
        {%if user.get_username %}
        <a class="btn btn-primary" href="{% url 'post_edit' pk=post.pk %}">
        
        <button type="submit" class="btn btn-primary"> Редактировать <i class="bi bi-pencil"></i></button>
        </a>
        {%endif%}
    </div>
    <div class="card-footer">
        {{post.published_date}}
    </div>
  </div>
```

my-first-blog/blog/templates/blog/post_detail.html:
```
{% extends 'blog/base.html' %}

{% block content %}
<br>
<nav>
<div class="container">
</nav>
    <div class="card">
        <div class="card-body">
            <a class = "nav-link1">{{ post.title }}</a>
            <p>{{ post.text|linebreaksbr }}</p>
            {% if user.is_authenticated %}
            <a class="btn btn-primary" href="{% url 'post_edit' pk=post.pk %}">
                <button type="submit" class="btn btn-primary"> Редактировать <i class="bi bi-pencil"></i></button>
            </a>
            {% endif %}
        </div>
        <div class="card-footer">
            {% if post.published_date %}
            {{post.published_date}}
            {% endif %}
        </div>
    </div>
    <br>
    <div id="footer">
        НИЯУ МИФИ &middot; &copy; 2022
    </div>
</div>
{% endblock %}
```

my-first-blog/blog/templates/blog/post_edit.html:
```
{% extends 'blog/base.html' %}

{% block content %}
    <h1>Новый пост</h1>
    <form method="POST" class="post-form">{% csrf_token %}
        {{ form.as_p }}
        <button type="submit" class="btn btn-primary">Сохранить</button>
    <br>
    <br>
    <br>
    <div id="footer">
        НИЯУ МИФИ &middot; &copy; 2022
    </div>
    </form>
{% endblock %}
```

my-first-blog/blog/templates/blog/post_list.html:
```
{% extends 'blog/base.html' %}
{% load static %}
{% block content %}
{% for post in posts %}
{% include 'blog/post.html' with post=post %}
{% endfor %}
{% endblock %}
{% block footer %}
<br>
<div id="footer">
    НИЯУ МИФИ &middot; &copy; 2022
        <p class='text-muted'>
        Количество посещений данной страницы: {{num_visits}}
        </p>
</div>
{% endblock %}
```

my-first-blog/blog/views.py:
```
from django.shortcuts import render
from django.utils import timezone
from .models import Post
from django.shortcuts import render, get_object_or_404
from .forms import PostForm
from django.shortcuts import redirect

def post_list(request):
    posts = Post.objects.filter(published_date__lte=timezone.now()).order_by('-published_date')
    num_visits = request.session.get('num_visits', 0)
    num_visits += 1
    request.session['num_visits'] = num_visits
    return render(request, 'blog/post_list.html', {"posts": posts, "num_visits": num_visits})
def post_detail(request, pk):
    post = get_object_or_404(Post, pk=pk)
    return render(request, 'blog/post_detail.html', {'post': post})
def post_new(request):
    if request.method == "POST":
        form = PostForm(request.POST)
        if form.is_valid():
            post = form.save(commit=False)
            post.author = request.user
            post.published_date = timezone.now()
            post.save()
            return redirect('post_detail', pk=post.pk)
    else:
        form = PostForm()
    return render(request, 'blog/post_edit.html', {'form': form})
def post_edit(request, pk):
    post = get_object_or_404(Post, pk=pk)
    if request.method == "POST":
        form = PostForm(request.POST, instance=post)
        if form.is_valid():
            post = form.save(commit=False)
            post.author = request.user
            post.published_date = timezone.now()
            post.save()
            return redirect('post_detail', pk=post.pk)
    else:
        form = PostForm(instance=post)
    return render(request, 'blog/post_edit.html', {'form': form})
```
my-first-blog/blog/urls.py: 
```
from django.urls import path 
from . import views

urlpatterns = [ 
    path('', views.post_list, name='post_list'),
    path('post/<int:pk>/', views.post_detail, name='post_detail'), 
    path('post/new/', views.post_new, name='post_new'), 
    path('post/<int:pk>/edit/', views.post_edit, name='post_edit'),

]
```
my-first-blog/blog/models.py
```
from django.conf import settings
from django.db import models
from django.utils import timezone


class Post(models.Model):
    author = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    title = models.CharField(max_length=200)
    text = models.TextField()
    created_date = models.DateTimeField(default=timezone.now)
    published_date = models.DateTimeField(blank=True, null=True)

    def publish(self):
        self.published_date = timezone.now()
        self.save()

    def __str__(self):
        return self.title

```

my-first-blog/blog/forms.py:
```
from django import forms

from .models import Post

class PostForm(forms.ModelForm):

    class Meta:
        model = Post
        fields = ('title', 'text',)
```
