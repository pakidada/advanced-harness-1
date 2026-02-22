---
name: nextjs-frontend-guidelines
description: Next.js 15 frontend development guidelines for 연결사 (yeongyeolsa) React 19/TypeScript application. Modern patterns including App Router, Server/Client Components, shadcn/ui components, Tailwind CSS 4, JWT + Firebase authentication, and Korean localization. Use when creating components, pages, API routes, fetching data, styling, or working with frontend code.
---

# Next.js 15 Frontend Development Guidelines for 연결사

## Purpose

Comprehensive guide for 연결사 (yeongyeolsa) frontend development with Next.js 15, React 19, emphasizing App Router patterns, Server/Client component separation, shadcn/ui components, Tailwind CSS 4 styling, JWT + Firebase authentication, and Korean localization.

## When to Use This Skill

- Creating new components or pages
- Building new features with App Router
- Fetching data with Server Components or client-side patterns
- Styling components with shadcn/ui and Tailwind CSS 4
- Setting up API routes or Server Actions
- Authentication flows (email/password JWT, Firebase Google)
- Performance optimization
- Organizing frontend code
- TypeScript best practices

---

## Quick Start

### New Component Checklist

Creating a component? Follow this checklist:

- [ ] Determine if Server or Client Component
- [ ] Use `'use client'` directive only when needed
- [ ] Props type with TypeScript interface
- [ ] Use `@/` import alias for project imports
- [ ] Use shadcn/ui components where applicable
- [ ] Use `cn()` utility for conditional classes
- [ ] Named export for components
- [ ] Async Server Components for data fetching when possible
- [ ] Client Components for interactivity (useState, useEffect, event handlers)
- [ ] Korean text for UI labels

### New Feature Checklist

Creating a feature? Set up this structure:

- [ ] Create `src/components/{feature-name}/` directory
- [ ] Separate Server and Client components
- [ ] Create API route if needed: `src/app/api/{feature}/route.ts`
- [ ] Set up TypeScript types in `src/types/`
- [ ] Create route in `src/app/{feature-name}/page.tsx`
- [ ] Use Server Components by default
- [ ] Add Client Components only for interactivity
- [ ] Use Server Actions for mutations when appropriate
- [ ] Add constants/enums to `src/constants/enums.ts` if needed

---

## Project Structure

연결사 프로젝트 구조 (`@/` alias로 import):

```
src/
├── app/                        # Next.js App Router
│   ├── page.tsx                # Home/Landing page
│   ├── layout.tsx              # Root layout (AuthProvider, Pretendard font)
│   ├── error.tsx               # Global error boundary
│   ├── loading.tsx             # Global loading state
│   ├── globals.css             # Tailwind CSS v4 theme (@theme syntax)
│   ├── login/
│   │   └── page.tsx            # Login + Sign-up page
│   └── api/
│       └── auth/session/
│           └── route.ts        # HTTP-only cookie session sync (POST/DELETE/GET)
├── components/                 # React components
│   ├── layout/                 # Layout components
│   │   ├── Navbar.tsx          # Responsive nav with auth state
│   │   └── Footer.tsx          # Footer with copyright
│   └── ui/                     # shadcn/ui components (11)
│       ├── alert.tsx
│       ├── badge.tsx           # Variants: default, secondary, destructive, outline, success, warning, info
│       ├── button.tsx          # Variants: default, destructive, outline, secondary, ghost, link, accent
│       ├── card.tsx
│       ├── checkbox.tsx
│       ├── dialog.tsx
│       ├── input.tsx
│       ├── label.tsx
│       ├── select.tsx
│       ├── skeleton.tsx
│       └── textarea.tsx
├── constants/                  # Constants & enums
│   └── enums.ts                # Enum options with Korean labels + helper functions
├── fonts/                      # Local fonts
│   └── PretendardVariable.woff2
├── lib/                        # Core utilities
│   ├── api.ts                  # API client with JWT token management
│   └── utils.ts                # cn() helper (clsx + tailwind-merge)
├── providers/                  # Context providers
│   └── AuthProvider.tsx        # Auth state context (email + Firebase Google)
├── types/                      # TypeScript definitions
│   └── index.ts                # NavItem, Feature, Review interfaces
└── middleware.ts               # Route protection middleware
```

---

## Import Patterns

