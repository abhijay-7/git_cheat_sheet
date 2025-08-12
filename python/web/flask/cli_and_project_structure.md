# Flask CLI Commands Cheatsheet

## Core Flask Commands

### flask run
Start the development server

```bash
flask run                          # Start server on default host/port (127.0.0.1:5000)
flask run --host=0.0.0.0          # Make server accessible from any IP
flask run --port=8000             # Run on specific port
flask run --debug                 # Enable debug mode (auto-reload + debugger)
flask run --no-reload             # Disable auto-reload
flask run --no-debugger           # Disable debugger
flask run --reload                # Force enable auto-reload
flask run --eager-loading         # Disable lazy loading
flask run --with-threads          # Enable threaded mode
flask run --without-threads       # Disable threaded mode
```

### flask shell
Open interactive Python shell with app context

```bash
flask shell                       # Open shell with app context loaded
```

### flask routes
Display all registered routes

```bash
flask routes                      # Show all routes
flask routes --sort=endpoint      # Sort by endpoint name
flask routes --sort=methods       # Sort by HTTP methods
flask routes --sort=rule          # Sort by URL rule
flask routes --all-methods        # Show all methods for each route
```

## Database Commands (Flask-Migrate)

### flask db init
Initialize migration repository

```bash
flask db init                     # Create migrations folder
flask db init --directory=custom  # Use custom migrations directory
```

### flask db migrate
Create new migration

```bash
flask db migrate                  # Auto-generate migration
flask db migrate -m "Add users"   # Add message to migration
flask db migrate --rev-id=001     # Set custom revision ID
flask db migrate --branch-label=feature  # Add branch label
flask db migrate --splice         # Allow migration from multiple heads
flask db migrate --head=head      # Migrate from specific head
```

### flask db upgrade
Apply migrations

```bash
flask db upgrade                  # Upgrade to latest migration
flask db upgrade head             # Upgrade to head (same as above)
flask db upgrade +2               # Upgrade 2 revisions forward
flask db upgrade ae10+2           # Upgrade 2 revisions from ae10
flask db upgrade ae10:head        # Upgrade from ae10 to head
flask db upgrade --sql            # Show SQL instead of executing
flask db upgrade --tag=production # Upgrade with tag
```

### flask db downgrade
Revert migrations

```bash
flask db downgrade               # Downgrade one revision
flask db downgrade base          # Downgrade to initial state
flask db downgrade -1            # Downgrade one revision
flask db downgrade ae10          # Downgrade to specific revision
flask db downgrade --sql         # Show SQL instead of executing
flask db downgrade --tag=staging # Downgrade with tag
```

### flask db current
Show current revision

```bash
flask db current                 # Show current migration revision
flask db current --verbose       # Show verbose output
```

### flask db history
Show migration history

```bash
flask db history                 # Show all migration history
flask db history --rev-range=base:head  # Show range of revisions
flask db history -r base:head    # Short form of rev-range
flask db history --verbose       # Show detailed history
flask db history -v              # Short form of verbose
```

### flask db show
Show migration details

```bash
flask db show                    # Show current revision details
flask db show ae10               # Show specific revision details
flask db show head               # Show head revision details
```

### flask db stamp
Mark database with revision

```bash
flask db stamp head              # Mark database as current with head
flask db stamp ae10              # Mark database with specific revision
flask db stamp --sql             # Show SQL instead of executing
```

### flask db edit
Edit migration file

```bash
flask db edit                    # Edit current migration
flask db edit ae10               # Edit specific migration
```

### flask db merge
Merge multiple heads

```bash
flask db merge                   # Merge heads into single revision
flask db merge -m "Merge heads"  # Add message to merge
flask db merge --rev-id=merge1   # Set custom revision ID
```

### flask db branches
Show branch information

```bash
flask db branches               # Show branch information
flask db branches --verbose     # Show detailed branch info
```

### flask db heads
Show current heads

```bash
flask db heads                  # Show current head revisions
flask db heads --verbose        # Show detailed head info
```

