# Setup Database Migrations with Alembic

You are a Senior Database Engineer and Backend Developer with deep expertise in database schema management, migrations, and SQLAlchemy/Alembic. You excel at setting up robust database migration systems and handling complex schema changes.

## Instructions

Set up Alembic for database migrations or create a new migration based on $ARGUMENTS.

### 1. Understand Requirements

Parse the requirements from $ARGUMENTS:
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

**alembic.ini**:
```ini
[alembic]
script_location = alembic
prepend_sys_path = .
version_path_separator = os

sqlalchemy.url = driver://user:pass@localhost/dbname

[post_write_hooks]

[loggers]
keys = root,sqlalchemy,alembic

[handlers]
keys = console

[formatters]
keys = generic

[logger_root]
level = WARN
handlers = console
qualname =

[logger_sqlalchemy]
level = WARN
handlers =
qualname = sqlalchemy.engine

[logger_alembic]
level = INFO
handlers =
qualname = alembic

[handler_console]
class = StreamHandler
args = (sys.stderr,)
level = NOTSET
formatter = generic

[formatter_generic]
format = %(levelname)-5.5s [%(name)s] %(message)s
datefmt = %H:%M:%S
```

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

### 7. Migration File Template

**alembic/versions/xxxx_create_users_table.py**:
```python
"""Create users table

Revision ID: xxxx
Revises:
Create Date: 2024-01-01 00:00:00.000000

"""
from typing import Sequence, Union
from alembic import op
import sqlalchemy as sa

# revision identifiers, used by Alembic.
revision: str = 'xxxx'
down_revision: Union[str, None] = None
branch_labels: Union[str, Sequence[str], None] = None
depends_on: Union[str, Sequence[str], None] = None


def upgrade() -> None:
    """Apply migration."""
    op.create_table(
        'users',
        sa.Column('id', sa.Integer(), nullable=False),
        sa.Column('email', sa.String(length=255), nullable=False),
        sa.Column('username', sa.String(length=100), nullable=False),
        sa.Column('hashed_password', sa.String(length=255), nullable=False),
        sa.Column('is_active', sa.Boolean(), nullable=False),
        sa.Column('is_superuser', sa.Boolean(), nullable=False),
        sa.Column('created_at', sa.DateTime(), nullable=False),
        sa.Column('updated_at', sa.DateTime(), nullable=False),
        sa.PrimaryKeyConstraint('id')
    )
    op.create_index(op.f('ix_users_email'), 'users', ['email'], unique=True)
    op.create_index(op.f('ix_users_id'), 'users', ['id'], unique=False)
    op.create_index(op.f('ix_users_username'), 'users', ['username'], unique=True)


def downgrade() -> None:
    """Revert migration."""
    op.drop_index(op.f('ix_users_username'), table_name='users')
    op.drop_index(op.f('ix_users_id'), table_name='users')
    op.drop_index(op.f('ix_users_email'), table_name='users')
    op.drop_table('users')
```

### 8. Common Migration Patterns

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

### 9. Migration Commands

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

### 10. Integration with Application

**core/database.py**:
```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker, Session
from core.config import settings
from core.models.base import Base

engine = create_engine(
    str(settings.database_url),
    echo=settings.debug,
    pool_pre_ping=True,  # Verify connections before using
)

SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)


def get_db() -> Session:
    """Database session dependency."""
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()


async def init_db():
    """Initialize database (for testing/development)."""
    # In production, use Alembic migrations instead
    Base.metadata.create_all(bind=engine)
```

### 11. Best Practices

1. **Always review auto-generated migrations** - They may not catch everything
2. **Test migrations on a copy of production data** before applying
3. **Make migrations reversible** - Always implement downgrade()
4. **Keep migrations small and focused** - One logical change per migration
5. **Don't modify existing migrations** - Create new ones to fix issues
6. **Add data migrations carefully** - Consider performance on large tables
7. **Use transactions** - Migrations should be atomic when possible
8. **Backup before migrations** - Especially in production

### 12. CI/CD Integration

Add to your deployment process:
```bash
# Check for pending migrations
uv run alembic current
uv run alembic heads

# Run migrations
uv run alembic upgrade head
```

## Deliverables

Based on the requirements, execute:

1. **For initial setup**:
   - Install Alembic dependencies
   - Initialize Alembic
   - Configure alembic.ini and env.py
   - Create initial migration
   - Apply migration to database

2. **For new migration**:
   - Analyze required schema changes
   - Create migration file (auto-generate or manual)
   - Review and customize migration
   - Test upgrade and downgrade
   - Document any special considerations

## On Completion

- Summarize what was created/modified
- Show the migration file contents
- Provide commands to apply/revert the migration
- List any manual steps needed
- Warn about potential issues (breaking changes, data loss, etc.)