| Pattern | Usage | Example |
|---------|-------|---------|
| `@/` | Project imports (primary) | `import { api } from '@/lib/api'` |
| Relative | Same directory | `import { Component } from './Component'` |
| `type` | Type-only imports | `import type { NavItem } from '@/types'` |

---

## Common Imports Cheatsheet

```typescript
// Server Component (no 'use client')
import { Suspense } from 'react';
import { redirect } from 'next/navigation';
import { cookies } from 'next/headers';
import { Button } from '@/components/ui/button';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import type { Metadata } from 'next';

// Client Component
'use client';

import { useState, useEffect, useCallback } from 'react';
import { useRouter, useSearchParams } from 'next/navigation';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Dialog, DialogContent, DialogHeader, DialogTitle, DialogFooter } from '@/components/ui/dialog';
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select';
import { Badge } from '@/components/ui/badge';
import { Skeleton } from '@/components/ui/skeleton';
import { useAuth, useIsAuthenticated } from '@/providers/AuthProvider';
import { api } from '@/lib/api';
import { cn } from '@/lib/utils';
import { Loader2, AlertCircle } from 'lucide-react';

// Types
import type { NavItem, Feature, Review } from '@/types';

// Constants
import { USER_STATUS_OPTIONS, GENDER_OPTIONS, getEnumLabel } from '@/constants/enums';
```

---

## Topic Guides

### shadcn/ui Overview

**What is shadcn/ui?**
- Beautifully designed, accessible components built on Radix UI
- Copy/paste components into your project (not a npm package dependency)
- Fully customizable with Tailwind CSS
- TypeScript-first with full type safety

**Key Concepts:**
- Components live in `src/components/ui/`
- Use `cn()` utility for class merging (clsx + tailwind-merge)
- Variants via class-variance-authority (cva)
- Follows Radix UI accessibility patterns

**Available Components:**
- alert, badge, button, card, checkbox, dialog, input, label, select, skeleton, textarea

**Adding New Components:**
```bash
# 반드시 pnpm dlx 사용 (npx 아님)
pnpm dlx shadcn@latest add button
pnpm dlx shadcn@latest add form
pnpm dlx shadcn@latest add table
```

**Badge Variants:**
```typescript
<Badge variant="default" />      // Blue/primary
<Badge variant="secondary" />    // Gray
<Badge variant="destructive" />  // Red
<Badge variant="outline" />      // Border only
<Badge variant="success" />      // Green
<Badge variant="warning" />      // Yellow
<Badge variant="info" />         // Light blue
```

**Button Variants:**
```typescript
<Button variant="default" />     // Primary blue
<Button variant="destructive" /> // Red
<Button variant="outline" />     // Border
<Button variant="secondary" />   // Gray
<Button variant="ghost" />       // No background
<Button variant="link" />        // Underline
<Button variant="accent" />      // Yellow accent
```

---

### Component Patterns

**Server vs Client Components:**
- **Server Components (default)**: Data fetching, static content, no interactivity
- **Client Components ('use client')**: State, effects, event handlers, browser APIs

**Key Concepts:**
- Server Components are async and fetch data directly
- Client Components need 'use client' directive at the top
- Minimize Client Components for better performance
- Pass data from Server to Client Components via props
- Component structure: Props → Hooks → Handlers → Render → Export

**프로젝트 패턴:**
- Layout components (Navbar, Footer)는 Client Components (auth state 사용)
- Landing page는 Server Component (static content)
- 폼은 manual state management 사용 (react-hook-form 없음)

**[Complete Guide: resources/component-patterns.md](resources/component-patterns.md)**

---

### Authentication (연결사-Specific)

**인증 방식:**
1. **Email/Password**: 이메일 + 비밀번호 로그인/회원가입
2. **Firebase Google**: Google 소셜 로그인
3. **JWT**: 12시간 access token, 30일 refresh token

**토큰 저장:**
- localStorage: `app_access_token`, `app_refresh_token`
- HTTP-only cookies: SSR/middleware 접근용 (`/api/auth/session` 통해 동기화)

