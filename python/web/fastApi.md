# FastAPI In-Depth Cheatsheet

## Installation and Setup

```bash
# Install FastAPI with ASGI server
pip install fastapi uvicorn

# Install with common dependencies
pip install fastapi uvicorn sqlalchemy alembic python-multipart python-jose[cryptography] passlib[bcrypt]

# Install for development
pip install fastapi uvicorn[standard] pytest httpx

# Run the application
uvicorn main:app --reload
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

## Basic FastAPI Application

### Minimal Application
```python
# main.py
from fastapi import FastAPI
from typing import Optional

app = FastAPI(
    title="My API",
    description="A sample FastAPI application",
    version="1.0.0"
)

@app.get("/")
async def root():
    return {"message": "Hello World"}

@app.get("/items/{item_id}")
async def read_item(item_id: int, q: Optional[str] = None):
    return {"item_id": item_id, "q": q}

# Run with: uvicorn main:app --reload
```

### Application Structure with Routers
```python
# app/main.py
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from app.routers import users, items, auth
from app.core.config import settings
from app.database import engine
from app import models

# Create tables
models.Base.metadata.create_all(bind=engine)

app = FastAPI(
    title=settings.PROJECT_NAME,
    version=settings.VERSION,
    description="A comprehensive FastAPI application",
    openapi_url=f"{settings.API_V1_STR}/openapi.json"
)

