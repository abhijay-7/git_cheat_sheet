# Flask In-Depth Cheatsheet

## Installation and Setup

```bash
# Install Flask
pip install flask

# Install with common extensions
pip install flask flask-sqlalchemy flask-migrate flask-login flask-wtf flask-mail

# Basic project structure
mkdir myflaskapp
cd myflaskapp
mkdir app static templates
touch app.py config.py requirements.txt
```

## Basic Flask Application

### Minimal Application
```python
# app.py
from flask import Flask, render_template, request, redirect, url_for, flash, jsonify

app = Flask(__name__)
app.config['SECRET_KEY'] = 'your-secret-key-here'

@app.route('/')
def index():
    return '<h1>Hello, World!</h1>'

@app.route('/user/<name>')
def user(name):
    return f'<h1>Hello, {name}!</h1>'

if __name__ == '__main__':
    app.run(debug=True)
```

### Application Factory Pattern
```python
# app/__init__.py
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from flask_migrate import Migrate
from flask_login import LoginManager
from config import Config

db = SQLAlchemy()
migrate = Migrate()
login = LoginManager()
login.login_view = 'auth.login'
login.login_message = 'Please log in to access this page.'

def create_app(config_class=Config):
    app = Flask(__name__)
    app.config.from_object(config_class)
    
    # Initialize extensions
    db.init_app(app)
    migrate.init_app(app, db)
    login.init_app(app)
    
    # Register blueprints
    from app.main import bp as main_bp
    app.register_blueprint(main_bp)
    
    from app.auth import bp as auth_bp
    app.register_blueprint(auth_bp, url_prefix='/auth')
    
    from app.api import bp as api_bp
    app.register_blueprint(api_bp, url_prefix='/api')
    
    return app

from app import models
```

### Configuration Management
```python
# config.py
import os
from dotenv import load_dotenv

basedir = os.path.abspath(os.path.dirname(__file__))
load_dotenv(os.path.join(basedir, '.env'))

class Config:
    SECRET_KEY = os.environ.get('SECRET_KEY') or 'hard-to-guess-string'
    SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL') or \
        'sqlite:///' + os.path.join(basedir, 'app.db')
    SQLALCHEMY_TRACK_MODIFICATIONS = False
    
    # Mail settings
    MAIL_SERVER = os.environ.get('MAIL_SERVER')
    MAIL_PORT = int(os.environ.get('MAIL_PORT') or 587)
    MAIL_USE_TLS = os.environ.get('MAIL_USE_TLS', 'true').lower() in ['true', 'on', '1']
    MAIL_USERNAME = os.environ.get('MAIL_USERNAME')
    MAIL_PASSWORD = os.environ.get('MAIL_PASSWORD')
    ADMINS = ['admin@example.com']
    
    # Pagination
    POSTS_PER_PAGE = 10
    
    # File uploads
    UPLOAD_FOLDER = 'uploads'
    MAX_CONTENT_LENGTH = 16 * 1024 * 1024  # 16MB

class DevelopmentConfig(Config):
    DEBUG = True

class ProductionConfig(Config):
    DEBUG = False

class TestingConfig(Config):
    TESTING = True
    SQLALCHEMY_DATABASE_URI = 'sqlite://'

config = {
    'development': DevelopmentConfig,
    'production': ProductionConfig,
    'testing': TestingConfig,
    'default': DevelopmentConfig
}
```

## Database Models with SQLAlchemy

### Model Definition
```python
# app/models.py
from datetime import datetime
from flask_sqlalchemy import SQLAlchemy
from flask_login import UserMixin
from werkzeug.security import generate_password_hash, check_password_hash
from app import db, login

class User(UserMixin, db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(64), index=True, unique=True, nullable=False)
    email = db.Column(db.String(120), index=True, unique=True, nullable=False)
    password_hash = db.Column(db.String(128))
    first_name = db.Column(db.String(64))
    last_name = db.Column(db.String(64))
    bio = db.Column(db.Text)
    avatar = db.Column(db.String(200))
    is_active = db.Column(db.Boolean, default=True)
    is_admin = db.Column(db.Boolean, default=False)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    last_seen = db.Column(db.DateTime, default=datetime.utcnow)
    
    posts = db.relationship('Post', backref='author', lazy='dynamic', cascade='all, delete-orphan')
    comments = db.relationship('Comment', backref='author', lazy='dynamic', cascade='all, delete-orphan')
    
    def __repr__(self):
        return f'<User {self.username}>'
    
    def set_password(self, password):
        self.password_hash = generate_password_hash(password)
    
    def check_password(self, password):
        return check_password_hash(self.password_hash, password)
    
    @property
    def full_name(self):
        return f"{self.first_name} {self.last_name}".strip()
    
    def to_dict(self):
        return {
            'id': self.id,
            'username': self.username,
            'email': self.email,
            'full_name': self.full_name,
            'bio': self.bio,
            'avatar': self.avatar,
            'created_at': self.created_at.isoformat() + 'Z',
            'last_seen': self.last_seen.isoformat() + 'Z'
        }

@login.user_loader
def load_user(id):
    return User.query.get(int(id))

# Association table for many-to-many relationship
post_tags = db.Table('post_tags',
    db.Column('post_id', db.Integer, db.ForeignKey('post.id'), primary_key=True),
    db.Column('tag_id', db.Integer, db.ForeignKey('tag.id'), primary_key=True)
)

class Post(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(200), nullable=False)
    slug = db.Column(db.String(200), unique=True, nullable=False)
    content = db.Column(db.Text, nullable=False)
    excerpt = db.Column(db.Text)
    featured_image = db.Column(db.String(200))
    status = db.Column(db.String(20), default='draft')  # draft, published, archived
    views = db.Column(db.Integer, default=0)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    updated_at = db.Column(db.DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)
    published_at = db.Column(db.DateTime)
    
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'), nullable=False)
    category_id = db.Column(db.Integer, db.ForeignKey('category.id'))
    
    tags = db.relationship('Tag', secondary=post_tags, lazy='subquery',
                          backref=db.backref('posts', lazy=True))
    comments = db.relationship('Comment', backref='post', lazy='dynamic', cascade='all, delete-orphan')
    
    def __repr__(self):
        return f'<Post {self.title}>'
    
    @property
    def is_published(self):
        return self.status == 'published'
    
    def increment_views(self):
        self.views += 1
        db.session.commit()
    
    def to_dict(self):
        return {
            'id': self.id,
            'title': self.title,
            'slug': self.slug,
            'content': self.content,
            'excerpt': self.excerpt,
            'status': self.status,
            'views': self.views,
            'created_at': self.created_at.isoformat() + 'Z',
            'updated_at': self.updated_at.isoformat() + 'Z',
            'author': self.author.username,
            'category': self.category.name if self.category else None,
            'tags': [tag.name for tag in self.tags]
        }

class Category(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100), unique=True, nullable=False)
    slug = db.Column(db.String(100), unique=True, nullable=False)
    description = db.Column(db.Text)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    
    posts = db.relationship('Post', backref='category', lazy='dynamic')
    
    def __repr__(self):
        return f'<Category {self.name}>'

class Tag(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(50), unique=True, nullable=False)
    slug = db.Column(db.String(50), unique=True, nullable=False)
    
    def __repr__(self):
        return f'<Tag {self.name}>'

class Comment(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    content = db.Column(db.Text, nullable=False)
    is_approved = db.Column(db.Boolean, default=False)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'), nullable=False)
    post_id = db.Column(db.Integer, db.ForeignKey('post.id'), nullable=False)
    parent_id = db.Column(db.Integer, db.ForeignKey('comment.id'))
    
    replies = db.relationship('Comment', backref=db.backref('parent', remote_side=[id]))
    
    def __repr__(self):
        return f'<Comment {self.id}>'
```

