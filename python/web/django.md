# Django In-Depth Cheatsheet

## Installation and Setup

```bash
# Install Django
pip install django

# Install with database drivers
pip install django psycopg2-binary  # PostgreSQL
pip install django mysqlclient       # MySQL

# Create new project
django-admin startproject myproject
cd myproject

# Create new app
python manage.py startapp myapp

# Run development server
python manage.py runserver
python manage.py runserver 0.0.0.0:8000  # All interfaces
python manage.py runserver 8080           # Custom port
```

## Project Structure and Configuration

### Settings Configuration
```python
# settings.py
import os
from pathlib import Path

BASE_DIR = Path(__file__).resolve().parent.parent

# Security settings
SECRET_KEY = os.environ.get('SECRET_KEY', 'your-secret-key-here')
DEBUG = os.environ.get('DEBUG', 'False').lower() == 'true'
ALLOWED_HOSTS = ['localhost', '127.0.0.1', '.herokuapp.com']

# Application definition
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'rest_framework',  # Django REST Framework
    'corsheaders',     # CORS headers
    'myapp',           # Custom app
]

MIDDLEWARE = [
    'corsheaders.middleware.CorsMiddleware',
    'django.middleware.security.SecurityMiddleware',
    'whitenoise.middleware.WhiteNoiseMiddleware',  # Static files
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

ROOT_URLCONF = 'myproject.urls'

# Database configuration
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.environ.get('DB_NAME', 'mydb'),
        'USER': os.environ.get('DB_USER', 'myuser'),
        'PASSWORD': os.environ.get('DB_PASSWORD', 'mypassword'),
        'HOST': os.environ.get('DB_HOST', 'localhost'),
        'PORT': os.environ.get('DB_PORT', '5432'),
    }
}

# Static files configuration
STATIC_URL = '/static/'
STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
STATICFILES_DIRS = [os.path.join(BASE_DIR, 'static')]

# Media files configuration
MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')

# Internationalization
LANGUAGE_CODE = 'en-us'
TIME_ZONE = 'UTC'
USE_I18N = True
USE_TZ = True

# Security settings for production
SECURE_SSL_REDIRECT = not DEBUG
SECURE_HSTS_SECONDS = 31536000 if not DEBUG else 0
SECURE_HSTS_INCLUDE_SUBDOMAINS = not DEBUG
SECURE_HSTS_PRELOAD = not DEBUG
SESSION_COOKIE_SECURE = not DEBUG
CSRF_COOKIE_SECURE = not DEBUG
```

### Environment Variables (.env)
```bash
# .env file
SECRET_KEY=your-very-secret-key-here
DEBUG=True
DB_NAME=myproject_db
DB_USER=myuser
DB_PASSWORD=mypassword
DB_HOST=localhost
DB_PORT=5432

# Load in settings.py
import os
from dotenv import load_dotenv
load_dotenv()
```

## Models and Database

### Model Definition
```python
# models.py
from django.db import models
from django.contrib.auth.models import User
from django.urls import reverse
from django.utils import timezone

class Category(models.Model):
    name = models.CharField(max_length=100, unique=True)
    slug = models.SlugField(unique=True)
    description = models.TextField(blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        verbose_name_plural = "Categories"
        ordering = ['name']
    
    def __str__(self):
        return self.name
    
    def get_absolute_url(self):
        return reverse('category_detail', kwargs={'slug': self.slug})

class Post(models.Model):
    STATUS_CHOICES = [
        ('draft', 'Draft'),
        ('published', 'Published'),
        ('archived', 'Archived'),
    ]
    
    title = models.CharField(max_length=200)
    slug = models.SlugField(unique=True)
    content = models.TextField()
    excerpt = models.TextField(max_length=300, blank=True)
    author = models.ForeignKey(User, on_delete=models.CASCADE, related_name='posts')
    category = models.ForeignKey(Category, on_delete=models.SET_NULL, null=True, blank=True)
    tags = models.ManyToManyField('Tag', blank=True)
    status = models.CharField(max_length=10, choices=STATUS_CHOICES, default='draft')
    featured_image = models.ImageField(upload_to='posts/', blank=True, null=True)
    views = models.PositiveIntegerField(default=0)
    likes = models.ManyToManyField(User, through='PostLike', related_name='liked_posts')
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    published_at = models.DateTimeField(null=True, blank=True)
    
    class Meta:
        ordering = ['-created_at']
        indexes = [
            models.Index(fields=['status', 'published_at']),
            models.Index(fields=['author', 'created_at']),
        ]
    
    def __str__(self):
        return self.title
    
    def save(self, *args, **kwargs):
        if self.status == 'published' and not self.published_at:
            self.published_at = timezone.now()
        super().save(*args, **kwargs)
    
    def get_absolute_url(self):
        return reverse('post_detail', kwargs={'slug': self.slug})
    
    @property
    def is_published(self):
        return self.status == 'published'
    
    def get_like_count(self):
        return self.likes.count()

class Tag(models.Model):
    name = models.CharField(max_length=50, unique=True)
    slug = models.SlugField(unique=True)
    
    def __str__(self):
        return self.name

class PostLike(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    post = models.ForeignKey(Post, on_delete=models.CASCADE)
    created_at = models.DateTimeField(auto_now_add=True)
    
    class Meta:
        unique_together = ('user', 'post')

class Comment(models.Model):
    post = models.ForeignKey(Post, on_delete=models.CASCADE, related_name='comments')
    author = models.ForeignKey(User, on_delete=models.CASCADE)
    content = models.TextField()
    parent = models.ForeignKey('self', on_delete=models.CASCADE, null=True, blank=True, related_name='replies')
    is_approved = models.BooleanField(default=False)
    created_at = models.DateTimeField(auto_now_add=True)
    
    class Meta:
        ordering = ['created_at']
    
    def __str__(self):
        return f'Comment by {self.author.username} on {self.post.title}'
```