# CORS middleware
app.add_middleware(
    CORSMiddleware,
    allow_origins=settings.BACKEND_CORS_ORIGINS,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Include routers
app.include_router(auth.router, prefix=f"{settings.API_V1_STR}/auth", tags=["auth"])
app.include_router(users.router, prefix=f"{settings.API_V1_STR}/users", tags=["users"])
app.include_router(items.router, prefix=f"{settings.API_V1_STR}/items", tags=["items"])

@app.get("/")
async def root():
    return {"message": "Welcome to FastAPI"}

# Health check endpoint
@app.get("/health")
async def health_check():
    return {"status": "healthy"}
```

### Configuration Management
```python
# app/core/config.py
from pydantic import BaseSettings, AnyHttpUrl, validator
from typing import List, Union, Optional
import secrets

class Settings(BaseSettings):
    API_V1_STR: str = "/api/v1"
    SECRET_KEY: str = secrets.token_urlsafe(32)
    PROJECT_NAME: str = "FastAPI App"
    VERSION: str = "1.0.0"
    
    # Security
    ACCESS_TOKEN_EXPIRE_MINUTES: int = 60 * 24 * 8  # 8 days
    ALGORITHM: str = "HS256"
    
    # Database
    DATABASE_URL: Optional[str] = "sqlite:///./test.db"
    
    # CORS
    BACKEND_CORS_ORIGINS: List[AnyHttpUrl] = []
    
    @validator("BACKEND_CORS_ORIGINS", pre=True)
    def assemble_cors_origins(cls, v: Union[str, List[str]]) -> Union[List[str], str]:
        if isinstance(v, str) and not v.startswith("["):
            return [i.strip() for i in v.split(",")]
        elif isinstance(v, (list, str)):
            return v
        raise ValueError(v)
    
    # Email settings
    SMTP_TLS: bool = True
    SMTP_PORT: Optional[int] = None
    SMTP_HOST: Optional[str] = None
    SMTP_USER: Optional[str] = None
    SMTP_PASSWORD: Optional[str] = None
    EMAILS_FROM_EMAIL: Optional[str] = None
    EMAILS_FROM_NAME: Optional[str] = None
    
    # First superuser
    FIRST_SUPERUSER: str = "admin@example.com"
    FIRST_SUPERUSER_PASSWORD: str = "changethis"
    
    class Config:
        env_file = ".env"
        case_sensitive = True

settings = Settings()

# .env file
SECRET_KEY=your-super-secret-key-here
DATABASE_URL=postgresql://user:password@localhost/dbname
BACKEND_CORS_ORIGINS=http://localhost:3000,http://localhost:8080
FIRST_SUPERUSER=admin@example.com
FIRST_SUPERUSER_PASSWORD=changethis
```

## Pydantic Models and Validation

### Request and Response Models
```python
# app/schemas.py
from pydantic import BaseModel, EmailStr, validator, Field
from typing import Optional, List
from datetime import datetime
from enum import Enum

class UserRole(str, Enum):
    admin = "admin"
    user = "user"
    moderator = "moderator"

class ItemStatus(str, Enum):
    active = "active"
    inactive = "inactive"
    draft = "draft"

# Base models
class UserBase(BaseModel):
    email: EmailStr
    first_name: str = Field(..., min_length=1, max_length=50)
    last_name: str = Field(..., min_length=1, max_length=50)
    is_active: bool = True
    role: UserRole = UserRole.user

class UserCreate(UserBase):
    password: str = Field(..., min_length=8, max_length=100)
    
    @validator('password')
    def validate_password(cls, v):
        if not any(c.isupper() for c in v):
            raise ValueError('Password must contain at least one uppercase letter')
        if not any(c.islower() for c in v):
            raise ValueError('Password must contain at least one lowercase letter')
        if not any(c.isdigit() for c in v):
            raise ValueError('Password must contain at least one digit')
        return v

class UserUpdate(BaseModel):
    first_name: Optional[str] = Field(None, min_length=1, max_length=50)
    last_name: Optional[str] = Field(None, min_length=1, max_length=50)
    is_active: Optional[bool] = None
    role: Optional[UserRole] = None

class UserInDB(UserBase):
    id: int
    created_at: datetime
    updated_at: Optional[datetime] = None
    
    class Config:
        orm_mode = True

class User(UserInDB):
    pass

# Item models
class ItemBase(BaseModel):
    title: str = Field(..., min_length=1, max_length=200)
    description: Optional[str] = Field(None, max_length=1000)
    price: float = Field(..., gt=0, description="Price must be greater than zero")
    tax: Optional[float] = Field(None, ge=0, le=100)
    status: ItemStatus = ItemStatus.active
    tags: List[str] = []

class ItemCreate(ItemBase):
    pass

class ItemUpdate(BaseModel):
    title: Optional[str] = Field(None, min_length=1, max_length=200)
    description: Optional[str] = Field(None, max_length=1000)
    price: Optional[float] = Field(None, gt=0)
    tax: Optional[float] = Field(None, ge=0, le=100)
    status: Optional[ItemStatus] = None
    tags: Optional[List[str]] = None

class ItemInDB(ItemBase):
    id: int
    owner_id: int
    created_at: datetime
    updated_at: Optional[datetime] = None
    
    class Config:
        orm_mode = True

class Item(ItemInDB):
    owner: User

# Authentication models
class Token(BaseModel):
    access_token: str
    token_type: str = "bearer"
    expires_in: int

class TokenData(BaseModel):
    email: Optional[str] = None

class UserLogin(BaseModel):
    email: EmailStr
    password: str

# Response models with pagination
class PaginatedResponse(BaseModel):
    items: List[dict]
    total: int
    page: int
    size: int
    pages: int

class UserListResponse(BaseModel):
    users: List[User]
    total: int
    page: int
    size: int
    pages: int

class ItemListResponse(BaseModel):
    items: List[Item]
    total: int
    page: int
    size: int
    pages: int

# Error response models
class HTTPError(BaseModel):
    detail: str

class ValidationError(BaseModel):
    detail: List[dict]

# Custom validators
class CreateUserRequest(BaseModel):
    email: EmailStr
    password: str
    first_name: str
    last_name: str
    
    @validator('email')
    def email_must_be_valid(cls, v):
        # Additional email validation if needed
        return v.lower()
    
    @validator('first_name', 'last_name')
    def name_must_not_be_empty(cls, v):
        if not v.strip():
            raise ValueError('Name cannot be empty or whitespace')
        return v.strip().title()

# Advanced validation with field dependencies
class AdvancedItemCreate(BaseModel):
    name: str
    price: float
    discount_price: Optional[float] = None
    
    @validator('discount_price')
    def discount_must_be_less_than_price(cls, v, values):
        if v is not None and 'price' in values and v >= values['price']:
            raise ValueError('Discount price must be less than regular price')
        return v
```

### Custom Validators and Types
```python
# app/schemas/validators.py
from pydantic import validator, BaseModel
import re
from typing import Optional

class PhoneNumber(str):
    @classmethod
    def __get_validators__(cls):
        yield cls.validate
    
    @classmethod
    def validate(cls, v):
        if not isinstance(v, str):
            raise TypeError('string required')
        
        # Remove all non-digit characters
        digits = re.sub(r'\D', '', v)
        
        if len(digits) < 10:
            raise ValueError('Phone number must have at least 10 digits')
        
        return cls(digits)

class UserProfile(BaseModel):
    name: str
    phone: Optional[PhoneNumber] = None
    website: Optional[str] = None
    
    @validator('website')
    def validate_website(cls, v):
        if v and not v.startswith(('http://', 'https://')):
            return f'https://{v}'
        return v
    
    @validator('name')
    def validate_name(cls, v):
        if len(v.strip()) < 2:
            raise ValueError('Name must be at least 2 characters long')
        return v.strip().title()

# Custom field types
from pydantic import Field
from datetime import datetime, date

class EventCreate(BaseModel):
    title: str = Field(..., min_length=3, max_length=100)
    description: str = Field(..., min_length=10, max_length=500)
    start_date: datetime = Field(..., description="Event start date and time")
    end_date: datetime = Field(..., description="Event end date and time")
    max_participants: int = Field(..., gt=0, le=1000)
    
    @validator('end_date')
    def end_date_must_be_after_start_date(cls, v, values):
        if 'start_date' in values and v <= values['start_date']:
            raise ValueError('End date must be after start date')
        return v
    
    @validator('start_date')
    def start_date_must_be_future(cls, v):
        if v <= datetime.now():
            raise ValueError('Start date must be in the future')
        return v
```

## Database Integration with SQLAlchemy

### Database Models
```python
# app/database.py
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
from app.core.config import settings

engine = create_engine(settings.DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

Base = declarative_base()

# Dependency to get DB session
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

# app/models.py
from sqlalchemy import Column, Integer, String, Boolean, DateTime, Float, Text, ForeignKey, Table
from sqlalchemy.orm import relationship
from sqlalchemy.sql import func
from app.database import Base
import bcrypt

# Association table for many-to-many relationship
item_tags = Table(
    'item_tags',
    Base.metadata,
    Column('item_id', Integer, ForeignKey('items.id'), primary_key=True),
    Column('tag_id', Integer, ForeignKey('tags.id'), primary_key=True)
)

class User(Base):
    __tablename__ = "users"
    
    id = Column(Integer, primary_key=True, index=True)
    email = Column(String, unique=True, index=True, nullable=False)
    first_name = Column(String, nullable=False)
    last_name = Column(String, nullable=False)
    hashed_password = Column(String, nullable=False)
    is_active = Column(Boolean, default=True)
    role = Column(String, default="user")
    created_at = Column(DateTime(timezone=True), server_default=func.now())
    updated_at = Column(DateTime(timezone=True), onupdate=func.now())
    
    # Relationships
    items = relationship("Item", back_populates="owner", cascade="all, delete-orphan")
    
    def verify_password(self, password: str) -> bool:
        return bcrypt.checkpw(password.encode('utf-8'), self.hashed_password.encode('utf-8'))
    
    @staticmethod
    def hash_password(password: str) -> str:
        return bcrypt.hashpw(password.encode('utf-8'), bcrypt.gensalt()).decode('utf-8')
    
    @property
    def full_name(self) -> str:
        return f"{self.first_name} {self.last_name}"

class Item(Base):
    __tablename__ = "items"
    
    id = Column(Integer, primary_key=True, index=True)
    title = Column(String, index=True, nullable=False)
    description = Column(Text)
    price = Column(Float, nullable=False)
    tax = Column(Float, default=0.0)
    status = Column(String, default="active")
    owner_id = Column(Integer, ForeignKey("users.id"), nullable=False)
    created_at = Column(DateTime(timezone=True), server_default=func.now())
    updated_at = Column(DateTime(timezone=True), onupdate=func.now())
    
    # Relationships
    owner = relationship("User", back_populates="items")
    tags = relationship("Tag", secondary=item_tags, back_populates="items")
    
    @property
    def total_price(self) -> float:
        return self.price + (self.price * (self.tax or 0) / 100)

class Tag(Base):
    __tablename__ = "tags"
    
    id = Column(Integer, primary_key=True, index=True)
    name = Column(String, unique=True, index=True, nullable=False)
    description = Column(Text)
    created_at = Column(DateTime(timezone=True), server_default=func.now())
    
    # Relationships
    items = relationship("Item", secondary=item_tags, back_populates="tags")

class Category(Base):
    __tablename__ = "categories"
    
    id = Column(Integer, primary_key=True, index=True)
    name = Column(String, unique=True, index=True, nullable=False)
    slug = Column(String, unique=True, index=True, nullable=False)
    description = Column(Text)
    is_active = Column(Boolean, default=True)
    parent_id = Column(Integer, ForeignKey("categories.id"))
    created_at = Column(DateTime(timezone=True), server_default=func.now())
    
    # Self-referential relationship
    parent = relationship("Category", remote_side=[id], back_populates="children")
    children = relationship("Category", back_populates="parent")
```

### Database Operations (CRUD)
```python
# app/crud/base.py
from typing import Any, Dict, Generic, List, Optional, Type, TypeVar, Union
from fastapi.encoders import jsonable_encoder
from pydantic import BaseModel
from sqlalchemy.orm import Session
from app.database import Base

ModelType = TypeVar("ModelType", bound=Base)
CreateSchemaType = TypeVar("CreateSchemaType", bound=BaseModel)
UpdateSchemaType = TypeVar("UpdateSchemaType", bound=BaseModel)

class CRUDBase(Generic[ModelType, CreateSchemaType, UpdateSchemaType]):
    def __init__(self, model: Type[ModelType]):
        self.model = model
    
    def get(self, db: Session, id: Any) -> Optional[ModelType]:
        return db.query(self.model).filter(self.model.id == id).first()
    
    def get_multi(
        self, db: Session, *, skip: int = 0, limit: int = 100
    ) -> List[ModelType]:
        return db.query(self.model).offset(skip).limit(limit).all()
    
    def create(self, db: Session, *, obj_in: CreateSchemaType) -> ModelType:
        obj_in_data = jsonable_encoder(obj_in)
        db_obj = self.model(**obj_in_data)
        db.add(db_obj)
        db.commit()
        db.refresh(db_obj)
        return db_obj
    
    def update(
        self,
        db: Session,
        *,
        db_obj: ModelType,
        obj_in: Union[UpdateSchemaType, Dict[str, Any]]
    ) -> ModelType:
        obj_data = jsonable_encoder(db_obj)
        if isinstance(obj_in, dict):
            update_data = obj_in
        else:
            update_data = obj_in.dict(exclude_unset=True)
        for field in obj_data:
            if field in update_data:
                setattr(db_obj, field, update_data[field])
        db.add(db_obj)
        db.commit()
        db.refresh(db_obj)
        return db_obj
    
    def remove(self, db: Session, *, id: int) -> ModelType:
        obj = db.query(self.model).get(id)
        db.delete(obj)
        db.commit()
        return obj

# app/crud/crud_user.py
from typing import Optional
from sqlalchemy.orm import Session
from app.crud.base import CRUDBase
from app.models import User
from app.schemas import UserCreate, UserUpdate

class CRUDUser(CRUDBase[User, UserCreate, UserUpdate]):
    def get_by_email(self, db: Session, *, email: str) -> Optional[User]:
        return db.query(User).filter(User.email == email).first()
    
    def create(self, db: Session, *, obj_in: UserCreate) -> User:
        db_obj = User(
            email=obj_in.email,
            first_name=obj_in.first_name,
            last_name=obj_in.last_name,
            hashed_password=User.hash_password(obj_in.password),
            role=obj_in.role,
            is_active=obj_in.is_active,
        )
        db.add(db_obj)
        db.commit()
        db.refresh(db_obj)
        return db_obj
    
    def authenticate(self, db: Session, *, email: str, password: str) -> Optional[User]:
        user = self.get_by_email(db, email=email)
        if not user:
            return None
        if not user.verify_password(password):
            return None
        return user
    
    def is_active(self, user: User) -> bool:
        return user.is_active
    
    def is_superuser(self, user: User) -> bool:
        return user.role == "admin"

user = CRUDUser(User)

# app/crud/crud_item.py
from typing import List, Optional
from sqlalchemy.orm import Session
from app.crud.base import CRUDBase
from app.models import Item, User
from app.schemas import ItemCreate, ItemUpdate

class CRUDItem(CRUDBase[Item, ItemCreate, ItemUpdate]):
    def create_with_owner(
        self, db: Session, *, obj_in: ItemCreate, owner_id: int
    ) -> Item:
        obj_in_data = jsonable_encoder(obj_in)
        db_obj = self.model(**obj_in_data, owner_id=owner_id)
        db.add(db_obj)
        db.commit()
        db.refresh(db_obj)
        return db_obj
    
    def get_multi_by_owner(
        self, db: Session, *, owner_id: int, skip: int = 0, limit: int = 100
    ) -> List[Item]:
        return (
            db.query(self.model)
            .filter(Item.owner_id == owner_id)
            .offset(skip)
            .limit(limit)
            .all()
        )
    
    def get_by_status(
        self, db: Session, *, status: str, skip: int = 0, limit: int = 100
    ) -> List[Item]:
        return (
            db.query(self.model)
            .filter(Item.status == status)
            .offset(skip)
            .limit(limit)
            .all()
        )

item = CRUDItem(Item)
```

### Database Migrations with Alembic
```bash
# Initialize Alembic
alembic init alembic

# Create migration
alembic revision --autogenerate -m "Initial migration"

# Apply migration
alembic upgrade head

# Downgrade migration
alembic downgrade -1

# Show current revision
alembic current

# Show migration history
alembic history
```

```python
# alembic/env.py configuration
from app.database import Base
from app.models import *  # Import all models

target_metadata = Base.metadata
```

## Authentication and Authorization

### JWT Authentication
```python
# app/core/security.py
from datetime import datetime, timedelta
from typing import Optional
from jose import JWTError, jwt
from passlib.context import CryptContext
from app.core.config import settings

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def create_access_token(data: dict, expires_delta: Optional[timedelta] = None):
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(minutes=settings.ACCESS_TOKEN_EXPIRE_MINUTES)
    
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, settings.SECRET_KEY, algorithm=settings.ALGORITHM)
    return encoded_jwt

def verify_token(token: str) -> Optional[str]:
    try:
        payload = jwt.decode(token, settings.SECRET_KEY, algorithms=[settings.ALGORITHM])
        email: str = payload.get("sub")
        if email is None:
            return None
        return email
    except JWTError:
        return None

def verify_password(plain_password: str, hashed_password: str) -> bool:
    return pwd_context.verify(plain_password, hashed_password)

def get_password_hash(password: str) -> str:
    return pwd_context.hash(password)

# app/core/deps.py
from typing import Generator, Optional
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from jose import jwt, JWTError
from sqlalchemy.orm import Session
from app import crud, models
from app.core import security
from app.core.config import settings
from app.database import get_db

security_scheme = HTTPBearer()

def get_current_user(
    db: Session = Depends(get_db),
    credentials: HTTPAuthorizationCredentials = Depends(security_scheme)
) -> models.User:
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
        headers={"WWW-Authenticate": "Bearer"},
    )
    
    try:
        payload = jwt.decode(
            credentials.credentials, settings.SECRET_KEY, algorithms=[settings.ALGORITHM]
        )
        email: str = payload.get("sub")
        if email is None:
            raise credentials_exception
    except JWTError:
        raise credentials_exception
    
    user = crud.user.get_by_email(db, email=email)
    if user is None:
        raise credentials_exception
    return user