## Environment Variables

Set these before running Flask commands:

```bash
export FLASK_APP=app.py          # Specify app file/module
export FLASK_ENV=development     # Set environment (deprecated)
export FLASK_DEBUG=1             # Enable debug mode
export FLASK_RUN_HOST=0.0.0.0    # Default host for flask run
export FLASK_RUN_PORT=8000       # Default port for flask run
export DATABASE_URL=sqlite:///app.db  # Database connection string
```

## Custom Commands

Create custom commands in your Flask app:

```python
@app.cli.command()
def init_db():
    """Initialize the database."""
    pass

@app.cli.group()
def user():
    """User management commands."""
    pass

@user.command()
def create():
    """Create a user."""
    pass
```

Then use:
```bash
flask init-db                   # Run custom init-db command
flask user create               # Run custom user create command
```

## Common Patterns

```bash
# Development workflow
flask db init                    # First time only
flask db migrate -m "Initial migration"
flask db upgrade
flask run --debug

# Production deployment
flask db upgrade
flask run --host=0.0.0.0 --port=80

# Reset database
flask db downgrade base
flask db upgrade

# Check migration status
flask db current
flask db history
flask routes
```

## Flask Project Folder Structures

### 1. Simple Single-File Structure
For small projects and prototypes:
```
myproject/
├── app.py              # Main application file
├── requirements.txt    # Dependencies
├── .env               # Environment variables
├── config.py          # Configuration settings
└── static/            # CSS, JS, images
    ├── css/
    ├── js/
    └── images/
└── templates/         # Jinja2 templates
    ├── base.html
    ├── index.html
    └── about.html
```

### 2. Package Structure (Small to Medium Apps)
When your app grows beyond a single file:
```
myproject/
├── app/
│   ├── __init__.py    # App factory
│   ├── models.py      # Database models
│   ├── views.py       # Route handlers
│   ├── forms.py       # WTForms
│   ├── static/        # Static files
│   └── templates/     # Templates
├── migrations/        # Database migrations
├── tests/            # Unit tests
├── venv/             # Virtual environment
├── config.py         # Configuration
├── requirements.txt  # Dependencies
├── .env              # Environment variables
├── .flaskenv         # Flask-specific env vars
└── run.py            # Application entry point
```

### 3. Blueprint Structure (Medium to Large Apps)
Organized with blueprints for modularity:
```
myproject/
├── app/
│   ├── __init__.py           # App factory
│   ├── models.py             # Database models
│   ├── cli.py                # Custom CLI commands
│   ├── extensions.py         # Extension initialization
│   ├── auth/                 # Authentication blueprint
│   │   ├── __init__.py
│   │   ├── views.py
│   │   ├── forms.py
│   │   └── templates/
│   ├── main/                 # Main blueprint
│   │   ├── __init__.py
│   │   ├── views.py
│   │   ├── forms.py
│   │   └── templates/
│   ├── api/                  # API blueprint
│   │   ├── __init__.py
│   │   ├── views.py
│   │   └── errors.py
│   ├── static/               # Static files
│   └── templates/            # Base templates
│       ├── base.html
│       ├── auth/
│       └── errors/
├── migrations/               # Database migrations
├── tests/                    # Unit tests
│   ├── __init__.py
│   ├── test_auth.py
│   ├── test_main.py
│   └── test_api.py
├── logs/                     # Application logs
├── config.py                 # Configuration classes
├── requirements.txt          # Dependencies
├── .env                      # Environment variables
├── .flaskenv                 # Flask environment
└── wsgi.py                   # WSGI entry point
```

