# Service Layer - FastAPI

## Service Pattern

Services contain business logic and orchestrate repositories.

### UserService Example

```python
# backend/domain/user/service.py
from typing import List, Optional
from datetime import datetime, timezone
from sqlmodel.ext.asyncio.session import AsyncSession

from backend.domain.user.repository import UserRepository, UserDataLoader
from backend.domain.user.model import User
from backend.dtos.user import (
    UserCreateRequest,
    UserResponse,
    UserUpdateRequest,
    UserListResponse,
)
from backend.error import NotFoundError, ConflictError

class UserService:
    def __init__(self, session: AsyncSession):
        self.session = session
        self._repository = UserRepository(session)
        self._data_loader = UserDataLoader(session)

    async def get_user(self, user_id: str) -> UserResponse:
        """Get user by ID"""
        user = await self._repository.get_async(user_id)
        if not user:
            raise NotFoundError(f"User {user_id} not found")
        return UserResponse.model_validate(user)

    async def create_user(self, request: UserCreateRequest) -> UserResponse:
        """Create new user with business validation"""
        # Business rule: Phone must be unique
        existing = await self._repository.find_by_phone(request.phone)
        if existing:
            raise ConflictError("Phone number already registered")

        user = await self._repository.create_async(**request.model_dump())
        return UserResponse.model_validate(user)

    async def update_user(self, user_id: str, request: UserUpdateRequest) -> UserResponse:
        """Update user (only non-None fields)"""
        user = await self._repository.get_async(user_id)
        if not user:
            raise NotFoundError(f"User {user_id} not found")

        update_data = request.model_dump(exclude_unset=True)
        updated = await self._repository.update_async(user_id, **update_data)
        return UserResponse.model_validate(updated)

    async def delete_user(self, user_id: str) -> None:
        """Soft delete user"""
        user = await self._repository.get_async(user_id)
        if not user:
            raise NotFoundError(f"User {user_id} not found")
        await self._repository.update_async(
            user_id, deleted_at=datetime.now(tz=timezone.utc)
        )
```

### AuthService (backend/domain/user/auth_service.py)

```python
# backend/domain/user/auth_service.py
from datetime import datetime, timezone, timedelta
from sqlmodel.ext.asyncio.session import AsyncSession
from fastapi import Depends
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials

from backend.domain.user.repository import UserRepository
from backend.dtos.auth import EmailLoginRequestDto, EmailSignUpRequestDto, LoginResponseDto
from backend.error import UnauthorizedError, ConflictError
from backend.utils.password import hash_password, verify_password
from backend.core.config import settings
import jwt

security = HTTPBearer()

class AuthService:
    def __init__(self, session: AsyncSession):
        self.session = session
        self._user_repository = UserRepository(session)

    async def email_sign_up(self, request: EmailSignUpRequestDto) -> LoginResponseDto:
        """이메일 회원가입 및 JWT 발급"""
        existing = await self._user_repository.find_by_email(request.email)
        if existing:
            raise ConflictError("Email already registered")

        hashed_pw = hash_password(request.password)
        user = await self._user_repository.create_async(
            email=request.email,
            hashed_password=hashed_pw,
            name=request.username,
        )
        return self._build_login_response(user)

    async def email_login(self, request: EmailLoginRequestDto) -> LoginResponseDto:
        """이메일/비밀번호 로그인"""
        user = await self._user_repository.find_by_email(request.email)
        if not user or not verify_password(request.password, user.hashed_password):
            raise UnauthorizedError("Invalid credentials")
        return self._build_login_response(user)

    def _build_login_response(self, user) -> LoginResponseDto:
        now = datetime.now(tz=timezone.utc)
        access_payload = {
            "sub": user.id,
            "type": "access",
            "exp": now + timedelta(minutes=settings.access_token_expire_minutes),
        }
        refresh_payload = {
            "sub": user.id,
            "type": "refresh",
            "exp": now + timedelta(days=settings.refresh_token_expire_days),
        }
        return LoginResponseDto(
            user_id=user.id,
            app_auth_token=jwt.encode(access_payload, settings.jwt_secret_key, algorithm=settings.jwt_algorithm),
            refresh_token=jwt.encode(refresh_payload, settings.jwt_secret_key, algorithm=settings.jwt_algorithm),
        )


async def get_user_id(
    credentials: HTTPAuthorizationCredentials = Depends(security),
) -> str:
    """JWT 토큰에서 user_id 추출 (FastAPI 의존성)"""
    # MOCK_AUTH_ENABLED=true 시 mock-user-001 반환
    if settings.mock_auth_enabled:
        return "mock-user-001"
    try:
        payload = jwt.decode(
            credentials.credentials,
            settings.jwt_secret_key,
            algorithms=[settings.jwt_algorithm],
        )
        return payload["sub"]
    except jwt.ExpiredSignatureError:
        raise UnauthorizedError("Token has expired")
    except jwt.InvalidTokenError:
        raise UnauthorizedError("Invalid token")
```

## Service Responsibilities

1. **Business Logic**: Implement domain rules
2. **Validation**: Business-level validation (beyond DTO)
3. **Orchestration**: Coordinate multiple repositories
4. **Transformation**: Model ↔ DTO conversion
5. **Error Handling**: Raise domain exceptions

## Multi-Repository Service Pattern (asyncio.gather 활용)

```python
import asyncio

class UserService:
    def __init__(self, session: AsyncSession):
        self.session = session
        self._repository = UserRepository(session)
        self._data_loader = UserDataLoader(session)

    async def get_user_detail(self, user_id: str) -> UserDetailResponse:
        """Relations을 병렬로 로드하여 N+1 방지"""
        user_with_relations = await self._data_loader.load_user_with_relations(
            user_id,
            load_profile=True,
            load_photos=True,
        )
        if not user_with_relations:
            raise NotFoundError(f"User {user_id} not found")
        return UserDetailResponse.model_validate(user_with_relations.user)

    async def get_stats(self) -> dict:
        """독립적인 쿼리를 병렬 실행"""
        total, active, pending = await asyncio.gather(
            self._repository.count_async(),
            self._repository.count_async(filters={"status": "active"}),
            self._repository.count_async(filters={"status": "pending"}),
        )
        return {"total": total, "active": active, "pending": pending}
```

## Best Practices

1. **One service per domain**: UserService for user domain, AuthService in auth_service.py
2. **Inject session**: Accept AsyncSession in constructor
3. **Return DTOs**: Never return models directly to router (use `model_validate()`)
4. **Raise exceptions**: Use domain exceptions for errors
5. **Business rules**: Enforce in service, not repository
6. **Parallel queries**: Use `asyncio.gather()` for independent queries
7. **Data loader**: Use UserDataLoader for loading user relations (N+1 방지)
8. **Timezone-aware timestamps**: `datetime.now(tz=timezone.utc)` — `datetime.utcnow()` 사용 금지