### Database Operations
```python
# Database initialization and migrations
from flask_migrate import Migrate

# Initialize database
flask db init

# Create migration
flask db migrate -m "Initial migration"

# Apply migration
flask db upgrade

# Downgrade
flask db downgrade

# Database operations in views
from app import db
from app.models import User, Post

# Create
user = User(username='john', email='john@example.com')
user.set_password('password123')
db.session.add(user)
db.session.commit()

# Read
users = User.query.all()
user = User.query.filter_by(username='john').first()
posts = Post.query.filter(Post.status == 'published').order_by(Post.created_at.desc()).all()

# Update
user = User.query.get(1)
user.bio = 'Updated bio'
db.session.commit()

# Delete
user = User.query.get(1)
db.session.delete(user)
db.session.commit()

# Complex queries
from sqlalchemy import and_, or_, func

# Join queries
posts_with_authors = db.session.query(Post, User).join(User).all()

# Aggregation
user_post_counts = db.session.query(
    User.username, 
    func.count(Post.id).label('post_count')
).join(Post).group_by(User.username).all()

# Pagination
page = request.args.get('page', 1, type=int)
posts = Post.query.filter_by(status='published').paginate(
    page=page, per_page=10, error_out=False
)
```

## Views and Routing

### Route Patterns and Methods
```python
# app/main/routes.py
from flask import Blueprint, render_template, request, redirect, url_for, flash, jsonify, abort
from flask_login import login_required, current_user
from app import db
from app.models import Post, Category, User
from app.main.forms import PostForm, CommentForm

bp = Blueprint('main', __name__)

@bp.route('/')
@bp.route('/index')
def index():
    page = request.args.get('page', 1, type=int)
    posts = Post.query.filter_by(status='published').order_by(
        Post.created_at.desc()
    ).paginate(
        page=page, per_page=10, error_out=False
    )
    
    categories = Category.query.all()
    return render_template('index.html', posts=posts, categories=categories)

@bp.route('/post/<slug>')
def post(slug):
    post = Post.query.filter_by(slug=slug, status='published').first_or_404()
    post.increment_views()
    
    # Get related posts
    related_posts = Post.query.filter(
        Post.category_id == post.category_id,
        Post.id != post.id,
        Post.status == 'published'
    ).limit(3).all()
    
    comments = Comment.query.filter_by(
        post_id=post.id, 
        is_approved=True, 
        parent_id=None
    ).order_by(Comment.created_at.desc()).all()
    
    form = CommentForm()
    return render_template('post.html', post=post, related_posts=related_posts, 
                         comments=comments, form=form)

@bp.route('/create_post', methods=['GET', 'POST'])
@login_required
def create_post():
    form = PostForm()
    if form.validate_on_submit():
        post = Post(
            title=form.title.data,
            slug=generate_slug(form.title.data),
            content=form.content.data,
            excerpt=form.excerpt.data,
            status=form.status.data,
            category_id=form.category.data,
            author=current_user
        )
        
        # Handle tags
        tag_names = [tag.strip() for tag in form.tags.data.split(',') if tag.strip()]
        for tag_name in tag_names:
            tag = Tag.query.filter_by(name=tag_name).first()
            if not tag:
                tag = Tag(name=tag_name, slug=slugify(tag_name))
                db.session.add(tag)
            post.tags.append(tag)
        
        db.session.add(post)
        db.session.commit()
        flash('Your post has been created!', 'success')
        return redirect(url_for('main.post', slug=post.slug))
    
    return render_template('create_post.html', form=form)

@bp.route('/edit_post/<int:id>', methods=['GET', 'POST'])
@login_required
def edit_post(id):
    post = Post.query.get_or_404(id)
    if post.author != current_user and not current_user.is_admin:
        abort(403)
    
    form = PostForm(obj=post)
    if form.validate_on_submit():
        post.title = form.title.data
        post.content = form.content.data
        post.excerpt = form.excerpt.data
        post.status = form.status.data
        post.category_id = form.category.data
        
        # Update tags
        post.tags.clear()
        tag_names = [tag.strip() for tag in form.tags.data.split(',') if tag.strip()]
        for tag_name in tag_names:
            tag = Tag.query.filter_by(name=tag_name).first()
            if not tag:
                tag = Tag(name=tag_name, slug=slugify(tag_name))
                db.session.add(tag)
            post.tags.append(tag)
        
        db.session.commit()
        flash('Your post has been updated!', 'success')
        return redirect(url_for('main.post', slug=post.slug))
    
    return render_template('edit_post.html', form=form, post=post)

@bp.route('/category/<slug>')
def category(slug):
    category = Category.query.filter_by(slug=slug).first_or_404()
    page = request.args.get('page', 1, type=int)
    posts = Post.query.filter_by(
        category=category, 
        status='published'
    ).order_by(Post.created_at.desc()).paginate(
        page=page, per_page=10, error_out=False
    )
    return render_template('category.html', category=category, posts=posts)

@bp.route('/search')
def search():
    query = request.args.get('q', '')
    if not query:
        return redirect(url_for('main.index'))
    
    page = request.args.get('page', 1, type=int)
    posts = Post.query.filter(
        Post.title.contains(query) | Post.content.contains(query),
        Post.status == 'published'
    ).order_by(Post.created_at.desc()).paginate(
        page=page, per_page=10, error_out=False
    )
    
    return render_template('search.html', posts=posts, query=query)

# AJAX endpoints
@bp.route('/add_comment', methods=['POST'])
@login_required
def add_comment():
    post_id = request.form.get('post_id')
    content = request.form.get('content')
    parent_id = request.form.get('parent_id', type=int)
    
    if not post_id or not content:
        return jsonify({'error': 'Missing required fields'}), 400
    
    post = Post.query.get_or_404(post_id)
    comment = Comment(
        content=content,
        author=current_user,
        post=post,
        parent_id=parent_id
    )
    
    db.session.add(comment)
    db.session.commit()
    
    return jsonify({
        'success': True,
        'comment': {
            'id': comment.id,
            'content': comment.content,
            'author': comment.author.username,
            'created_at': comment.created_at.strftime('%Y-%m-%d %H:%M')
        }
    })

@bp.route('/delete_post/<int:id>', methods=['POST'])
@login_required
def delete_post(id):
    post = Post.query.get_or_404(id)
    if post.author != current_user and not current_user.is_admin:
        abort(403)
    
    db.session.delete(post)
    db.session.commit()
    flash('Post deleted successfully!', 'success')
    return redirect(url_for('main.index'))

# Error handlers
@bp.errorhandler(404)
def not_found_error(error):
    return render_template('errors/404.html'), 404

@bp.errorhandler(500)
def internal_error(error):
    db.session.rollback()
    return render_template('errors/500.html'), 500

# Utility functions
def generate_slug(title):
    import re
    slug = re.sub(r'[^\w\s-]', '', title.lower())
    slug = re.sub(r'[-\s]+', '-', slug)
    return slug.strip('-')

def slugify(text):
    import re
    text = re.sub(r'[^\w\s-]', '', text.lower())
    return re.sub(r'[-\s]+', '-', text).strip('-')
```