### Advanced Model Features
```python
# Custom model managers
class PublishedManager(models.Manager):
    def get_queryset(self):
        return super().get_queryset().filter(status='published')

class Post(models.Model):
    # ... other fields ...
    
    objects = models.Manager()  # Default manager
    published = PublishedManager()  # Custom manager
    
    # Custom methods
    @classmethod
    def get_popular_posts(cls, limit=5):
        return cls.published.order_by('-views')[:limit]
    
    def increment_views(self):
        self.views = models.F('views') + 1
        self.save(update_fields=['views'])

# Abstract base models
class TimeStampedModel(models.Model):
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        abstract = True

class Post(TimeStampedModel):
    # Inherits created_at and updated_at fields
    title = models.CharField(max_length=200)
    # ... other fields ...

# Model validation
from django.core.exceptions import ValidationError

def validate_file_size(value):
    filesize = value.size
    if filesize > 5 * 1024 * 1024:  # 5MB
        raise ValidationError("File size cannot exceed 5MB")

class Document(models.Model):
    title = models.CharField(max_length=200)
    file = models.FileField(upload_to='documents/', validators=[validate_file_size])
    
    def clean(self):
        if self.title and len(self.title) < 5:
            raise ValidationError({'title': 'Title must be at least 5 characters long'})
```

### Database Operations
```python
# Database migrations
python manage.py makemigrations
python manage.py makemigrations myapp
python manage.py migrate
python manage.py migrate --fake  # Mark as applied without running
python manage.py showmigrations
python manage.py sqlmigrate myapp 0001

# Custom migration
from django.db import migrations, models

def populate_categories(apps, schema_editor):
    Category = apps.get_model('myapp', 'Category')
    categories = ['Technology', 'Science', 'Arts']
    for cat_name in categories:
        Category.objects.create(name=cat_name, slug=cat_name.lower())

class Migration(migrations.Migration):
    dependencies = [
        ('myapp', '0001_initial'),
    ]
    
    operations = [
        migrations.RunPython(populate_categories),
    ]

# QuerySet operations
# Basic queries
posts = Post.objects.all()
published_posts = Post.objects.filter(status='published')
recent_posts = Post.objects.filter(created_at__gte=timezone.now() - timezone.timedelta(days=7))

# Complex queries
from django.db.models import Q, F, Count, Avg

# Q objects for complex conditions
popular_posts = Post.objects.filter(
    Q(views__gte=1000) | Q(likes__count__gte=50)
).distinct()

# F expressions for field comparisons
trending_posts = Post.objects.filter(views__gt=F('likes__count') * 10)

# Annotations and aggregations
posts_with_stats = Post.objects.annotate(
    comment_count=Count('comments'),
    like_count=Count('likes'),
    avg_comment_length=Avg('comments__content__length')
).filter(comment_count__gt=0)

# Prefetch related objects
posts_with_relations = Post.objects.select_related('author', 'category').prefetch_related('tags', 'comments')

# Raw SQL queries
posts = Post.objects.raw('SELECT * FROM myapp_post WHERE views > %s', [1000])

# Bulk operations
Post.objects.bulk_create([
    Post(title='Post 1', content='Content 1'),
    Post(title='Post 2', content='Content 2'),
])

Post.objects.filter(status='draft').bulk_update(
    [post1, post2], ['status'], batch_size=100
)
```

## Views and URL Routing

### Function-Based Views (FBV)
```python
# views.py
from django.shortcuts import render, get_object_or_404, redirect
from django.http import JsonResponse, Http404
from django.contrib.auth.decorators import login_required
from django.views.decorators.http import require_http_methods
from django.core.paginator import Paginator
from django.contrib import messages
from django.db.models import Q

def post_list(request):
    posts_list = Post.published.all()
    
    # Search functionality
    query = request.GET.get('q')
    if query:
        posts_list = posts_list.filter(
            Q(title__icontains=query) | Q(content__icontains=query)
        )
    
    # Pagination
    paginator = Paginator(posts_list, 10)  # 10 posts per page
    page_number = request.GET.get('page')
    posts = paginator.get_page(page_number)
    
    context = {
        'posts': posts,
        'query': query,
    }
    return render(request, 'blog/post_list.html', context)

def post_detail(request, slug):
    post = get_object_or_404(Post, slug=slug, status='published')
    
    # Increment views
    post.increment_views()
    
    # Get related posts
    related_posts = Post.published.filter(
        category=post.category
    ).exclude(id=post.id)[:3]
    
    # Handle comments
    if request.method == 'POST' and request.user.is_authenticated:
        content = request.POST.get('content')
        if content:
            Comment.objects.create(
                post=post,
                author=request.user,
                content=content
            )
            messages.success(request, 'Comment added successfully!')
            return redirect('post_detail', slug=slug)
    
    comments = post.comments.filter(is_approved=True, parent=None)
    
    context = {
        'post': post,
        'related_posts': related_posts,
        'comments': comments,
    }
    return render(request, 'blog/post_detail.html', context)

@login_required
@require_http_methods(["POST"])
def toggle_like(request, post_id):
    post = get_object_or_404(Post, id=post_id)
    like, created = PostLike.objects.get_or_create(
        user=request.user,
        post=post
    )
    
    if not created:
        like.delete()
        liked = False
    else:
        liked = True
    
    return JsonResponse({
        'liked': liked,
        'like_count': post.get_like_count()
    })
```