def get_current_active_user(
    current_user: models.User = Depends(get_current_user),
) -> models.User:
    if not crud.user.is_active(current_user):
        raise HTTPException(status_code=400, detail="Inactive user")
    return current_user

def get_current_active_superuser(
    current_user: models.User = Depends(get_current_user),
) -> models.User:
    if not crud.user.is_superuser(current_user):
        raise HTTPException(
            status_code=400, detail="The user doesn't have enough privileges"
        )
    return current_user

# Optional authentication (for public endpoints that can benefit from user context)
def get_optional_current_user(
    db: Session = Depends(get_db),
    credentials: Optional[HTTPAuthorizationCredentials] = Depends(HTTPBearer(auto_error=False))
) -> Optional[models.User]:
    if credentials is None:
        return None
    
    try:
        payload = jwt.decode(
            credentials.credentials, settings.SECRET_KEY, algorithms=[settings.ALGORITHM]
        )
        email: str = payload.get("sub")
        if email is None:
            return None
        
        user = crud.user.get_by_email(db, email=email)
        return user if user and crud.user.is_active(user) else None
    except JWTError:
        return None
```

### Permission-Based Authorization
```python
# app/core/permissions.py
from enum import Enum
from typing import List
from fastapi import HTTPException, status
from app.models import User