**AuthProvider Context:**
```typescript
interface AuthContextType {
  user: UserInfo | null;
  isLoading: boolean;
  isAuthenticated: boolean;
  emailLogin: (email: string, password: string) => Promise<void>;
  emailSignUp: (email: string, password: string, username: string) => Promise<void>;
  login: (loginResponse: LoginResponse) => void;
  logout: () => Promise<void>;
  refreshUser: () => Promise<void>;
}

interface LoginResponse {
  user_id: string;
  app_auth_token: string;
  refresh_token: string;
  nickname: string | null;
}

interface UserInfo {
  id: string;
  nickname: string;
  email: string | null;
  auth_type: string;
  is_admin: boolean;
  is_premium: boolean;
}
```

**useAuth() 사용 패턴:**
```typescript
'use client';

import { useAuth } from '@/providers/AuthProvider';

export function MyComponent() {
  const {
    user,
    isLoading,
    isAuthenticated,
    emailLogin,
    emailSignUp,
    logout,
    refreshUser,
  } = useAuth();

  if (isLoading) return <Skeleton className="h-8 w-32" />;
  if (!isAuthenticated) return <LoginPrompt />;

  return <div>안녕하세요, {user?.nickname}님</div>;
}
```

**useIsAuthenticated() 훅 (간단한 체크용):**
```typescript
import { useIsAuthenticated } from '@/providers/AuthProvider';

export function ProtectedButton() {
  const isAuthenticated = useIsAuthenticated();

  if (!isAuthenticated) return null;
  return <Button>보호된 액션</Button>;
}
```

**Hydration Protection Pattern (Navbar 등):**
```typescript
'use client';

import { useState, useEffect } from 'react';
import { useAuth } from '@/providers/AuthProvider';

export function Navbar() {
  const { user, isLoading, isAuthenticated } = useAuth();
  const [mounted, setMounted] = useState(false);

  useEffect(() => {
    setMounted(true);
  }, []);

  return (
    <nav>
      {/* 정적 콘텐츠는 항상 렌더링 */}
      <Logo />

      {/* auth 상태 의존 콘텐츠는 mount 후에만 */}
      {mounted && !isLoading ? (
        isAuthenticated ? <UserMenu user={user} /> : <LoginButton />
      ) : (
        <div className="w-20 h-10" /> // 플레이스홀더
      )}
    </nav>
  );
}
```

**이메일 로그인/회원가입 폼:**
```typescript
'use client';

import { useState } from 'react';
import { useAuth } from '@/providers/AuthProvider';
import { useRouter } from 'next/navigation';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';

export function LoginForm() {
  const { emailLogin } = useAuth();
  const router = useRouter();
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState('');

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setLoading(true);
    setError('');
    try {
      await emailLogin(email, password);
      router.push('/');
    } catch (err) {
      setError('이메일 또는 비밀번호가 올바르지 않습니다.');
    } finally {
      setLoading(false);
    }
  };

  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      <Input
        type="email"
        value={email}
        onChange={e => setEmail(e.target.value)}
        placeholder="이메일"
        required
      />
      <Input
        type="password"
        value={password}
        onChange={e => setPassword(e.target.value)}
        placeholder="비밀번호"
        required
      />
      {error && <p className="text-sm text-red-500">{error}</p>}
      <Button type="submit" disabled={loading} className="w-full">
        {loading ? '로그인 중...' : '로그인'}
      </Button>
    </form>
  );
}
```

---

### Data Fetching

**Server Component Data Fetching (권장):**
```typescript
// app/some-page/page.tsx
import { api } from '@/lib/api';

export default async function SomePage() {
  // Server Component에서는 직접 fetch 또는 api 호출
  const data = await api.get<DataType>('/api/v1/some-endpoint');

  return <DataDisplay data={data} />;
}
```

**Client-Side Data Fetching:**
```typescript
'use client';

import { useState, useEffect, useCallback } from 'react';
import { api } from '@/lib/api';
import { Skeleton } from '@/components/ui/skeleton';
import { Button } from '@/components/ui/button';

interface DataItem {
  id: string;
  title: string;
}

export function DataList() {
  const [items, setItems] = useState<DataItem[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  const fetchData = useCallback(async () => {
    try {
      setLoading(true);
      const response = await api.get<{ items: DataItem[] }>('/api/v1/items');
      setItems(response.items);
    } catch (err) {
      setError(err instanceof Error ? err.message : '데이터를 불러오는데 실패했습니다.');
    } finally {
      setLoading(false);
    }
  }, []);

  useEffect(() => {
    fetchData();
  }, [fetchData]);

  if (loading) {
    return (
      <div className="space-y-4">
        {[...Array(5)].map((_, i) => (
          <Skeleton key={i} className="h-16 w-full" />
        ))}
      </div>
    );
  }

  if (error) {
    return (
      <div className="text-center py-8">
        <p className="text-red-500 mb-4">{error}</p>
        <Button onClick={fetchData}>다시 시도</Button>
      </div>
    );
  }

  return (
    <div className="space-y-4">
      {items.map(item => (
        <div key={item.id} className="p-4 border rounded-lg">
          {item.title}
        </div>
      ))}
    </div>
  );
}
```