### Class-Based Views (CBV)
```python
from django.views.generic import ListView, DetailView, CreateView, UpdateView, DeleteView
from django.contrib.auth.mixins import LoginRequiredMixin, UserPassesTestMixin
from django.urls import reverse_lazy

class PostListView(ListView):
    model = Post
    template_name = 'blog/post_list.html'
    context_object_name = 'posts'
    paginate_by = 10
    queryset = Post.published.all()
    
    def get_queryset(self):
        queryset = super().get_queryset()
        query = self.request.GET.get('q')
        if query:
            queryset = queryset.filter(
                Q(title__icontains=query) | Q(content__icontains=query)
            )
        return queryset
    
    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['query'] = self.request.GET.get('q', '')
        return context

class PostDetailView(DetailView):
    model = Post
    template_name = 'blog/post_detail.html'
    context_object_name = 'post'
    
    def get_queryset(self):
        return Post.published.all()
    
    def get_object(self):
        obj = super().get_object()
        obj.increment_views()
        return obj
    
    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        post = self.get_object()
        context['related_posts'] = Post.published.filter(
            category=post.category
        ).exclude(id=post.id)[:3]
        context['comments'] = post.comments.filter(
            is_approved=True,
            parent=None
        )
        return context

class PostCreateView(LoginRequiredMixin, CreateView):
    model = Post
    template_name = 'blog/post_form.html'
    fields = ['title', 'content', 'category', 'tags', 'featured_image']
    
    def form_valid(self, form):
        form.instance.author = self.request.user
        return super().form_valid(form)

class PostUpdateView(LoginRequiredMixin, UserPassesTestMixin, UpdateView):
    model = Post
    template_name = 'blog/post_form.html'
    fields = ['title', 'content', 'category', 'tags', 'featured_image', 'status']
    
    def test_func(self):
        post = self.get_object()
        return self.request.user == post.author or self.request.user.is_staff

class PostDeleteView(LoginRequiredMixin, UserPassesTestMixin, DeleteView):
    model = Post
    template_name = 'blog/post_confirm_delete.html'
    success_url = reverse_lazy('post_list')
    
    def test_func(self):
        post = self.get_object()
        return self.request.user == post.author or self.request.user.is_staff

# Custom mixins
class AuthorRequiredMixin(UserPassesTestMixin):
    def test_func(self):
        obj = self.get_object()
        return obj.author == self.request.user

class AjaxResponseMixin:
    def dispatch(self, request, *args, **kwargs):
        if not request.is_ajax():
            return JsonResponse({'error': 'AJAX request required'}, status=400)
        return super().dispatch(request, *args, **kwargs)
```

### URL Configuration
```python
# urls.py (project level)
from django.contrib import admin
from django.urls import path, include
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('blog.urls')),
    path('api/', include('api.urls')),
    path('auth/', include('django.contrib.auth.urls')),
]

# Serve media files in development
if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)

# urls.py (app level)
from django.urls import path
from . import views

app_name = 'blog'

urlpatterns = [
    path('', views.PostListView.as_view(), name='post_list'),
    path('post/<slug:slug>/', views.PostDetailView.as_view(), name='post_detail'),
    path('post/create/', views.PostCreateView.as_view(), name='post_create'),
    path('post/<slug:slug>/edit/', views.PostUpdateView.as_view(), name='post_update'),
    path('post/<slug:slug>/delete/', views.PostDeleteView.as_view(), name='post_delete'),
    path('category/<slug:slug>/', views.CategoryDetailView.as_view(), name='category_detail'),
    path('api/post/<int:post_id>/like/', views.toggle_like, name='toggle_like'),
    
    # Advanced URL patterns
    path('posts/<int:year>/', views.posts_by_year, name='posts_by_year'),
    path('posts/<int:year>/<int:month>/', views.posts_by_month, name='posts_by_month'),
    path('search/', views.search_posts, name='search_posts'),
    
    # API endpoints
    path('api/posts/', views.PostListAPIView.as_view(), name='api_post_list'),
    path('api/posts/<int:pk>/', views.PostDetailAPIView.as_view(), name='api_post_detail'),
]
```

## Forms and Form Handling