class Permission(str, Enum):
    READ_USERS = "read:users"
    WRITE_USERS = "write:users"
    DELETE_USERS = "delete:users"
    READ_ITEMS = "read:items"
    WRITE_ITEMS = "write:items"
    DELETE_ITEMS = "delete:items"
    ADMIN_ACCESS = "admin:access"

# Role-based permissions
ROLE_PERMISSIONS = {
    "user": [Permission.READ_ITEMS, Permission.WRITE_ITEMS],
    "moderator": [
        Permission.READ_ITEMS, Permission.WRITE_ITEMS, Permission.DELETE_ITEMS,
        Permission.READ_USERS
    ],
    "admin": [
        Permission.READ_USERS, Permission.WRITE_USERS, Permission.DELETE_USERS,
        Permission.READ_ITEMS, Permission.WRITE_ITEMS, Permission.DELETE_ITEMS,
        Permission.ADMIN_ACCESS
    ]
}

def has_permission(user: User, permission: Permission) -> bool:
    user_permissions = ROLE_PERMISSIONS.get(user.role, [])
    return permission in user_permissions

def require_permissions(required_permissions: List[Permission]):
    def permission_checker(current_user: User):
        for permission in required_permissions:
            if not has_permission(current_user, permission):
                raise HTTPException(
                    status_code=status.HTTP_403_FORBIDDEN,
                    detail="Insufficient permissions"
                )
        return current_user
    return permission_checker

# Usage in dependencies
from functools import partial

require_admin = partial(require_permissions, [Permission.ADMIN_ACCESS])
require_write_items = partial(require_permissions, [Permission.WRITE_ITEMS])
require_delete_users = partial(require_permissions, [Permission.DELETE_USERS])
```

## API Routes and Endpoints

### User Management Routes
```python
# app/routers/auth.py
from datetime import timedelta
from fastapi import APIRouter, Depends, HTTPException, status
from fastapi.security import OAuth2PasswordRequestForm
from sqlalchemy.orm import Session
from app import crud, schemas
from app.core import security
from app.core.config import settings
from app.database import get_db

router = APIRouter()

@router.post("/login", response_model=schemas.Token)
async def login(
    db: Session = Depends(get_db),
    form_data: OAuth2PasswordRequestForm = Depends()
):
    user = crud.user.authenticate(
        db, email=form_data.username, password=form_data.password
    )
    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Incorrect email or password",
            headers={"WWW-Authenticate": "Bearer"},
        )
    elif not crud.user.is_active(user):
        raise HTTPException(status_code=400, detail="Inactive user")
    
    access_token_expires = timedelta(minutes=settings.ACCESS_TOKEN_EXPIRE_MINUTES)
    access_token = security.create_access_token(
        data={"sub": user.email}, expires_delta=access_token_expires
    )
    return {
        "access_token": access_token,
        "token_type": "bearer",
        "expires_in": settings.ACCESS_TOKEN_EXPIRE_MINUTES * 60
    }

@router.post("/register", response_model=schemas.User)
async def register(
    user_in: schemas.UserCreate,
    db: Session = Depends(get_db)
):
    user = crud.user.get_by_email(db, email=user_in.email)
    if user:
        raise HTTPException(
            status_code=400,
            detail="The user with this email already exists in the system.",
        )
    user = crud.user.create(db, obj_in=user_in)
    return user

@router.post("/refresh-token", response_model=schemas.Token)
async def refresh_token(
    current_user: schemas.User = Depends(get_current_active_user)
):
    access_token_expires = timedelta(minutes=settings.ACCESS_TOKEN_EXPIRE_MINUTES)
    access_token = security.create_access_token(
        data={"sub": current_user.email}, expires_delta=access_token_expires
    )
    return {
        "access_token": access_token,
        "token_type": "bearer",
        "expires_in": settings.ACCESS_TOKEN_EXPIRE_MINUTES * 60
    }

# app/routers/users.py
from typing import List
from fastapi import APIRouter, Depends, HTTPException, status, Query
from sqlalchemy.orm import Session
from app import crud, models, schemas
from app.core.deps import get_current_active_user, get_current_active_superuser
from app.core.permissions import require_admin, require_delete_users
from app.database import get_db

router = APIRouter()

@router.get("/", response_model=schemas.UserListResponse)
async def read_users(
    db: Session = Depends(get_db),
    skip: int = Query(0, ge=0),
    limit: int = Query(100, ge=1, le=100),
    current_user: models.User = Depends(get_current_active_superuser)
):
    users = crud.user.get_multi(db, skip=skip, limit=limit)
    total = db.query(models.User).count()
    return schemas.UserListResponse(
        users=users,
        total=total,
        page=skip // limit + 1,
        size=limit,
        pages=(total + limit - 1) // limit
    )

@router.get("/me", response_model=schemas.User)
async def read_user_me(
    current_user: models.User = Depends(get_current_active_user)
):
    return current_user

@router.put("/me", response_model=schemas.User)
async def update_user_me(
    user_in: schemas.UserUpdate,
    db: Session = Depends(get_db),
    current_user: models.User = Depends(get_current_active_user)
):
    user = crud.user.update(db, db_obj=current_user, obj_in=user_in)
    return user

@router.get("/{user_id}", response_model=schemas.User)
async def read_user(
    user_id: int,
    db: Session = Depends(get_db),
    current_user: models.User = Depends(get_current_active_superuser)
):
    user = crud.user.get(db, id=user_id)
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    return user

@router.put("/{user_id}", response_model=schemas.User)
async def update_user(
    user_id: int,
    user_in: schemas.UserUpdate,
    db: Session = Depends(get_db),
    current_user: models.User = Depends(get_current_active_superuser)
):
    user = crud.user.get(db, id=user_id)
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    user = crud.user.update(db, db_obj=user, obj_in=user_in)
    return user

@router.delete("/{user_id}")
async def delete_user(
    user_id: int,
    db: Session = Depends(get_db),
    current_user: models.User = Depends(require_delete_users)
):
    user = crud.user.get(db, id=user_id)
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    
    if user.id == current_user.id:
        raise HTTPException(
            status_code=400, detail="Cannot delete your own user account"
        )
    
    crud.user.remove(db, id=user_id)
    return {"message": "User deleted successfully"}