### Request Handling and Context
```python
# Request object usage
from flask import request, g, session

@bp.before_request
def before_request():
    # Code that runs before each request
    if current_user.is_authenticated:
        current_user.last_seen = datetime.utcnow()
        db.session.commit()
    
    # Store commonly used data in g
    g.search_form = SearchForm()

@bp.route('/upload', methods=['POST'])
def upload_file():
    if 'file' not in request.files:
        flash('No file selected', 'error')
        return redirect(request.url)
    
    file = request.files['file']
    if file.filename == '':
        flash('No file selected', 'error')
        return redirect(request.url)
    
    if file and allowed_file(file.filename):
        filename = secure_filename(file.filename)
        file.save(os.path.join(app.config['UPLOAD_FOLDER'], filename))
        flash('File uploaded successfully!', 'success')
        return redirect(url_for('main.index'))

def allowed_file(filename):
    ALLOWED_EXTENSIONS = {'txt', 'pdf', 'png', 'jpg', 'jpeg', 'gif'}
    return '.' in filename and \
           filename.rsplit('.', 1)[1].lower() in ALLOWED_EXTENSIONS

# Session management
@bp.route('/set_preference')
def set_preference():
    session['theme'] = request.args.get('theme', 'light')
    return redirect(request.referrer or url_for('main.index'))

# Cookie handling
from flask import make_response

@bp.route('/set_cookie')
def set_cookie():
    response = make_response('Cookie set!')
    response.set_cookie('user_preference', 'dark_mode', max_age=60*60*24*30)  # 30 days
    return response

# JSON responses
@bp.route('/api/posts')
def api_posts():
    posts = Post.query.filter_by(status='published').all()
    return jsonify([post.to_dict() for post in posts])

# Custom response headers
@bp.after_request
def after_request(response):
    response.headers['X-Custom-Header'] = 'Flask App'
    return response
```

## Forms and Validation with Flask-WTF