### Model Forms
```python
# forms.py
from django import forms
from django.contrib.auth.forms import UserCreationForm
from django.contrib.auth.models import User
from .models import Post, Comment, Category

class PostForm(forms.ModelForm):
    class Meta:
        model = Post
        fields = ['title', 'content', 'category', 'tags', 'featured_image', 'status']
        widgets = {
            'title': forms.TextInput(attrs={'class': 'form-control', 'placeholder': 'Enter title'}),
            'content': forms.Textarea(attrs={'class': 'form-control', 'rows': 10}),
            'category': forms.Select(attrs={'class': 'form-control'}),
            'tags': forms.CheckboxSelectMultiple(),
            'featured_image': forms.FileInput(attrs={'class': 'form-control'}),
            'status': forms.Select(attrs={'class': 'form-control'}),
        }
    
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.fields['category'].queryset = Category.objects.all()
        self.fields['title'].help_text = "Enter a descriptive title for your post"
    
    def clean_title(self):
        title = self.cleaned_data['title']
        if len(title) < 5:
            raise forms.ValidationError("Title must be at least 5 characters long")
        return title
    
    def clean(self):
        cleaned_data = super().clean()
        title = cleaned_data.get('title')
        content = cleaned_data.get('content')
        
        if title and content and title.lower() in content.lower():
            raise forms.ValidationError("Title should not be repeated in content")
        
        return cleaned_data

class CommentForm(forms.ModelForm):
    class Meta:
        model = Comment
        fields = ['content']
        widgets = {
            'content': forms.Textarea(attrs={
                'class': 'form-control',
                'rows': 4,
                'placeholder': 'Write your comment here...'
            })
        }

class SearchForm(forms.Form):
    query = forms.CharField(
        max_length=255,
        widget=forms.TextInput(attrs={
            'class': 'form-control',
            'placeholder': 'Search posts...',
            'autocomplete': 'off'
        })
    )
    category = forms.ModelChoiceField(
        queryset=Category.objects.all(),
        required=False,
        empty_label="All Categories",
        widget=forms.Select(attrs={'class': 'form-control'})
    )

class CustomUserCreationForm(UserCreationForm):
    email = forms.EmailField(required=True)
    first_name = forms.CharField(max_length=30, required=True)
    last_name = forms.CharField(max_length=30, required=True)
    
    class Meta:
        model = User
        fields = ('username', 'first_name', 'last_name', 'email', 'password1', 'password2')
    
    def save(self, commit=True):
        user = super().save(commit=False)
        user.email = self.cleaned_data['email']
        user.first_name = self.cleaned_data['first_name']
        user.last_name = self.cleaned_data['last_name']
        if commit:
            user.save()
        return user

# Form handling in views
def create_post(request):
    if request.method == 'POST':
        form = PostForm(request.POST, request.FILES)
        if form.is_valid():
            post = form.save(commit=False)
            post.author = request.user
            post.save()
            form.save_m2m()  # Save many-to-many relationships
            messages.success(request, 'Post created successfully!')
            return redirect('post_detail', slug=post.slug)
    else:
        form = PostForm()
    
    return render(request, 'blog/post_create.html', {'form': form})
```

### Advanced Forms
```python
# Dynamic forms
class DynamicPostForm(forms.ModelForm):
    def __init__(self, user=None, *args, **kwargs):
        super().__init__(*args, **kwargs)
        
        if user and not user.is_staff:
            # Limit status choices for non-staff users
            self.fields['status'].choices = [('draft', 'Draft'), ('published', 'Published')]
        
        # Dynamic category filtering
        if user and hasattr(user, 'profile'):
            allowed_categories = user.profile.allowed_categories.all()
            self.fields['category'].queryset = allowed_categories

# Formsets for handling multiple forms
from django.forms import formset_factory, modelformset_factory

# Simple formset
CommentFormSet = formset_factory(CommentForm, extra=3)

def post_with_comments(request, slug):
    post = get_object_or_404(Post, slug=slug)
    
    if request.method == 'POST':
        formset = CommentFormSet(request.POST)
        if formset.is_valid():
            for form in formset:
                if form.cleaned_data:  # Skip empty forms
                    comment = form.save(commit=False)
                    comment.post = post
                    comment.author = request.user
                    comment.save()
            return redirect('post_detail', slug=slug)
    else:
        formset = CommentFormSet()
    
    return render(request, 'blog/post_comments.html', {
        'post': post,
        'formset': formset
    })

# Model formset
PostFormSet = modelformset_factory(Post, fields=['title', 'status'], extra=0)

def bulk_edit_posts(request):
    queryset = Post.objects.filter(author=request.user)
    
    if request.method == 'POST':
        formset = PostFormSet(request.POST, queryset=queryset)
        if formset.is_valid():
            formset.save()
            messages.success(request, 'Posts updated successfully!')
            return redirect('post_list')
    else:
        formset = PostFormSet(queryset=queryset)
    
    return render(request, 'blog/bulk_edit.html', {'formset': formset})

# Custom form fields
class TagField(forms.CharField):
    def to_python(self, value):
        if not value:
            return []
        return [tag.strip() for tag in value.split(',') if tag.strip()]
    
    def validate(self, value):
        super().validate(value)
        if len(value) > 10:
            raise forms.ValidationError("Maximum 10 tags allowed")

class PostFormWithTags(forms.ModelForm):
    tags_input = TagField(
        required=False,
        help_text="Enter tags separated by commas"
    )
    
    class Meta:
        model = Post
        fields = ['title', 'content', 'category']
    
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        if self.instance.pk:
            self.fields['tags_input'].initial = ', '.join(
                self.instance.tags.values_list('name', flat=True)
            )
    
    def save(self, commit=True):
        instance = super().save(commit=commit)
        if commit:
            # Handle tags
            tag_names = self.cleaned_data.get('tags_input', [])
            instance.tags.clear()
            for tag_name in tag_names:
                tag, created = Tag.objects.get_or_create(name=tag_name)
                instance.tags.add(tag)
        return instance
```

## Templates and Static Files

