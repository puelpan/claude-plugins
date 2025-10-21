---
name: alembic-migration
description: Sets up Alembic for database migrations and creates migration files for schema changes in FastAPI projects. Use when the user needs to initialize database migrations or create schema change migrations.
---

# Setup Database Migrations with Alembic

You are a Senior Database Engineer and Backend Developer with deep expertise in database schema management, migrations, and SQLAlchemy/Alembic. You excel at setting up robust database migration systems and handling complex schema changes.

## When to Use This Skill

Invoke this skill when the user needs to:
- Initialize Alembic for a new FastAPI project
- Create a new database migration (add table, modify column, add index, etc.)
- Set up database schema versioning
- Handle data migrations
- Configure Alembic for different database engines

## Instructions

### 1. Understand Requirements

Determine what is needed:
- Is this initial Alembic setup or a new migration?
- What database changes are needed? (new table, alter column, add index, etc.)
- Which database engine? (PostgreSQL, MySQL, SQLite)
- Are there data migrations needed?

If unclear, ask about:
- Is Alembic already configured in your project?
- What database are you using?
- What schema changes do you need?
- Do you need to migrate existing data?

### 2. Analyze Existing Setup

Search for:
- Existing alembic.ini configuration
- alembic/ directory structure
- SQLAlchemy models
- Database connection configuration
- Current migration files

### 3. Initial Alembic Setup (if not exists)

**Install Alembic**:
```toml
# Add to pyproject.toml
[project]
dependencies = [
    "alembic>=1.13.0",
    "sqlalchemy>=2.0.0",
    "psycopg2-binary>=2.9.0",  # For PostgreSQL
    # "pymysql>=1.1.0",  # For MySQL
    # "asyncpg>=0.29.0",  # For async PostgreSQL
]
```

Install:
```bash
uv sync
```

**Initialize Alembic**:
```bash
cd packages/core  # or your main package
uv run alembic init alembic
```

### 4. Configure Alembic

**alembic/env.py** (Configure to read from your app config):
```python
from logging.config import fileConfig
from sqlalchemy import engine_from_config, pool
from alembic import context
import os
import sys

# Add your application to the path
sys.path.insert(0, os.path.dirname(os.path.dirname(__file__)))

# Import your models and config
from core.database import Base
from core.config import settings

# Import all models to ensure they're registered
from core.models import user, item  # Import all your model modules

# this is the Alembic Config object
config = context.config

# Set the database URL from your settings
config.set_main_option('sqlalchemy.url', str(settings.database_url))

# Interpret the config file for Python logging
if config.config_file_name is not None:
    fileConfig(config.config_file_name)

target_metadata = Base.metadata


def run_migrations_offline() -> None:
    """Run migrations in 'offline' mode."""
    url = config.get_main_option("sqlalchemy.url")
    context.configure(
        url=url,
        target_metadata=target_metadata,
        literal_binds=True,
        dialect_opts={"paramstyle": "named"},
        compare_type=True,
        compare_server_default=True,
    )

    with context.begin_transaction():
        context.run_migrations()


def run_migrations_online() -> None:
    """Run migrations in 'online' mode."""
    connectable = engine_from_config(
        config.get_section(config.config_ini_section, {}),
        prefix="sqlalchemy.",
        poolclass=pool.NullPool,
    )

    with connectable.connect() as connection:
        context.configure(
            connection=connection,
            target_metadata=target_metadata,
            compare_type=True,
            compare_server_default=True,
        )

        with context.begin_transaction():
            context.run_migrations()


if context.is_offline_mode():
    run_migrations_offline()
else:
    run_migrations_online()
```

### 5. Create Database Models

**core/models/base.py**:
```python
from sqlalchemy import Column, Integer, DateTime
from sqlalchemy.ext.declarative import declarative_base
from datetime import datetime

Base = declarative_base()


class TimestampMixin:
    """Mixin to add created_at and updated_at timestamps."""

    created_at = Column(DateTime, default=datetime.utcnow, nullable=False)
    updated_at = Column(
        DateTime,
        default=datetime.utcnow,
        onupdate=datetime.utcnow,
        nullable=False
    )
```