### Form Classes
```python
# app/main/forms.py
from flask_wtf import FlaskForm
from flask_wtf.file import FileField, FileAllowed
from wtforms import StringField, TextAreaField, SelectField, BooleanField, SubmitField
from wtforms.validators import DataRequired, Length, Email, ValidationError, Optional
from app.models import User, Category

class PostForm(FlaskForm):
    title = StringField('Title', validators=[DataRequired(), Length(1, 200)])
    content = TextAreaField('Content', validators=[DataRequired()])
    excerpt = TextAreaField('Excerpt', validators=[Length(0, 300)])
    category = SelectField('Category', coerce=int, validators=[Optional()])
    tags = StringField('Tags', validators=[Optional()])
    status = SelectField('Status', choices=[
        ('draft', 'Draft'),
        ('published', 'Published'),
        ('archived', 'Archived')
    ], default='draft')
    featured_image = FileField('Featured Image', validators=[
        FileAllowed(['jpg', 'png', 'gif'], 'Images only!')
    ])
    submit = SubmitField('Save Post')
    
    def __init__(self, *args, **kwargs):
        super(PostForm, self).__init__(*args, **kwargs)
        self.category.choices = [(0, 'Select Category')] + [
            (c.id, c.name) for c in Category.query.all()
        ]
    
    def validate_title(self, title):
        # Check if title already exists (for new posts)
        if not hasattr(self, 'post') or self.post.title != title.data:
            post = Post.query.filter_by(title=title.data).first()
            if post:
                raise ValidationError('A post with this title already exists.')

class CommentForm(FlaskForm):
    content = TextAreaField('Comment', validators=[
        DataRequired(),
        Length(min=10, max=1000, message='Comment must be between 10 and 1000 characters.')
    ])
    submit = SubmitField('Add Comment')

class SearchForm(FlaskForm):
    q = StringField('Search', validators=[DataRequired()])
    
    def __init__(self, *args, **kwargs):
        if 'formdata' not in kwargs:
            kwargs['formdata'] = request.args
        if 'csrf_enabled' not in kwargs:
            kwargs['csrf_enabled'] = False
        super(SearchForm, self).__init__(*args, **kwargs)

class UserProfileForm(FlaskForm):
    username = StringField('Username', validators=[DataRequired(), Length(1, 64)])
    email = StringField('Email', validators=[DataRequired(), Email()])
    first_name = StringField('First Name', validators=[Length(0, 64)])
    last_name = StringField('Last Name', validators=[Length(0, 64)])
    bio = TextAreaField('Bio', validators=[Length(0, 500)])
    avatar = FileField('Avatar', validators=[
        FileAllowed(['jpg', 'png', 'gif'], 'Images only!')
    ])
    submit = SubmitField('Update Profile')
    
    def __init__(self, original_username, original_email, *args, **kwargs):
        super(UserProfileForm, self).__init__(*args, **kwargs)
        self.original_username = original_username
        self.original_email = original_email
    
    def validate_username(self, username):
        if username.data != self.original_username:
            user = User.query.filter_by(username=self.username.data).first()
            if user is not None:
                raise ValidationError('Please use a different username.')
    
    def validate_email(self, email):
        if email.data != self.original_email:
            user = User.query.filter_by(email=self.email.data).first()
            if user is not None:
                raise ValidationError('Please use a different email address.')

# Custom validators
def unique_username(form, field):
    user = User.query.filter_by(username=field.data).first()
    if user:
        raise ValidationError('Username already exists.')

def strong_password(form, field):
    password = field.data
    if len(password) < 8:
        raise ValidationError('Password must be at least 8 characters long.')
    if not any(c.isupper() for c in password):
        raise ValidationError('Password must contain at least one uppercase letter.')
    if not any(c.islower() for c in password):
        raise ValidationError('Password must contain at least one lowercase letter.')
    if not any(c.isdigit() for c in password):
        raise ValidationError('Password must contain at least one digit.')

class RegistrationForm(FlaskForm):
    username = StringField('Username', validators=[
        DataRequired(),
        Length(4, 20),
        unique_username
    ])
    email = StringField('Email', validators=[DataRequired(), Email()])
    password = PasswordField('Password', validators=[DataRequired(), strong_password])
    password2 = PasswordField('Repeat Password', validators=[
        DataRequired(),
        EqualTo('password')
    ])
    submit = SubmitField('Register')
```

### Form Handling in Templates
```html
<!-- templates/forms/form_macros.html -->
{% macro render_field(field, class_="form-control") %}
    <div class="mb-3">
        {{ field.label(class="form-label") }}
        {% if field.type == "BooleanField" %}
            {{ field(class="form-check-input") }}
        {% elif field.type == "SelectField" %}
            {{ field(class="form-select") }}
        {% elif field.type == "TextAreaField" %}
            {{ field(class="form-control", rows="5") }}
        {% else %}
            {{ field(class=class_) }}
        {% endif %}
        
        {% if field.errors %}
            <div class="invalid-feedback d-block">
                {% for error in field.errors %}
                    {{ error }}
                {% endfor %}
            </div>
        {% endif %}
        
        {% if field.description %}
            <div class="form-text">{{ field.description }}</div>
        {% endif %}
    </div>
{% endmacro %}

<!-- templates/create_post.html -->
{% extends "base.html" %}
{% from 'forms/form_macros.html' import render_field %}

{% block content %}
<div class="container">
    <h1>Create New Post</h1>
    
    <form method="POST" enctype="multipart/form-data">
        {{ form.hidden_tag() }}
        
        {{ render_field(form.title) }}
        {{ render_field(form.content) }}
        {{ render_field(form.excerpt) }}
        {{ render_field(form.category) }}
        {{ render_field(form.tags) }}
        {{ render_field(form.status) }}
        {{ render_field(form.featured_image) }}
        
        <div class="mb-3">
            {{ form.submit(class="btn btn-primary") }}
            <a href="{{ url_for('main.index') }}" class="btn btn-secondary">Cancel</a>
        </div>
    </form>
</div>

<script>
// Client-side validation
document.addEventListener('DOMContentLoaded', function() {
    const form = document.querySelector('form');
    const titleField = document.querySelector('#title');
    const contentField = document.querySelector('#content');
    
    form.addEventListener('submit', function(e) {
        if (titleField.value.length < 5) {
            e.preventDefault();
            alert('Title must be at least 5 characters long.');
            titleField.focus();
        }
        
        if (contentField.value.length < 50) {
            e.preventDefault();
            alert('Content must be at least 50 characters long.');
            contentField.focus();
        }
    });
});
</script>
{% endblock %}
```

## Authentication with Flask-Login