### Template Structure
```html
<!-- base.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}My Blog{% endblock %}</title>
    
    {% load static %}
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" rel="stylesheet">
    <link href="{% static 'css/style.css' %}" rel="stylesheet">
    
    {% block extra_css %}{% endblock %}
</head>
<body>
    <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
        <div class="container">
            <a class="navbar-brand" href="{% url 'blog:post_list' %}">My Blog</a>
            <div class="navbar-nav ms-auto">
                {% if user.is_authenticated %}
                    <a class="nav-link" href="{% url 'blog:post_create' %}">Create Post</a>
                    <a class="nav-link" href="{% url 'admin:index' %}">Admin</a>
                    <a class="nav-link" href="{% url 'logout' %}">Logout ({{ user.username }})</a>
                {% else %}
                    <a class="nav-link" href="{% url 'login' %}">Login</a>
                    <a class="nav-link" href="{% url 'signup' %}">Sign Up</a>
                {% endif %}
            </div>
        </div>
    </nav>
    
    <main class="container mt-4">
        {% if messages %}
            {% for message in messages %}
                <div class="alert alert-{{ message.tags }} alert-dismissible fade show" role="alert">
                    {{ message }}
                    <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
                </div>
            {% endfor %}
        {% endif %}
        
        {% block content %}{% endblock %}
    </main>
    
    <footer class="bg-dark text-light mt-5 py-4">
        <div class="container">
            <p>&copy; 2024 My Blog. All rights reserved.</p>
        </div>
    </footer>
    
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/js/bootstrap.bundle.min.js"></script>
    <script src="{% static 'js/main.js' %}"></script>
    {% block extra_js %}{% endblock %}
</body>
</html>

<!-- post_list.html -->
{% extends 'base.html' %}
{% load static %}

{% block title %}Posts - {{ block.super }}{% endblock %}

{% block content %}
<div class="row">
    <div class="col-md-8">
        <h1>Latest Posts</h1>
        
        <!-- Search form -->
        <form method="get" class="mb-4">
            <div class="input-group">
                <input type="text" name="q" value="{{ query }}" class="form-control" placeholder="Search posts...">
                <button class="btn btn-primary" type="submit">Search</button>
            </div>
        </form>
        
        {% for post in posts %}
            <article class="card mb-4">
                {% if post.featured_image %}
                    <img src="{{ post.featured_image.url }}" class="card-img-top" alt="{{ post.title }}">
                {% endif %}
                <div class="card-body">
                    <h2 class="card-title">
                        <a href="{{ post.get_absolute_url }}">{{ post.title }}</a>
                    </h2>
                    <p class="card-text">{{ post.excerpt|default:post.content|truncatewords:30 }}</p>
                    <div class="d-flex justify-content-between align-items-center">
                        <small class="text-muted">
                            By {{ post.author.get_full_name|default:post.author.username }}
                            on {{ post.published_at|date:"M d, Y" }}
                        </small>
                        <div>
                            <span class="badge bg-secondary">{{ post.views }} views</span>
                            <span class="badge bg-primary">{{ post.get_like_count }} likes</span>
                        </div>
                    </div>
                </div>
            </article>
        {% empty %}
            <p>No posts found.</p>
        {% endfor %}
        
        <!-- Pagination -->
        {% if posts.has_other_pages %}
            <nav aria-label="Posts pagination">
                <ul class="pagination justify-content-center">
                    {% if posts.has_previous %}
                        <li class="page-item">
                            <a class="page-link" href="?page=1{% if query %}&q={{ query }}{% endif %}">First</a>
                        </li>
                        <li class="page-item">
                            <a class="page-link" href="?page={{ posts.previous_page_number }}{% if query %}&q={{ query }}{% endif %}">Previous</a>
                        </li>
                    {% endif %}
                    
                    <li class="page-item active">
                        <span class="page-link">Page {{ posts.number }} of {{ posts.paginator.num_pages }}</span>
                    </li>
                    
                    {% if posts.has_next %}
                        <li class="page-item">
                            <a class="page-link" href="?page={{ posts.next_page_number }}{% if query %}&q={{ query }}{% endif %}">Next</a>
                        </li>
                        <li class="page-item">
                            <a class="page-link" href="?page={{ posts.paginator.num_pages }}{% if query %}&q={{ query }}{% endif %}">Last</a>
                        </li>
                    {% endif %}
                </ul>
            </nav>
        {% endif %}
    </div>
    
    <div class="col-md-4">
        <!-- Sidebar -->
        <div class="card">
            <div class="card-header">
                <h5>Categories</h5>
            </div>
            <div class="card-body">
                {% for category in categories %}
                    <a href="{{ category.get_absolute_url }}" class="btn btn-outline-primary btn-sm me-2 mb-2">
                        {{ category.name }}
                    </a>
                {% endfor %}
            </div>
        </div>
        
        <div class="card mt-4">
            <div class="card-header">
                <h5>Popular Posts</h5>
            </div>
            <div class="card-body">
                {% for post in popular_posts %}
                    <p><a href="{{ post.get_absolute_url }}">{{ post.title }}</a></p>
                {% endfor %}
            </div>
        </div>
    </div>
</div>
{% endblock %}
```

### Custom Template Tags and Filters
```python
# templatetags/blog_extras.py
from django import template
from django.utils.safestring import mark_safe
from django.utils.html import format_html
import markdown

register = template.Library()

@register.filter
def markdown_to_html(text):
    """Convert markdown text to HTML"""
    return mark_safe(markdown.markdown(text))

@register.simple_tag
def total_posts():
    """Get total number of published posts"""
    from blog.models import Post
    return Post.published.count()

@register.simple_tag(takes_context=True)
def url_replace(context, **kwargs):
    """Replace URL parameters while keeping existing ones"""
    query = context['request'].GET.copy()
    for key, value in kwargs.items():
        query[key] = value
    return query.urlencode()

@register.inclusion_tag('blog/tags/recent_posts.html')
def show_recent_posts(count=5):
    """Include template tag for recent posts"""
    from blog.models import Post
    posts = Post.published.all()[:count]
    return {'posts': posts}

@register.filter
def add_class(field, css_class):
    """Add CSS class to form field"""
    return field.as_widget(attrs={"class": css_class})

# Usage in templates
# {% load blog_extras %}
# {{ post.content|markdown_to_html }}
# {% total_posts %}
# {% show_recent_posts 3 %}
# {{ form.title|add_class:"form-control" }}
```