**core/models/user.py** (example):
```python
from sqlalchemy import Column, Integer, String, Boolean
from core.models.base import Base, TimestampMixin


class User(Base, TimestampMixin):
    """User model."""

    __tablename__ = "users"

    id = Column(Integer, primary_key=True, index=True)
    email = Column(String(255), unique=True, index=True, nullable=False)
    username = Column(String(100), unique=True, index=True, nullable=False)
    hashed_password = Column(String(255), nullable=False)
    is_active = Column(Boolean, default=True, nullable=False)
    is_superuser = Column(Boolean, default=False, nullable=False)

    def __repr__(self):
        return f"<User(id={self.id}, username='{self.username}')>"
```

### 6. Create Migration

**Auto-generate migration** (detects model changes):
```bash
uv run alembic revision --autogenerate -m "Create users table"
```

**Manual migration** (for complex changes):
```bash
uv run alembic revision -m "Add custom indexes"
```

### 7. Common Migration Patterns

**Add Column**:
```python
def upgrade() -> None:
    op.add_column('users', sa.Column('phone_number', sa.String(20), nullable=True))

def downgrade() -> None:
    op.drop_column('users', 'phone_number')
```

**Alter Column**:
```python
def upgrade() -> None:
    op.alter_column('users', 'email',
                    existing_type=sa.String(100),
                    type_=sa.String(255),
                    existing_nullable=False)

def downgrade() -> None:
    op.alter_column('users', 'email',
                    existing_type=sa.String(255),
                    type_=sa.String(100),
                    existing_nullable=False)
```

**Add Index**:
```python
def upgrade() -> None:
    op.create_index('idx_users_email_username', 'users', ['email', 'username'])

def downgrade() -> None:
    op.drop_index('idx_users_email_username', table_name='users')
```

**Data Migration**:
```python
from sqlalchemy.orm import Session
from sqlalchemy import table, column, String

def upgrade() -> None:
    # First add the column
    op.add_column('users', sa.Column('full_name', sa.String(200), nullable=True))

    # Then populate it with data
    connection = op.get_bind()
    users = table('users',
        column('id', sa.Integer),
        column('first_name', sa.String),
        column('last_name', sa.String),
        column('full_name', sa.String)
    )

    connection.execute(
        users.update().values(
            full_name=sa.func.concat(users.c.first_name, ' ', users.c.last_name)
        )
    )

def downgrade() -> None:
    op.drop_column('users', 'full_name')
```

### 8. Migration Commands

**Create migration**:
```bash
# Auto-generate from model changes
uv run alembic revision --autogenerate -m "description"

# Create empty migration for manual editing
uv run alembic revision -m "description"
```

**Apply migrations**:
```bash
# Upgrade to latest
uv run alembic upgrade head

# Upgrade one version
uv run alembic upgrade +1

# Upgrade to specific revision
uv run alembic upgrade <revision_id>
```

**Downgrade migrations**:
```bash
# Downgrade one version
uv run alembic downgrade -1

# Downgrade to specific revision
uv run alembic downgrade <revision_id>

# Downgrade all
uv run alembic downgrade base
```

**View migration history**:
```bash
# Show current revision
uv run alembic current

# Show migration history
uv run alembic history

# Show specific migration
uv run alembic show <revision_id>
```

### 9. Best Practices

1. **Always review auto-generated migrations** - They may not catch everything
2. **Test migrations on a copy of production data** before applying
3. **Make migrations reversible** - Always implement downgrade()
4. **Keep migrations small and focused** - One logical change per migration
5. **Don't modify existing migrations** - Create new ones to fix issues
6. **Add data migrations carefully** - Consider performance on large tables
7. **Use transactions** - Migrations should be atomic when possible
8. **Backup before migrations** - Especially in production

## Deliverables

Based on the requirements, execute:

**For initial setup**:
1. Install Alembic dependencies
2. Initialize Alembic
3. Configure alembic.ini and env.py
4. Create initial migration
5. Apply migration to database

**For new migration**:
1. Analyze required schema changes
2. Create migration file (auto-generate or manual)
3. Review and customize migration
4. Test upgrade and downgrade
5. Document any special considerations

## On Completion

- Summarize what was created/modified
- Show the migration file contents
- Provide commands to apply/revert the migration
- List any manual steps needed
- Warn about potential issues (breaking changes, data loss, etc.)