```

### Item Management Routes
```python
# app/routers/items.py
from typing import List, Optional
from fastapi import APIRouter, Depends, HTTPException, Query, Path, status
from sqlalchemy.orm import Session
from app import crud, models, schemas
from app.core.deps import get_current_active_user, get_optional_current_user
from app.core.permissions import require_write_items, require_delete_items
from app.database import get_db

router = APIRouter()

@router.get("/", response_model=schemas.ItemListResponse)
async def read_items(
    db: Session = Depends(get_db),
    skip: int = Query(0, ge=0, description="Number of items to skip"),
    limit: int = Query(100, ge=1, le=100, description="Number of items to return"),
    status_filter: Optional[str] = Query(None, description="Filter by status"),
    current_user: Optional[models.User] = Depends(get_optional_current_user)
):
    if status_filter:
        items = crud.item.get_by_status(db, status=status_filter, skip=skip, limit=limit)
        total = db.query(models.Item).filter(models.Item.status == status_filter).count()
    else:
        items = crud.item.get_multi(db, skip=skip, limit=limit)
        total = db.query(models.Item).count()
    
    return schemas.ItemListResponse(
        items=items,
        total=total,
        page=skip // limit + 1,
        size=limit,
        pages=(total + limit - 1) // limit
    )

@router.post("/", response_model=schemas.Item, status_code=status.HTTP_201_CREATED)
async def create_item(
    item_in: schemas.ItemCreate,
    db: Session = Depends(get_db),
    current_user: models.User = Depends(require_write_items)
):
    item = crud.item.create_with_owner(db, obj_in=item_in, owner_id=current_user.id)
    return item

@router.get("/{item_id}", response_model=schemas.Item)
async def read_item(
    item_id: int = Path(..., description="The ID of the item to retrieve"),
    db: Session = Depends(get_db),
    current_user: Optional[models.User] = Depends(get_optional_current_user)
):
    item = crud.item.get(db, id=item_id)
    if not item:
        raise HTTPException(status_code=404, detail="Item not found")
    return item

@router.put("/{item_id}", response_model=schemas.Item)
async def update_item(
    item_id: int,
    item_in: schemas.ItemUpdate,
    db: Session = Depends(get_db),
    current_user: models.User = Depends(get_current_active_user)
):
    item = crud.item.get(db, id=item_id)
    if not item:
        raise HTTPException(status_code=404, detail="Item not found")
    
    # Check ownership or admin privileges
    if item.owner_id != current_user.id and not crud.user.is_superuser(current_user):
        raise HTTPException(status_code=403, detail="Not enough permissions")
    
    item = crud.item.update(db, db_obj=item, obj_in=item_in)
    return item

@router.delete("/{item_id}")
async def delete_item(
    item_id: int,
    db: Session = Depends(get_db),
    current_user: models.User = Depends(get_current_active_user)
):
    item = crud.item.get(db, id=item_id)
    if not item:
        raise HTTPException(status_code=404, detail="Item not found")
    
    # Check ownership or admin privileges
    if item.owner_id != current_user.id and not crud.user.is_superuser(current_user):
        raise HTTPException(status_code=403, detail="Not enough permissions")
    
    crud.item.remove(db, id=item_id)
    return {"message": "Item deleted successfully"}

@router.get("/user/me", response_model=List[schemas.Item])
async def read_user_items(
    db: Session = Depends(get_db),
    skip: int = Query(0, ge=0),
    limit: int = Query(100, ge=1, le=100),
    current_user: models.User = Depends(get_current_active_user)
):
    items = crud.item.get_multi_by_owner(
        db, owner_id=current_user.id, skip=skip, limit=limit
    )
    return items

# Advanced search endpoint
@router.get("/search/", response_model=List[schemas.Item])
async def search_items(
    q: str = Query(..., description="Search query"),
    db: Session = Depends(get_db),
    skip: int = Query(0, ge=0),
    limit: int = Query(100, ge=1, le=100),
    min_price: Optional[float] = Query(None, ge=0),
    max_price: Optional[float] = Query(None, ge=0),
    current_user: Optional[models.User] = Depends(get_optional_current_user)
):
    query = db.query(models.Item)
    
    # Text search
    query = query.filter(
        models.Item.title.contains(q) | models.Item.description.contains(q)
    )
    
    # Price filters
    if min_price is not None:
        query = query.filter(models.Item.price >= min_price)
    if max_price is not None:
        query = query.filter(models.Item.price <= max_price)
    
    items = query.offset(skip).limit(limit).all()
    return items
```

## File Upload and Media Handling

### File Upload Endpoints
```python
# app/routers/upload.py
import os
import shutil
from typing import List
from fastapi import APIRouter, Depends, HTTPException, UploadFile, File, Form
from fastapi.responses import FileResponse
from pathlib import Path
from app.core.deps import get_current_active_user
from app.models import User
from app.core.config import settings

router = APIRouter()

# Create upload directories
UPLOAD_DIR = Path("uploads")
UPLOAD_DIR.mkdir(exist_ok=True)
(UPLOAD_DIR / "avatars").mkdir(exist_ok=True)
(UPLOAD_DIR / "items").mkdir(exist_ok=True)

# Allowed file types
ALLOWED_IMAGE_TYPES = {"image/jpeg", "image/png", "image/gif", "image/webp"}
ALLOWED_DOCUMENT_TYPES = {"application/pdf", "text/plain", "application/msword"}
MAX_FILE_SIZE = 10 * 1024 * 1024  # 10MB

def validate_file(file: UploadFile, allowed_types: set, max_size: int = MAX_FILE_SIZE):
    if file.content_type not in allowed_types:
        raise HTTPException(
            status_code=400,
            detail=f"File type {file.content_type} not allowed. Allowed types: {allowed_types}"
        )
    
    # Note: file.size is not always available, so we'll check during read
    return True

@router.post("/avatar/")
async def upload_avatar(
    file: UploadFile = File(...),
    current_user: User = Depends(get_current_active_user)
):
    validate_file(file, ALLOWED_IMAGE_TYPES)
    
    # Generate unique filename
    file_extension = Path(file.filename).suffix
    filename = f"avatar_{current_user.id}_{int(time.time())}{file_extension}"
    file_path = UPLOAD_DIR / "avatars" / filename
    
    # Save file
    try:
        with open(file_path, "wb") as buffer:
            content = await file.read()
            if len(content) > MAX_FILE_SIZE:
                raise HTTPException(status_code=400, detail="File too large")
            buffer.write(content)
    except Exception as e:
        raise HTTPException(status_code=500, detail=f"Could not save file: {str(e)}")
    
    # Update user avatar in database
    # current_user.avatar = str(file_path)
    # db.commit()
    
    return {
        "filename": filename,
        "content_type": file.content_type,
        "size": len(content),
        "url": f"/uploads/avatars/{filename}"
    }