## Authentication and Authorization

### User Authentication
```python
# views.py
from django.contrib.auth import login, authenticate
from django.contrib.auth.views import LoginView, LogoutView
from django.contrib.auth.decorators import login_required, permission_required
from django.contrib.auth.mixins import LoginRequiredMixin, PermissionRequiredMixin

class CustomLoginView(LoginView):
    template_name = 'registration/login.html'
    redirect_authenticated_user = True
    
    def get_success_url(self):
        return self.get_redirect_url() or reverse_lazy('blog:post_list')

def signup(request):
    if request.method == 'POST':
        form = CustomUserCreationForm(request.POST)
        if form.is_valid():
            user = form.save()
            username = form.cleaned_data.get('username')
            messages.success(request, f'Account created for {username}!')
            # Auto-login after signup
            user = authenticate(username=username, password=form.cleaned_data.get('password1'))
            if user:
                login(request, user)
            return redirect('blog:post_list')
    else:
        form = CustomUserCreationForm()
    return render(request, 'registration/signup.html', {'form': form})

# Permission-based access control
@permission_required('blog.add_post')
def create_post(request):
    # Only users with 'add_post' permission can access
    pass

class PostCreateView(PermissionRequiredMixin, CreateView):
    permission_required = 'blog.add_post'
    model = Post
    fields = ['title', 'content']

# Custom permissions
from django.contrib.auth.models import Permission
from django.contrib.contenttypes.models import ContentType

# Create custom permission
content_type = ContentType.objects.get_for_model(Post)
permission = Permission.objects.create(
    codename='can_feature_post',
    name='Can feature post',
    content_type=content_type,
)

# Check custom permission
@permission_required('blog.can_feature_post')
def feature_post(request, post_id):
    post = get_object_or_404(Post, id=post_id)
    post.is_featured = True
    post.save()
    return redirect('post_detail', slug=post.slug)
```

### User Profiles
```python
# models.py
from django.contrib.auth.models import User
from django.db.models.signals import post_save
from django.dispatch import receiver

class UserProfile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE, related_name='profile')
    bio = models.TextField(max_length=500, blank=True)
    avatar = models.ImageField(upload_to='avatars/', blank=True, null=True)
    website = models.URLField(blank=True)
    twitter = models.CharField(max_length=50, blank=True)
    github = models.CharField(max_length=50, blank=True)
    location = models.CharField(max_length=100, blank=True)
    birth_date = models.DateField(null=True, blank=True)
    
    def __str__(self):
        return f"{self.user.username}'s Profile"

@receiver(post_save, sender=User)
def create_user_profile(sender, instance, created, **kwargs):
    if created:
        UserProfile.objects.create(user=instance)

@receiver(post_save, sender=User)
def save_user_profile(sender, instance, **kwargs):
    if hasattr(instance, 'profile'):
        instance.profile.save()

# Profile views
@login_required
def profile_edit(request):
    if request.method == 'POST':
        user_form = UserForm(request.POST, instance=request.user)
        profile_form = UserProfileForm(request.POST, request.FILES, instance=request.user.profile)
        
        if user_form.is_valid() and profile_form.is_valid():
            user_form.save()
            profile_form.save()
            messages.success(request, 'Profile updated successfully!')
            return redirect('profile')
    else:
        user_form = UserForm(instance=request.user)
        profile_form = UserProfileForm(instance=request.user.profile)
    
    return render(request, 'registration/profile_edit.html', {
        'user_form': user_form,
        'profile_form': profile_form
    })
```

## Admin Interface