### User Authentication System
```python
# app/auth/routes.py
from flask import Blueprint, render_template, redirect, url_for, flash, request
from flask_login import login_user, logout_user, login_required, current_user
from werkzeug.urls import url_parse
from app import db
from app.auth.forms import LoginForm, RegistrationForm, ResetPasswordRequestForm, ResetPasswordForm
from app.models import User
from app.auth.email import send_password_reset_email

bp = Blueprint('auth', __name__)

@bp.route('/login', methods=['GET', 'POST'])
def login():
    if current_user.is_authenticated:
        return redirect(url_for('main.index'))
    
    form = LoginForm()
    if form.validate_on_submit():
        user = User.query.filter_by(username=form.username.data).first()
        
        if user is None or not user.check_password(form.password.data):
            flash('Invalid username or password', 'error')
            return redirect(url_for('auth.login'))
        
        if not user.is_active:
            flash('Your account has been deactivated. Please contact support.', 'error')
            return redirect(url_for('auth.login'))
        
        login_user(user, remember=form.remember_me.data)
        
        # Redirect to next page or index
        next_page = request.args.get('next')
        if not next_page or url_parse(next_page).netloc != '':
            next_page = url_for('main.index')
        
        flash(f'Welcome back, {user.username}!', 'success')
        return redirect(next_page)
    
    return render_template('auth/login.html', title='Sign In', form=form)

@bp.route('/logout')
@login_required
def logout():
    logout_user()
    flash('You have been logged out.', 'info')
    return redirect(url_for('main.index'))

@bp.route('/register', methods=['GET', 'POST'])
def register():
    if current_user.is_authenticated:
        return redirect(url_for('main.index'))
    
    form = RegistrationForm()
    if form.validate_on_submit():
        user = User(
            username=form.username.data,
            email=form.email.data
        )
        user.set_password(form.password.data)
        
        db.session.add(user)
        db.session.commit()
        
        flash('Congratulations, you are now registered!', 'success')
        return redirect(url_for('auth.login'))
    
    return render_template('auth/register.html', title='Register', form=form)

@bp.route('/profile')
@login_required
def profile():
    posts = Post.query.filter_by(author=current_user).order_by(
        Post.created_at.desc()
    ).all()
    return render_template('auth/profile.html', user=current_user, posts=posts)

@bp.route('/edit_profile', methods=['GET', 'POST'])
@login_required
def edit_profile():
    form = UserProfileForm(current_user.username, current_user.email)
    
    if form.validate_on_submit():
        current_user.username = form.username.data
        current_user.email = form.email.data
        current_user.first_name = form.first_name.data
        current_user.last_name = form.last_name.data
        current_user.bio = form.bio.data
        
        # Handle avatar upload
        if form.avatar.data:
            avatar_file = form.avatar.data
            filename = secure_filename(avatar_file.filename)
            avatar_file.save(os.path.join(app.config['UPLOAD_FOLDER'], 'avatars', filename))
            current_user.avatar = filename
        
        db.session.commit()
        flash('Your profile has been updated.', 'success')
        return redirect(url_for('auth.profile'))
    
    elif request.method == 'GET':
        form.username.data = current_user.username
        form.email.data = current_user.email
        form.first_name.data = current_user.first_name
        form.last_name.data = current_user.last_name
        form.bio.data = current_user.bio
    
    return render_template('auth/edit_profile.html', title='Edit Profile', form=form)

# Password reset functionality
@bp.route('/reset_password_request', methods=['GET', 'POST'])
def reset_password_request():
    if current_user.is_authenticated:
        return redirect(url_for('main.index'))
    
    form = ResetPasswordRequestForm()
    if form.validate_on_submit():
        user = User.query.filter_by(email=form.email.data).first()
        if user:
            send_password_reset_email(user)
        flash('Check your email for the instructions to reset your password', 'info')
        return redirect(url_for('auth.login'))
    
    return render_template('auth/reset_password_request.html', title='Reset Password', form=form)

@bp.route('/reset_password/<token>', methods=['GET', 'POST'])
def reset_password(token):
    if current_user.is_authenticated:
        return redirect(url_for('main.index'))
    
    user = User.verify_reset_password_token(token)
    if not user:
        return redirect(url_for('main.index'))
    
    form = ResetPasswordForm()
    if form.validate_on_submit():
        user.set_password(form.password.data)
        db.session.commit()
        flash('Your password has been reset.', 'success')
        return redirect(url_for('auth.login'))
    
    return render_template('auth/reset_password.html', form=form)
```

### Role-Based Access Control
```python
# Custom decorators for role-based access
from functools import wraps
from flask import abort
from flask_login import current_user

def admin_required(f):
    @wraps(f)
    def decorated_function(*args, **kwargs):
        if not current_user.is_authenticated or not current_user.is_admin:
            abort(403)
        return f(*args, **kwargs)
    return decorated_function

def author_required(f):
    """Allow access to post authors or admins"""
    @wraps(f)
    def decorated_function(*args, **kwargs):
        post_id = kwargs.get('id')
        if post_id:
            post = Post.query.get_or_404(post_id)
            if current_user != post.author and not current_user.is_admin:
                abort(403)
        return f(*args, **kwargs)
    return decorated_function

# Usage in views
@bp.route('/admin')
@login_required
@admin_required
def admin_dashboard():
    users = User.query.all()
    posts = Post.query.all()
    return render_template('admin/dashboard.html', users=users, posts=posts)

@bp.route('/edit_post/<int:id>')
@login_required
@author_required
def edit_post(id):
    post = Post.query.get_or_404(id)
    # Edit logic here
    pass
```

## Templates with Jinja2

