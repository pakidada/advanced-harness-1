---
name: copyright-safe-style-transfer
description: Replicate의 FLUX Pro 계열 모델(Kontext, Redux, Depth)을 사용하여 이미지 스타일을 저작권에 안전하게 변환. 스타일 변환, style transfer, FLUX, 저작권, copyright-safe, 이미지 변환, image transformation 관련 키워드로 트리거됨.
---

# Copyright-Safe Style Transfer

Replicate의 FLUX Pro 계열 모델을 사용하여 이미지 스타일을 저작권에 안전하게 변환하는 스킬.

---

## Purpose

- 이미지의 스타일을 저작권 문제 없이 변환
- 원본 구조는 유지하면서 완전히 새로운 아트스타일 적용
- FLUX Pro 모델 시리즈를 활용한 고품질 스타일 변환

---

## When to Use

자동으로 활성화되는 조건:
- "스타일 변환", "style transfer" 언급
- "저작권", "copyright" 관련 우려
- "FLUX", "flux pro" 모델 언급
- 이미지 스타일 변경 요청
- "이미지 리스타일", "restyle" 작업

---

## Prerequisites

### Replicate API Token 설정

**방법 1: 환경 변수**
```bash
export REPLICATE_API_TOKEN="r8_your_token_here"
```

**방법 2: MCP 설정 (`.mcp.json`)**
```json
{
  "mcpServers": {
    "replicate": {
      "type": "sse",
      "url": "https://mcp.replicate.com/sse",
      "env": {
        "REPLICATE_API_TOKEN": "r8_your_token_here"
      }
    }
  }
}
```

---

## Available FLUX Pro Models for Style Transfer

### 1. FLUX.1 Kontext Pro (Recommended)
**Model**: `black-forest-labs/flux-kontext-pro`

- 텍스트 기반 이미지 편집 및 스타일 변환
- 원본 구조를 이해하고 새로운 스타일로 재생성
- 저작권 안전: 완전히 새로운 이미지 생성

**Best For**: 전체적인 스타일 변환, 아트스타일 변경

### 2. FLUX.1 Redux Pro
**Model**: `black-forest-labs/flux-redux-pro`

- 이미지 변형 및 리스타일링 전문
- 입력 이미지를 기반으로 새로운 버전 생성
- 빠른 변환 속도

**Best For**: 빠른 변형, 색상 스타일 변경

### 3. FLUX.1 Depth Pro
**Model**: `black-forest-labs/flux-depth-pro`

- 3D 구조를 유지하면서 텍스처 변경
- 깊이 정보 보존하며 스타일 변환
- 구조적 일관성 보장

**Best For**: 깊이감 유지가 중요한 변환

### 4. FLUX.2 Pro (Latest)
**Model**: `black-forest-labs/flux-2-pro`

- 최신 모델, 최고 품질
- 최대 8개 참조 이미지 지원
- 6초 이내 생성

**Best For**: 최고 품질이 필요한 작업

---

## Copyright-Safe Transformation Strategy

### Core Principles

1. **Complete Regeneration**: 원본 픽셀을 복사하지 않고 완전히 새로 생성
2. **Style Abstraction**: 특정 아티스트 스타일이 아닌 일반적 스타일 적용
3. **Structural Reference Only**: 구조/구도만 참조하고 시각적 요소는 새로 생성

### Safe Prompt Patterns

```text
✅ SAFE - 일반적 스타일 설명:
- "in watercolor painting style"
- "as digital illustration"
- "in minimalist vector art style"
- "with oil painting texture"
- "in anime-inspired style" (특정 작가 미언급)

❌ AVOID - 특정 아티스트/작품 언급:
- "in the style of [specific artist]"
- "like [specific artwork]"
- "copy the style of [brand]"
```

### Transformation Levels

| Level | Description | Copyright Safety |
|-------|-------------|------------------|
| **Full Restyle** | 완전히 다른 스타일로 변환 | Very Safe |
| **Medium Restyle** | 비슷한 구도, 다른 표현 | Safe |
| **Light Restyle** | 색상/텍스처만 변경 | Moderate |

---

## Usage Examples

### Example 1: 사진을 일러스트 스타일로 변환

**프롬프트**:
```text
Transform this photo into a clean digital illustration style.
Use flat colors, clean lines, and simplified shapes.
Maintain the subject's pose and composition but completely
recreate all visual elements in illustration style.
```

**파라미터**:
```json
{
  "model": "black-forest-labs/flux-kontext-pro",
  "prompt": "위 프롬프트",
  "image": "입력_이미지_URL",
  "guidance": 3.5,
  "output_format": "png"
}
```

### Example 2: 수채화 스타일 변환

**프롬프트**:
```text
Reimagine this image as a soft watercolor painting.
Add subtle color bleeds, paper texture, and flowing brushstrokes.
Keep the general composition but create entirely new artistic
interpretation with watercolor medium characteristics.
```