### Custom Admin Configuration
```python
# admin.py
from django.contrib import admin
from django.utils.html import format_html
from django.urls import reverse
from django.utils.safestring import mark_safe
from .models import Post, Category, Tag, Comment

@admin.register(Category)
class CategoryAdmin(admin.ModelAdmin):
    list_display = ['name', 'slug', 'post_count', 'created_at']
    prepopulated_fields = {'slug': ('name',)}
    search_fields = ['name', 'description']
    list_filter = ['created_at']
    
    def post_count(self, obj):
        count = obj.post_set.count()
        url = reverse('admin:blog_post_changelist') + f'?category={obj.id}'
        return format_html('<a href="{}">{} posts</a>', url, count)
    post_count.short_description = 'Posts'

class CommentInline(admin.TabularInline):
    model = Comment
    extra = 0
    readonly_fields = ['author', 'created_at']
    fields = ['author', 'content', 'is_approved', 'created_at']

@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    list_display = ['title', 'author', 'category', 'status', 'views', 'created_at', 'featured_image_preview']
    list_filter = ['status', 'category', 'created_at', 'author']
    search_fields = ['title', 'content', 'author__username']
    prepopulated_fields = {'slug': ('title',)}
    date_hierarchy = 'created_at'
    ordering = ['-created_at']
    filter_horizontal = ['tags']
    inlines = [CommentInline]
    
    fieldsets = (
        ('Content', {
            'fields': ('title', 'slug', 'content', 'excerpt')
        }),
        ('Classification', {
            'fields': ('category', 'tags', 'status')
        }),
        ('Media', {
            'fields': ('featured_image',)
        }),
        ('Metadata', {
            'fields': ('author', 'views', 'created_at', 'updated_at'),
            'classes': ('collapse',)
        }),
    )
    readonly_fields = ['created_at', 'updated_at', 'views']
    
    def featured_image_preview(self, obj):
        if obj.featured_image:
            return format_html(
                '<img src="{}" style="width: 50px; height: 50px; object-fit: cover;" />',
                obj.featured_image.url
            )
        return "No image"
    featured_image_preview.short_description = 'Preview'
    
    def save_model(self, request, obj, form, change):
        if not change:  # Creating new object
            obj.author = request.user
        super().save_model(request, obj, form, change)
    
    def get_queryset(self, request):
        qs = super().get_queryset(request)
        if request.user.is_superuser:
            return qs
        return qs.filter(author=request.user)

@admin.register(Comment)
class CommentAdmin(admin.ModelAdmin):
    list_display = ['post', 'author', 'content_preview', 'is_approved', 'created_at']
    list_filter = ['is_approved', 'created_at']
    search_fields = ['content', 'author__username', 'post__title']
    actions = ['approve_comments', 'reject_comments']
    
    def content_preview(self, obj):
        return obj.content[:50] + "..." if len(obj.content) > 50 else obj.content
    content_preview.short_description = 'Content'
    
    def approve_comments(self, request, queryset):
        queryset.update(is_approved=True)
        self.message_user(request, f'{queryset.count()} comments approved.')
    approve_comments.short_description = 'Approve selected comments'
    
    def reject_comments(self, request, queryset):
        queryset.update(is_approved=False)
        self.message_user(request, f'{queryset.count()} comments rejected.')
    reject_comments.short_description = 'Reject selected comments'

# Custom admin site
class BlogAdminSite(admin.AdminSite):
    site_header = 'Blog Administration'
    site_title = 'Blog Admin'
    index_title = 'Welcome to Blog Administration'

blog_admin_site = BlogAdminSite(name='blog_admin')
blog_admin_site.register(Post, PostAdmin)
blog_admin_site.register(Category, CategoryAdmin)
```

## Django REST Framework Integration

### API Views and Serializers
```python
# serializers.py
from rest_framework import serializers
from .models import Post, Category, Comment

class CategorySerializer(serializers.ModelSerializer):
    class Meta:
        model = Category
        fields = ['id', 'name', 'slug', 'description']

class PostSerializer(serializers.ModelSerializer):
    author = serializers.StringRelatedField(read_only=True)
    category = CategorySerializer(read_only=True)
    like_count = serializers.SerializerMethodField()
    comment_count = serializers.SerializerMethodField()
    
    class Meta:
        model = Post
        fields = [
            'id', 'title', 'slug', 'content', 'excerpt',
            'author', 'category', 'status', 'featured_image',
            'views', 'like_count', 'comment_count',
            'created_at', 'updated_at'
        ]
        read_only_fields = ['author', 'views', 'created_at', 'updated_at']
    
    def get_like_count(self, obj):
        return obj.get_like_count()
    
    def get_comment_count(self, obj):
        return obj.comments.filter(is_approved=True).count()

class CommentSerializer(serializers.ModelSerializer):
    author = serializers.StringRelatedField(read_only=True)
    
    class Meta:
        model = Comment
        fields = ['id', 'content', 'author', 'created_at', 'is_approved']
        read_only_fields = ['author', 'created_at', 'is_approved']

# api_views.py
from rest_framework import generics, permissions, status
from rest_framework.decorators import api_view, permission_classes
from rest_framework.response import Response
from django_filters.rest_framework import DjangoFilterBackend
from rest_framework.filters import SearchFilter, OrderingFilter

class PostListCreateAPIView(generics.ListCreateAPIView):
    serializer_class = PostSerializer
    permission_classes = [permissions.IsAuthenticatedOrReadOnly]
    filter_backends = [DjangoFilterBackend, SearchFilter, OrderingFilter]
    filterset_fields = ['category', 'status']
    search_fields = ['title', 'content']
    ordering_fields = ['created_at', 'views', 'title']
    ordering = ['-created_at']
    
    def get_queryset(self):
        if self.request.user.is_authenticated and self.request.user.is_staff:
            return Post.objects.all()
        return Post.published.all()
    
    def perform_create(self, serializer):
        serializer.save(author=self.request.user)

class PostDetailAPIView(generics.RetrieveUpdateDestroyAPIView):
    serializer_class = PostSerializer
    permission_classes = [permissions.IsAuthenticatedOrReadOnly]
    lookup_field = 'slug'
    
    def get_queryset(self):
        if self.request.user.is_authenticated and self.request.user.is_staff:
            return Post.objects.all()
        return Post.published.all()
    
    def get_object(self):
        obj = super().get_object()
        # Increment views for GET requests
        if self.request.method == 'GET':
            obj.increment_views()
        return obj

@api_view(['POST'])
@permission_classes([permissions.IsAuthenticated])
def toggle_like_api(request, post_id):
    try:
        post = Post.objects.get(id=post_id, status='published')
    except Post.DoesNotExist:
        return Response({'error': 'Post not found'}, status=status.HTTP_404_NOT_FOUND)
    
    like, created = PostLike.objects.get_or_create(
        user=request.user,
        post=post
    )
    
    if not created:
        like.delete()
        liked = False
    else:
        liked = True
    
    return Response({
        'liked': liked,
        'like_count': post.get_like_count()
    })

# API URLs
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from . import api_views

router = DefaultRouter()
# router.register(r'posts', api_views.PostViewSet)

urlpatterns = [
    path('api/', include(router.urls)),
    path('api/posts/', api_views.PostListCreateAPIView.as_view(), name='api_post_list'),
    path('api/posts/<slug:slug>/', api_views.PostDetailAPIView.as_view(), name='api_post_detail'),
    path('api/posts/<int:post_id>/like/', api_views.toggle_like_api, name='api_toggle_like'),
]
```