### Template Inheritance and Structure
```html
<!-- templates/base.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}Flask Blog{% endblock %}</title>
    
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" rel="stylesheet">
    <link href="{{ url_for('static', filename='css/style.css') }}" rel="stylesheet">
    {% block styles %}{% endblock %}
</head>
<body>
    <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
        <div class="container">
            <a class="navbar-brand" href="{{ url_for('main.index') }}">Flask Blog</a>
            
            <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav">
                <span class="navbar-toggler-icon"></span>
            </button>
            
            <div class="collapse navbar-collapse" id="navbarNav">
                <ul class="navbar-nav me-auto">
                    <li class="nav-item">
                        <a class="nav-link" href="{{ url_for('main.index') }}">Home</a>
                    </li>
                    {% if current_user.is_authenticated %}
                        <li class="nav-item">
                            <a class="nav-link" href="{{ url_for('main.create_post') }}">Write</a>
                        </li>
                    {% endif %}
                </ul>
                
                <!-- Search form -->
                <form class="d-flex me-3" method="GET" action="{{ url_for('main.search') }}">
                    <input class="form-control me-2" type="search" name="q" placeholder="Search..." 
                           value="{{ request.args.get('q', '') }}">
                    <button class="btn btn-outline-light" type="submit">Search</button>
                </form>
                
                <ul class="navbar-nav">
                    {% if current_user.is_authenticated %}
                        <li class="nav-item dropdown">
                            <a class="nav-link dropdown-toggle" href="#" id="navbarDropdown" 
                               role="button" data-bs-toggle="dropdown">
                                {{ current_user.username }}
                            </a>
                            <ul class="dropdown-menu">
                                <li><a class="dropdown-item" href="{{ url_for('auth.profile') }}">Profile</a></li>
                                <li><a class="dropdown-item" href="{{ url_for('auth.edit_profile') }}">Settings</a></li>
                                {% if current_user.is_admin %}
                                    <li><hr class="dropdown-divider"></li>
                                    <li><a class="dropdown-item" href="{{ url_for('admin.dashboard') }}">Admin</a></li>
                                {% endif %}
                                <li><hr class="dropdown-divider"></li>
                                <li><a class="dropdown-item" href="{{ url_for('auth.logout') }}">Logout</a></li>
                            </ul>
                        </li>
                    {% else %}
                        <li class="nav-item">
                            <a class="nav-link" href="{{ url_for('auth.login') }}">Login</a>
                        </li>
                        <li class="nav-item">
                            <a class="nav-link" href="{{ url_for('auth.register') }}">Register</a>
                        </li>
                    {% endif %}
                </ul>
            </div>
        </div>
    </nav>
    
    <main class="container mt-4">
        {% with messages = get_flashed_messages(with_categories=true) %}
            {% if messages %}
                {% for category, message in messages %}
                    <div class="alert alert-{{ 'danger' if category == 'error' else category }} alert-dismissible fade show">
                        {{ message }}
                        <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
                    </div>
                {% endfor %}
            {% endif %}
        {% endwith %}
        
        {% block content %}{% endblock %}
    </main>
    
    <footer class="bg-dark text-light mt-5 py-4">
        <div class="container">
            <div class="row">
                <div class="col-md-6">
                    <p>&copy; {{ moment().format('YYYY') }} Flask Blog. All rights reserved.</p>
                </div>
                <div class="col-md-6 text-end">
                    <a href="#" class="text-light me-3">Privacy</a>
                    <a href="#" class="text-light">Terms</a>
                </div>
            </div>
        </div>
    </footer>
    
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/js/bootstrap.bundle.min.js"></script>
    <script src="{{ url_for('static', filename='js/main.js') }}"></script>
    {% block scripts %}{% endblock %}
</body>
</html>

<!-- templates/index.html -->
{% extends "base.html" %}

{% block content %}
<div class="row">
    <div class="col-md-8">
        <h1>Latest Posts</h1>
        
        {% for post in posts.items %}
            <article class="card mb-4">
                {% if post.featured_image %}
                    <img src="{{ url_for('static', filename='uploads/' + post.featured_image) }}" 
                         class="card-img-top" alt="{{ post.title }}">
                {% endif %}
                
                <div class="card-body">
                    <h2 class="card-title">
                        <a href="{{ url_for('main.post', slug=post.slug) }}" class="text-decoration-none">
                            {{ post.title }}
                        </a>
                    </h2>
                    
                    <p class="card-text">
                        {{ post.excerpt or post.content|truncate(150) }}
                    </p>
                    
                    <div class="d-flex justify-content-between align-items-center">
                        <small class="text-muted">
                            By <strong>{{ post.author.full_name or post.author.username }}</strong>
                            on {{ moment(post.created_at).format('MMMM Do, YYYY') }}
                        </small>
                        
                        <div>
                            {% if post.category %}
                                <span class="badge bg-primary">{{ post.category.name }}</span>
                            {% endif %}
                            <span class="badge bg-secondary">{{ post.views }} views</span>
                        </div>
                    </div>
                    
                    {% if post.tags %}
                        <div class="mt-2">
                            {% for tag in post.tags %}
                                <span class="badge bg-light text-dark me-1">#{{ tag.name }}</span>
                            {% endfor %}
                        </div>
                    {% endif %}
                </div>
            </article>
        {% else %}
            <div class="alert alert-info">
                <h4>No posts yet!</h4>
                <p>Be the first to share something interesting.</p>
                {% if current_user.is_authenticated %}
                    <a href="{{ url_for('main.create_post') }}" class="btn btn-primary">Write a Post</a>
                {% endif %}
            </div>
        {% endfor %}
        
        <!-- Pagination -->
        {% if posts.pages > 1 %}
            <nav aria-label="Posts pagination">
                <ul class="pagination justify-content-center">
                    {% if posts.has_prev %}
                        <li class="page-item">
                            <a class="page-link" href="{{ url_for('main.index', page=1) }}">First</a>
                        </li>
                        <li class="page-item">
                            <a class="page-link" href="{{ url_for('main.index', page=posts.prev_num) }}">Previous</a>
                        </li>
                    {% endif %}
                    
                    {% for page_num in posts.iter_pages() %}
                        {% if page_num %}
                            {% if page_num != posts.page %}
                                <li class="page-item">
                                    <a class="page-link" href="{{ url_for('main.index', page=page_num) }}">{{ page_num }}</a>
                                </li>
                            {% else %}
                                <li class="page-item active">
                                    <span class="page-link">{{ page_num }}</span>
                                </li>
                            {% endif %}
                        {% else %}
                            <li class="page-item disabled">
                                <span class="page-link">...</span>
                            </li>
                        {% endif %}
                    {% endfor %}
                    
                    {% if posts.has_next %}
                        <li class="page-item">
                            <a class="page-link" href="{{ url_for('main.index', page=posts.next_num) }}">Next</a>
                        </li>
                        <li class="page-item">
                            <a class="page-link" href="{{ url_for('main.index', page=posts.pages) }}">Last</a>
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
                    <a href="{{ url_for('main.category', slug=category.slug) }}" 
                       class="btn btn-outline-primary btn-sm me-2 mb-2">
                        {{ category.name }}
                    </a>
                {% endfor %}
            </div>
        </div>
        
        <div class="card mt-4">
            <div class="card-header">
                <h5>Recent Posts</h5>
            </div>
            <div class="card-body">
                {% for post in recent_posts %}
                    <div class="mb-2">
                        <a href="{{ url_for('main.post', slug=post.slug) }}" class="text-decoration-none">
                            {{ post.title }}
                        </a>
                        <small class="text-muted d-block">{{ moment(post.created_at).fromNow() }}</small>
                    </div>
                {% endfor %}
            </div>
        </div>
    </div>
</div>
{% endblock %}
```