**[Complete Guide: resources/data-fetching.md](resources/data-fetching.md)**

---

### API Client Patterns

**API Client (`lib/api.ts`) 주요 기능:**
- 환경별 API URL 자동 선택 (`NEXT_PUBLIC_ENV` 기반)
- localStorage에서 토큰 자동 첨부 (Bearer)
- 401 시 자동 토큰 갱신
- HTTP-only 쿠키 동기화 (`/api/auth/session`)

```typescript
import { api, emailLogin, emailSignUp, getCurrentUser, refreshTokens } from '@/lib/api';
import { getAccessToken, getRefreshToken, setTokens, clearTokens, hasTokens } from '@/lib/api';

// Generic HTTP methods
const data = await api.get<ResponseType>('/api/v1/endpoint');
await api.post('/api/v1/endpoint', body);
await api.put('/api/v1/endpoint', body);
await api.patch('/api/v1/endpoint', changes);
await api.delete('/api/v1/endpoint');

// Auth methods
await emailLogin(email, password);          // → LoginResponse
await emailSignUp(email, password, name);   // → LoginResponse
const user = await getCurrentUser();        // → UserInfo
await refreshTokens();                      // 토큰 갱신

// Token management
const token = getAccessToken();
const refresh = getRefreshToken();
setTokens(accessToken, refreshToken);
clearTokens();
const authenticated = hasTokens();
```

**환경 변수:**
```bash
# frontend/.env
NEXT_PUBLIC_API_URL_DEV=http://localhost:28080   # 개발 서버 URL
NEXT_PUBLIC_API_URL_PROD=https://api.example.com # 프로덕션 URL
NEXT_PUBLIC_ENV=development                       # 'development' | 'production'
```

**ApiError 처리:**
```typescript
import { api, ApiError } from '@/lib/api';

try {
  await api.post('/api/v1/auth/login', { email, password });
} catch (err) {
  if (err instanceof ApiError) {
    console.log(err.status);   // HTTP status code
    console.log(err.message);  // Error message
  }
}
```

---

### Constants & Enums Pattern

**`constants/enums.ts` 현재 정의:**
```typescript
// User status
export const USER_STATUS_OPTIONS = [
  { value: "draft", label: "상담 전" },
  { value: "pending_review", label: "상담 예정" },
  { value: "active", label: "상담 완료" },
  { value: "suspended", label: "정지" },
  { value: "withdrawn", label: "탈퇴" },
] as const;

// Gender
export const GENDER_OPTIONS = [
  { value: "male", label: "남성" },
  { value: "female", label: "여성" },
] as const;

// Education, Smoking, Religion, Tattoo, Car, DINK, Divorce, Long-distance...
// (상세 옵션은 enums.ts 참조)

// Document status
export const DOCUMENT_STATUS_OPTIONS = [
  { value: "pending", label: "검토 중" },
  { value: "approved", label: "승인됨" },
  { value: "rejected", label: "반려됨" },
] as const;

// Helper functions
export function getEnumLabel(
  options: readonly { value: string; label: string }[],
  value: string | null | undefined
): string {
  if (!value) return "-";
  return options.find(o => o.value === value)?.label ?? value;
}

// Shorthand helpers
export function getUserStatusLabel(value: string | null | undefined): string
export function getGenderLabel(value: string | null | undefined): string
export function getEducationLabel(value: string | null | undefined): string
export function getSmokingLabel(value: string | null | undefined): string
export function getReligionLabel(value: string | null | undefined): string
export function getDocumentStatusLabel(value: string | null | undefined): string
```