## Deployment and Production

### Production Settings
```python
# settings/production.py
from .base import *
import dj_database_url

DEBUG = False
ALLOWED_HOSTS = ['yourdomain.com', 'www.yourdomain.com']

# Database
DATABASES = {
    'default': dj_database_url.parse(os.environ.get('DATABASE_URL'))
}

# Static files (WhiteNoise)
MIDDLEWARE.insert(1, 'whitenoise.middleware.WhiteNoiseMiddleware')
STATICFILES_STORAGE = 'whitenoise.storage.CompressedManifestStaticFilesStorage'

# Security settings
SECURE_PROXY_SSL_HEADER = ('HTTP_X_FORWARDED_PROTO', 'https')
SECURE_SSL_REDIRECT = True
SECURE_HSTS_SECONDS = 31536000
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
SECURE_HSTS_PRELOAD = True
SECURE_CONTENT_TYPE_NOSNIFF = True
SECURE_BROWSER_XSS_FILTER = True
X_FRAME_OPTIONS = 'DENY'

# Email settings
EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
EMAIL_HOST = 'smtp.gmail.com'
EMAIL_PORT = 587
EMAIL_USE_TLS = True
EMAIL_HOST_USER = os.environ.get('EMAIL_HOST_USER')
EMAIL_HOST_PASSWORD = os.environ.get('EMAIL_HOST_PASSWORD')

# Logging
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'verbose': {
            'format': '{levelname} {asctime} {module} {process:d} {thread:d} {message}',
            'style': '{',
        },
    },
    'handlers': {
        'file': {
            'level': 'INFO',
            'class': 'logging.FileHandler',
            'filename': '/path/to/django.log',
            'formatter': 'verbose',
        },
    },
    'root': {
        'handlers': ['file'],
        'level': 'INFO',
    },
}

# Caching
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.redis.RedisCache',
        'LOCATION': os.environ.get('REDIS_URL', 'redis://127.0.0.1:6379/1'),
    }
}
```

### Management Commands
```python
# management/commands/populate_data.py
from django.core.management.base import BaseCommand
from blog.models import Category, Post
from django.contrib.auth.models import User

class Command(BaseCommand):
    help = 'Populate database with sample data'
    
    def add_arguments(self, parser):
        parser.add_argument('--posts', type=int, default=10, help='Number of posts to create')
    
    def handle(self, *args, **options):
        # Create categories
        categories = ['Technology', 'Science', 'Arts', 'Sports']
        for cat_name in categories:
            category, created = Category.objects.get_or_create(
                name=cat_name,
                defaults={'slug': cat_name.lower()}
            )
            if created:
                self.stdout.write(f'Created category: {cat_name}')
        
        # Create posts
        author = User.objects.first()
        if not author:
            self.stdout.write(self.style.ERROR('No users found. Create a user first.'))
            return
        
        for i in range(options['posts']):
            post, created = Post.objects.get_or_create(
                title=f'Sample Post {i+1}',
                defaults={
                    'slug': f'sample-post-{i+1}',
                    'content': f'This is the content for post {i+1}',
                    'author': author,
                    'status': 'published'
                }
            )
            if created:
                self.stdout.write(f'Created post: {post.title}')
        
        self.stdout.write(self.style.SUCCESS('Data population completed!'))

# Usage: python manage.py populate_data --posts 20
```

### Performance Optimization
```python
# Caching views
from django.views.decorators.cache import cache_page
from django.core.cache import cache

@cache_page(60 * 15)  # Cache for 15 minutes
def post_list(request):
    posts = Post.published.all()
    return render(request, 'blog/post_list.html', {'posts': posts})

# Manual caching
def get_popular_posts():
    cache_key = 'popular_posts'
    posts = cache.get(cache_key)
    if posts is None:
        posts = list(Post.published.order_by('-views')[:5])
        cache.set(cache_key, posts, 60 * 30)  # Cache for 30 minutes
    return posts

# Database optimization
# Use select_related for ForeignKey relationships
posts = Post.objects.select_related('author', 'category').all()

# Use prefetch_related for Many-to-Many relationships
posts = Post.objects.prefetch_related('tags', 'comments').all()

# Use only() to fetch specific fields
posts = Post.objects.only('title', 'slug', 'created_at').all()

# Use defer() to exclude fields
posts = Post.objects.defer('content').all()

# Bulk operations
Post.objects.bulk_create([
    Post(title='Post 1', content='Content 1', author=user),
    Post(title='Post 2', content='Content 2', author=user),
])

# Pagination for large datasets
from django.core.paginator import Paginator

def post_list(request):
    post_list = Post.published.all()
    paginator = Paginator(post_list, 25)  # Show 25 posts per page
    page_number = request.GET.get('page')
    posts = paginator.get_page(page_number)
    return render(request, 'blog/post_list.html', {'posts': posts})
```