### Custom Template Filters and Functions
```python
# app/__init__.py - Register custom filters
@app.template_filter('datetime')
def datetime_filter(datetime):
    return datetime.strftime('%Y-%m-%d %H:%M')

@app.template_filter('markdown')
def markdown_filter(text):
    import markdown
    return Markup(markdown.markdown(text))

# Template context processors
@app.context_processor
def inject_now():
    return {'now': datetime.utcnow()}

@app.context_processor
def inject_config():
    return {'config': app.config}

# Custom template functions
@app.template_global()
def get_post_count():
    return Post.query.filter_by(status='published').count()

@app.template_global()
def get_user_count():
    return User.query.count()

# Usage in templates:
# {{ post.created_at|datetime }}
# {{ post.content|markdown|safe }}
# {{ get_post_count() }} posts published
```

## API Development with Flask-RESTful

### RESTful API Endpoints
```python
# app/api/routes.py
from flask import Blueprint, jsonify, request, g
from flask_login import login_required, current_user
from app import db
from app.models import Post, User, Category
from app.api.auth import token_auth
from app.api.errors import bad_request, error_response

bp = Blueprint('api', __name__)

# Token-based authentication
@bp.route('/tokens', methods=['POST'])
def get_token():
    username = request.json.get('username')
    password = request.json.get('password')
    
    if not username or not password:
        return bad_request('Username and password required')
    
    user = User.query.filter_by(username=username).first()
    if not user or not user.check_password(password):
        return error_response(401, 'Invalid credentials')
    
    token = user.get_token()
    db.session.commit()
    return jsonify({'token': token})

@bp.route('/tokens', methods=['DELETE'])
@token_auth.login_required
def revoke_token():
    g.current_user.revoke_token()
    db.session.commit()
    return '', 204

# Posts API
@bp.route('/posts', methods=['GET'])
def get_posts():
    page = request.args.get('page', 1, type=int)
    per_page = min(request.args.get('per_page', 10, type=int), 100)
    
    posts = Post.query.filter_by(status='published').order_by(
        Post.created_at.desc()
    ).paginate(
        page=page, per_page=per_page, error_out=False
    )
    
    return jsonify({
        'posts': [post.to_dict() for post in posts.items],
        'pagination': {
            'page': page,
            'pages': posts.pages,
            'per_page': per_page,
            'total': posts.total,
            'has_next': posts.has_next,
            'has_prev': posts.has_prev
        }
    })

@bp.route('/posts/<int:id>', methods=['GET'])
def get_post(id):
    post = Post.query.filter_by(id=id, status='published').first()
    if not post:
        return error_response(404, 'Post not found')
    
    post.increment_views()
    return jsonify(post.to_dict())

@bp.route('/posts', methods=['POST'])
@token_auth.login_required
def create_post():
    data = request.get_json() or {}
    
    required_fields = ['title', 'content']
    for field in required_fields:
        if field not in data:
            return bad_request(f'Must include {field} field')
    
    post = Post()
    post.from_dict(data, new_post=True)
    post.author = g.current_user
    
    db.session.add(post)
    db.session.commit()
    
    response = jsonify(post.to_dict())
    response.status_code = 201
    response.headers['Location'] = url_for('api.get_post', id=post.id)
    return response

@bp.route('/posts/<int:id>', methods=['PUT'])
@token_auth.login_required
def update_post(id):
    post = Post.query.get_or_404(id)
    
    if g.current_user != post.author and not g.current_user.is_admin:
        return error_response(403, 'Access denied')
    
    data = request.get_json() or {}
    post.from_dict(data, new_post=False)
    db.session.commit()
    
    return jsonify(post.to_dict())

@bp.route('/posts/<int:id>', methods=['DELETE'])
@token_auth.login_required
def delete_post(id):
    post = Post.query.get_or_404(id)
    
    if g.current_user != post.author and not g.current_user.is_admin:
        return error_response(403, 'Access denied')
    
    db.session.delete(post)
    db.session.commit()
    
    return '', 204

# Users API
@bp.route('/users', methods=['GET'])
def get_users():
    page = request.args.get('page', 1, type=int)
    per_page = min(request.args.get('per_page', 10, type=int), 100)
    
    users = User.query.paginate(
        page=page, per_page=per_page, error_out=False
    )
    
    return jsonify({
        'users': [user.to_dict() for user in users.items],
        'pagination': {
            'page': page,
            'pages': users.pages,
            'per_page': per_page,
            'total': users.total
        }
    })

@bp.route('/users/<int:id>', methods=['GET'])
def get_user(id):
    user = User.query.get_or_404(id)
    return jsonify(user.to_dict())

@bp.route('/users', methods=['POST'])
def create_user():
    data = request.get_json() or {}
    
    required_fields = ['username', 'email', 'password']
    for field in required_fields:
        if field not in data:
            return bad_request(f'Must include {field} field')
    
    if User.query.filter_by(username=data['username']).first():
        return bad_request('Username already exists')
    
    if User.query.filter_by(email=data['email']).first():
        return bad_request('Email already exists')
    
    user = User()
    user.from_dict(data, new_user=True)
    
    db.session.add(user)
    db.session.commit()
    
    response = jsonify(user.to_dict())
    response.status_code = 201
    response.headers['Location'] = url_for('api.get_user', id=user.id)
    return response

# API Error handling
@bp.errorhandler(404)
def not_found(error):
    return error_response(404, 'Resource not found')

@bp.errorhandler(500)
def internal_error(error):
    db.session.rollback()
    return error_response(500, 'Internal server error')
```