**Select 컴포넌트에서 사용:**
```typescript
import { USER_STATUS_OPTIONS, getEnumLabel } from '@/constants/enums';

<Select value={status} onValueChange={setStatus}>
  <SelectTrigger>
    <SelectValue placeholder="상태 선택" />
  </SelectTrigger>
  <SelectContent>
    {USER_STATUS_OPTIONS.map(option => (
      <SelectItem key={option.value} value={option.value}>
        {option.label}
      </SelectItem>
    ))}
  </SelectContent>
</Select>

// 레이블 표시
<span>{getEnumLabel(USER_STATUS_OPTIONS, item.status)}</span>
```

---

### Form Patterns

**Manual State Management (프로젝트 표준):**
```typescript
'use client';

import { useState, useCallback } from 'react';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { api } from '@/lib/api';
import { cn } from '@/lib/utils';
import { Loader2 } from 'lucide-react';

interface FormData {
  email: string;
  username: string;
}

interface FormErrors {
  email?: string;
  username?: string;
}

export function SampleForm() {
  const [formData, setFormData] = useState<FormData>({ email: '', username: '' });
  const [errors, setErrors] = useState<FormErrors>({});
  const [loading, setLoading] = useState(false);

  const validate = (): boolean => {
    const newErrors: FormErrors = {};
    if (!formData.email.includes('@')) {
      newErrors.email = "올바른 이메일 형식이 아닙니다.";
    }
    if (formData.username.length < 2) {
      newErrors.username = "이름은 2자 이상이어야 합니다.";
    }
    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };

  const handleChange = useCallback((field: keyof FormData, value: string) => {
    setFormData(prev => ({ ...prev, [field]: value }));
    if (errors[field]) {
      setErrors(prev => ({ ...prev, [field]: undefined }));
    }
  }, [errors]);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    if (!validate()) return;

    setLoading(true);
    try {
      await api.post('/api/v1/some-endpoint', formData);
    } catch (err) {
      setErrors({ email: "제출에 실패했습니다. 다시 시도해주세요." });
    } finally {
      setLoading(false);
    }
  };

  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      <div className="space-y-2">
        <Input
          type="email"
          value={formData.email}
          onChange={e => handleChange('email', e.target.value)}
          placeholder="이메일"
          className={cn(errors.email && "border-red-500")}
        />
        {errors.email && <p className="text-sm text-red-500">{errors.email}</p>}
      </div>
      <div className="space-y-2">
        <Input
          value={formData.username}
          onChange={e => handleChange('username', e.target.value)}
          placeholder="이름"
          className={cn(errors.username && "border-red-500")}
        />
        {errors.username && <p className="text-sm text-red-500">{errors.username}</p>}
      </div>
      <Button type="submit" disabled={loading} className="w-full">
        {loading && <Loader2 className="animate-spin mr-2 h-4 w-4" />}
        {loading ? "처리 중..." : "제출"}
      </Button>
    </form>
  );
}
```

**Modal Dialog Form:**
```typescript
'use client';

import { useState, useEffect } from 'react';
import { Dialog, DialogContent, DialogHeader, DialogTitle, DialogFooter } from '@/components/ui/dialog';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';

interface EditModalProps {
  isOpen: boolean;
  onClose: () => void;
  onSuccess: () => void;
  initialValue: string;
}

export function EditModal({ isOpen, onClose, onSuccess, initialValue }: EditModalProps) {
  const [value, setValue] = useState('');
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState('');

  useEffect(() => {
    if (isOpen) {
      setValue(initialValue);
      setError('');
    }
  }, [isOpen, initialValue]);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setLoading(true);
    setError('');
    try {
      await api.patch('/api/v1/item', { value });
      onSuccess();
      onClose();
    } catch (err) {
      setError(err instanceof Error ? err.message : '수정에 실패했습니다.');
    } finally {
      setLoading(false);
    }
  };

  return (
    <Dialog open={isOpen} onOpenChange={onClose}>
      <DialogContent>
        <DialogHeader>
          <DialogTitle>수정</DialogTitle>
        </DialogHeader>
        <form onSubmit={handleSubmit} className="space-y-4">
          <Input value={value} onChange={e => setValue(e.target.value)} />
          {error && <p className="text-sm text-red-500">{error}</p>}
          <DialogFooter>
            <Button type="button" variant="outline" onClick={onClose}>취소</Button>
            <Button type="submit" disabled={loading}>
              {loading ? '저장 중...' : '저장'}
            </Button>
          </DialogFooter>
        </form>
      </DialogContent>
    </Dialog>
  );
}
```

---

### Styling