### Example 3: 미니멀 벡터 아트

**프롬프트**:
```text
Convert to minimalist vector art style with:
- Limited color palette (max 5 colors)
- Clean geometric shapes
- No gradients, flat fills only
- Bold outlines where appropriate
Completely recreate the visual elements, do not trace.
```

### Example 4: 애니메이션 스타일

**프롬프트**:
```text
Transform into anime-inspired illustration style with:
- Clean cel-shaded coloring
- Expressive linework
- Vibrant color palette
- Stylized features
Create an original interpretation, not copying any specific anime.
```

---

## API Implementation

### Using Replicate MCP

```python
# MCP 도구 사용 예시
mcp__replicate__create_predictions(
    model="black-forest-labs/flux-kontext-pro",
    input={
        "prompt": "Transform into watercolor painting style...",
        "image": "https://example.com/input.jpg",
        "guidance": 3.5,
        "output_format": "png"
    }
)
```

### Using Python SDK

```python
import replicate

def copyright_safe_style_transfer(
    image_url: str,
    target_style: str,
    model: str = "black-forest-labs/flux-kontext-pro"
) -> str:
    """저작권 안전한 스타일 변환 수행"""

    # 안전한 프롬프트 템플릿
    safe_prompt = f"""Transform this image into {target_style}.
    Completely regenerate all visual elements in the new style.
    Maintain composition but create entirely new artistic interpretation.
    Do not copy or replicate any copyrighted elements."""

    output = replicate.run(
        model,
        input={
            "prompt": safe_prompt,
            "image": image_url,
            "guidance": 3.5,
            "output_format": "png",
            "safety_tolerance": 2
        }
    )

    return output[0] if output else None

# 사용 예시
result = copyright_safe_style_transfer(
    image_url="https://example.com/photo.jpg",
    target_style="soft watercolor painting style with muted colors"
)
```

### Using curl

```bash
curl -s -X POST https://api.replicate.com/v1/predictions \
  -H "Authorization: Bearer $REPLICATE_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "black-forest-labs/flux-kontext-pro",
    "input": {
      "prompt": "Transform into clean digital illustration style",
      "image": "https://example.com/input.jpg",
      "guidance": 3.5,
      "output_format": "png"
    }
  }'
```

---

## Style Preset Library

### Ready-to-Use Prompts

**Illustration Styles**:
```text
# Clean Digital
"clean digital illustration with flat colors and crisp edges"

# Soft Render
"soft-rendered digital art with subtle gradients and gentle lighting"

# Line Art
"detailed line art illustration with clean ink-like strokes"
```

**Painting Styles**:
```text
# Watercolor
"traditional watercolor painting with paper texture and color bleeding"

# Oil Paint
"classical oil painting with visible brushstrokes and rich colors"

# Gouache
"gouache painting with opaque colors and matte finish"
```

**Modern Styles**:
```text
# Minimalist
"minimalist design with geometric shapes and limited color palette"

# Retro
"retro vintage style with muted colors and grain texture"

# Neon
"neon-lit cyberpunk aesthetic with glowing colors and dark background"
```

---

## Pricing Reference

| Model | Speed | Price |
|-------|-------|-------|
| FLUX Kontext Pro | ~6s | $0.04/image |
| FLUX Redux Pro | ~4s | $0.025/image |
| FLUX Depth Pro | ~5s | $0.03/image |
| FLUX.2 Pro | ~6s | $0.03/megapixel |

---

## Troubleshooting

### Common Issues

1. **스타일이 적용되지 않음**
   - `guidance` 값 증가 (3.0 → 4.0)
   - 프롬프트에 스타일 설명 더 구체적으로

2. **원본과 너무 다름**
   - `guidance` 값 감소 (3.5 → 2.5)
   - FLUX Depth Pro 사용 (구조 보존)

3. **저작권 우려**
   - 특정 아티스트/브랜드 언급 제거
   - "inspired by" 대신 일반적 스타일 설명 사용

4. **API 인증 오류**
   - `REPLICATE_API_TOKEN` 환경변수 확인
   - 토큰이 `r8_`으로 시작하는지 확인

---

## Legal Disclaimer

이 스킬은 저작권 법적 조언을 제공하지 않습니다. 스타일 변환 결과물의 저작권 문제는 사용자가 책임집니다. 상업적 사용 시 법률 전문가와 상담하세요.

**Safe Practices**:
- 특정 아티스트 스타일 명시 금지
- 완전한 재생성 (트레이싱 아님) 보장
- 변환 결과물에 대한 독자적 권리 확보

---

## Related Resources

- [FLUX Models on Replicate](https://replicate.com/collections/flux)
- [FLUX Kontext Blog](https://replicate.com/blog/flux-kontext)
- [Black Forest Labs](https://blackforestlabs.ai/)

---

**Skill Status**: ACTIVE
**Line Count**: < 500 (following 500-line rule)
**Last Updated**: 2026-01