@router.post("/items/{item_id}/images/")
async def upload_item_images(
    item_id: int,
    files: List[UploadFile] = File(...),
    current_user: User = Depends(get_current_active_user)
):
    if len(files) > 5:
        raise HTTPException(status_code=400, detail="Maximum 5 files allowed")
    
    uploaded_files = []
    
    for file in files:
        validate_file(file, ALLOWED_IMAGE_TYPES)
        
        file_extension = Path(file.filename).suffix
        filename = f"item_{item_id}_{int(time.time())}_{len(uploaded_files)}{file_extension}"
        file_path = UPLOAD_DIR / "items" / filename
        
        try:
            with open(file_path, "wb") as buffer:
                content = await file.read()
                if len(content) > MAX_FILE_SIZE:
                    raise HTTPException(status_code=400, detail=f"File {file.filename} too large")
                buffer.write(content)
            
            uploaded_files.append({
                "filename": filename,
                "original_name": file.filename,
                "content_type": file.content_type,
                "size": len(content),
                "url": f"/uploads/items/{filename}"
            })
        except Exception as e:
            # Cleanup already uploaded files
            for uploaded in uploaded_files:
                try:
                    os.remove(UPLOAD_DIR / "items" / uploaded["filename"])
                except:
                    pass
            raise HTTPException(status_code=500, detail=f"Could not save files: {str(e)}")
    
    return {"uploaded_files": uploaded_files}

@router.get("/files/{category}/{filename}")
async def get_file(category: str, filename: str):
    if category not in ["avatars", "items"]:
        raise HTTPException(status_code=400, detail="Invalid category")
    
    file_path = UPLOAD_DIR / category / filename
    
    if not file_path.exists():
        raise HTTPException(status_code=404, detail="File not found")
    
    return FileResponse(file_path)

@router.delete("/files/{category}/{filename}")
async def delete_file(
    category: str,
    filename: str,
    current_user: User = Depends(get_current_active_user)
):
    if category not in ["avatars", "items"]:
        raise HTTPException(status_code=400, detail="Invalid category")
    
    file_path = UPLOAD_DIR / category / filename
    
    if not file_path.exists():
        raise HTTPException(status_code=404, detail="File not found")
    
    # Add ownership checks here
    # For avatar: check if it's the user's avatar
    # For item images: check if user owns the item
    
    try:
        os.remove(file_path)
        return {"message": "File deleted successfully"}
    except Exception as e:
        raise HTTPException(status_code=500, detail=f"Could not delete file: {str(e)}")

# Advanced file processing
@router.post("/process-image/")
async def process_image(
    file: UploadFile = File(...),
    resize_width: int = Form(None),
    resize_height: int = Form(None),
    quality: int = Form(85),
    current_user: User = Depends(get_current_active_user)
):
    validate_file(file, ALLOWED_IMAGE_TYPES)
    
    try:
        from PIL import Image
        import io
        
        # Read and process image
        content = await file.read()
        image = Image.open(io.BytesIO(content))
        
        # Resize if requested
        if resize_width or resize_height:
            if resize_width and resize_height:
                image = image.resize((resize_width, resize_height), Image.Resampling.LANCZOS)
            elif resize_width:
                ratio = resize_width / image.width
                new_height = int(image.height * ratio)
                image = image.resize((resize_width, new_height), Image.Resampling.LANCZOS)
            elif resize_height:
                ratio = resize_height / image.height
                new_width = int(image.width * ratio)
                image = image.resize((new_width, resize_height), Image.Resampling.LANCZOS)
        
        # Save processed image
        filename = f"processed_{int(time.time())}.jpg"
        file_path = UPLOAD_DIR / "processed" / filename
        file_path.parent.mkdir(exist_ok=True)
        
        image.save(file_path, "JPEG", quality=quality, optimize=True)
        
        return {
            "filename": filename,
            "url": f"/uploads/processed/{filename}",
            "original_size": f"{Image.open(io.BytesIO(content)).size}",
            "new_size": f"{image.size}",
            "quality": quality
        }
        
    except ImportError:
        raise HTTPException(
            status_code=500, 
            detail="Image processing not available. Install Pillow: pip install Pillow"
        )
    except Exception as e:
        raise HTTPException(status_code=500, detail=f"Could not process image: {str(e)}")
```

## Middleware and Error Handling

### Custom Middleware
```python
# app/middleware.py
import time
from fastapi import Request, Response
from fastapi.middleware.base import BaseHTTPMiddleware
from fastapi.responses import JSONResponse
from starlette.middleware.base import BaseHTTPMiddleware
import logging

logger = logging.getLogger(__name__)

class TimingMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        start_time = time.time()
        response = await call_next(request)
        process_time = time.time() - start_time
        response.headers["X-Process-Time"] = str(process_time)
        return response

class LoggingMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        start_time = time.time()
        
        # Log request
        logger.info(f"Request: {request.method} {request.url}")
        
        try:
            response = await call_next(request)
            process_time = time.time() - start_time
            
            # Log response
            logger.info(
                f"Response: {response.status_code} - {process_time:.4f}s"
            )
            return response
        except Exception as e:
            process_time = time.time() - start_time
            logger.error(f"Request failed: {str(e)} - {process_time:.4f}s")
            raise

class RateLimitMiddleware(BaseHTTPMiddleware):
    def __init__(self, app, max_requests: int = 100, window_seconds: int = 60):
        super().__init__(app)
        self.max_requests = max_requests
        self.window_seconds = window_seconds
        self.requests = {}
    
    async def dispatch(self, request: Request, call_next):
        client_ip = request.client.host
        current_time = time.time()
        
        # Clean old requests
        self.requests = {
            ip: times for ip, times in self.requests.items()
            if any(t > current_time - self.window_seconds for t in times)
        }
        
        # Check rate limit
        if client_ip in self.requests:
            self.requests[client_ip] = [
                t for t in self.requests[client_ip]
                if t > current_time - self.window_seconds
            ]
            
            if len(self.requests[client_ip]) >= self.max_requests:
                return JSONResponse(
                    status_code=429,
                    content={"detail": "Rate limit exceeded"}
                )
        else:
            self.requests[client_ip] = []
        
        # Add current request
        self.requests[client_ip].append(current_time)
        
        response = await call_next(request)
        return response

# Add middleware to app
from app.middleware import TimingMiddleware, LoggingMiddleware, RateLimitMiddleware

app.add_middleware(TimingMiddleware)
app.add_middleware(LoggingMiddleware)
app.add_middleware(RateLimitMiddleware, max_requests=100, window_seconds=60)
```

### Error Handling
```python
# app/core/exceptions.py
from fastapi import HTTPException, Request, status
from fastapi.responses import JSONResponse
from fastapi.exceptions import RequestValidationError
from sqlalchemy.exc import IntegrityError
import logging

logger = logging.getLogger(__name__)

class CustomHTTPException(HTTPException):
    def __init__(self, status_code: int, detail: str, error_code: str = None):
        super().__init__(status_code, detail)
        self.error_code = error_code

class DatabaseException(Exception):
    def __init__(self, message: str, error_code: str = "DATABASE_ERROR"):
        self.message = message
        self.error_code = error_code
        super().__init__(self.message)