**Tailwind CSS v4 (`globals.css` @theme 방식):**
```css
/* globals.css - @theme 블록으로 커스텀 변수 정의 */
@theme {
  --color-primary: #4A6CF7;     /* Blue - 메인 브랜드 */
  --color-accent: #FFE066;      /* Yellow - 강조 */
  --color-dark: #1A1A2E;        /* Dark navy */
  --font-pretendard: 'Pretendard Variable', sans-serif;
}
```

**cn() Utility:**
```typescript
import { cn } from '@/lib/utils';

// 조건부 클래스
<div className={cn(
  "flex items-center gap-2",
  isActive && "bg-primary text-white",
  isScrolled && "shadow-sm",
  className
)}>

// 상태별 스타일링
const statusColors: Record<string, string> = {
  draft: "bg-slate-100 text-slate-600",
  pending_review: "bg-yellow-100 text-yellow-700",
  active: "bg-green-100 text-green-700",
  suspended: "bg-red-100 text-red-600",
  withdrawn: "bg-gray-100 text-gray-500",
};

<Badge className={cn(statusColors[status] || "bg-gray-100")}>
  {getStatusLabel(status)}
</Badge>
```

**브랜드 컬러:**
```typescript
// 메인 블루 (#4A6CF7)
className="bg-primary text-white"
className="text-primary"
className="border-primary"

// 악센트 옐로우 (#FFE066)
className="bg-accent"

// 다크 네이비 (#1A1A2E)
className="bg-dark text-white"

// 그라디언트
className="bg-gradient-to-r from-primary to-blue-600"
```

**반응형 패턴:**
```typescript
// Grid
className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6"

// 텍스트
className="text-2xl sm:text-3xl md:text-4xl lg:text-5xl"

// Display 토글
className="hidden md:flex"   // 모바일 숨김
className="md:hidden"        // 모바일만 표시

// 패딩
className="px-4 sm:px-6 lg:px-8"
```

**글꼴 (Pretendard - Korean 최적화):**
```typescript
// layout.tsx에서 로컬 폰트로 로드됨
// 별도 import 불필요 - CSS 변수로 자동 적용
className="font-pretendard"  // or just 기본 font-family
```

**[Complete Guide: resources/styling-guide.md](resources/styling-guide.md)**

---

### File Organization

**App Router 구조:**
```
src/
  app/
    page.tsx              # 홈 페이지 (/)
    layout.tsx            # Root layout
    error.tsx             # Global error boundary
    loading.tsx           # Global loading
    {route}/
      page.tsx            # 라우트 페이지
      layout.tsx          # 라우트 레이아웃 (선택)
      loading.tsx         # 라우트 로딩 (선택)
      error.tsx           # 라우트 에러 (선택)
    api/
      {route}/
        route.ts          # API route handler
  components/
    ui/                   # shadcn/ui 컴포넌트 (수정하지 말 것)
    layout/               # 레이아웃 컴포넌트 (Navbar, Footer)
    {feature}/            # 기능별 컴포넌트 디렉토리
      FeatureComponent.tsx
      FeatureModal.tsx
```

**컴포넌트 배치 원칙:**
- shadcn/ui 컴포넌트 → `src/components/ui/` (직접 수정 최소화)
- 레이아웃 컴포넌트 → `src/components/layout/`
- 기능별 컴포넌트 → `src/components/{feature}/`
- Server/Client 컴포넌트는 파일명으로 구분 가능 (필요 시 접미사 활용)

**[Complete Guide: resources/file-organization.md](resources/file-organization.md)**

---

### Loading & Error States

**App Router Conventions:**

**Global Loading (`app/loading.tsx`):**
```typescript
// 현재 구현 패턴
export default function Loading() {
  return (
    <div className="flex items-center justify-center min-h-screen">
      <div className="text-center">
        <div className="animate-spin rounded-full h-8 w-8 border-b-2 border-primary mx-auto mb-4" />
        <p className="text-gray-500">로딩 중...</p>
      </div>
    </div>
  );
}
```

**Skeleton 로딩 (컴포넌트 내):**
```typescript
import { Skeleton } from '@/components/ui/skeleton';

// 카드 스켈레톤
function CardSkeleton() {
  return (
    <div className="space-y-4">
      <Skeleton className="h-8 w-48" />
      <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
        {[...Array(3)].map((_, i) => (
          <Skeleton key={i} className="h-24" />
        ))}
      </div>
    </div>
  );
}
```