### 4. Large Application Structure (Enterprise)
For complex applications with multiple services:
```
myproject/
├── app/
│   ├── __init__.py
│   ├── extensions.py         # Extension initialization
│   ├── cli.py               # Custom commands
│   ├── utils/               # Utility functions
│   │   ├── __init__.py
│   │   ├── decorators.py
│   │   ├── helpers.py
│   │   └── validators.py
│   ├── models/              # Database models
│   │   ├── __init__.py
│   │   ├── user.py
│   │   ├── post.py
│   │   └── mixins.py
│   ├── auth/                # Authentication module
│   │   ├── __init__.py
│   │   ├── views.py
│   │   ├── forms.py
│   │   ├── models.py
│   │   └── templates/
│   ├── admin/               # Admin interface
│   │   ├── __init__.py
│   │   ├── views.py
│   │   └── templates/
│   ├── api/                 # REST API
│   │   ├── __init__.py
│   │   ├── v1/
│   │   │   ├── __init__.py
│   │   │   ├── auth.py
│   │   │   ├── users.py
│   │   │   └── posts.py
│   │   ├── v2/
│   │   └── errors.py
│   ├── main/                # Main application
│   │   ├── __init__.py
│   │   ├── views.py
│   │   ├── forms.py
│   │   └── templates/
│   ├── static/
│   │   ├── css/
│   │   ├── js/
│   │   ├── images/
│   │   └── vendor/          # Third-party assets
│   └── templates/
│       ├── base.html
│       ├── macros.html
│       ├── auth/
│       ├── admin/
│       ├── main/
│       └── errors/
├── migrations/
├── tests/
│   ├── __init__.py
│   ├── conftest.py          # Pytest configuration
│   ├── unit/
│   │   ├── test_models.py
│   │   └── test_utils.py
│   ├── integration/
│   └── functional/
├── docs/                    # Documentation
├── scripts/                 # Deployment scripts
├── logs/
├── instance/                # Instance-specific files
├── config/
│   ├── __init__.py
│   ├── development.py
│   ├── production.py
│   └── testing.py
├── requirements/            # Split requirements
│   ├── base.txt
│   ├── dev.txt
│   └── prod.txt
├── docker-compose.yml       # Docker configuration
├── Dockerfile
├── .env.example
├── .flaskenv
└── wsgi.py
```

### 5. Microservices Structure
For microservices architecture:
```
myproject/
├── services/
│   ├── auth-service/
│   │   ├── app/
│   │   ├── requirements.txt
│   │   ├── Dockerfile
│   │   └── wsgi.py
│   ├── user-service/
│   │   ├── app/
│   │   ├── requirements.txt
│   │   ├── Dockerfile
│   │   └── wsgi.py
│   └── api-gateway/
│       ├── app/
│       ├── requirements.txt
│       ├── Dockerfile
│       └── wsgi.py
├── shared/                  # Shared utilities
│   ├── __init__.py
│   ├── database.py
│   ├── auth.py
│   └── utils.py
├── docker-compose.yml
├── kubernetes/              # K8s manifests
└── scripts/                # Deployment scripts
```

### 6. Factory Pattern Structure
Using application factory pattern:
```
myproject/
├── flaskr/                  # Main package
│   ├── __init__.py          # Application factory
│   ├── db.py                # Database setup
│   ├── schema.sql           # Database schema
│   ├── auth.py              # Authentication views
│   ├── blog.py              # Blog views
│   ├── static/
│   └── templates/
│       ├── base.html
│       ├── auth/
│       └── blog/
├── tests/
│   ├── conftest.py
│   ├── data.sql
│   ├── test_factory.py
│   ├── test_db.py
│   ├── test_auth.py
│   └── test_blog.py
├── instance/                # Instance folder
├── .venv/                   # Virtual environment
├── setup.py                 # Package installation
├── MANIFEST.in              # Include additional files
└── requirements.txt
```

### Key Structure Guidelines

- **Small projects**: Use single-file or simple package structure
- **Medium projects**: Implement blueprints for organization
- **Large projects**: Use modular structure with clear separation
- **Always include**: `requirements.txt`, `.env` files, tests folder
- **Consider**: Docker files, CI/CD configs, documentation folder
- **Instance folder**: For deployment-specific configurations
- **Migrations folder**: Created automatically with `flask db init`