class BusinessLogicException(Exception):
    def __init__(self, message: str, error_code: str = "BUSINESS_LOGIC_ERROR"):
        self.message = message
        self.error_code = error_code
        super().__init__(self.message)

# Exception handlers
async def validation_exception_handler(request: Request, exc: RequestValidationError):
    logger.warning(f"Validation error: {exc.errors()}")
    return JSONResponse(
        status_code=status.HTTP_422_UNPROCESSABLE_ENTITY,
        content={
            "detail": "Validation error",
            "errors": exc.errors(),
            "error_code": "VALIDATION_ERROR"
        }
    )

async def http_exception_handler(request: Request, exc: HTTPException):
    logger.warning(f"HTTP error: {exc.status_code} - {exc.detail}")
    return JSONResponse(
        status_code=exc.status_code,
        content={
            "detail": exc.detail,
            "error_code": getattr(exc, 'error_code', f"HTTP_{exc.status_code}")
        }
    )

async def database_exception_handler(request: Request, exc: DatabaseException):
    logger.error(f"Database error: {exc.message}")
    return JSONResponse(
        status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
        content={
            "detail": "Database operation failed",
            "error_code": exc.error_code
        }
    )

async def business_logic_exception_handler(request: Request, exc: BusinessLogicException):
    logger.warning(f"Business logic error: {exc.message}")
    return JSONResponse(
        status_code=status.HTTP_400_BAD_REQUEST,
        content={
            "detail": exc.message,
            "error_code": exc.error_code
        }
    )

async def integrity_error_handler(request: Request, exc: IntegrityError):
    logger.error(f"Database integrity error: {str(exc)}")
    return JSONResponse(
        status_code=status.HTTP_409_CONFLICT,
        content={
            "detail": "Resource already exists or constraint violation",
            "error_code": "INTEGRITY_ERROR"
        }
    )

async def general_exception_handler(request: Request, exc: Exception):
    logger.error(f"Unhandled error: {str(exc)}", exc_info=True)
    return JSONResponse(
        status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
        content={
            "detail": "Internal server error",
            "error_code": "INTERNAL_ERROR"
        }
    )

# Register exception handlers
from fastapi.exceptions import RequestValidationError
from sqlalchemy.exc import IntegrityError

app.add_exception_handler(RequestValidationError, validation_exception_handler)
app.add_exception_handler(HTTPException, http_exception_handler)
app.add_exception_handler(DatabaseException, database_exception_handler)
app.add_exception_handler(BusinessLogicException, business_logic_exception_handler)
app.add_exception_handler(IntegrityError, integrity_error_handler)
app.add_exception_handler(Exception, general_exception_handler)
```

## Testing

### Unit Testing with Pytest
```python
# tests/conftest.py
import pytest
from fastapi.testclient import TestClient
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from app.main import app
from app.database import get_db, Base
from app import crud, schemas
from app.core.config import settings

# Test database
SQLALCHEMY_DATABASE_URL = "sqlite:///./test.db"
engine = create_engine(
    SQLALCHEMY_DATABASE_URL, connect_args={"check_same_thread": False}
)
TestingSessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

def override_get_db():
    try:
        db = TestingSessionLocal()
        yield db
    finally:
        db.close()

app.dependency_overrides[get_db] = override_get_db

@pytest.fixture(scope="session")
def db():
    Base.metadata.create_all(bind=engine)
    yield
    Base.metadata.drop_all(bind=engine)

@pytest.fixture
def client():
    return TestClient(app)

@pytest.fixture
def test_user():
    db = TestingSessionLocal()
    user_in = schemas.UserCreate(
        email="test@example.com",
        password="testpassword123",
        first_name="Test",
        last_name="User"
    )
    user = crud.user.create(db, obj_in=user_in)
    db.close()
    return user

@pytest.fixture
def admin_user():
    db = TestingSessionLocal()
    user_in = schemas.UserCreate(
        email="admin@example.com",
        password="adminpassword123",
        first_name="Admin",
        last_name="User",
        role="admin"
    )
    user = crud.user.create(db, obj_in=user_in)
    db.close()
    return user

@pytest.fixture
def authenticated_client(client, test_user):
    # Login and get token
    response = client.post(
        "/api/v1/auth/login",
        data={"username": test_user.email, "password": "testpassword123"}
    )
    token = response.json()["access_token"]
    
    # Set authorization header
    client.headers.update({"Authorization": f"Bearer {token}"})
    return client

# tests/test_auth.py
def test_login_success(client, test_user):
    response = client.post(
        "/api/v1/auth/login",
        data={"username": test_user.email, "password": "testpassword123"}
    )
    assert response.status_code == 200
    data = response.json()
    assert "access_token" in data
    assert data["token_type"] == "bearer"

def test_login_invalid_credentials(client):
    response = client.post(
        "/api/v1/auth/login",
        data={"username": "wrong@example.com", "password": "wrongpassword"}
    )
    assert response.status_code == 401

def test_register_success(client):
    user_data = {
        "email": "newuser@example.com",
        "password": "newpassword123",
        "first_name": "New",
        "last_name": "User"
    }
    response = client.post("/api/v1/auth/register", json=user_data)
    assert response.status_code == 200
    data = response.json()
    assert data["email"] == user_data["email"]
    assert "id" in data

def test_register_duplicate_email(client, test_user):
    user_data = {
        "email": test_user.email,
        "password": "password123",
        "first_name": "Duplicate",
        "last_name": "User"
    }
    response = client.post("/api/v1/auth/register", json=user_data)
    assert response.status_code == 400

# tests/test_users.py
def test_read_users_as_admin(client, admin_user):
    # Login as admin
    response = client.post(
        "/api/v1/auth/login",
        data={"username": admin_user.email, "password": "adminpassword123"}
    )
    token = response.json()["access_token"]
    
    # Get users
    response = client.get(
        "/api/v1/users/",
        headers={"Authorization": f"Bearer {token}"}
    )
    assert response.status_code == 200
    data = response.json()
    assert "users" in data
    assert "total" in data

def test_read_users_as_regular_user(authenticated_client):
    response = authenticated_client.get("/api/v1/users/")
    assert response.status_code == 403

def test_read_user_me(authenticated_client, test_user):
    response = authenticated_client.get("/api/v1/users/me")
    assert response.status_code == 200
    data = response.json()
    assert data["email"] == test_user.email

def test_update_user_me(authenticated_client):
    update_data = {"first_name": "Updated", "last_name": "Name"}
    response = authenticated_client.put("/api/v1/users/me", json=update_data)
    assert response.status_code == 200
    data = response.json()
    assert data["first_name"] == "Updated"
    assert data["last_name"] == "Name"

# tests/test_items.py
def test_create_item(authenticated_client):
    item_data = {
        "title": "Test Item",
        "description": "Test Description",
        "price": 99.99,
        "tax": 10.0
    }
    response = authenticated_client.post("/api/v1/items/", json=item_data)
    assert response.status_code == 201
    data = response.json()
    assert data["title"] == item_data["title"]
    assert data["price"] == item_data["price"]