**Error Handling (Korean):**
```typescript
// error.tsx (route-level)
'use client';

import { Button } from '@/components/ui/button';
import Link from 'next/link';

export default function Error({ error, reset }: { error: Error; reset: () => void }) {
  return (
    <div className="flex flex-col items-center justify-center min-h-screen py-12">
      <h2 className="text-2xl font-bold mb-4">오류가 발생했습니다</h2>
      <p className="text-gray-500 mb-6">페이지를 불러오는 중 문제가 발생했습니다.</p>
      <div className="flex gap-4">
        <Button onClick={reset}>다시 시도</Button>
        <Button variant="outline" asChild>
          <Link href="/">홈으로 돌아가기</Link>
        </Button>
      </div>
    </div>
  );
}
```

**[Complete Guide: resources/loading-and-error-states.md](resources/loading-and-error-states.md)**

---

### Performance

**Next.js 15 최적화:**
- Server Components by default (zero JS to client)
- Dynamic imports: `const Heavy = dynamic(() => import('./Heavy'))`
- Image optimization: `next/image` with AVIF + WebP
- Font: 로컬 Pretendard WOFF2 (network 요청 없음)
- Turbopack: 빠른 개발 빌드 (기본 활성화)
- CSS chunking: strict mode 활성화

**이미지 최적화:**
```typescript
import Image from 'next/image';

// 원격 이미지 (허용된 도메인: lh3.googleusercontent.com, prod-apne2-ygs.s3.amazonaws.com)
<Image
  src="https://lh3.googleusercontent.com/..."
  alt="사용자 프로필"
  width={40}
  height={40}
  className="rounded-full"
/>
```

**React 19 패턴:**
- `useMemo`: 비용이 큰 연산
- `useCallback`: 자식에게 전달하는 이벤트 핸들러
- `React.memo`: 불필요한 재렌더링 방지

**차트 (Recharts):**
```typescript
import { LineChart, Line, XAxis, YAxis, CartesianGrid, Tooltip, ResponsiveContainer } from 'recharts';

export function SimpleChart({ data }: { data: { date: string; count: number }[] }) {
  return (
    <ResponsiveContainer width="100%" height={300}>
      <LineChart data={data}>
        <CartesianGrid strokeDasharray="3 3" />
        <XAxis dataKey="date" />
        <YAxis />
        <Tooltip />
        <Line type="monotone" dataKey="count" stroke="#4A6CF7" />
      </LineChart>
    </ResponsiveContainer>
  );
}
```

**[Complete Guide: resources/performance.md](resources/performance.md)**

---

### TypeScript

**Standards:**
- Strict mode 활성화 (tsconfig)
- 함수 명시적 반환 타입
- Type imports: `import type { NavItem } from '@/types'`
- No `any` type (필요 시 `unknown` 사용)

**현재 `types/index.ts` 정의:**
```typescript
interface NavItem {
  label: string;
  href: string;
}

interface Feature {
  title: string;
  description: string;
  icon: string;
}

interface Review {
  content: string;
  author: string;
  role: string;
  avatarUrl: string;
  rating: number;
}
```

**새 타입 추가 패턴:**
```typescript
// types/index.ts에 추가하거나 기능별로 types/{feature}.ts 생성
export interface SomeFeature {
  id: string;
  name: string;
  status: 'active' | 'inactive';
  createdAt: string;
}

// API 응답 타입
export interface PaginatedResponse<T> {
  items: T[];
  total: number;
  skip: number;
  limit: number;
}

// 컴포넌트 Props 타입
export interface SomeComponentProps {
  data: SomeFeature;
  onUpdate?: (updated: SomeFeature) => void;
  className?: string;
}
```

**[Complete Guide: resources/typescript-standards.md](resources/typescript-standards.md)**

---

## Navigation Guide

| Need to... | Read this resource |
|------------|-------------------|
| Create a component | [component-patterns.md](resources/component-patterns.md) |
| Fetch data | [data-fetching.md](resources/data-fetching.md) |
| Organize files/folders | [file-organization.md](resources/file-organization.md) |
| Style components | [styling-guide.md](resources/styling-guide.md) |
| Set up routing | [routing-guide.md](resources/routing-guide.md) |
| Handle loading/errors | [loading-and-error-states.md](resources/loading-and-error-states.md) |
| Optimize performance | [performance.md](resources/performance.md) |
| TypeScript types | [typescript-standards.md](resources/typescript-standards.md) |
| Forms/Auth/API Routes | [common-patterns.md](resources/common-patterns.md) |
| See full examples | [complete-examples.md](resources/complete-examples.md) |