### API Authentication
```python
# app/api/auth.py
from flask_httpauth import HTTPTokenAuth
from app.models import User

token_auth = HTTPTokenAuth()

@token_auth.verify_token
def verify_token(token):
    user = User.check_token(token) if token else None
    return user

@token_auth.error_handler
def token_auth_error(status):
    return error_response(status, 'Invalid token')

# Add token methods to User model
class User(UserMixin, db.Model):
    # ... existing fields ...
    token = db.Column(db.String(32), index=True, unique=True)
    token_expiration = db.Column(db.DateTime)
    
    def get_token(self, expires_in=3600):
        now = datetime.utcnow()
        if self.token and self.token_expiration > now + timedelta(seconds=60):
            return self.token
        self.token = base64.b64encode(os.urandom(24)).decode('utf-8')
        self.token_expiration = now + timedelta(seconds=expires_in)
        db.session.add(self)
        return self.token
    
    def revoke_token(self):
        self.token_expiration = datetime.utcnow() - timedelta(seconds=1)
    
    @staticmethod
    def check_token(token):
        user = User.query.filter_by(token=token).first()
        if user is None or user.token_expiration < datetime.utcnow():
            return None
        return user
```

## Testing

### Unit Testing
```python
# tests/test_models.py
import unittest
from app import create_app, db
from app.models import User, Post
from config import TestingConfig

class UserModelCase(unittest.TestCase):
    def setUp(self):
        self.app = create_app(TestingConfig)
        self.app_context = self.app.app_context()
        self.app_context.push()
        db.create_all()
    
    def tearDown(self):
        db.session.remove()
        db.drop_all()
        self.app_context.pop()
    
    def test_password_hashing(self):
        user = User(username='test')
        user.set_password('password')
        
        self.assertFalse(user.check_password('wrongpassword'))
        self.assertTrue(user.check_password('password'))
    
    def test_user_creation(self):
        user = User(username='test', email='test@example.com')
        db.session.add(user)
        db.session.commit()
        
        self.assertEqual(User.query.count(), 1)
        self.assertEqual(user.username, 'test')
    
    def test_post_creation(self):
        user = User(username='test', email='test@example.com')
        post = Post(title='Test Post', content='Test content', author=user)
        
        db.session.add(user)
        db.session.add(post)
        db.session.commit()
        
        self.assertEqual(Post.query.count(), 1)
        self.assertEqual(post.author, user)

# tests/test_routes.py
class RoutesTestCase(unittest.TestCase):
    def setUp(self):
        self.app = create_app(TestingConfig)
        self.app_context = self.app.app_context()
        self.app_context.push()
        self.client = self.app.test_client()
        db.create_all()
        
        # Create test user
        self.user = User(username='test', email='test@example.com')
        self.user.set_password('password')
        db.session.add(self.user)
        db.session.commit()
    
    def tearDown(self):
        db.session.remove()
        db.drop_all()
        self.app_context.pop()
    
    def login(self, username='test', password='password'):
        return self.client.post('/auth/login', data={
            'username': username,
            'password': password
        }, follow_redirects=True)
    
    def logout(self):
        return self.client.get('/auth/logout', follow_redirects=True)
    
    def test_home_page(self):
        response = self.client.get('/')
        self.assertEqual(response.status_code, 200)
        self.assertIn(b'Latest Posts', response.data)
    
    def test_login_logout(self):
        # Test login
        response = self.login()
        self.assertEqual(response.status_code, 200)
        self.assertIn(b'Welcome back', response.data)
        
        # Test logout
        response = self.logout()
        self.assertEqual(response.status_code, 200)
        self.assertIn(b'logged out', response.data)
    
    def test_create_post(self):
        self.login()
        response = self.client.post('/create_post', data={
            'title': 'Test Post',
            'content': 'This is a test post content.',
            'status': 'published'
        }, follow_redirects=True)
        
        self.assertEqual(response.status_code, 200)
        self.assertIn(b'Test Post', response.data)

if __name__ == '__main__':
    unittest.main()
```

## Deployment and Production

### Production Configuration
```python
# config.py - Production settings
class ProductionConfig(Config):
    DEBUG = False
    
    # Database
    SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL') or \
        'postgresql://user:password@localhost/flaskblog'
    
    # Security
    SSL_REDIRECT = True
    
    # Email (using SendGrid or similar)
    MAIL_SERVER = 'smtp.sendgrid.net'
    MAIL_PORT = 587
    MAIL_USE_TLS = True
    MAIL_USERNAME = 'apikey'
    MAIL_PASSWORD = os.environ.get('SENDGRID_API_KEY')
    
    # File uploads (use cloud storage)
    UPLOAD_FOLDER = 's3://mybucket/uploads'
    
    # Caching
    CACHE_TYPE = 'redis'
    CACHE_REDIS_URL = os.environ.get('REDIS_URL')
    
    # Logging
    LOG_TO_STDOUT = os.environ.get('LOG_TO_STDOUT')

# wsgi.py
from app import create_app
import os

app = create_app(os.getenv('FLASK_CONFIG') or 'default')

if __name__ == "__main__":
    app.run()

# Dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

ENV FLASK_APP=wsgi.py

EXPOSE 5000

CMD ["gunicorn", "--bind", "0.0.0.0:5000", "wsgi:app"]
```

### Performance Optimization
```python
# Caching with Flask-Caching
from flask_caching import Cache

cache = Cache()

def create_app(config_class=Config):
    app = Flask(__name__)
    app.config.from_object(config_class)
    
    cache.init_app(app)
    # ... other initializations

# Cache views
@bp.route('/')
@cache.cached(timeout=300)  # Cache for 5 minutes
def index():
    posts = Post.query.filter_by(status='published').all()
    return render_template('index.html', posts=posts)

# Cache expensive queries
@cache.memoize(timeout=600)
def get_popular_posts(limit=5):
    return Post.query.filter_by(status='published').order_by(
        Post.views.desc()
    ).limit(limit).all()

# Database query optimization
# Use lazy loading for relationships
posts = Post.query.options(db.joinedload(Post.author)).all()

# Pagination for large datasets
def get_posts_paginated(page=1, per_page=20):
    return Post.query.filter_by(status='published').paginate(
        page=page, per_page=per_page, error_out=False
    )
```