def test_read_items(client):
    response = client.get("/api/v1/items/")
    assert response.status_code == 200
    data = response.json()
    assert "items" in data

def test_read_item_not_found(client):
    response = client.get("/api/v1/items/999")
    assert response.status_code == 404

# Run tests
# pytest -v
# pytest --cov=app
# pytest -k "test_auth"
```

### Integration Testing
```python
# tests/test_integration.py
import pytest
from fastapi.testclient import TestClient

def test_full_user_workflow(client):
    # Register user
    user_data = {
        "email": "workflow@example.com",
        "password": "password123",
        "first_name": "Workflow",
        "last_name": "User"
    }
    response = client.post("/api/v1/auth/register", json=user_data)
    assert response.status_code == 200
    
    # Login
    response = client.post(
        "/api/v1/auth/login",
        data={"username": user_data["email"], "password": user_data["password"]}
    )
    assert response.status_code == 200
    token = response.json()["access_token"]
    
    headers = {"Authorization": f"Bearer {token}"}
    
    # Create item
    item_data = {
        "title": "Workflow Item",
        "description": "Created in workflow test",
        "price": 29.99
    }
    response = client.post("/api/v1/items/", json=item_data, headers=headers)
    assert response.status_code == 201
    item_id = response.json()["id"]
    
    # Read item
    response = client.get(f"/api/v1/items/{item_id}")
    assert response.status_code == 200
    
    # Update item
    update_data = {"title": "Updated Workflow Item"}
    response = client.put(f"/api/v1/items/{item_id}", json=update_data, headers=headers)
    assert response.status_code == 200
    assert response.json()["title"] == update_data["title"]
    
    # Delete item
    response = client.delete(f"/api/v1/items/{item_id}", headers=headers)
    assert response.status_code == 200
    
    # Verify deletion
    response = client.get(f"/api/v1/items/{item_id}")
    assert response.status_code == 404
```

## Production Deployment

### Docker Configuration
```dockerfile
# Dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

# Install Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Create non-root user
RUN useradd --create-home --shell /bin/bash app && chown -R app:app /app
USER app

# Expose port
EXPOSE 8000

# Run the application
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]

# docker-compose.yml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://user:password@db:5432/fastapi_db
      - SECRET_KEY=your-super-secret-key
    depends_on:
      - db
      - redis
    volumes:
      - ./uploads:/app/uploads

  db:
    image: postgres:13
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=fastapi_db
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  redis:
    image: redis:6-alpine
    ports:
      - "6379:6379"

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/ssl
    depends_on:
      - app

volumes:
  postgres_data:
```

### Production Settings
```python
# app/core/config.py - Production additions
class ProductionSettings(Settings):
    # Security
    SECRET_KEY: str = Field(..., env="SECRET_KEY")
    ALLOWED_HOSTS: List[str] = Field(default_factory=list, env="ALLOWED_HOSTS")
    
    # Database
    DATABASE_URL: str = Field(..., env="DATABASE_URL")
    
    # Redis
    REDIS_URL: str = Field(default="redis://localhost:6379", env="REDIS_URL")
    
    # Logging
    LOG_LEVEL: str = Field(default="INFO", env="LOG_LEVEL")
    
    # File storage (S3, etc.)
    USE_S3: bool = Field(default=False, env="USE_S3")
    AWS_ACCESS_KEY_ID: Optional[str] = Field(None, env="AWS_ACCESS_KEY_ID")
    AWS_SECRET_ACCESS_KEY: Optional[str] = Field(None, env="AWS_SECRET_ACCESS_KEY")
    AWS_S3_BUCKET_NAME: Optional[str] = Field(None, env="AWS_S3_BUCKET_NAME")
    
    # Monitoring
    SENTRY_DSN: Optional[str] = Field(None, env="SENTRY_DSN")
    
    class Config:
        env_file = ".env.production"

# Performance and monitoring
import sentry_sdk
from sentry_sdk.integrations.fastapi import FastApiIntegration

if settings.SENTRY_DSN:
    sentry_sdk.init(
        dsn=settings.SENTRY_DSN,
        integrations=[FastApiIntegration(auto_enabling=True)],
        traces_sample_rate=0.1,
    )

# Caching with Redis
import redis
from functools import wraps
import json
import pickle

redis_client = redis.from_url(settings.REDIS_URL)

def cache_result(expire_time: int = 300):
    def decorator(func):
        @wraps(func)
        async def wrapper(*args, **kwargs):
            # Create cache key
            cache_key = f"{func.__name__}:{hash(str(args) + str(kwargs))}"
            
            # Try to get from cache
            cached_result = redis_client.get(cache_key)
            if cached_result:
                return pickle.loads(cached_result)
            
            # Execute function and cache result
            result = await func(*args, **kwargs)
            redis_client.setex(cache_key, expire_time, pickle.dumps(result))
            return result
        return wrapper
    return decorator

# Usage
@cache_result(expire_time=600)  # 10 minutes
async def get_popular_items(db: Session):
    return db.query(Item).order_by(Item.views.desc()).limit(10).all()
```

### Performance Optimization
```python
# app/core/performance.py
from sqlalchemy.orm import selectinload, joinedload
from functools import lru_cache
import asyncio
from concurrent.futures import ThreadPoolExecutor

# Database query optimization
def get_items_optimized(db: Session, skip: int = 0, limit: int = 100):
    return (
        db.query(Item)
        .options(
            selectinload(Item.tags),  # Efficient for one-to-many
            joinedload(Item.owner)    # Efficient for many-to-one
        )
        .offset(skip)
        .limit(limit)
        .all()
    )

# Background tasks
from fastapi import BackgroundTasks

def send_email_notification(email: str, subject: str, body: str):
    # Simulate sending email
    import time
    time.sleep(2)  # Simulate email sending delay
    print(f"Email sent to {email}: {subject}")

@router.post("/items/")
async def create_item_with_notification(
    item_in: schemas.ItemCreate,
    background_tasks: BackgroundTasks,
    db: Session = Depends(get_db),
    current_user: models.User = Depends(get_current_active_user)
):
    item = crud.item.create_with_owner(db, obj_in=item_in, owner_id=current_user.id)
    
    # Send notification in background
    background_tasks.add_task(
        send_email_notification,
        current_user.email,
        "Item Created",
        f"Your item '{item.title}' has been created successfully."
    )
    
    return item

# Async database operations
import asyncpg
import databases

# For async database operations (alternative to SQLAlchemy)
database = databases.Database(settings.DATABASE_URL)

@app.on_event("startup")
async def startup():
    await database.connect()

@app.on_event("shutdown")
async def shutdown():
    await database.disconnect()

# Connection pooling and performance settings
engine = create_engine(
    settings.DATABASE_URL,
    pool_size=20,
    max_overflow=0,
    pool_pre_ping=True,
    pool_recycle=300,
)
```