---

## Core Principles

1. **Server Components First**: Server Components 기본, Client Components는 상호작용 필요 시만
2. **Async Data Fetching**: Server Component에서 직접 데이터 fetch
3. **Minimize Client JS**: 브라우저에 전송되는 JS 최소화 = 성능 향상
4. **App Router Conventions**: loading.tsx, error.tsx, layout.tsx 적절히 활용
5. **shadcn/ui Components**: `@/components/ui/`의 pre-built 컴포넌트 우선 사용
6. **cn() for Classes**: 조건부/병합 클래스는 반드시 `cn()` 사용
7. **Import with @/ alias**: 프로젝트 전반에 걸쳐 일관된 import 경로
8. **Type Safety**: 엄격한 TypeScript, 명시적 타입
9. **Korean Localization**: 사용자 대면 텍스트는 모두 한국어
10. **AuthProvider**: 클라이언트 auth 상태는 `useAuth()` 훅 사용
11. **pnpm**: 패키지 관리자는 반드시 pnpm 사용 (`pnpm dlx shadcn@latest add`)

---

## Quick Reference: Templates

### Server Component Template

```typescript
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { api } from '@/lib/api';
import type { Metadata } from 'next';

export const metadata: Metadata = {
  title: '페이지 제목 | 연결사',
};

interface DataType {
  id: string;
  name: string;
}

export default async function ExamplePage() {
  // Server Component에서 직접 데이터 fetch
  const data = await api.get<DataType[]>('/api/v1/items');

  return (
    <div className="container mx-auto py-8 px-4">
      <h1 className="text-2xl font-bold mb-6">페이지 제목</h1>
      <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
        {data.map(item => (
          <Card key={item.id}>
            <CardHeader>
              <CardTitle>{item.name}</CardTitle>
            </CardHeader>
            <CardContent>
              <p className="text-muted-foreground">내용</p>
            </CardContent>
          </Card>
        ))}
      </div>
    </div>
  );
}
```

### Client Component Template

```typescript
'use client';

import { useState, useCallback, useEffect } from 'react';
import { Button } from '@/components/ui/button';
import { Skeleton } from '@/components/ui/skeleton';
import { api } from '@/lib/api';
import { cn } from '@/lib/utils';
import { Loader2 } from 'lucide-react';

interface DataItem {
  id: string;
  name: string;
}

interface ExampleListProps {
  className?: string;
}

export function ExampleList({ className }: ExampleListProps) {
  const [items, setItems] = useState<DataItem[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  const fetchItems = useCallback(async () => {
    try {
      setLoading(true);
      const data = await api.get<DataItem[]>('/api/v1/items');
      setItems(data);
    } catch (err) {
      setError(err instanceof Error ? err.message : '데이터를 불러오는데 실패했습니다.');
    } finally {
      setLoading(false);
    }
  }, []);

  useEffect(() => {
    fetchItems();
  }, [fetchItems]);

  if (loading) {
    return (
      <div className="space-y-4">
        {[...Array(5)].map((_, i) => (
          <Skeleton key={i} className="h-16 w-full" />
        ))}
      </div>
    );
  }

  if (error) {
    return (
      <div className="text-center py-8">
        <p className="text-red-500 mb-4">{error}</p>
        <Button onClick={fetchItems}>다시 시도</Button>
      </div>
    );
  }

  return (
    <div className={cn("space-y-4", className)}>
      {items.map(item => (
        <div key={item.id} className="p-4 border rounded-lg hover:bg-gray-50 transition-colors">
          <p className="font-medium">{item.name}</p>
        </div>
      ))}
    </div>
  );
}
```

For complete examples, see [resources/complete-examples.md](resources/complete-examples.md)

---

## Related Skills

- **fastapi-backend-guidelines**: 프론트엔드가 소비하는 백엔드 API 패턴
- **pytest-backend-testing**: 백엔드 테스트 (프론트엔드는 Playwright E2E)

---

**Skill Status**: 현재 연결사(yeongyeolsa) 프로젝트 구조 기반으로 업데이트됨 (2026-02-22)
